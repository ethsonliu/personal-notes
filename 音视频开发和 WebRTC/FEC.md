## 目录

- [基础知识](#基础知识)
- [webrtc 中的 fec](#webrtc-中的-fec)

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

## webrtc 中的 fec

解 fec 包，源码实现在 ulpfec_receiver_impl.cc，如下，

```
//初始化对象
void init() {
    std::unique_ptr<webrtc::UlpfecReceiver> fec_receiver_;
 
    fec_receiver_.reset(webrtc::UlpfecReceiver::Create(this));
}  
 
//添加rtp包并解析
void addAndParse(packet) {
    webrtc::RTPHeader hacky_header;
    hacky_header.headerLength = rtp_header->getHeaderLength();
    hacky_header.sequenceNumber = rtp_header->getSeqNumber();
    if (fec_receiver_->AddReceivedRedPacket(hacky_header,(const uint8_t*) packet->data, packet->length, external_ulp_pt_) == 0 {
   fec_receiver_->ProcessReceivedFec(); //会间接调用到callback
    }
}
 
//callback 
//需要继承类 ： public webrtc::RtpData
OnRecoveredPacket(const uint8_t* rtp_packet, size_t rtp_packet_length) {
// 处理rtp包
}
```

封fec包，一般都使用red封装格式，如下，

```
void init() {
 
    webrtc::FecProtectionParams key_fec_params_{1, 60, webrtc::kFecMaskRandom}
    fec_generator_.reset(new webrtc::UlpfecGenerator());
        fec_generator_->SetFecParameters(key_fec_params_);
}
 
void buildFec(rtp) {
    red_packet = buildRedPacket(rtp);
 
    //发送red_packet
 
    fec_generator_->AddRtpPacketAndGenerateFec((const uint8_t *)rtp->data, payload_length, header_length);
 
uint16_t num_fec_packets = fec_generator_->NumAvailableFecPackets();
 
    if (num_fec_packets > 0) {
        uint16_t first_fec_sequence_number = AllocateSequenceNumber(num_fec_packets); //fec 包分配序列号， 紧随在原始rtp包之后
 
        fec_packets = fec_generator_->GetUlpfecPacketsAsRed(external_red_pt_, external_ulp_pt_, first_fec_sequence_number,header_length);
    }
    for (const auto& fec_packet : fec_packets) {
    //直接发送fec包
    ｝
}
```
