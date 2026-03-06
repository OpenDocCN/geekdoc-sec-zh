# 宣布 web3j Gradle 插件

> 原文：<https://blog.web3labs.com/web3development/announcing-the-web3j-gradle-plugin>

在这里的 [blk.io](https://blk.io/) 我们很高兴地宣布 web3j [Gradle 插件](https://github.com/web3j/web3j-gradle-plugin)的全面上市。自从 web3j 增加了对智能合同绑定的支持，我们就一直希望有一个 Gradle 插件。我们很幸运 [Heinz Marti](https://github.com/h2mch) 开发了(并继续维护)一个 [Maven 插件](https://github.com/web3j/web3j-maven-plugin)，但是作为一个 Gradle 的长期用户，它真的觉得这是 Java、Android 和 Kotlin 开发者缺少的一个关键工具。

该插件自动负责编译和创建 Java 绑定，因此您可以在 IDE 中做任何事情(或者如果您喜欢 vim/emacs，只需两个终端窗口)。

## ![Web3j Gradle plugin](img/16639deca75704da8a9865706d40dff8.png)

## 入门指南

要开始，只需使用插件 DSL 语法将 web3j 插件添加到您的`gradle.build`文件中(推荐):

```java
plugins {
    id 'org.web3j' version '0.1.4'
}
```

或者，如果您使用的是旧风格的插件约定:

```java
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
           classpath 'org.web3j:web3j-gradle-plugin:0.1.4'
    }
}

apply plugin: 'web3j'
```

现在，我们的项目只需要将我们的 Solidity 代码放在项目的一个`solidity`目录中。

![Run gradlew build](img/c64ff02d30ee31f4ed0fb2542c825d3c.png)

现在只需运行 gradlew build 让插件完成它的工作！

当插件作为构建阶段的一部分运行时，您可以看到由 Solidity 编译器生成的 Solidity 二进制文件和 ABI 文件，以及项目`build`目录中的 web3j 智能契约绑定。

![Solidity ABI and binary files, plus Java contract bindings](img/3732f30ece22c0246c328007c20cfc08.png)

*Our automatically generated Solidity ABI and binary files, plus Java contract bindings*

## 示例项目

在这个例子中，我们使用了 web3j [示例项目](https://github.com/web3j/sample-project-gradle)，我们已经对其进行了更新，以使用该插件！

## 在幕后

该插件向 Gradle 构建阶段添加了许多任务，可以通过运行 grad le tasks-all 来查看这些任务。

```java
compileSolidity - Compiles Solidity contracts for main source set.
compileTestJava - Compiles test Java source.
compileTestSolidity - Compiles Solidity contracts for test source set.
generateContractWrappers - Generates web3j contract wrappers for main source set.
generateTestContractWrappers - Generates web3j contract wrappers for test source set.
```

这些任务是不言自明的，然而，插件本身是模块化的，因为实度编译阶段是由一个附加的 web3j [实度梯度](https://github.com/web3j/solidity-gradle-plugin)插件执行的。这意味着，如果你只是想在你的项目中执行可靠性编译，你可以使用专用插件。

如果您的项目不使用默认的标准配置，您也可以配置许多属性。如果您有任何进一步改进的建议，请告诉我们！

最后感谢 [Xavier](https://github.com/xaviarias) 做了所有的重活！