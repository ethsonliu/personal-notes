


```c++
class RtpPacketHistory {
 public:
  enum class StorageMode {
    kDisabled,     // Don't store any packets.
    kStoreAndCull  // Store up to `number_to_store` packets, but try to remove
                   // packets as they time out or as signaled as received.
  };

  // Maximum number of packets we ever allow in the history.
  static constexpr size_t kMaxCapacity = 9600;
  // Maximum number of entries in prioritized queue of padding packets.
  static constexpr size_t kMaxPaddingHistory = 63;
  // Don't remove packets within max(1 second, 3x RTT).
  static constexpr TimeDelta kMinPacketDuration = TimeDelta::Seconds(1);
  static constexpr int kMinPacketDurationRtt = 3;
  // With kStoreAndCull, always remove packets after 3x max(1000ms, 3x rtt).
  static constexpr int kPacketCullingDelayFactor = 3;
  
  // 包结构体
  class StoredPacket {
   public:
    StoredPacket() = default;
    StoredPacket(std::unique_ptr<RtpPacketToSend> packet,
                 Timestamp send_time,
                 uint64_t insert_order);
    StoredPacket(StoredPacket&&);
    StoredPacket& operator=(StoredPacket&&);
    ~StoredPacket();

    uint64_t insert_order() const { return insert_order_; }
    size_t times_retransmitted() const { return times_retransmitted_; }
    void IncrementTimesRetransmitted(PacketPrioritySet* priority_set);

    // The time of last transmission, including retransmissions.
    Timestamp send_time() const { return send_time_; }
    void set_send_time(Timestamp value) { send_time_ = value; }

    // The actual packet.
    std::unique_ptr<RtpPacketToSend> packet_;

    // True if the packet is currently in the pacer queue pending transmission.
    bool pending_transmission_;

   private:
    Timestamp send_time_ = Timestamp::Zero();

    // Unique number per StoredPacket, incremented by one for each added
    // packet. Used to sort on insert order.
    uint64_t insert_order_;

    // Number of times RE-transmitted, ie excluding the first transmission.
    size_t times_retransmitted_;
  };
  
  // set 的比较器
  struct MoreUseful {
    bool operator()(StoredPacket* lhs, StoredPacket* rhs) const;
  };
  
   const bool enable_padding_prio_;
  mutable Mutex lock_;
  size_t number_to_store_ RTC_GUARDED_BY(lock_);
  StorageMode mode_ RTC_GUARDED_BY(lock_);
  TimeDelta rtt_ RTC_GUARDED_BY(lock_);

  // Queue of stored packets, ordered by sequence number, with older packets in
  // the front and new packets being added to the back. Note that there may be
  // wrap-arounds so the back may have a lower sequence number.
  // Packets may also be removed out-of-order, in which case there will be
  // instances of StoredPacket with `packet_` set to nullptr. The first and last
  // entry in the queue will however always be populated.
  std::deque<StoredPacket> packet_history_ RTC_GUARDED_BY(lock_);

  // Total number of packets with inserted.
  uint64_t packets_inserted_ RTC_GUARDED_BY(lock_);
  
  // Objects from `packet_history_` ordered by "most likely to be useful", used
  // in GetPayloadPaddingPacket().
  using PacketPrioritySet = std::set<StoredPacket*, MoreUseful>;
  PacketPrioritySet padding_priority_ RTC_GUARDED_BY(lock_);
};
```

具体实现如下，

```c++
bool RtpPacketHistory::MoreUseful::operator()(StoredPacket* lhs,
                                              StoredPacket* rhs) const {
  // Prefer to send packets we haven't already sent as padding.
  if (lhs->times_retransmitted() != rhs->times_retransmitted()) {
    return lhs->times_retransmitted() < rhs->times_retransmitted();
  }
  // All else being equal, prefer newer packets.
  return lhs->insert_order() > rhs->insert_order();
}

// 保存 rtp 包
void RtpPacketHistory::PutRtpPacket(std::unique_ptr<RtpPacketToSend> packet,
                                    Timestamp send_time) {
  RTC_DCHECK(packet);
  MutexLock lock(&lock_);
  if (mode_ == StorageMode::kDisabled) {
    return;
  }

  RTC_DCHECK(packet->allow_retransmission());
  CullOldPackets();

  // Store packet.
  const uint16_t rtp_seq_no = packet->SequenceNumber();
  int packet_index = GetPacketIndex(rtp_seq_no);
  if (packet_index >= 0 &&
      static_cast<size_t>(packet_index) < packet_history_.size() &&
      packet_history_[packet_index].packet_ != nullptr) {
    RTC_LOG(LS_WARNING) << "Duplicate packet inserted: " << rtp_seq_no;
    // Remove previous packet to avoid inconsistent state.
    RemovePacket(packet_index);
    packet_index = GetPacketIndex(rtp_seq_no);
  }

  // Packet to be inserted ahead of first packet, expand front.
  for (; packet_index < 0; ++packet_index) {
    packet_history_.emplace_front();
  }
  // Packet to be inserted behind last packet, expand back.
  while (static_cast<int>(packet_history_.size()) <= packet_index) {
    packet_history_.emplace_back();
  }

  RTC_DCHECK_GE(packet_index, 0);
  RTC_DCHECK_LT(packet_index, packet_history_.size());
  RTC_DCHECK(packet_history_[packet_index].packet_ == nullptr);

  packet_history_[packet_index] =
      StoredPacket(std::move(packet), send_time, packets_inserted_++);

  if (enable_padding_prio_) {
    if (padding_priority_.size() >= kMaxPaddingHistory - 1) {
      padding_priority_.erase(std::prev(padding_priority_.end()));
    }
    auto prio_it = padding_priority_.insert(&packet_history_[packet_index]);
    RTC_DCHECK(prio_it.second) << "Failed to insert packet into prio set.";
  }
}

// 清除超时包
void RtpPacketHistory::CullOldPackets() {
  Timestamp now = clock_->CurrentTime();
  TimeDelta packet_duration =
      rtt_.IsFinite()
          ? std::max(kMinPacketDurationRtt * rtt_, kMinPacketDuration)
          : kMinPacketDuration;
  while (!packet_history_.empty()) {
    if (packet_history_.size() >= kMaxCapacity) {
      // We have reached the absolute max capacity, remove one packet
      // unconditionally.
      RemovePacket(0);
      continue;
    }

    const StoredPacket& stored_packet = packet_history_.front();
    if (stored_packet.pending_transmission_) {
      // Don't remove packets in the pacer queue, pending tranmission.
      return;
    }

    if (stored_packet.send_time() + packet_duration > now) {
      // Don't cull packets too early to avoid failed retransmission requests.
      return;
    }

    if (packet_history_.size() >= number_to_store_ ||
        stored_packet.send_time() +
                (packet_duration * kPacketCullingDelayFactor) <=
            now) {
      // Too many packets in history, or this packet has timed out. Remove it
      // and continue.
      RemovePacket(0);
    } else {
      // No more packets can be removed right now.
      return;
    }
  }
}

// 删除包
std::unique_ptr<RtpPacketToSend> RtpPacketHistory::RemovePacket(
    int packet_index) {
  // Move the packet out from the StoredPacket container.
  std::unique_ptr<RtpPacketToSend> rtp_packet =
      std::move(packet_history_[packet_index].packet_);

  // Erase from padding priority set, if eligible.
  if (enable_padding_prio_) {
    padding_priority_.erase(&packet_history_[packet_index]);
  }

  if (packet_index == 0) {
    while (!packet_history_.empty() &&
           packet_history_.front().packet_ == nullptr) {
      packet_history_.pop_front();
    }
  }

  return rtp_packet;
}

// 获取某个包的索引
int RtpPacketHistory::GetPacketIndex(uint16_t sequence_number) const {
  if (packet_history_.empty()) {
    return 0;
  }

  RTC_DCHECK(packet_history_.front().packet_ != nullptr);
  int first_seq = packet_history_.front().packet_->SequenceNumber();
  if (first_seq == sequence_number) {
    return 0;
  }

  int packet_index = sequence_number - first_seq;
  constexpr int kSeqNumSpan = std::numeric_limits<uint16_t>::max() + 1;

  if (IsNewerSequenceNumber(sequence_number, first_seq)) {
    if (sequence_number < first_seq) {
      // Forward wrap.
      packet_index += kSeqNumSpan;
    }
  } else if (sequence_number > first_seq) {
    // Backwards wrap.
    packet_index -= kSeqNumSpan;
  }

  return packet_index;
}

RtpPacketHistory::StoredPacket* RtpPacketHistory::GetStoredPacket(
    uint16_t sequence_number) {
  int index = GetPacketIndex(sequence_number);
  if (index < 0 || static_cast<size_t>(index) >= packet_history_.size() ||
      packet_history_[index].packet_ == nullptr) {
    return nullptr;
  }
  return &packet_history_[index];
}
```
