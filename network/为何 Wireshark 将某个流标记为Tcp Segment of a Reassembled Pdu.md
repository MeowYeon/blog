# 为何 Wireshark 将某个流标记为Tcp Segment of a Reassembled Pdu
本文翻译自：https://www.pico.net/kb/how-does-wireshark-annotate-some-packets-with-tcp-segment-of-a-reassembled-pdu

简单来说，当某个TCP流中的负载消息是一条长的应用消息的一部分，或者待传输的文档在后续包中才能传输完成 时，wireshark会将该TCP报文标记为：Tcp Segment of a Reassembled Pdu

比这个简单解释更丰富的解释需要稍微深入TCP的操作。分布式应用通过交换消息或者文档来彼此通信，系统层面上来说就是通过TCP socket连接来通信。TCP socket提供的API不了解并且不关心应用消息格式，而是提供了使用字节流通信的通用模型。

TCP 会将数据重新分段然后封装到IP数据报中在网络中传输，与数据原本消息边界完全解耦。通常TCP 会在写入数据时尽快将数据发送出去，但是：
- 它在等待响应者的ACK消息时需要缓存数据，从而可以在同一个网络包中传输多条应用消息 （小消息合并）
- 应用消息可能超过单条TCP包的大小 （大消息拆分）
因此，TCP socket提供的API提供的字节流模型会在应用消息与TCP报文之间呈现一对多和多对一两种关系。

wireshark 提供的捕获数据包的视图包含两个层级：它展示单个报文，同时也装配了插件解析报文中的应用消息。虽然插件可以获取报文中的数据，但是它也依赖于wireshark实现的TCP报文重新组装：对单向（TCP全双工）的TCP连接，wireshark获取每个报文中的数据，然后根据序列号（sequence number）排序各个数据报，最后拼接并重组字节流。当数据段组装之后，流会提供解剖器可以查看里面的应用消息或者文档。如果原始数据被拆分成多个数据报，解剖器会在展示数据之前等待wireshark处理完最后一个报文并完成数据重组。

特别是，解剖器在数据完整 之前 不会展示任何数据。为了向用户标识这种报文，wireshark会将这些报文标记为：Tcp Segment of a Reassembled Pdu
- Segment 是TCP术语，代表了带TCP头部的一段数据（实际上是 packet 的同义词，尽管技术上是一个单独实体。例如，有可能一个大的TCP段会被分配到多个IP数据报中，尽管TCP努力避免这种情况）。
- PDU 是"Protocol Data Unit"的首字母缩写，在这个语境中，它代表被wireshark解剖的一条应用消息或者文档。

一旦最后一个报文到达并且完整数据被解析，wireshark会在最后一个报文中显示完整解析数据，并显示各组分的原始数据负载。

所以，总结，当TCP中的负载数据是更大的应用消息的一部分时，wireshark将TCP报标记为"TCP segment of a reassembled PDU"。

## 时间线
> 2020.11.12 yeon.guo ShangHai YangPu
> 2023.05.28 yeon.guo ShangHai MinHang 迁移
