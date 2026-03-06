# ABIv2 来到 Web3j

> 原文：<https://blog.web3labs.com/web3development/abiv2-comes-to-web3j>

我们刚刚发布了 Web3j 的一个重要里程碑——支持 Solidity ABIv2 规范！在这篇文章中，我将深入探讨 ABIv2 对作为智能合约开发者的你意味着什么，以及 Web3j 是如何支持它的。

Web3j 中最强大的特性之一是生成智能契约包装器的能力，它提供了在 Java 中以类型安全的方式使用 Solidity 智能契约的能力。这是可能的，因为 solidity 编译器的输出提供了一个 ABI(应用程序二进制接口),它本质上是一个描述智能合约中的函数的 JSON 文件。Solidity 编译器中的 ABI 编码器允许将结构用作输入/输出/事件参数。然而，对于“pragma experimental abiencoder v2；”指令可以启用这个特性。

## 一个例子

当在智能协定中启用 ABIv2 时，您可以开始使用结构作为函数或事件的参数。使用这些功能的一个简单的契约示例如下所示:

```java
pragma solidity ^0.6.5;
pragma experimental ABIEncoderV2;

contract ExampleContract {

struct Foo {
    string id;
    string name;
}

struct Bar {
    uint id;
    string data;
}

Foo foo;
Bar bar;

constructor(Foo memory _foo, Bar memory_bar) public {
foo = _foo;
bar = _bar;
emit Event(msg.sender, _foo, _bar);
}


function setFoo(Foo memory _toSet) public {
     foo = _toSet;
}


function setBar(Bar memory _toSet) public {
     bar = _toSet;
}


function getFooBar() public view returns (Foo memory, Bar memory) {
     return (foo, bar);
}


event Event(address indexed _address, Foo _foo, Bar _bar);
}
```

这个契约有两个结构叫做 **Foo** 和 **Bar** 。 **Foo** 包含两个都是字符串的字段，而 **Bar** 包含一个无符号整数和一个字符串。这些只是简单的示例结构，但也可能使用其他类型，如字节数组、地址，甚至可能有任意嵌套的结构。换句话说，可以定义一个结构 A，它有一个由结构 b 表示的字段。

```java
constructor(Foo memory _foo, Bar memory _bar) public {
    foo = _foo;
    bar = _bar;
    emit Event(msg.sender, _foo, _bar);
}
```

上面的例子有一个构造函数，它将一个 **Foo** 和一个 **Bar** struct 作为输入参数。构造函数将输入参数设置为契约中的全局字段，然后发出一个事件，该事件存储传入的 **Foo** 和 **Bar** 结构。

```java
function setFoo(Foo memory _toSet) public {
    foo = _toSet;
}


function setBar(Bar memory _toSet) public {
    bar = _toSet;
}


function getFooBar() public view returns (Foo memory, Bar memory) {
    return (foo, bar);
}
```

还有三个函数，其中两个分别是 **Foo** 和 **Bar** 结构的设置器。它们将 struct 作为输入参数。最后一个函数是 getter，它返回一个表示契约中全局字段的 **Foo** 和 **Bar** 结构的元组。

## ABI

一旦用 Solidity 编译器编译了上述契约，它将生成一个 ABI 文件，该文件实质上是一个 JSON 对象列表，代表了示例契约中的每个函数和事件。它为每个函数和事件定义了每个输入和输出参数。在上面的示例契约中，代表 **setFoo** 函数的 JSON 对象将如下所示:

```java
{
  "inputs": [
    {
      "components": [
        {
          "internalType": "string",
          "name": "id",
          "type": "string"
        },
        {
          "internalType": "string",
          "name": "name",
          "type": "string"
      }
    ],
    "internalType": "struct ExampleContract.Foo",
    "name": "_toSet",
    "type": "tuple"
   }
  ],
  "name": "setFoo",
  "outputs": [],
  "stateMutability": "nonpayable",
  "type": "function"
}
```

这个 JSON 对象有一个带有单个对象的**输入**属性。你可以看到输入的**类型**属性是“元组”，但这并没有太大的帮助，因为我们只能通过确定不同的元组有不同的组成来区分它们。自从 Solidity 版本 0.6.0 以来，还有一个**内部类型**属性，它为我们提供了可以在 Solidity 上下文中找到的参数类型。

## Web3j

要为这个函数生成类型安全的 Java 代码，我们必须生成一个对应于 struct **Foo** **的类。** Web3j 的智能合同生成器将生成这样一个类:

```java
public static class Foo extends DynamicStruct {
  public String id;


public String name;


public Foo(String id, String name) {
    super(new org.web3j.abi.datatypes.Utf8String(id), new org.web3j.abi.datatypes.Utf8String(name));
    this.id = id;
    this.name = name;
}


public Foo(Utf8String id, Utf8String name) {
    super(id, name);
    this.id = id.getValue();
    this.name = name.getValue();
  }
}
```

Struct **Foo** 包含两个动态类型的字符串。因此 java 类 **Foo** 扩展了 **DynamicType** 。这使得类型编码器/解码器能够以相同的方式处理所有动态结构。

有两个构造器，一个具有 java 类型，一个具有 Web3j Solidity 包装器类型(即 **Utf8String** 实现**类型**)。Java 类型是为了方便开发人员，所以可以很容易地构造一个 **Foo** 的实例来进行契约调用。类型解码器使用另一个构造函数从约定调用的响应中构造一个 **Foo** 的实例。

## 筑巢挑战

构建一个支持结构的类型安全生成器面临许多挑战。最大的挑战之一是解析 ABI 以找到唯一的嵌套结构。这对于用 Solidity v 0.6.x 编译的契约来说相对容易，因为 **internalType** 字段公开了关于使其唯一的结构的信息。然而，如果 ABI 没有这一领域，问题就变得复杂得多。

### 获得# Buidling

从版本 4.6.0 开始，Web3j 中提供了 ABIv2 支持。我们鼓励你开始使用它，并充分利用这个伟大的新功能。

如果您想快速启动并运行，我鼓励您查看我们的 [Epirus SDK](https://www.web3labs.com/epirus-platform) ,它让项目变得非常简单！