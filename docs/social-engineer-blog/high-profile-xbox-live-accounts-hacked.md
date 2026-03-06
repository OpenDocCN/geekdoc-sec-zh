# “高调”Xbox Live 帐户被黑

> 原文：<https://www.social-engineer.org/social-engineering/high-profile-xbox-live-accounts-hacked/>

the Verge[报道称](https://www.theverge.com/2013/3/19/4125886/microsoft-confirms-high-profile-employee-xbox-live-accounts-hacked "The Verge")黑客已经瞄准并成功入侵了微软过去和现在的高级员工的 Xbox Live 账户。看起来，社会工程攻击是使用一种称为链或串的技术完成的，或者我们称之为多层社会工程攻击。多层社会工程攻击的工作原理如下:

1.  攻击者有[xyz]信息
2.  攻击者使用[xyz]对公司 A 进行社交工程，使其提供[abc]信息
3.  攻击者使用[abc]对公司 B 进行社交工程，使其提供[mno]信息
4.  攻击者使用[xyz]、[abc]和[mno]信息来访问 C 公司的帐户

据来自 KrebsOnSecurity.com 的 Brian Krebs 称，Xbox live 的攻击是这样进行的:

黑客利用 http://ssndob.ru 获得了在 Xbox Live 平台上工作的微软员工的 SSN 信息。然后，黑客打电话给电话公司，使用 SSN 来转移电话。然后，他们打电话给 Xbox Live，拨打他们文件上的号码(该号码从电话公司被重定向到黑客的手机上)来验证他们的帐户信息。验证完成后，帐户密码被重置，黑客可以访问 Xbox Live 帐户。太棒了。

我们[在 2012 年 8 月 7 日](https://www.social-engineer.org/general-blog/social-engineering-and-weak-security-affect-apple-and-amazon/ "Social-Engineer.Org Blog")报道了这种类型的攻击，当时 Mat Honan 的 iCloud 账户被黑，他的整个数字生活被摧毁。似乎同一个被称为“恐惧症”的黑客可能对这两起多层社会工程攻击负责。此外，Ars Technia 报告称，Phobia 也可能对针对 Ars 站点发起的拒绝服务攻击负责，该攻击也是由 Brian Krebs 发现的。

在给 The Verge 的一份声明中，微软承认“少数由现任和前任微软员工持有的高调 Xbox LIVE 账户”遭到了泄露。微软随后就此事件发表了一份声明。

“我们意识到，一群攻击者正在使用几种字符串社会工程技术来破坏少数由现任和前任微软员工持有的高调 Xbox LIVE 帐户的帐户。我们正在与执法部门和其他受影响的公司积极合作，以禁用这种当前的攻击方法，并防止其进一步使用。安全对我们来说至关重要，我们每天都在努力为我们的会员带来新形式的保护。”

“Microsoft 不会在其服务中收集或使用社会安全号码，包括 Xbox LIVE 玩家代号或 Microsoft 帐户。攻击者通过社交工程将目标锁定在知名的微软员工，其他公司也使用这些数据来拦截来自微软的安全证明，以危及账户安全。”

多层次的[社会工程攻击](https://www.social-engineer.org/framework/general-discussion/categories-social-engineers/hackers/ "Hackers")越来越受欢迎，并带来了毁灭性的结果。随着公司开始了解社会工程攻击，并实施适用于自己公司的安全程序，需要将思考置于更大的全球背景下。如果从组织外部恶意获取[xyz]数据，我们的安全协议是否足够？这些载体刚刚开始渗透到负责保护我们的数据和数字生活的安全专业人员和组织的思想中。