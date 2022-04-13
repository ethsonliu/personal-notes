音频 nack 与类 NackTracker 有关。

## 接收

nack 包作为 rtcp 包被发送端收到，首先是 ModuleRtpRtcpImpl::IncomingRtcpPacket 处理，

```c++
int32_t ModuleRtpRtcpImpl::IncomingRtcpPacket(
    const uint8_t* rtcp_packet,
    const size_t length) {
  return rtcp_receiver_.IncomingPacket(rtcp_packet, length) ? 0 : -1;
}
```

接着是 RTCPReceiver::IncomingPacket 处理，

```c++
void RTCPReceiver::IncomingPacket(rtc::ArrayView<const uint8_t> packet) {
  if (packet.empty()) {
    RTC_LOG(LS_WARNING) << "Incoming empty RTCP packet";
    return;
  }

  PacketInformation packet_information;
  if (!ParseCompoundPacket(packet, &packet_information)) // 解析 rtcp 包内容（计算 rtt，如果是nack包就取出rtp丢失序列号），并保存到 packet_information 对象里
    return;
  TriggerCallbacksFromRtcpPacket(packet_information);
}
```

再接着继续往下处理，

```c++
void RTCPReceiver::TriggerCallbacksFromRtcpPacket(
    const PacketInformation& packet_information) {
  // Process TMMBR and REMB first to avoid multiple callbacks
  // to OnNetworkChanged.
  if (packet_information.packet_type_flags & kRtcpTmmbr) {
    // Might trigger a OnReceivedBandwidthEstimateUpdate.
    NotifyTmmbrUpdated();
  }

  if (!receiver_only_ && (packet_information.packet_type_flags & kRtcpSrReq)) {
    rtp_rtcp_->OnRequestSendReport();
  }
  if (!receiver_only_ && (packet_information.packet_type_flags & kRtcpNack)) {
    if (!packet_information.nack_sequence_numbers.empty()) {
      RTC_LOG(LS_VERBOSE) << "Incoming NACK length: "
                          << packet_information.nack_sequence_numbers.size();
      rtp_rtcp_->OnReceivedNack(packet_information.nack_sequence_numbers); // nack 包的话会进入这里
    }
  }

......
}
```

继续往下走，

```c++
void ModuleRtpRtcpImpl::OnReceivedNack(
    const std::vector<uint16_t>& nack_sequence_numbers) {
  if (!rtp_sender_)
    return;

  if (!StorePackets() || nack_sequence_numbers.empty()) {
    return;
  }
  // Use RTT from RtcpRttStats class if provided.
  int64_t rtt = rtt_ms();
  if (rtt == 0) {
    rtcp_receiver_.RTT(rtcp_receiver_.RemoteSSRC(), NULL, &rtt, NULL, NULL); // 计算平均 rtt，公式 avg_rtt_ms = report_block->sum_rtt_ms / report_block->num_rtts
  }
  rtp_sender_->packet_generator.OnReceivedNack(nack_sequence_numbers, rtt);
}
```
再接着处理 nack 包，

```c++
void RTPSender::OnReceivedNack(
    const std::vector<uint16_t>& nack_sequence_numbers,
    int64_t avg_rtt) {
  packet_history_->SetRtt(TimeDelta::Millis(5 + avg_rtt));
  for (uint16_t seq_no : nack_sequence_numbers) {
    const int32_t bytes_sent = ReSendPacket(seq_no);
    if (bytes_sent < 0) {
      // Failed to send one Sequence number. Give up the rest in this nack.
      RTC_LOG(LS_WARNING) << "Failed resending RTP packet " << seq_no
                          << ", Discard rest of packets.";
      break;
    }
  }
}

int32_t RTPSender::ReSendPacket(uint16_t packet_id) {
  int32_t packet_size = 0;
  const bool rtx = (RtxStatus() & kRtxRetransmitted) > 0;

  std::unique_ptr<RtpPacketToSend> packet =
      packet_history_->GetPacketAndMarkAsPending(
          packet_id, [&](const RtpPacketToSend& stored_packet) {
            // Check if we're overusing retransmission bitrate.
            // TODO(sprang): Add histograms for nack success or failure
            // reasons.
            packet_size = stored_packet.size();
            std::unique_ptr<RtpPacketToSend> retransmit_packet;
            if (retransmission_rate_limiter_ &&
                !retransmission_rate_limiter_->TryUseRate(packet_size)) {
              return retransmit_packet;
            }
            if (rtx) {
              retransmit_packet = BuildRtxPacket(stored_packet);
            } else {
              retransmit_packet =
                  std::make_unique<RtpPacketToSend>(stored_packet);
            }
            if (retransmit_packet) {
              retransmit_packet->set_retransmitted_sequence_number(
                  stored_packet.SequenceNumber());
            }
            return retransmit_packet;
          });
  if (packet_size == 0) {
    // Packet not found or already queued for retransmission, ignore.
    RTC_DCHECK(!packet);
    return 0;
  }
  if (!packet) {
    // Packet was found, but lambda helper above chose not to create
    // `retransmit_packet` out of it.
    return -1;
  }
  packet->set_packet_type(RtpPacketMediaType::kRetransmission);
  packet->set_fec_protect_packet(false);
  std::vector<std::unique_ptr<RtpPacketToSend>> packets;
  packets.emplace_back(std::move(packet));
  paced_sender_->EnqueuePackets(std::move(packets));

  return packet_size;
}

std::unique_ptr<RtpPacketToSend> RtpPacketHistory::GetPacketAndMarkAsPending(
    uint16_t sequence_number,
    rtc::FunctionView<std::unique_ptr<RtpPacketToSend>(const RtpPacketToSend&)>
        encapsulate) {
  MutexLock lock(&lock_);
  if (mode_ == StorageMode::kDisabled) {
    return nullptr;
  }

  StoredPacket* packet = GetStoredPacket(sequence_number);
  if (packet == nullptr) {
    return nullptr;
  }

  if (packet->pending_transmission_) {
    // Packet already in pacer queue, ignore this request.
    return nullptr;
  }

  if (!VerifyRtt(*packet)) {
    // Packet already resent within too short a time window, ignore.
    return nullptr;
  }

  // Copy and/or encapsulate packet.
  std::unique_ptr<RtpPacketToSend> encapsulated_packet =
      encapsulate(*packet->packet_);
  if (encapsulated_packet) {
    packet->pending_transmission_ = true;
  }

  return encapsulated_packet;
}

bool RtpPacketHistory::VerifyRtt(
    const RtpPacketHistory::StoredPacket& packet) const {
  if (packet.times_retransmitted() > 0 &&
      clock_->CurrentTime() - packet.send_time() < rtt_) {
    // This packet has already been retransmitted once, and the time since
    // that even is lower than on RTT. Ignore request as this packet is
    // likely already in the network pipe.
    return false;
  }

  return true;
}
```

## 发送

首先是 NackTracker 的创建，

```c++
class NackTracker {
 public:
  // A limit for the size of the NACK list.
  static const size_t kNackListSizeLimit = 500;  // 10 seconds for 20 ms frame
                                                 // packets.
  NackTracker();
};

void NetEqImpl::EnableNack(size_t max_nack_list_size) {
  rtc::CritScope lock(&crit_sect_);
  if (!nack_enabled_) {
    const int kNackThresholdPackets = 2;  创建时这个值=2，是表示有 2 个 seq 间隔是就要重传了，很重要
    nack_.reset(NackTracker::Create(kNackThresholdPackets));
    nack_enabled_ = true;
    nack_->UpdateSampleRate(fs_hz_);
  }
  nack_->SetMaxNackListSize(max_nack_list_size);
}
```

有新的包到来时，需要 insert 进去，

```c++
void NackTracker::UpdateLastReceivedPacket(uint16_t sequence_number, uint32_t timestamp)
{
  // Just record the value of sequence number and timestamp if this is the
  // first packet.
  // 如果是第一个包，就只是保存
  if (!any_rtp_received_) {
    sequence_num_last_received_rtp_ = sequence_number;
    timestamp_last_received_rtp_ = timestamp;
    any_rtp_received_ = true;
    // If no packet is decoded, to have a reasonable estimate of time-to-play
    // use the given values.
    if (!any_rtp_decoded_) {
      sequence_num_last_decoded_rtp_ = sequence_number;
      timestamp_last_decoded_rtp_ = timestamp;
    }
    return;
  }

  // 如果是上一个包，就不保存了
  if (sequence_number == sequence_num_last_received_rtp_)
    return;

  // 从nack 的list 中，重传的包到了之后，从nack 的list 中去掉
  // Received RTP should not be in the list.
  nack_list_.erase(sequence_number);

  // 和上一个包进行对比，如果是false，走下边
  // If this is an old sequence number, no more action is required, return.
  if (IsNewerSequenceNumber(sequence_num_last_received_rtp_, sequence_number))
    return;

  // 计算samples_per_packet_ 每个包的平均增长数
  UpdateSamplesPerPacket(sequence_number, timestamp);
  
  // 更新list，很重要
  UpdateList(sequence_number);
  
  // 保存这次的信息
  sequence_num_last_received_rtp_ = sequence_number;
  timestamp_last_received_rtp_ = timestamp;
  
  // 根据limit 删除不要nack 的包，及丢帧了
  LimitNackListSize();
}

void NackTracker::UpdateList(uint16_t sequence_number_current_received_rtp)
{
  // Some of the packets which were considered late, now are considered missing.
  ChangeFromLateToMissing(sequence_number_current_received_rtp);

  if (IsNewerSequenceNumber(sequence_number_current_received_rtp,
                            sequence_num_last_received_rtp_ + 1))
    // 添加到list 中
    AddToList(sequence_number_current_received_rtp);
}

void NackTracker::ChangeFromLateToMissing(uint16_t sequence_number_current_received_rtp)
{
  // 根据2分法，如果有 nack_threshold_packets_ = 2个间隔
  NackList::const_iterator lower_bound =
      nack_list_.lower_bound(static_cast<uint16_t>(
          sequence_number_current_received_rtp - nack_threshold_packets_));

  // 前面的is_missing 就设置成true
  for (NackList::iterator it = nack_list_.begin(); it != lower_bound; ++it)
    it->second.is_missing = true;
}

void NackTracker::AddToList(uint16_t sequence_number_current_received_rtp)
{
  assert(!any_rtp_decoded_ ||
         IsNewerSequenceNumber(sequence_number_current_received_rtp,
                               sequence_num_last_decoded_rtp_));

  // Packets with sequence numbers older than |upper_bound_missing| are
  // considered missing, and the rest are considered late.
  uint16_t upper_bound_missing =
      sequence_number_current_received_rtp - nack_threshold_packets_;

  for (uint16_t n = sequence_num_last_received_rtp_ + 1;
       IsNewerSequenceNumber(sequence_number_current_received_rtp, n); ++n) {
    bool is_missing = IsNewerSequenceNumber(upper_bound_missing, n);
    
    uint32_t timestamp = EstimateTimestamp(n);
    NackElement nack_element(TimeToPlay(timestamp), timestamp, is_missing);
    nack_list_.insert(nack_list_.end(), std::make_pair(n, nack_element));
  }
}

void NackTracker::LimitNackListSize()
{
  // 这里丢弃小于 limit 的数据
  uint16_t limit = sequence_num_last_received_rtp_ -
                   static_cast<uint16_t>(max_nack_list_size_) - 1;
  nack_list_.erase(nack_list_.begin(), nack_list_.upper_bound(limit));
}

std::vector<uint16_t> NackTracker::GetNackList(int64_t round_trip_time_ms) const
{
  RTC_DCHECK_GE(round_trip_time_ms, 0);
  std::vector<uint16_t> sequence_numbers;
  for (NackList::const_iterator it = nack_list_.begin(); it != nack_list_.end();
       ++it) {
    if (it->second.is_missing &&
        it->second.time_to_play_ms > round_trip_time_ms)
      sequence_numbers.push_back(it->first);
  }
  return sequence_numbers;
}
```

当 GetNackList() 之后，就需要发送，发送的接口如下，

```c++
// Send a Negative acknowledgment packet.
int32_t ModuleRtpRtcpImpl::SendNACK(const uint16_t* nack_list,
                                    const uint16_t size) {
  // 保存记录
  for (int i = 0; i < size; ++i) {
    receive_loss_stats_.AddLostPacket(nack_list[i]);
  }

  uint16_t nack_length = size;
  uint16_t start_id = 0;
  int64_t now = clock_->TimeInMilliseconds();
  if (TimeToSendFullNackList(now)) {
    nack_last_time_sent_full_ = now;
    nack_last_time_sent_full_prev_ = now;
  } else {
    // 如果是已经发送过的Seq，就直接返回
    // Only send extended list.
    if (nack_last_seq_number_sent_ == nack_list[size - 1]) {
      // Last sequence number is the same, do not send list.
      return 0;
    }

     // 获取新的Seq list 位置，这里只发送新的Seq，已经发送过的，就不在发送了
    // Send new sequence numbers.
    for (int i = 0; i < size; ++i) {
      if (nack_last_seq_number_sent_ == nack_list[i]) {
        start_id = i + 1;
        break;
      }
    }
    nack_length = size - start_id;
  }

  // 最大重传数目
  // Our RTCP NACK implementation is limited to kRtcpMaxNackFields sequence
  // numbers per RTCP packet.
  if (nack_length > kRtcpMaxNackFields) {
    nack_length = kRtcpMaxNackFields;
  }
  
  // 记录本次发送的最大重传Seq
  nack_last_seq_number_sen_ = nack_list[start_id + nack_length - 1];
  
  // 重传
  return rtcp_sender_.SendRTCP(GetFeedbackState(), kRtcpNack, nack_length,
                               &nack_list[start_id]);
}

```

IsNewerSequenceNumber 的含义如下，

```
t1 = last
t2 = current

if t1 > t2 && |t1-t2| == kBreakPoint;
if t1 = max && t2 = 0 return true
if t1 = 0 && t2 = max return false
else
if t1 - t2 < kBreakPoint return true
if t1 - t2 > kBreakPoint return false
总结：t1 < t2 && t1 - t2 < kBreakpoint return false，这个时候t2 是 t1 的下一个包
```

最后，这里有两个问题，

**nack 包什么时候发送呢？**

因为报文有可能出现乱序抖动情况，不能说检测出丢包就立即重传，需要等待 rtt_ms。因为 NACK 产生的延时主要在 RTT 环路延时上，所以再次重传的时间一定要大于 rtt_ms。

**rtp 的最大重传次数是多少？**

```c++
const int kMaxPacketAge = 10000;
const int kMaxNackPackets = 1000;
const int kDefaultRttMs = 100;
const int kMaxNackRetries = 10; // 最大重传10次
const int kMaxReorderedPackets = 128;
const int kNumReorderingBuckets = 10;
const int kDefaultSendNackDelayMs = 0;
```

**NackTracker 的一个 bug**

如果第一个 rtp 包没收到，nack 是处理不了的。
