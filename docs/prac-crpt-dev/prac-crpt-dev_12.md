# 量子安全加密

> 原文：[`cryptobook.nakov.com/quantum-safe-cryptography`](https://cryptobook.nakov.com/quantum-safe-cryptography)

**量子计算机**是 ...

+   TODO

+   TODO

查看此页面：[`ianix.com/pqcrypto/pqcrypto-deployment.html`](https://ianix.com/pqcrypto/pqcrypto-deployment.html)

+   TODO

**量子计算**是一种基于量子物理学的计算模型，其工作方式与经典计算机不同，可以完成经典计算机无法完成的事情，例如有效地破解 RSA 和 ECC。**量子计算机**不是“更快的计算机”，它们不是全能的，也不能更快地完成任何计算任务。量子计算机在某些问题上非常高效，而在其他问题上则相当弱。

在计算机科学中，众所周知，**量子计算机将破坏一些加密算法**，特别是依赖于**IFP**（整数分解问题）、**DLP**（离散对数问题）和**ECDLP**（椭圆曲线离散对数问题）的公钥加密系统，如**RSA**、**ECC**和**ECDSA**。量子算法不会是密码学的终结，因为：

+   只有某些加密系统是**量子不安全的**（如 RSA、DHKE、ECC、ECDSA 和 ECDH）。

+   一些加密系统是**量子安全的**，并且只会受到轻微的影响（如加密散列、MAC 算法和对称密钥加密）。

让我们详细讨论这个问题。

## [](#quantum-safe-and-quantum-broken-crypto-algorithms)量子安全和量子破坏的加密算法

大多数加密**散列**（如 SHA2、SHA3、BLAKE2）、**MAC**算法（如 HMAC 和 CMAK）、**密钥派生函数**（bcrypt、Scrypt、Argon2）基本上是**量子安全的**（仅受到量子计算的一点点影响）。

+   使用 384 位或更多来确保量子安全（256 位对于长期来说应该足够）

**对称加密**（如 AES-256、Twofish-256）是**量子安全的**。

+   使用 256 位或更长的密钥长度（不要使用 128 位 AES）

大多数流行的**公钥加密系统**（如 RSA、DSA、ECDSA、EdDSA、DHKE、ECDH、ElGamal）都是**量子破坏的**！

+   大多数**数字签名**算法（如 RSA、ECDSA、EdDSA）都是**量子破坏的**！

+   **量子安全的签名算法**和公钥加密系统已经开发出来（例如基于格或基于散列的签名），但由于比 ECC 有更长的密钥和更长的签名，所以并没有被广泛使用。

查看[`en.wikipedia.org/wiki/Post-quantum_cryptography`](https://en.wikipedia.org/wiki/Post-quantum_cryptography)

...

量子抵抗加密算法

...

### [](#ecc-cryptography-and-most-digital-signatures-are-quantum-broken)ECC 密码学和大多数数字签名都是量子破坏的！

...

一个**k**位数字可以使用**5k+1**个**量子比特**的量子计算机在**O(k³**)的时间内进行分解（使用 Shor 算法）。

+   查看[`www.theory.caltech.edu/~preskill/pubs/preskill-1996-networks.pdf`](http://www.theory.caltech.edu/~preskill/pubs/preskill-1996-networks.pdf)

256 位数字（例如比特币公钥）可以使用 1281 个量子比特在 72*256³ 次量子操作中分解。

+   ~ 1.2 亿次操作 == ~ 不到 1 秒使用好的机器

ECDSA、DSA、RSA、ElGamal、DHKE、ECDH 密码系统都是量子破解的

结论：发布已签名的交易（如以太坊所做）不是量子安全的 -> 避免泄露 ECC 公钥

### [](#hashes-are-quantum-safe)哈希是量子安全的

密码学**哈希**（如 SHA2、SHA3、BLAKE2）被认为是**量子安全**的：

+   在传统计算机上，找到 256 位哈希的碰撞需要√2²⁵⁶ 步（使用[**生日攻击**](https://en.wikipedia.org/wiki/Birthday_attack)) -> SHA256 有 2¹²⁸ 的密码强度

+   量子计算机可能在∛2²⁵⁶ 次操作中找到哈希碰撞（参见[the BHT 算法](https://arxiv.org/pdf/quant-ph/9705002.pdf))，但这是有争议的（参见[Bernstein 2009] - [`cr.yp.to/hash/collisioncost-20090823.pdf`](http://cr.yp.to/hash/collisioncost-20090823.pdf)）

+   理论上可能需要 2⁸⁵ 次量子操作来找到 SHA256 / SHA3-256 的碰撞，但在实践中可能需要更多。

结论：SHA256 / SHA3-256 很可能是量子安全的

+   SHA384、SHA512 和 SHA3-384、SHA3-512 是量子安全的

...

### [](#symmetric-ciphers-are-quantum-safe)对称密码是量子安全的

...

大多数对称密码（如 AES 和 ChaCha20）是量子安全的：

+   [Grover 算法](https://en.wikipedia.org/wiki/Grover's_algorithm)通过√𝑁量子操作找到 AES 密钥

+   量子时代将对称加密的密钥大小**加倍**，参见[`cr.yp.to/codes/grovercode-20100303.pdf`](http://cr.yp.to/codes/grovercode-20100303.pdf%29%29)

后量子时代中的 AES-256 就像之前的 AES-128 一样

+   128 位或更少对称加密可被量子攻击

结论：256 位对称加密通常量子安全

+   AES-256, ChaCha20-256, Twofish-256, Camellia-256 被认为是量子安全的

## [](#post-quantum-cryptography)后量子密码学

...

量子安全密钥协商：[`en.wikipedia.org/wiki/CECPQ1`](https://en.wikipedia.org/wiki/CECPQ1)

[`ianix.com/pqcrypto/pqcrypto-deployment.html`](https://ianix.com/pqcrypto/pqcrypto-deployment.html)

[`pqcrypto.org`](https://pqcrypto.org/)

后量子签名方案 XMSS：

+   [`tools.ietf.org/html/rfc8391`](https://legacy.gitbook.com/book/svetlin-nakov/practical-cryptography-for-developers/edit#)

+   JS XMSS - [`www.npmjs.com/package/xmss`](https://www.npmjs.com/package/xmss)

+   后量子密钥协商方案 McEliece 和 NewHope

后量子签名和密钥协商（XMSS、McEliece、NewHope）：[`github.com/randombit/botan`](https://github.com/randombit/botan)

QC-MDPC 和 libPQC 已被量子破解：[`eprint.iacr.org/2016/858.pdf`](https://eprint.iacr.org/2016/858.pdf)

### [](#hash-based-public-key-cryptography)基于哈希的公钥密码学

...

### [](#code-based-public-key-cryptography)基于代码的公钥密码学

...

### [](#lattice-based-public-key-cryptography)基于格的公钥密码学

...

**GLYPH**签名（基于格的环-LWE 格，环-LWE，环错误学习）

+   Go 语言实现：[`github.com/AidosKuneen/glyph`](https://github.com/AidosKuneen/glyph)

**BLISS** - [`bliss.di.ens.fr`](http://bliss.di.ens.fr)

+   Go 语言实现：[`github.com/HcashOrg/bliss/blob/master/demo_test.go`](https://github.com/HcashOrg/bliss/blob/master/demo_test.go)

**NewHope**

+   Go 语言实现：[`github.com/Yawning/newhope`](https://github.com/Yawning/newhope)

+   Python 实现：[`github.com/scottwn/PyNewHope`](https://github.com/scottwn/PyNewHope)

+   Python 实现：[`github.com/anupsv/NewHope-Key-Exchange`](https://github.com/anupsv/NewHope-Key-Exchange)

**XMSS**

+   Python 实现：[`github.com/theQRL/QRL/blob/master/src/qrl/crypto/xmss.py`](https://github.com/theQRL/QRL/blob/master/src/qrl/crypto/xmss.py)

**NTRU**：NTRUEncrypt 和 NTRUSign

+   [`en.wikipedia.org/wiki/NTRUEncrypt`](https://en.wikipedia.org/wiki/NTRUEncrypt)

### [](#zero-knowledge-proof-based)基于零知识证明

PICNIC - [`github.com/Microsoft/Picnic`](https://github.com/Microsoft/Picnic)

### [](#multivariate-quadratic-equations-public-key-cryptography)多元二次方程公钥密码学

彩虹：[`github.com/bcgit/bc-java/tree/master/core/src/main/java/org/bouncycastle/pqc/crypto/rainbow`](https://github.com/bcgit/bc-java/tree/master/core/src/main/java/org/bouncycastle/pqc/crypto/rainbow)

...

## [](#quantum-resistant-cryptography-libraries)量子抗性加密库

量子安全加密技术仍在发展中，尚未成熟，并且目前大多数加密库和工具（如 Web 浏览器、OpenSSL、OpenSSH 等）尚未广泛支持。以下是一些发展良好的量子加密算法库列表：

+   **liboqs** (Open Quantum Safe) - [`github.com/open-quantum-safe/liboqs`](https://github.com/open-quantum-safe/liboqs)

+   **Bouncy Castle PQC** - [`github.com/bcgit/bc-java/tree/master/core/src/main/java/org/bouncycastle/pqc/crypto`](https://github.com/bcgit/bc-java/tree/master/core/src/main/java/org/bouncycastle/pqc/crypto)

## [](#sphincs-signatures-in-python)Python 中的 SPHINCS+签名

[`github.com/sphincs/pyspx`](https://github.com/sphincs/pyspx)

[`pypi.org/project/PySPX`](https://pypi.org/project/PySPX/)

### [](#newhope-key-exchange-in-python)Python 中的 NewHope 密钥交换

[`github.com/anupsv/NewHope-Key-Exchange`](https://github.com/anupsv/NewHope-Key-Exchange)

[`github.com/scottwn/PyNewHope`](https://github.com/scottwn/PyNewHope)
