# 如何安装 Web3j

> 原文：<https://blog.web3labs.com/web3development/how-to-install-web3j>

If you are wondering how to install web3j on your machine then look no further. This post will walk you through everything you need to know about the various ways you can get web3j into your project in a few easy steps.

## 项目相关性

Web3j 发布在 Maven 上的 [Maven Repository](https://mvnrepository.com/artifact/org.web3j) 中，因此可以作为一个依赖项添加到任何项目中，该项目实际上使用了任何 Java 开发中已知的构建工具。
一些例子包括:

### **胃**

要将 web3j 添加到 maven 项目中，可以在 pom.xml 文件中包含以下内容:

```java
<!-- https://mvnrepository.com/artifact/org.web3j/core -->
<dependency>
<groupId>org.web3j</groupId>
<artifactId>core</artifactId>
<version>4.9.1</version>
</dependency>
```

```java
implementation group: 'org.web3j', name: 'core', version: '4.9.1’
```

**等级**:】每个模块负责一个功能领域，如核心功能、编码和解码等。每个模块都被配置为单独发布。您可能已经注意到在上面的例子中引用了核心模块。根据项目的需要，每个模块都可以从 Maven 中单独取出。模块的完整列表可以在 [Maven 仓库中找到。](https://mvnrepository.com/artifact/org.web3j)

**通过插件**
注入的 Web3j Gradle 插件可以让你编译 Solidity 智能合约，生成 Java 包装器。这些包装器是 java 类，使开发人员可以轻松地部署他们的链上智能契约或与之交互。该插件还将 web3j 注入到用户的项目中，使其易于与以太坊交互。

要通过 gradle 将插件添加到您的项目中，请在 gradle.build 文件的 plugin 部分添加以下内容:

```java
id "org.web3j" version "4.9.0"
```

通过将插件添加到您的项目中，一些新的 Gradle 任务应该可供使用。可以在这里详细查看: [Gradle 插件提供任务编译 Solidity 契约](https://github.com/web3j/solidity-gradle-plugin)。

**使用 Web3j-CLI**
使用 Web3j 的另一个好方法是通过命令行界面。CLI 捆绑了强大的功能，旨在通过提供大量入门所需的脚手架来简化开发过程。要安装 CLI，您需要在基于 Unix 的系统的终端上粘贴以下行:

```java
curl -L get.web3j.io | sh
```

**Powershell 中的 Windows**

```java
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/web3j/web3j-installer/master/installer.ps1'))
```

使用命令行界面，用户可以生成一个 wallet 或一个新项目，甚至可以为一个 ERC20 令牌设置整个后端。更多信息请访问[命令行工具- Web3j](https://docs.web3j.io/4.8.7/command_line_tools/) 。

有任何问题或意见吗？我们希望收到您的来信！如果你想更多地了解区块链，它的发展和最新的发展，那就去看看我们的 [dev 博客](/web3development)或者听听我们启发性的 [Web3 创新者播客](https://podcast.web3labs.com/)。