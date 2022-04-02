## 目录

- [基础知识](#基础知识)
- [EC 纠删码与 Reed-Solomon Codes](#EC-纠删码与-Reed-Solomon-Codes)
- [webrtc 中的 fec](#webrtc-中的-fec)
- [带内和带外fec](#带内和带外fec)

## 基础知识

FEC（Forward Error Correction）前向纠错，是一种通过在网络传输中增加数据包的冗余信息，使得接收端能够在网络发生丢包后利用这些冗余信息直接恢复出丢失的数据包的一种方法。

**FEC 的基础理论是异或**。异或的规则如下，

两个值不相等则为 1，相等则为 0，

```
0 ^ 0 = 0
1 ^ 1 = 0
0 ^ 1 = 1
1 ^ 0 = 1
```

注：按位异或 ^，则是把两个数转换为二进制，按位进行异或运算。

异或的特性

```
恒等律：X ^ 0 = X
归零律：X ^ X = 0
交换律：A ^ B = B ^ A
结合律：A ^ (B ^ C) = (A ^ B) ^ C
```

注：可以通过数学方法推导证明，我们这里只需要记住这些规则即可，后面有大量的应用。

**XOR 的应用案例**

有了这些 XOR 的基础理论，我们看看它是怎么应用到实际中的 “校验” 和 “纠错” 的。

一：奇偶校验（Parity Check）

判断一个二进制数中 1 的数量是奇数还是偶数（应用了异或的 恒等律 和 归零律）：
```
// 例如：求 10100001 中 1 的数量是奇数还是偶数
// 结果为 1 就是奇数个 1，结果为 0 就是偶数个 1
1 ^ 0 ^ 1 ^ 0 ^ 0 ^ 0 ^ 0 ^ 1 = 1    
```

这条性质可用于奇偶校验（Parity Check），每个字节的数据都计算一个校验位，数据和校验位一起发送出去，这样接收方可以根据校验位粗略地判断接收到的数据是否有误。

二：磁盘阵列-RAID5

使用 3 块磁盘（A、B、C）组成RAID5 阵列来存储用户的数据，把每份数据切分为 A、B 两部分，然后把 A xor B 的结果作为 C ，分别写入 A、B、C 三块磁盘。最终，任意一块磁盘出错，都是可以通过另外两块磁盘的数据进行恢复的。

实现原理：应用了异或的 恒等律 和 结合律
```
c = a ^ b
a = a ^ (b ^ b) = (a ^ b) ^ b = c ^ b
b = (a ^ a) ^ b = a ^ c
```

三：基于 XOR 的 FEC

假设网络通信有 N 个 packet 需要发送，那么，可以类似上述 RAID5 的策略，每 2 个 packet 生成一个 FEC packet，这样，连续的 3 个 packet 的任意一个 packet 丢失，都能通过另外 2 个恢复出来的。

但考虑到每 2 个 packet 就产生 1 个 fec packet，冗余度可能有点高（比较浪费带宽），我们能否每 3 个或者每 N 个 packet 再产生一个 fec packet 呢？当然可以，我们以每 3 个 packet（A、B、C） 产生 1 个 fec packet（D）为例来推导一下：
```
d = a ^ b ^ c
a = a ^ (b ^ b) ^ (c ^ c) = (b ^ c) ^ (a ^ b ^ c) = b ^ c ^ d
b = (a ^ a) ^ b ^ (c ^ c) = (a ^ c) ^ (a ^ b ^ c) = a ^ c ^ d
c = (a ^ a) ^ (b ^ b) ^ c = (a ^ b) ^ (a ^ b ^ c) = a ^ b ^ d
```
由上述公式推导即可知道，这 4 个 packet，任意丢失 1 个 packet，均可以由其他 3 个 packet 恢复出来。

## EC 纠删码与 Reed-Solomon Codes

一些互联网云计算公司提供的对象存储服务，都会宣称自己具有极高的数据可靠性，使用了如三副本技术、EC 纠删码技术等等，后者大致方案如图所示：

![image](https://user-images.githubusercontent.com/33995130/146896567-1954cec8-4652-4a80-b768-e8ccfa33334d.png)

图中采用的是 8+4 的纠删码策略（即：原始数据切割为 8 份，计算出 4 份冗余信息），将这 12 份分别存储在 不同机柜的 12 台不同节点上，即使同一时刻出现多台节点（至多 4 台）损坏或不可访问，只要有不少于 8 个节点可用，数据即可恢复。

不知道大家看出来点什么没有？相比于上面基于 N 个 packet 产生 1 个 FEC packet 的方案，这种 K + M 的纠删码策略具有更好的扛丢失能力，总结下来就是：

通过 K个有效数据，产生 M 个 FEC 冗余包，这 K + M 个数据，任意丢失 M 个数据，都能把 K 个有效数据恢复出来。
其实这种方案，最早也是应用于网络传输领域的，只不过被借用到存储领域来提高磁盘的利用率。要实现这种 K + M 的 FEC 策略，使用简单的 XOR 异或来推导比较难，需要借助矩阵相关的计算，实现方案有很多种，下面简单介绍下最著名和常用的 Reed-solomon codes。

里德-所罗门码（Reed-solomon codes，简称 RS codes），利用该原理实现的 FEC 策略，通常也叫做 RS-FEC。网上关于它的介绍特别多，本文就不详细展开了，仅简单以示意图的形式给出大致的原理：

**RS codes 编码过程：**

![image](https://user-images.githubusercontent.com/33995130/146896636-f1122c70-622b-48bd-a7b8-e538efc838ee.png)

大致原理如下：假设有效数据有 K 个，期望生成 M 个 FEC 数据

1. 把 K 个有效数据组成一个单位向量 D

2. 生成一个变换矩阵 B：由一个 K 阶的单位矩阵 和一个 K * M 的范德蒙特 矩阵（Vandemode）组成

3. 两个矩阵相乘得到的矩阵 G，即包含了 M 个冗余的 FEC 数据

**RS codes 解码过程：**

![image](https://user-images.githubusercontent.com/33995130/146896683-9617f2aa-ce37-4102-a151-c13cf8c52494.png)

假设数据 D1，D4，C2 丢失了，

1. 对矩阵 B 和 D，分别取没有丢失的行构成 B‘ 和 G’
2. 根据如下公式，即可计算恢复出有效数据向量 D，即 `D = B' x G'`

## webrtc 中的 fec

**webrtc 中使用的是 ULP FEC**，ULP 是 Uneven Level Protection 的缩写，意为不均等保护，可以针对数据的重要程度提供不同级别的保护。一般音视频的媒体数据，不同部分的重要程度是不一样的，越靠前的数据越重要，对靠前的数据使用更多的 FEC 包来保护，靠后的数据使用更少的 FEC 包来保护，这样就可更充分的利用带宽。

大致原理如下，

![image](https://user-images.githubusercontent.com/33995130/147198071-fe0ce2ff-1793-4694-8e54-dcf6e5b697de.png)

FEC 包 1 在 0 级别保护了数据包 A、B，

FEC 包 2 在 0 级别保护了数据包 C、D，同时在 1 级别保护了数据包 A、B、C、D。

![image](https://user-images.githubusercontent.com/33995130/147198151-4bb898d5-5edc-4120-9efd-79960fb2b097.png)


**ulp fec 包结构**

![image](https://user-images.githubusercontent.com/33995130/146920102-5d82a821-f050-45c1-ab78-7fb1749eb6f2.png)

RTP Header 就是标准的 RTP 头， FEC Header 固定为 10 个字节，FEC Header 后面可以跟着多个 Level，每个 Level 保护着不同的数据。

**ulp fec header 结构**

![image](https://user-images.githubusercontent.com/33995130/146920171-afcadb41-71bf-4a6e-98fa-6fe6c6163122.png)

- E: 保留的扩展标志位，必须设置为 0
- L:长掩码标志位。当 L 设置为 0 时，ulp header 的 mask 长度为 16bits，当 L 设置为 1 时，ulp header 的 mask 长度为 48bits。
- P、X、CC、PT 的值由此 fec 包所保护的 RTP 包的对应值通过 XOR 运算得到。
- SN base: 设置为此 fec 包所保护的 RTP 包中的最小序列号。
- TS recovery: 由此 fec 包所保护的 RTP 包的 timestamps 通过 XOR 运算得到。
- Length recovery：由此 fec 包所保护的 RTP 包的长度通过 XOR 运算得到。

**ulp header 结构**

![image](https://user-images.githubusercontent.com/33995130/146920343-116339b9-21c3-4aac-8ad3-73dd8cf0a323.png)

- Protection Length: 此级别所保护的数据的长度。
- mask: 保护掩码，长度为 2 个字节或者 6 个字节（由 fec header 的 L 标志位决定）。通过 mask 可以知道此级别保护了哪些 RTP 包，例如 SN base 等于 100，mask 值等于 9b80，对应的二进制为  1001 1011 1000 0000，那么就可以知道第 0、3、4、6、7、8 个 RTP  包被此级别所保护，被保护的 RTP 包的序列号分别是 100、103、104、10106、107、108。

**webrtc 中 ulp fec 的启用**

如果采用 H264 编码，则不会启用 ULP FEC 功能，见代码 call/RtpVideoSender::ShouldDisableRedAndUlpfec，

编码生成 fec 包的个数由保护系数 protection_factor 和媒体包个数决定。protection_factor 是一个由丢包率和有效比率决定的值，计算公式：

```
protection_factor = kFecRateTable[k];
k = rate_i * 129 + loss_j;
loss_j = 0, 1, .. 128;
rate_i 是在某个范围内变化的值。
```

kFecRateTable 是一个静态数组，在 modules/video_coding/fec_rate_table.h 文件中定义，具体的 protection_factor 计算过程在 modules/video_coding/media_opt_util.cc 文件的 VCMFecMethod::ProtectionFactor 函数。

一般网络畅通的时候不会生成 fec 包，只有在丢包的情况下 protection_factor 才不会为 0，接下来才会生成 fec 编码包。

**掩码表的作用**

在 ulpfec 协议中，一个 fec 包可以保护多个媒体包，而一个媒体包也可以被多个 fec 包所保护。假设 m 个媒体包需要使用 k 个 fec 包来保护，则可以定义一个如下图所示的二维的 m * n 零一矩阵来描述媒体数据包在 fec 包中的保护分布情况：

![image](https://user-images.githubusercontent.com/33995130/147198439-f48d779b-b4c1-4959-b0df-2824e6156912.png)


1. 矩阵中元素 m[i, j] 置 1 表示第 j 个媒体数据包需要第 i 个 FEC 包保护。
2. 从行角度来看，第 i 行元素表示第 i 个 FEC 包保护的媒体数据包的集合；
3. 从列角度讲，第 j 列元素表示保护第 j 个媒体数据包的 FEC 包的集合。

由于该矩阵是零一矩阵，因此在存储上可以采用掩码来存储。这个掩码也就是 FEC Level Header 中所定义的 mask 掩码。

那么掩码中的 0、1 如何分布？现实世界中网络丢包分为随机丢包、突发丢包两种情况，FEC 包需要能够针对这两种情况对媒体数据包进行保护。WebRTC 预先构造两个掩码表 kPacketMaskRandomTbl 和 kPacketMaskBurstyTbl，以模拟在随机情况和突发情况下媒体数据包在 FEC 包中的保护分配情况。

假设在随机丢包场景下，对于 m * n 的情况，我们只需要从 kPacketMaskRandomTbl[m][n] 就可以获取 FEC 包所需要的全部掩码，然后该掩码为基础，构造 FEC 数据包。

webrtc 在 modules/rtp_rtcp_source/fec_private_tables_bursty 和 fec_private_tables_random 文件中预定义了两个掩码表 kPacketMaskBurstyTbl 和 kPacketMaskRandomTbl，其中 kPacketMaskBurstyTbl 用于阵发性或者连续性的网络丢包环境，而 kPacketMaskRandomTbl 用于随机性的丢包环境。

**webrtc 中 ulp fec 包的生成**

编码生成 fec 包的流程如下：

![image](https://user-images.githubusercontent.com/33995130/147025397-a9dda890-ae9d-47d5-8be0-dfd197f166b8.png)

下面是 EncodeFec 函数的实现代码：

```
int ForwardErrorCorrection::EncodeFec(const PacketList& media_packets,
                                      uint8_t protection_factor,
                                      int num_important_packets,
                                      bool use_unequal_protection,
                                      FecMaskType fec_mask_type,
                                      std::list<Packet*>* fec_packets) {
  const size_t num_media_packets = media_packets.size();
 
  // Sanity check arguments.
  RTC_DCHECK_GT(num_media_packets, 0);
  RTC_DCHECK_GE(num_important_packets, 0);
  RTC_DCHECK_LE(num_important_packets, num_media_packets);
  RTC_DCHECK(fec_packets->empty());
  const size_t max_media_packets = fec_header_writer_->MaxMediaPackets();
  if (num_media_packets > max_media_packets) {
    RTC_LOG(LS_WARNING) << "Can't protect " << num_media_packets
                        << " media packets per frame. Max is "
                        << max_media_packets << ".";
    return -1;
  }
 
  // Error check the media packets.
  for (const auto& media_packet : media_packets) {
    RTC_DCHECK(media_packet);
    if (media_packet->data.size() < kRtpHeaderSize) {
      RTC_LOG(LS_WARNING) << "Media packet " << media_packet->data.size()
                          << " bytes "
                             "is smaller than RTP header.";
      return -1;
    }
    // Ensure the FEC packets will fit in a typical MTU.
    if (media_packet->data.size() + MaxPacketOverhead() + kTransportOverhead >
        IP_PACKET_SIZE) {
      RTC_LOG(LS_WARNING) << "Media packet " << media_packet->data.size()
                          << " bytes "
                             "with overhead is larger than "
                          << IP_PACKET_SIZE << " bytes.";
    }
  }
 
  // Prepare generated FEC packets.
  int num_fec_packets = NumFecPackets(num_media_packets, protection_factor);
  if (num_fec_packets == 0) {
    return 0;
  }
  for (int i = 0; i < num_fec_packets; ++i) {
    generated_fec_packets_[i].data.EnsureCapacity(IP_PACKET_SIZE);
    memset(generated_fec_packets_[i].data.data(), 0, IP_PACKET_SIZE);
    // Use this as a marker for untouched packets.
    generated_fec_packets_[i].data.SetSize(0);
    fec_packets->push_back(&generated_fec_packets_[i]);
  }
 
  internal::PacketMaskTable mask_table(fec_mask_type, num_media_packets);
  packet_mask_size_ = internal::PacketMaskSize(num_media_packets);
  memset(packet_masks_, 0, num_fec_packets * packet_mask_size_);
  internal::GeneratePacketMasks(num_media_packets, num_fec_packets,
                                num_important_packets, use_unequal_protection,
                                &mask_table, packet_masks_);
 
  // Adapt packet masks to missing media packets.
  int num_mask_bits = InsertZerosInPacketMasks(media_packets, num_fec_packets);
  if (num_mask_bits < 0) {
    RTC_LOG(LS_INFO) << "Due to sequence number gaps, cannot protect media "
                        "packets with a single block of FEC packets.";
    fec_packets->clear();
    return -1;
  }
  packet_mask_size_ = internal::PacketMaskSize(num_mask_bits);
 
  // Write FEC packets to |generated_fec_packets_|.
  GenerateFecPayloads(media_packets, num_fec_packets);
  // TODO(brandtr): Generalize this when multistream protection support is
  // added.
  const uint32_t media_ssrc = ParseSsrc(media_packets.front()->data.data());
  const uint16_t seq_num_base =
      ParseSequenceNumber(media_packets.front()->data.data());
  FinalizeFecHeaders(num_fec_packets, media_ssrc, seq_num_base);
 
  return 0;
}
```

EncodeFec 函数的大概流程就是根据媒体包个数和保护系数 protection_factor 确定需要需要生成的 fec 包的个数 num_fec_packets，如果 num_fec_packets 为 0 则函数返回，否则获取生成 fec 包所需要的掩码表并存放在 packet_masks_ 变量中，然后调用 GenerateFecPayloads 生成所有的 fec 包，这个时候得到的 fec 包并不是最终的状态，还需要调用 FinalizeFecHeaders 来调整 fec 包的头部。

下面来看下 GenerateFecPayload 函数的实现：

```
void ForwardErrorCorrection::GenerateFecPayloads(
    const PacketList& media_packets,
    size_t num_fec_packets) {
  RTC_DCHECK(!media_packets.empty());
  for (size_t i = 0; i < num_fec_packets; ++i) {
    Packet* const fec_packet = &generated_fec_packets_[i];
    size_t pkt_mask_idx = i * packet_mask_size_;
    const size_t min_packet_mask_size = fec_header_writer_->MinPacketMaskSize(
        &packet_masks_[pkt_mask_idx], packet_mask_size_);
    const size_t fec_header_size =
        fec_header_writer_->FecHeaderSize(min_packet_mask_size);
 
    size_t media_pkt_idx = 0;
    auto media_packets_it = media_packets.cbegin();
    uint16_t prev_seq_num =
        ParseSequenceNumber((*media_packets_it)->data.data());
    while (media_packets_it != media_packets.end()) {
      Packet* const media_packet = media_packets_it->get();
      const uint8_t* media_packet_data = media_packet->data.cdata();
      // Should |media_packet| be protected by |fec_packet|?
      if (packet_masks_[pkt_mask_idx] & (1 << (7 - media_pkt_idx))) {
        size_t media_payload_length =
            media_packet->data.size() - kRtpHeaderSize;
 
        bool first_protected_packet = (fec_packet->data.size() == 0);
        size_t fec_packet_length = fec_header_size + media_payload_length;
        if (fec_packet_length > fec_packet->data.size()) {
          // Recall that XORing with zero (which the FEC packets are prefilled
          // with) is the identity operator, thus all prior XORs are
          // still correct even though we expand the packet length here.
          fec_packet->data.SetSize(fec_packet_length);
        }
        if (first_protected_packet) {
          uint8_t* data = fec_packet->data.data();
          // Write P, X, CC, M, and PT recovery fields.
          // Note that bits 0, 1, and 16 are overwritten in FinalizeFecHeaders.
          memcpy(&data[0], &media_packet_data[0], 2);
          // Write length recovery field. (This is a temporary location for
          // ULPFEC.)
          ByteWriter<uint16_t>::WriteBigEndian(&data[2], media_payload_length);
          // Write timestamp recovery field.
          memcpy(&data[4], &media_packet_data[4], 4);
          // Write payload.
          if (media_payload_length > 0) {
            memcpy(&data[fec_header_size], &media_packet_data[kRtpHeaderSize],
                   media_payload_length);
          }
        } else {
          XorHeaders(*media_packet, fec_packet);
          XorPayloads(*media_packet, media_payload_length, fec_header_size,
                      fec_packet);
        }
      }
      media_packets_it++;
      if (media_packets_it != media_packets.end()) {
        uint16_t seq_num =
            ParseSequenceNumber((*media_packets_it)->data.data());
        media_pkt_idx += static_cast<uint16_t>(seq_num - prev_seq_num);
        prev_seq_num = seq_num;
      }
      pkt_mask_idx += media_pkt_idx / 8;
      media_pkt_idx %= 8;
    }
    RTC_DCHECK_GT(fec_packet->data.size(), 0)
        << "Packet mask is wrong or poorly designed.";
  }
}
```

GenerateFecPayloads 函数的处理流程如下：

1. 取 fec 包列表 generated_fec_packets_ 中的一个包 fec_packet，计算此 fec 包所对应的掩码表的索引 pkt_mask_idx，计算此 fec 包的头部大小 fec_header_size，计算媒体包列表 media_packets 的第一个包的序号 prev_seq_num。
2. 取 media_packets_ 的一个包 media_packet 进行处理，通过掩码值和 media_packet 的序列号来判断此 media_packet 是否是被 fec_packet 所保护。如果不受保护则转到 5 处理。如果受保护，则判断此 media_packet 是否此 fec_packet 保护的第一个包，如果是则转到 3 处理，否则转到 4 处理。
3. 第一部分，media_packet 头两个字节拷贝 fec_packet 的头两个字节，因为 RTP 头部的第一个字节的开始两位是版本事情，而 FEC 头部第一个字节的开始两位是是 E 和 L，所以可以使用 memcpy 直接复制，后面调用 FinazlizeFecHeaders 函数再对 E、L 位进行修正。第二部分，将 media_packet 的负载长度临时写到 fec_packet 的第 3、4 个字节，后面调用 FinazlizeFecHeaders 函数再对长度进行修正。第三部分，将 media_packet 的时间戳写到 fec_packet。第四部分，将 media_packet 的负载数据写到 fec_packet。转到 5 处理。
4. 将 media_packet 与 fec_packet 的头部进行 xor 运算，运算结果写到 fec_packet 的头部。将 media_packet 与 fec_packet 的负载数据进行 xor 运算，运算结果写到 fec_packet 的负载部分。
5. 取 media_packets_ 的下一个包，转 2 继续处理。遍历 media_packets_ 后结束循环。
6. 取 generated_fec_packets_ 的下一个包，转 1 继续处理。遍历 generated_fec_packets 后结束循环，至此所有的 fec 包基本都构建好了。

接下来再看 FinalizeFecHeaders 函数的代码：

```
void ForwardErrorCorrection::FinalizeFecHeaders(size_t num_fec_packets,
                                                uint32_t media_ssrc,
                                                uint16_t seq_num_base) {
  for (size_t i = 0; i < num_fec_packets; ++i) {
    fec_header_writer_->FinalizeFecHeader(
        media_ssrc, seq_num_base, &packet_masks_[i * packet_mask_size_],
        packet_mask_size_, &generated_fec_packets_[i]);
  }
}
 
void UlpfecHeaderWriter::FinalizeFecHeader(
    uint32_t /* media_ssrc */,
    uint16_t seq_num_base,
    const uint8_t* packet_mask,
    size_t packet_mask_size,
    ForwardErrorCorrection::Packet* fec_packet) const {
  uint8_t* data = fec_packet->data.data();
  // Set E bit to zero.
  data[0] &= 0x7f;
  // Set L bit based on packet mask size. (Note that the packet mask
  // can only take on two discrete values.)
  bool l_bit = (packet_mask_size == kUlpfecPacketMaskSizeLBitSet);
  if (l_bit) {
    data[0] |= 0x40;  // Set the L bit.
  } else {
    RTC_DCHECK_EQ(packet_mask_size, kUlpfecPacketMaskSizeLBitClear);
    data[0] &= 0xbf;  // Clear the L bit.
  }
  // Copy length recovery field from temporary location.
  memcpy(&data[8], &data[2], 2);
  // Write sequence number base.
  ByteWriter<uint16_t>::WriteBigEndian(&data[2], seq_num_base);
  // Protection length is set to entire packet. (This is not
  // required in general.)
  const size_t fec_header_size = FecHeaderSize(packet_mask_size);
  ByteWriter<uint16_t>::WriteBigEndian(
      &data[10], fec_packet->data.size() - fec_header_size);
  // Copy the packet mask.
  memcpy(&data[12], packet_mask, packet_mask_size);
}
```

FinalizeFecHeader 函数修正了 fec 头的 E 和 L 标志位，同时写入了基准序列号，更正了 length recovery 字段的值，最后写入 ulp level header。ulp level header 由保护长度（2 个字节）和掩码（2 个或者 6 个字节）组成。由 FinilizeFecHeader 函数可以看出，虽然 ulp fec 协议支持在一个 fec 包里面封装多个保护级别的数据，但 webrtc 实际上只用到了一个级别。

**利用 ulp fec 包恢复丢失的 rtp 包**

fec 包的解析就是 fec 包封装的逆过程，代码就不贴了，可以参见 modules/rtc_rtcp/source/forward_error_correction.cc::DecodeFec 以及相关函数，下面简要下收到 RTP 包后恢复丢失的 RTP 包的处理流程。

1. 判断此 RTP 包是否是 fec 包，如果是那么把这个包放到队列 received_fec_packets_ 里面，并通过 fec 包里面的 mask 字段解析出此 fec 包所保护的所有 RTP 包，把这些被保护的 RTP 包存放到此 fec 包下面。
2. 如果此 RTP 包是媒体包，那么更新 received_fec_packets 中对应的 fec 包的保护包接收情况。
3. 尝试恢复丢失的包。遍历 received_fec_packets_，如果其中的一个 fec 包的所保护的 RTP 包的缺失数 packets_missing 刚好是 1，那么就利用此 fec 包恢复缺失的 RTP 包。如果 packets_missing 为 0，证明此 fec 包所保护的 RTP 包均已收到，丢弃此 fec 包。如果 packets_missing 大于 1 则处理下一个 fec 包。

**Red 打包格式**

Red(Redundant coding)编码是 webrtc 中采用的一种编码方式，虽然 modules/rtp_rtcp/source/ulpfec_receiver_impl.cc 文件中存在 red 格式的定义，如下：

     0                   1                    2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3  4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |F|   block PT  |  timestamp offset         |   block length    |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

但实际上调试发现 webrtc 在采用 red 编码时打包 RTP 包仅仅是改变了原 RTP 包的 payload 类型，同时在原有的 payload 前增加多一个字节的 red_payload。假设原有的 RTP 的的负载类型是 96，而 red 负载类型是 122，那么包结构如下：

```
原 rtp 包：        RTP Header(PT = 96  ) + payload
red 格式 rtp 包:  RTP Header(PT = 122) + red_payload(一字节, 值为 96) + payload
```

payload 部分没有任何改变，如果是 fec 类型的包，那么 red_payload = 122, 这样接收端就可以通过 red_payload 的值来区分此 RTP 包是普通媒体包还是 fec 包。

**fec 打包和 RTP 包加密的先后顺序**

fec 打包在加密操作之前进行。所有的音视频 RTP 包、fec 类型的 RTP 包都会被送到 pacing 模块，再由 pacing 模块传到 pc_network_thread 线程进行发送，在调用 socket 发送数据包之前才调用 libsrtp 模块进行加密，加密是针对 RTP 包的 payload 部分进行。收端接收到 RTP 包后需要先解密再进行解析。

**如何区分音频 RTP 包和视频 RTP 包**

当 webrtc 使用同一个 udp 端口来传输音视频数据时，需要能够区分音频、视频 RTP 包。可以通过 RTP 包中的SSRC来区分。音频、视频的 RTP 包的 SSRC 不同，接收端在得到 RTP 包的 SSRC 后，根据 SSRC 来进行不同的处理。

**如何区分 RTP 包和 RTCP 包**

通过负载类型来区分，

```
// For additional details, see http://tools.ietf.org/html/rfc5761.
bool IsRtcpPacket(rtc::ArrayView<const char> packet) {
  if (packet.size() < kMinRtcpPacketLen ||
      !HasCorrectRtpVersion(
          rtc::reinterpret_array_view<const uint8_t>(packet))) {
    return false;
  }
 
  char pt = packet[1] & 0x7F;
  return (63 < pt) && (pt < 96);
}
```
## 带内和带外fec

带内FEC使用了标准帧内空闲字节做纠错字节，信号速率没有变，频宽的利用率提高了。带内FEC符合现行标准ITU-T G.707，纠错能力较强，兼容性好，可平滑升级过渡，不需对设备进行改动；但由于可用于FEC的开销有限， FEC的纠错力有限。

带外FEC，是在原来帧结构外通过数字包封技术加入了纠错字节，信号的速率增大。它采用RS码进行编解码，符合标准ITU-T G.975或ITU-T G.709，纠错能力很强，在海底光缆等长距离通信方面得到了快速发展。由于该方案增加了线速率，因此不能实现无缝升级，需要对相应设备进行改动，投资相对较大。其优点是开销采用外加方式，不受帧格式限制，可方便地插入FEC开销，具有很大的灵活性，纠错能力可做到很强。

参考：

- https://blog.csdn.net/weixin_29405665/article/details/107250663
