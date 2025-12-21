# 更多加密概念

> 原文：[`cryptobook.nakov.com/more-cryptographic-concepts`](https://cryptobook.nakov.com/more-cryptographic-concepts)

...

## [](#digital-certificates-the-x.509-standard-and-pki)数字证书、X.509 标准和 PKI

...

[`cryptography.io/en/latest/x509/`](https://cryptography.io/en/latest/x509/)

## [](#transport-layer-security-tls-and-ssl)传输层安全性（TLS）和 SSL

...

[`en.wikipedia.org/wiki/Transport_Layer_Security`](https://en.wikipedia.org/wiki/Transport_Layer_Security)

密码套件是一组算法，有助于保护使用传输层安全性（TLS）或其已弃用的前身安全套接字层（SSL）的网络连接。密码套件通常包含的算法集包括：一个**密钥交换**算法、一个**对称加密**算法和一个消息**认证码**（MAC）算法。

## [](#external-authentication-and-oauth)外部身份验证和 OAuth

...

## [](#two-factor-authentication-and-one-time-passwords)双因素身份验证和一次性密码

多因素身份验证增加了额外的身份验证层。通常，这些因素应该添加到以下类别之一：

+   我所知道的

+   我拥有的

+   我是谁

双因素身份验证需要实施这三个类别中的两个。最常见的情况是**双因素身份验证**需要一个**用户密码**和一个设备，该设备将发送/生成**一次性密码**。生成一次性密码（OTP）时使用基于 HMAC 的一次性密码算法。

### [](#hmac-based-one-time-password-hotp)基于 HMAC 的一次性密码（HOTP）

**HOTP**算法基于[HMAC](https://en.wikipedia.org/wiki/HMAC)并提供了一种对称生成人类可读密码的方法，每个密码仅用于一次身份验证尝试。HOTP 的关键参数是一个必须在各方之间预先交换的秘密：

<template id="B:2"></template>
