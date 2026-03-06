# 宣布 Crux，定额组的安全飞地

> 原文：<https://blog.web3labs.com/web3development/announcing-crux-a-secure-enclave-for-quorum>

![Crux Data privacy for Quorum](img/4f218983ad3f0579693a661a0f8ce6d8.png)

自从 Quorum 于 2016 年 11 月公开以来，我们一直在与它合作，并以某种方式为其做出贡献。不管你对 JP 摩根的看法如何，让世界上最大的金融公司之一支持以太坊区块链技术并创建 Quorum 始终是一个重要的声明，它有助于进一步巩固我们对以太坊在企业中的潜力的看法。

Quorum 本身由两部分组成——Quorum 客户机是 Geth 的一个分支，以及一个用 Haskell 编写的安全区域。为了使 Constellation 与 Geth 和 Quorum 客户端保持一致，并进入重要的 Golang 社区，我们决定在 Go 中从头开始重写它，创建 [Crux](https://github.com/blk-io/crux) 。

症结是一个下降的替代星座在法定人数区块链。它支持与 Constellation 相同的配置参数，以确保直接向前迁移。

我们已经为将来的 Crux 计划了一些很棒的增强，我们真的很想围绕它发展一个强大的社区。为了尽可能直接地使用它，我们已经有了一个版本的 Quorum 7 节点示例，它使用了 GitHub 上的 Crux [。](https://github.com/blk-io/quorum-examples)

否则，您可以前往[回购](https://github.com/blk-io/crux)并挖掘代码。

```java
git clone https://github.com/blk-io/crux.git
cd crux
make setup && make
./bin/crux

Usage of ./crux:
    crux.config              Optional config file
    --alwayssendto string    List of public keys for nodes to send all
 transactions too

    --berkeleydb             Use Berkeley DB for working with an 
existing Constellation data store [experimental]

    --generate-keys string   Generate a new keypair

    --othernodes string      "Boot nodes" to connect to to discover 
the network

    --port int               The local port to listen on (default -1)

    --privatekeys string     Private keys hosted by this node

    --publickeys string      Public keys hosted by this node

    --socket string          IPC socket to create for access to the 
Private API (default "crux.ipc")

    --storage string         Database storage file name 
(default "crux.db")

    --url string             The URL to advertise to other nodes 
(reachable by them)

    --verbosity int          Verbosity level of logs (default 1)

    --workdir string         The folder to put stuff in (default: .) 
(default ".")
```

Crux 使用 constellation 支持的相同的 NaCl 加密库，您可以使用 *generate-keys* 参数生成新的密钥:

```java
crux --generate-keys myKey
```

然后，您可以用与 Constellation 几乎相同的方式运行它:

```java
crux --url=http://127.0.0.1:9001/ --port=9001 --workdir=crux --
publickeys=tm.pub --privatekeys=tm.key --
othernodes=https://127.0.0.1:9001/
```

我们已经从 BerkeleyDB 迁移到了 Geth 使用的默认 LevelDB。但是，如果您想使用现有的 Constellation 实例，BerkeleyDB 的绑定是可用的。

最后一件事，如果你愿意加入我们，并全职致力于构建 Crux — [我们正在招聘](https://angel.co/blk-io/jobs/355875-golang-engineers)！