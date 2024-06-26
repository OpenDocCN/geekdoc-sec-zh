# 序言

这是本书的第二版。 虽然网络安全问题不断增加，但关于网络安全的入门书籍仍然非常少。 原因很简单：

+   大多数网络安全专家拿得太多了，以至于没有时间写书。

+   我们中的大多数人确实很忙。

+   很少有人能够对自己所做的事情了解得足够透彻，以至于敢于写一本关于这个主题的书来冒险自己的声誉。

我们还必须保持更新。 这个主题领域正在迅速发展。

当我准备第二版时，全球仍然没有就如何书写“网络安全”一词达成共识。它是一个词还是两个词？在美国，国土安全部（DHS）、国家标准与技术研究所（NIST）和 ISACA（信息安全审计与控制协会）使用单个词版本。 这本书也是如此。在英国，他们仍然将该术语分成两个词：网络安全。

每年参加多次信息安全和网络安全会议，通常作为发言人，我开始意识到与成百上千的专业人士讨论时，公共领域中提供的简明可靠信息是多么少。 由于信息和网络安全专业人员不断努力跟上最新的威胁，以及如何有效地衡量、管理和监视它们，他们创建的书籍和其他资源往往面向其他信息技术（IT）专业人员。

但是现在科技和数字设备已经成为任何组织的核心部分，并且对大多数人在个人层面至关重要，我意识到几乎每个人都希望更好地了解这个主题。 由于网络安全现在影响着每个人，我看到了对非技术人员易于理解的基础信息的需求。

本书探讨了网络安全学科以及它在所有类型的企业中应该如何运作。 如果您只是寻求有关如何保护个人网络安全的指导-请尝试我的其他出版物“如何在线保护资料安全”。

我的目标是创造比其他可用文本更少技术性和更具信息性的东西，为您提供如何需要网络安全，其含义是什么以及如何有效控制和减轻相关问题的方法提供简单的洞见。

因此，本书旨在成为任何想要迅速全面了解该主题领域的人的一站式必读文本。 您不需要任何先前的技术知识来理解文本。 每当使用技术术语时，您通常会在其第一次使用下面找到一个简单的非技术英语定义（否则-您总是可以在书的末尾查找该术语）。

尽管我在安全和合规方面工作了十多年，但直到 2009 年我才开始需要专门审查和审核网络安全。我很幸运能够获得全球最大公司之一的赞助，以审查他们的内部控制和那些为他们最重要的供应商所需的控制。

我早期的一个任务之一是准备一份关于亚马逊网络服务、Salesforce 和其他基于互联网的平台的能力和限制的白皮书。赞助公司和主题的重要性使我能够接触到一些世界上最好的网络安全人才，并迅速而早期地了解网络风险以及如何减轻它们。

大约在同一时间，一家财富 50 强公司委托我编制了他们的第一个版本的一套同步的治理控制，以满足他们所有主要的全球安全、隐私和合规要求。这需要审查、组织、解构和重构超过 9,000 个控制。最终的库大小不到原始库的 5%（不到 400 个控制），但仍然满足了每一个相关的要求。

这种对控制的和谐性的练习，再加上我对操作环境的频繁实际审查，使我对已知的网络安全风险及其如何减轻或消除这些风险有了深入的了解。

快进到现在，任何组织现在都需要一个特定的网络安全政策文件。几年前没有人有这样的文件。

制定网络安全政策文档的目的是证明已充分考虑了与技术相关的所有适当的风险和控制因素。2012 年，令我惊讶的是，当我回顾 2009 年的治理控制时，几乎每个组成部分都需要一个网络安全政策所需的。几乎唯一缺少的细节是需要整合一个总体文档，以证明所有这些组成部分都存在。到了 2017 年，一个三年前的治理政策将会严重过时。不断演变的威胁现在经常导致新的和额外的安全要求。

尽管许多保护技术的主要方法保持不变，但攻击的复杂性正在推动安全控制的大幅扩展，以保护、检测、阻止、调查和恢复未经授权的访问尝试。

我们仍处于网络安全的黎明时期。仍然有许多防御不力的组织（和个人）继续受到更有经验的对手的威胁。现在技术发展的速度会使这种情况变得更糟而不是更好。如果无法安全地利用新技术，那么传统公司很容易落后。相反，更多你之前从未听说过的公司开始进入《财富》500 强、富时 100 和其他榜单，因为他们精通技术，并专注于安全地、更快速地利用和实施新技术。

全球范围内也有大量的网络安全工作机会 - 2016 年的专家估计大约有一百万个这样的工作岗位仍然空缺。几乎所有需要网络安全专业人员的组织都在努力寻找合适的候选人。事实上，直到 2013 年左右，很少有人专门从事这个领域的工作，合格专业人员的数量还没有跟上需求。

当组织在招聘岗位时，在必填部分写上‘必须至少有 10 年的网络安全经验’，这让网络安全人员感到好笑。这种经验水平很少存在，即使存在也几乎无关紧要。我们也不会申请这样的工作，除非我们‘喜欢受罪’（一种英国的讽刺表达，暗示这个人喜欢给自己带来痛苦和折磨）。谁愿意接受为一个连现代网络安全行业基本理解都缺乏的企业工作的挑战呢？没有人想成为一个替罪羊。

在接下来的几年里，我们将经常（目前几乎每天）看到一些真正引人注目的故事登上主流媒体，报道下一个因网络安全防御漏洞而受到影响的组织。自我写下这本书的第一版以来，这些数据泄露事件包括从美国人事管理办公室窃取的数百万个个人详细信息，以及从雅虎窃取的十亿个帐户详细信息。那么这是怎么发生的呢？

这种情况发生是因为，就像任何新事物一样，我们在使用数字设备方面还不够成熟和稳定。如果你想象一下汽车早期的情况，当时关于方向盘放在哪里都没有明确的想法，所以在许多车型上最初是放在车辆中心的。当时没有安全带、滚笼或气囊。人们只是惊讶于汽车在没有绑在前面的马的情况下移动，所以他们开始称之为‘无马马车’。

当前的数字时代很像汽车的早期时代。现在的技术世界就像是野蛮西部。一场新的淘金热。公司往往在不知不觉中把一切都押在他们连接到数字生态系统的每项新技术上。

大多数人在采用新技术方面所应用的速度和预算往往是一种以风险为基础的赌博。这本书将帮助你更好地了解这些风险以及如何控制它们。迅速使用正确的新技术，你可以获得巨大的好处。在使用技术及其安全性之前花费时间和金钱进行验证，你将更安全，但你可能会落后于竞争对手。

所以，如果你真的想要了解网络安全学科，包括其风险以及如何控制它们，请继续阅读。
