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








