# ProtoBuf

Protobuf 是 Google Protocol Buffers 的简称，是一款非常优秀的库，它定义了一种紧凑（compact，相对 XML 和 JSON 而言）的可扩展二进制消息格式，特别适合网络数据传输。它为多种语言提供 binding，大大方便了分布式程序的开发，让系统不再局限于用某一种语言来编写。

---


以下内容并非 protobuf 入门，是提出一个方案解决：通信双方在编译时就共享 proto 文件的情况下，接收方在收到 Protobuf 二进制数据流之后，如何自动创建具体类型的 Protobuf Message 对象，并用收到的数据填充该 Message 对象（即反序列化）的问题。


## 网络编程中使用Protobuf的两个先决条件

1. 长度，Protobuf 打包的数据没有自带长度信息或终结符，需要由应用程序自己在发生和接收的时候做正确的切分。
2. 类型，Protobuf 打包的数据没有自带类型信息，需要由发送方把类型信息传给给接收方，接收方创建具体的 Protobuf Message 对象，再做反序列化。

## 根据 type name 反射自动创建 Message 对象

[[不懂]]










[[参考]]