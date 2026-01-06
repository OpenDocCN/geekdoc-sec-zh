# 匿名通信

> 原文：[`intensecrypto.org/public/lec_23_anonymous.html`](https://intensecrypto.org/public/lec_23_anonymous.html)

*如有任何错误/错别字/令人困惑的解释？[在 GitHub 上打开问题](https://github.com/boazbk/crypto/issues/new)。您也可以在下面评论*

**★ 另请参阅本章的 [**PDF 版本**](https://files.boazbarak.org/crypto/lec_23_anonymous.pdf)（更好的格式/参考文献）★

加密旨在保护通信内容，但有时更大的秘密是通信本身的存在。如果一个告密者想要向《纽约时报》泄露一些信息，她发送电子邮件的事实本身就会揭露她的身份。有两个主要概念旨在实现匿名性：

+   *匿名路由* 是确保爱丽丝和鲍勃可以通信而不泄露这一事实。

+   *隐写术* 是关于爱丽丝和鲍勃在看似无害的对话中隐藏加密通信。

## 隐写术

在隐写通信中的目标是隐藏加密（或非加密）内容而不被察觉。想法很简单：让我们从**对称情况**开始，假设爱丽丝和鲍勃共享一个密钥 \(k\)，并且爱丽丝想要向鲍勃传输一个比特 \(b\)。我们假设爱丽丝有 \(t\) 个词 \(w_1,\ldots,w_t\) 的选择，这些词在这个对话的这个时候是合理的。爱丽丝将选择一个词 \(w_i\)，使得 \(f_k(w_i)=b\)，其中 \(\{ f_k \}\) 是一个伪随机函数集合。以概率 \(1-2^{-t}\) 存在这样的词。鲍勃将使用 \(f_k(w_i)\) 解码信息。爱丽丝和鲍勃可以使用纠错码来补偿爱丽丝被迫发送错误比特的概率 \(2^{-t}\)。

在**公钥设置**中，假设鲍勃发布了一个加密方案的公钥 \(e\)，该方案具有**伪随机密文**。也就是说，对于一个不知道密钥的第三方，加密是不可区分于随机字符串的。为了向鲍勃发送一些消息 \(m\)，爱丽丝计算 \(c = E_e(m)\) 并逐比特传输给鲍勃。给定 \(t\) 个词 \(w_1,\ldots,w_t\)，为了传输比特 \(c_j\)，爱丽丝选择一个词 \(w_i\)，使得 \(H(w_i)=c_j\)，其中 \(H:\{0,1\}^*\rightarrow\{0,1\}\) 是一个作为随机预言机建模的哈希函数。爱丽丝输出的词的分布 \(w¹,\ldots,w^\ell\) 在 \((H(w¹),\ldots,H(w^\ell))=c\) 的条件下是均匀的。但请注意，如果 \(H\) 是一个随机预言机，那么 \(H(w¹),\ldots,H(w^\ell)\) 将是均匀的，因此与 \(c\) 不可区分。

## 匿名路由

+   **低延迟通信:** Aqua, Crowds, LAP, ShadowWalker, Tarzan, Tor

+   **一次发送一条消息，防止时间/流量分析:** Mix-nets, e-voting, Dining Cryptographer network (DC net), Dissent, Herbivore, Riposte

## Tor

基本架构。攻击

## Telex

## Riposte

## 评论

评论通过[GitHub 仓库](https://github.com/boazbk/crypto/issues)使用[utteranc.es](https://utteranc.es)应用程序发布。发表评论需要 GitHub 登录。如果您不想授权应用程序代表您发布，您也可以直接在[此页面的 GitHub 问题](https://github.com/boazbk/crypto/issues?q=Anonymous%20communication%20in%3Atitle)上评论。

编译于 2021 年 11 月 17 日 22:36:10

版权所有 2021，Boaz Barak。![Creative Commons License](https://creativecommons.org/licenses/by-nc-nd/4.0/)

本作品受[Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/)许可。

使用[pandoc](https://pandoc.org/)和[panflute](http://scorreia.com/software/panflute/)以及从[gitbook](https://www.gitbook.com/)和[bookdown](https://bookdown.org/)中提取的模板制作。**
