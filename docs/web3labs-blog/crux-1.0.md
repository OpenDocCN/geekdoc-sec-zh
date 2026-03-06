# 症结 1.0

> 原文：<https://blog.web3labs.com/web3development/crux-1.0>

自从我们[宣布症结](https://medium.com/blk-io/announcing-crux-a-secure-enclave-for-quorum-61afbfdb79e4?source=collection_home---6------0----------------)以来，我们在过去的几个月里一直忙于 blk.io。我们很高兴地宣布[版本 1.0](https://github.com/blk-io/crux/releases/tag/v1.0.0) 的可用性。

这个版本更新了客户端和内部 API，利用了 Google 的协议缓冲区和 gRPC 技术。也完全支持 TLS。

![Crux Constellation](img/2eea7d54953782d6265c99949af5d01c.png)

*The Crux Constellation (Eckhard Slawik)*

## TL；速度三角形定位法(dead reckoning)

我们有一个 Docker 映像，它使用 Crux 构建了一个 4 节点法定网络。只需运行:

```java
git clone https://github.com/blk-io/crux.git
docker-compose -f docker/quorum-crux/docker-compose.yaml up
```

并利用此处列出的[节点端点进行黑客攻击。](https://github.com/blk-io/crux#4-node-quorum-network-with-crux)

## 协议缓冲区

Google 的 [protobuf](https://developers.google.com/protocol-buffers/) 库提供了高效的二进制编码能力，代码生成器适用于所有主要语言，包括 Java、C++、C#、JavaScript、Python 和 Golang。

它还由 [gRPC](https://grpc.io/) RPC 框架补充，该框架通过 HTTP/2 提供 protobuf 数据的快速双向流，同样为所有主要语言提供代码生成器。

这种组合使您能够在一个`.proto`文件中定义 API 端点和消息，然后代码生成器会处理剩下的事情。

在最近的一些项目中，我们与 protobuf 和 gRPC 合作，这极大地提高了生产率，因为您不再需要手工编写 API 集成代码。

## 例子

例如，要将一个事务发送到 Crux，我们需要以下消息:

```java
// Store a transaction payload
message SendRequest {
  // The payload of the transaction to be sent
  bytes payload = 1;

  // The key of the Sender
  string from = 2;  

 // The keys of all the Receipients who are privy to the transaction
  repeated string to = 3;
}

// Response from the server for the SendRequest
message SendResponse {
  // The transaction hash is returned
  bytes key = 1;
}
```

然后，要定义一个端点来使用这些消息，我们只需使用以下代码:

```java
service Client {

  // Used to store a transaction in the Chimera node
  rpc Send(SendRequest) returns (SendResponse); 

 // Other endpoint definitions...

}
```

## 完全向后兼容

仍然完全支持以前的 Constellation API，但是，向 protobuf+gRPC 的过渡极大地简化了 API 的实现。Quorum -> Constellation 和 Constellation to Constellation API 有一段时间对许多人来说是个谜。现在你可以看到消息定义为 [protobufs](https://github.com/blk-io/chimera-api/blob/master/proto/messages.proto) 和 [gRPC](https://github.com/blk-io/chimera-api/blob/master/proto/grpc.proto) 定义。

你会看到 protobufs 在一个单独的回购中，在未来几周会有更多的报道。

我们热切希望使用 enclave 技术的其他供应商将采用类似的定义良好的 API，而不是 protobuf+gRPC，这有助于确保项目之间更好的互操作。

## Docker 图像

我们还提供了 Docker 图像，使 Quorum 和 Crux 的工作变得非常简单。

正如本文开头提到的，我们有一个使用 Crux 的 4 节点法定网络示例。这是将 gRPC 用于所有仲裁到关键通信。

我们还有一个[双节点 Crux](https://github.com/blk-io/crux#2-node-crux-only-network) 专用网络，用于处理 Crux API。

最后，对于那些不能使用 Docker 的人，我们也有一个更新版本的 [Quorum 7-nodes 示例](https://github.com/blk-io/quorum-examples)在一个流浪虚拟机上使用 Crux。

## 前进

我们正在大力投资开发 Crux，因为我们相信企业将继续需要离线私人交易存储解决方案，这正是 Crux 的用武之地。

如果你想聊聊 Crux 或我们正在做的任何事情，请随时通过 [Quorum Slack](https://clh7rniov2.execute-api.us-east-1.amazonaws.com/Express/) #crux 频道联系，或发送电子邮件 [hi@blk.io](mailto:hi@blk.io) 。