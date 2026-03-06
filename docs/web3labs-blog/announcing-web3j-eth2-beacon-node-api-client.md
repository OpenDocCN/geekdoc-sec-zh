# 宣布 Web3j Eth2 信标节点 API 客户端

> 原文：<https://blog.web3labs.com/web3development/announcing-web3j-eth2-beacon-node-api-client>

继最近以太坊 2.0 信标链的[发布](https://beaconcha.in/)之后，Web3 实验室非常兴奋地宣布我们对 Eth2 生态系统的第一个贡献: [Web3j 信标节点 API 客户端](https://github.com/web3j/web3j-eth2/tree/master/beacon-node-api)。这个客户端是一个瘦库，它将允许您使用 [Eth2.0-API](https://ethereum.github.io/eth2.0-APIs/) 规范中可用的方法与新的 Eth2 节点流畅地交互，包括事件订阅。作为一个 Web3j 库，我们已经尽可能地让一切变得简单，以便用户快速使用。

这个客户端是一个瘦库，它将允许您使用 [Eth2.0-API](https://ethereum.github.io/eth2.0-APIs/) 规范中可用的方法流畅地与新的 Eth2 节点交互，包括事件订阅。作为一个 Web3j 库，我们已经尽可能地让一切变得简单，以便用户快速使用。

在本文中，我们将指导您完成让您的应用程序与任何 Eth2 信标链节点交互所需的几个步骤。

## 入门指南

首先，您需要将这个库添加到您的项目中，或者使用 Maven:

```java
<dependency>
    <groupId>org.web3j.eth2</groupId>
    <artifactId>beacon-node-api</artifactId>
    <version>1.0.0</version>
</dependency>
```

或者格雷尔:

```java
implementation 'org.web3j.eth2:beacon-node-api:1.0.0'
```

在撰写本文时，第一个发布的版本 1.0.0 对应于 Eth2 规范 v0.12.2。查看 project [releases 页面](https://github.com/web3j/web3j-eth2/releases)以获得最新的可用版本。

一旦将库添加到项目中，就可以用一种熟悉的 Web3j 风格的方式创建一个指向特定信标链节点的客户机实例:

```java
var service = new BeaconNodeService(“http://...”);
var client = BeaconNodeClientFactory.build(service);
```

就是这样！此时，您可以开始调用节点端点，例如检索 head 块:

```java
client.getBeacon().getBlocks()
    .findById(NamedBlockId.HEAD);
```

或者检索所有当前证明者斜杠:

```java
client.getBeacon().getPool()
    .getAttesterSlashings()
    .findAll();
```

如果您不知道可以将客户端代码指向哪个信标链节点，在下一节中，我们将指导您完成几个简单的步骤来建立并运行测试网络。

## 启动测试网络

在本节中，我们将使用[库特](https://github.com/ConsenSys/teku)信标链实现来启动一个本地测试网络，将您的客户端代码指向该网络(在此之前，确保您正在运行一个 [Docker](https://www.docker.com/products/docker-desktop) 环境)。

首先，在本地文件夹中克隆库特项目:

```java
$ git clone https://github.com/ConsenSys/teku.git
```

然后启动信标链本地网络(查看 Docker Testnet 上的[库特文档](https://github.com/ConsenSys/teku/tree/master/test-network)了解更多信息):

```java
$ teku/test-network/launch.sh
Recreating test-network_teku1_1       ... done
Starting test-network_prometheus_1    ... done
Starting test-network_node-exporter_1 ... done
Recreating test-network_teku4_1       ... done
Recreating test-network_teku2_1       ... done
Recreating test-network_teku3_1       ... done
Starting test-network_grafana_1       ... done
```

您将获得一堆日志，但是一旦您的网络完全启动，您将能够指向端口 19601、19602、19603 和 19604 上的四个节点中的任何一个。

## 监听事件

让我们创建一个客户端，并监听测试网络中发生的一些事件:

```java
var service = new BeaconNodeService("http://localhost:19601/");
var client = BeaconNodeClientFactory.build(service);

// We want to receive at least one event
var latch = new CountDownLatch(1);

// At the moment we are interested in any topic
var topics = EnumSet.allOf(BeaconEventType.class);

// Then subscribe to each event
client.getEvents().onEvent(topics, event -> {
     System.out.println("Received event: " + event);
     latch.countDown();
});

// Wait for the event
latch.await();
```

运行这段代码时，您应该能够看到一些日志，显示您的客户机和本地节点之间的交互:

```java
00:15:47.744 [jersey-client-async-executor-0] DEBUG org.web3j.eth2.api.BeaconNodeService - 2 * Sending client request on thread jersey-client-async-executor-0
2 > GET http://localhost:19601/eth/v1/events?topics=head%2Cblock%2Cattestation%2Cvoluntary_exit%2Cfinalized_checkpoint%2Cchain_reorg
2 > Accept: text/event-stream

00:15:51.420 [jersey-client-async-executor-0] DEBUG org.web3j.eth2.api.BeaconNodeService - 2 * Client response received on thread jersey-client-async-executor-0
2 < 200
2 < Cache-Control: no-cache
2 < Connection: close
2 < Content-Type: text/event-stream;charset=utf-8
2 < Date: Thu, 10 Dec 2020 23:15:47 GMT
2 < Server: Javalin
event: head
data: {"slot":"33207","block":"0x1829e81553bb76ed92a7ae2d671e018b219b7eeee0cea80fac12b2a7d6924826","state":"0xe7339bff5fbb2a30f039a900532cb936e6afbd40ef1fd41cb645687659d8833a","epoch_transition":false,"previous_duty_dependent_root":"0xd34981d5ab59d4d091992099cedea95236dfcc2252f55f53e282ee847fb426e8","current_duty_dependent_root":"0x626db3e484c19fa8f0c57ca6c390bfa8a61313564d61f4c68764239a4e0c966e"}
```

如您所见，在运行这个例子时，我们收到的第一个事件是一个 *head* 事件。这发生在验证器委员会在每个槽(通常每隔几秒钟)验证一个头块之后。通过指定不同的信标事件类型集，可以很容易地修改上面的示例，以处理其他主题，如纪元终结性检查点、验证器证明或新块。

您现在已经开始探索 Eth2 数据了！我们很想听听你是如何找到这个图书馆的，以及你正在用它建造什么。欢迎随时进入我们的[社区论坛](https://community.web3labs.com/c/web3j/)与我们讨论，并在 https://github.com/web3j/web3j-eth2/的[查看项目文件。](https://github.com/web3j/web3j-eth2/)

随着 Eth2 的发展，我们将继续通过 Web3 实验室的 Web3j 为其提供支持。我们希望您对我们的网络发布感到兴奋，并期待在未来看到更多伟大的功能出现！