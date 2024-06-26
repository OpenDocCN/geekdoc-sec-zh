# 第一章：社会工程的基本概念与坏人的使用

社会工程是一种极其有效的攻击过程，超过 80% 的网络攻击，其中超过 70% 是来自国家级别的，都是通过利用人类而不是计算机或网络安全漏洞发起和执行的。因此，要构建安全的网络系统，不仅需要保护构成这些系统的计算机和网络，还需要对其人类用户进行安全程序的教育和培训。

针对人类的攻击被称为社会工程，因为它们操纵或设计用户执行所需的行动或泄露敏感信息。最一般的社会工程攻击只是试图让毫不知情的互联网用户点击恶意链接。更有针对性的攻击试图诱使组织中的敏感信息，如密码或私人信息，或者通过获得不应有的信任而从特定个人那里窃取有价值的东西。

这些攻击通常要求人们执行攻击者希望诱使受害者执行的所需行为。为了做到这一点，他们需要受害者的信任，这通常是通过互动获得的，或者通过复制或窃取身份而被利用。根据攻击的复杂程度，这些攻击将针对个人、组织或广泛的人群。骗子经常使用熟悉的公司名称或假装是受害者认识的人。2018 年的一个现实例子利用了 Netflix 的名字，当时向未知数量的收件人发送了一封旨在窃取个人信息的电子邮件。该邮件声称用户的帐户因为 Netflix 当前的结算信息有些问题而暂停，并邀请用户点击链接更新他们的付款方式。¹

社会工程攻击之所以奏效，是因为用户很难验证他们收到的每一条通信。此外，验证需要一定水平的技术专业知识，而大多数用户缺乏这方面的知识。更糟糕的是，能够访问特权信息的用户数量通常很大，从而形成了相应大规模的攻击面。²

说服个人透露敏感信息并将其用于恶意企图的行为已有很长时间了。自互联网出现以来，社会工程学攻击就一直存在。但在互联网发展之前，犯罪分子利用电话、邮政服务或广告来冒充受信任的代理以获取信息。大多数人认为，“网络钓鱼”这个词起源于 20 世纪 90 年代中期，当时它被用来描述获取互联网服务提供商（ISP）账户信息。然而，这个术语已经发展到包括各种攻击，目标是个人或企业拥有的信息，包括电话、电子邮件、社交媒体、亲自观察、游戏平台、邮递信件或包裹的盗窃，以及古老的喜好，即垃圾箱或垃圾拾取。

## 1.1 了解社会工程学作为一种武器的广度

不论是哪个社交网络，用户在网上仍然会被声称是别人的人欺骗。与现实世界不同，个人在网上沟通时可以对自己的一切进行误导，不仅限于他们的姓名和业务关系（在现实中这也是相对容易做到的），还包括他们的性别、年龄和地点（这些在现实中更难伪造的标识）。多年前，调查人员将这些类型的人称为信心或骗子。

也许是由于高科技时代的结果，骗子现在被称为从事社会工程学。毫不奇怪地得知，联邦调查局（FBI）正在调查传统的投资欺诈计划，如庞氏计划，这些计划现在正在虚拟世界中进行。其他骗子能够通过在社交网络网站上误认自己，然后诱使受害者提供他们的账户名称和密码以及其他个人身份信息来实施身份盗窃犯罪。

除了身份盗窃犯罪，儿童掠夺者经常使用社交网络网站来找到和沟通未来的受害者和其他恋童癖者。在至少一起 2018 年的公开案件中，一个人在控制了少女的电子邮件和社交网络账户后企图勒索她们的裸照。那起 FBI 的调查导致犯罪者被判处 18 年的联邦刑期，反映出这些犯罪是严重的，不会被容忍。

社会工程学是指任何依赖欺骗人们采取行动或泄露信息的攻击的广义术语。社会工程学已被定义为几种方式，如盒子 1.1 所示。

盒子 1.1 定义社会工程学

通过与个人交往来获得信心和信任以揭示敏感信息的行为。

来源：社会工程下的 NIST SP 800-63-2 [已废弃]

试图欺骗某人透露可以用于攻击系统或网络的信息（例如密码）的尝试。

来源：CNSSI 4009-2015（NIST SP 800-61 Rev. 2）、NIST SP 800-61 Rev. 2 下的社会工程、NIST SP 800-82 Rev. 2 下的社会工程（NIST SP 800-61）

攻击者试图诱骗人们透露敏感信息或执行某些操作的一般术语，例如下载并执行看似良性但实际上是恶意的文件。

来源：社会工程下的 NIST SP 800-114 [已废弃]

尝试欺骗某人透露信息（例如密码）的过程。

来源：社会工程下的 NIST SP 800-115

欺骗个人以揭示敏感信息、获取未经授权的访问权或者通过与个人建立联系获得信任，从而实施欺诈行为的行为。

来源：社会工程下的 NIST SP 800-63-3

来源：词汇表。计算机安全资源中心。访问于 2019 年 2 月 1 日。[`csrc.nist.gov/glossary/term/social-engineering`](https://csrc.nist.gov)。

如今，钓鱼这个术语已经演变为涵盖各种针对个人或公司信息的攻击。最初，钓鱼被确定为使用电子邮件消息，设计成看起来像来自受信任代理人（如银行、拍卖网站或在线商务网站）的消息。这些消息通常要求用户采取某种形式的行动，例如验证其账户信息。它们通常使用一种紧急感（例如账户暂停的威胁）来激励用户采取行动。最近，已经出现了几种新的社会工程手段来欺骗毫无戒心的用户。这些包括提供填写在线银行网站调查的服务，并承诺如果用户包含账户信息，则会给予一笔货币奖励，以及声称来自酒店奖励俱乐部的电子邮件消息，要求用户验证客户可能在合法网站上为预订目的存储的信用卡信息。消息中包含一个供受害者使用的统一资源定位器（URL），然后将用户重定向到一个输入其个人信息的网站。该网站被精心制作，以尽可能地模仿合法网站的外观和感觉。然后，这些信息被犯罪分子收集并使用。随着时间的推移，这些虚假的电子邮件和网站已经演变成对于临时调查更具技术欺骗性的形式。⁴

## 1.2 社会工程欺诈方案

在任何时候，网络犯罪分子都在使用各种互联网欺诈方案。举例来说，最近的欺诈方案涉及网络犯罪分子获取对一个毫无戒心的用户的电子邮件账户或社交网络网站的访问权限。欺诈者声称是账户持有者，然后向用户的朋友发送消息。在消息中，欺诈者声称自己正在旅行，被抢劫了信用卡、护照、钱和手机，急需立即用钱。朋友们没有意识到消息是来自犯罪分子，就将钱汇到了海外账户，而没有验证这一要求。

钓鱼方案试图让互联网用户相信他们收到的电子邮件来自可信任的来源，尽管事实并非如此。社交网络网站用户的钓鱼攻击采用各种格式，包括社交网络网站内来自陌生人或被入侵的朋友账户的消息；社交网络网站个人资料中声称指向无害内容实际上是有害的链接或视频；或者发送给用户声称来自社交网络网站本身的电子邮件。社交网络网站用户由于在使用社交网络网站时通常显示出更高水平的信任，因此成为这些方案的受害者。用户经常接受他们实际上并不认识的人进入他们的私人网站，或者有时未能正确设置个人资料的隐私设置。这使得网络窃贼在试图通过各种钓鱼方案欺骗受害者时具有优势。

社交网络网站以及一般的企业网站为犯罪分子提供了大量信息，使他们能够制作看起来正式的文件并发送给对特定主题表现出兴趣的个人目标。这些信息的个人化和详细性侵蚀了受害者的警惕意识，导致他们打开恶意邮件。这样的邮件将包含一个附件，其中包含恶意软件，旨在让邮件发送者控制受害者整个计算机。当恶意软件感染被发现时，通常已经太迟保护数据免受损害。

网络犯罪分子设计先进的恶意软件，以精确行动，感染、隐藏访问、窃取或修改数据而不被察觉。先进恶意软件的编写者耐心，并且已知会测试网络及其用户以评估防御性响应。恶意软件，也称为恶意代码，指的是秘密插入另一个程序的程序，目的是破坏数据，运行破坏性或侵入性程序，或以其他方式妨碍受害者数据、应用程序或操作系统的机密性、完整性或可用性。恶意软件是大多数主机面临的最常见的外部威胁，造成广泛的损害和破坏，并需要大多数组织进行广泛的恢复工作。先进的恶意软件可能会使用分层方法来感染并在系统上获得提升的特权。通常，这些类型的攻击会与其他网络犯罪策略捆绑在一起，如社会工程或零日漏洞利用。恶意软件经常被用来盗用信息和数据，这些信息和数据可以被轻易地使用，如登录凭据、信用卡和银行账号，有时还有商业机密。

在恶意软件感染的第一阶段，用户可能会收到一封钓鱼邮件，该邮件获取用户的信息或使用用户的凭据进入系统。一旦网络犯罪分子与用户或系统建立连接，他们可以利用其他可能使他们更深入地访问系统资源的向量进一步利用它。在第二阶段，黑客可能会安装一个后门，以在网络上建立一个持久存在，这个存在不能再通过反病毒软件或防火墙发现。

网络窃贼利用社交网络网站上的数据挖掘来提取关于受害者的敏感信息。这可以由犯罪行为者在大规模或小规模上进行。例如，在大规模数据挖掘计划中，网络犯罪分子可能向大量社交网络用户发送“了解你”的问卷。虽然这些问题的答案在表面上看起来并不恶意，但它们通常模仿金融机构或电子邮件账户提供商在个人忘记密码时提出的相同问题。因此，电子邮件地址和问卷的答案可以为网络犯罪分子提供进入银行账户、电子邮件账户或信用卡账户的工具，以转账或吸收账户资金。如果社交网络网站用户没有正确保护自己的个人资料或对敏感信息的访问权限，那么小规模数据挖掘对网络犯罪分子也可能很容易。事实上，一些社交网络应用程序鼓励用户发布自己是否正在度假，同时让窃贼知道没有人在家。⁵

勒索软件诈骗涉及一种感染计算机并限制用户访问其文件或威胁永久销毁其信息的恶意软件，除非支付赎金——从几百到几千美元不等。勒索软件影响家庭计算机、企业、金融机构、政府机构和学术机构，其他组织也可能感染，导致敏感或专有信息的丢失、常规业务的中断、为恢复系统和文件而产生的财务损失，以及/或对组织声誉的潜在损害。

最近几年来，勒索软件一直存在，但近来，网络罪犯对其使用明显增加。FBI 与公共和私营部门合作，针对这些罪犯及其诈骗行为进行打击。当勒索软件首次出现时，计算机主要是通过用户打开包含恶意软件的电子邮件附件而感染的。但最近，我们看到越来越多的事件涉及所谓的驱动型勒索软件，用户只需点击被妥协的网站，通常是被欺骗性的电子邮件或弹出窗口引诱过去，就能感染他们的计算机。

另一个新的趋势涉及赎金支付方式。尽管早期的一些勒索软件诈骗涉及让受害者使用预付卡支付赎金，但现在越来越多的受害者被要求使用比特币支付，比特币是一种去中心化的虚拟货币网络，吸引罪犯的原因是该系统提供的匿名性。另一个不断增长的问题是勒索软件锁定移动电话并要求支付赎金以解锁它们。

FBI 和联邦、国际和私营部门合作伙伴采取了积极措施，通过打击促进勒索软件分发和运作的主要僵尸网络之一，来中和一些更严重的勒索软件诈骗行为。例如，Reveton 勒索软件，由一种称为 Citadel 的恶意软件传递，虚假警告受害者，称其计算机已被 FBI 或司法部确定与儿童色情网站或其他非法在线活动有关。2013 年 6 月，微软、FBI 和金融合作伙伴在 Citadel 恶意软件上建立了一个巨大的犯罪僵尸网络，从而阻止了 Reveton 的分发。Cryptolocker 是一种使用加密密钥对加密受害者计算机文件的高度复杂的勒索软件，并要求赎金以获取加密密钥。2014 年 6 月，FBI 在与 GameOver Zeus 僵尸网络干扰同时宣布，美国和国际执法官员已经查封了 Cryptolocker 的命令和控制服务器。⁶

## 1.3 网络地下世界

网络犯罪对个人和商业的影响可能是巨大的，后果从仅仅是一种不便到财务破产不等。巨大利润的潜力吸引了年轻的犯罪分子，并导致了一个被称为网络地下世界的庞大地下经济的产生。网络地下世界是一个普遍存在的市场，其规则和逻辑与合法商业世界的规则和逻辑非常相似，包括一种独特的语言、对成员行为的一套期望，以及基于知识和技能、活动和声誉的分层系统。

在网络地下世界中，网络犯罪分子之间进行交流的一种方式是在线论坛。在这些论坛上，网络犯罪分子买卖登录凭证（例如电子邮件、社交网络网站或金融账户的凭证）；钓鱼工具包、恶意软件和对僵尸网络的访问权限；以及受害者的社会安全号码（SSN）、信用卡和其他敏感信息。这些犯罪分子越来越专业化、有组织化，并具有独特或专业化的技能。

此外，网络犯罪越来越具有跨国性质，世界各地生活的个人共同参与同一计划。2008 年底，一个国际黑客团伙进行了有史以来最复杂和有组织的计算机欺诈攻击之一。该犯罪团伙使用复杂的黑客技术来破解用于保护 44 张工资借记卡上数据的加密，并提供了一个提款人网络，从全球至少 280 个城市的超过 2,100 台 ATM 机中提取了超过 900 万美元，包括美国、俄罗斯、乌克兰、爱沙尼亚、意大利、香港、日本和加拿大的城市。这 900 万美元的损失发生在不到 12 小时的时间内。网络地下世界促进了网络犯罪服务、工具、专业知识和资源的交流，从而使这种跨国犯罪行动能够在多个国家开展。

除了与社交网络网站相关的网络犯罪后果之外，军事或政府人员通过其社交网络网站个人资料可能无意中暴露了宝贵的信息。最近公开的一例案例中，一个人在多个社交网络网站上创建了一个假的个人资料，假扮成一名有吸引力的女情报分析员，并向政府承包商、军事人员和其他政府人员发送了好友请求。许多好友请求被接受了，尽管该个人资料是虚构的。据新闻报道，这种欺骗使其创作者获得了相当数量的敏感数据，包括一张在阿富汗巡逻中拍摄的士兵照片，其中包含了标识他精确位置的嵌入式数据。当被问及创建假社交网络个人资料的人试图证明什么时，他回答说：第一件事是信任问题以及它是多么容易被给予。第二件事是展示通过各种网络泄露出的不同信息量有多大。他还指出，尽管一些人认识到这些网站是假的，但他们没有一个集中的地方来警告其他人有关所认为的欺诈行为，从而确保了一个月内有 300 个新连接。

最后一点值得进一步探讨。一些社交网络网站自愿成为模范企业公民，通过提供功能让用户举报滥用行为。一些网站提供了易于使用的按钮或链接，只需单击一下即可向系统管理员发送消息，提醒他们潜在的非法或滥用内容。不幸的是，许多网站并未效仿这一做法。一些网站不允许用户举报滥用行为，而其他一些网站要么故意要么无意地通过要求用户完成一系列繁琐的步骤来举报滥用行为，从而阻止了举报行为。 ³

## 1.4 联邦调查局和战略合作伙伴关系

美国司法部（DOJ）领导全国打击网络犯罪的努力，而联邦调查局（FBI）与其他联邦执法机构合作进行网络犯罪调查。FBI 的网络犯罪使命包括四个方面：首先，阻止那些策划最严重的计算机入侵和恶意代码传播的人；其次，识别和阻止利用互联网与儿童接触和剥削、制作、拥有或分享儿童色情作品的在线性侵者；第三，打击针对美国知识产权的行动，从而危及国家安全和竞争力；第四，瓦解参与互联网欺诈的国家和跨国有组织犯罪企业。为此，FBI 在全国各地的 56 个办事处设立了网络犯罪小组，拥有 1000 多名经过专门培训的特工、分析师和数字取证专家。然而，FBI 无法独自应对这一威胁。

FBI 在打击任何犯罪问题方面拥有的一些最佳工具是与联邦、州、地方和国际执法机构以及私营部门和学术界的长期合作伙伴关系。在联邦层面，并经总统授权，FBI 领导了国家网络调查联合特别行动组（NCIJTF），作为协调、整合和共享与网络威胁调查相关的相关信息的多机构国家焦点，以确定网络威胁团体和个人的身份、位置、意图、动机、能力、联盟、资金和方法。在这样做的过程中，NCIJTF 的合作伙伴支持美国政府在国家力量的所有元素上的全面选择。

FBI 还与非营利组织密切合作，包括与国家白领犯罪中心（NW3C）建立互联网犯罪投诉中心（IC3）、国家网络取证和培训联盟（NCFTA）、建立 InfraGard 的 InfraGard 国家成员联盟、金融服务信息共享与分析中心（FS-ISAC）和国家失踪和被剥削儿童中心（NCMEC）等广泛合作。

一个协调的例子突显了当我们在这些紧密建立的合作伙伴关系中工作时我们是多么有效。2019 年初，罗马尼亚警方和检察官进行了罗马尼亚有史以来最大的一次警察行动之一——对从事互联网诈骗的有组织犯罪团伙进行调查。该调查动用了超过 700 名执法人员，在 103 个地点进行了搜查，导致 34 人被逮捕。这个罗马尼亚犯罪团伙的 600 多名受害者是美国公民。成功摧毁这个团伙的一部分原因在于罗马尼亚执法部门与美国国内联邦、州和地方合作伙伴之间的合作伙伴关系的强大。通过 FBI 驻布加勒斯特法律专员（legat）的广泛协调，互联网犯罪投诉中心向罗马尼亚人提供了从[www.IC3.gov](http://www.IC3.gov)报告门户提交的 600 多份投诉。此外，再次与 FBI 的法律专员密切协调，超过 45 个 FBI 现场办公室通过进行采访以获取罗马尼亚投诉表上的受害者声明，并通过获取警方报告和在其部门内进行其他调查线索来协助调查。

与他人密切合作，分享信息，并利用所有可用的资源和专业知识，FBI 及其合作伙伴在打击网络犯罪方面取得了重大进展。显然，还有更多工作要做，但通过协调的方式，美国在努力将最严重的违法者绳之以法方面变得更加灵活和响应迅速。 ³

## 1.5 对抗钓鱼攻击的基本步骤

钓鱼是指骗子使用欺诈性的电子邮件或短信，或者模仿网站，以获取您分享有价值的个人信息，比如账号、社会安全号码或登录 ID 和密码。骗子利用这些信息来窃取一个人的钱财，或身份，或两者兼而有之。骗子还利用钓鱼邮件来获取对计算机或网络的访问权限；然后安装像勒索软件这样的程序，可以锁定用户计算机上的重要文件。

钓鱼骗子通过复制熟悉、信任的已建立的合法公司的标志，将他们的目标引入一种虚假的安全感。或者他们假装是朋友或家人。钓鱼骗子让人觉得他们需要你的信息或别人的信息，而且需要快速提供，否则会发生糟糕的事情。他们可能会说你的账户将被冻结，你将无法获得税款退还，或者你的老板会生气，甚至家人会受伤或你可能会被逮捕。他们撒谎来获取你的信息。计算机用户应采取几种措施来防止钓鱼成功：

+   谨慎打开电子邮件中的附件或点击链接。即使是您朋友或家人的帐户也可能被黑客攻击。文件和链接可能包含可以削弱计算机安全性的恶意软件。

+   自己动手输入。如果您认识的公司或组织向您发送链接或电话号码，请不要点击。使用您喜欢的搜索引擎自行查找网站或电话号码。即使电子邮件中的链接或电话号码看起来像真的，骗子也可能隐藏真实目标。

+   如果您不确定，请打电话。不要回复任何要求个人或财务信息的电子邮件。钓鱼者使用压力策略并利用恐惧。如果您认为某家公司、朋友或家庭成员确实需要您的个人信息，请拿起电话自己打电话给他们，使用他们网站上或您地址簿中的号码，而不是电子邮件中的号码。

+   打开两步验证。对支持的帐户，两步验证需要您的密码和额外的信息才能登录您的帐户。第二部分可以是发送到您手机的代码，或者是应用程序或令牌生成的随机数。即使您的密码被泄露，这也可以保护您的帐户。

+   作为额外预防措施，您可能希望选择多种第二验证类型（例如，PIN），以防您的主要方法（如电话）不可用。

+   将文件备份到外部硬盘或云存储。定期备份文件可以保护自己免受病毒或勒索软件攻击。

+   保持您的安全更新。使用您信任的安全软件，并确保将其设置为自动更新。⁷

+   要对未经请求的电话、访问或电子邮件持怀疑态度，询问员工或其他内部信息的个人。如果一个未知的个人声称来自合法组织，请直接与该公司验证其身份。

+   不要提供个人信息或组织信息，包括其结构或网络，除非你确定某人有权获得该信息。

+   不要在电子邮件中透露个人或财务信息，也不要回复要求此信息的电子邮件。这包括按电子邮件发送的链接。

+   不要在未检查网站安全性的情况下通过互联网发送敏感信息。

+   注意网站的统一资源定位符。恶意网站可能与合法网站看起来相同，但 URL 可能使用拼写变体或不同的域（例如，.com 与 .net）。

+   如果您不确定电子邮件请求是否合法，请通过直接联系公司来验证。不要使用请求网站提供的联系信息；相反，请查看以前的声明以获取联系信息。

+   安装和维护防病毒软件、防火墙和电子邮件过滤器以减少某些流量。

+   利用您的电子邮件客户端和网络浏览器提供的任何防钓鱼功能。

+   报告钓鱼邮件和短信（Box 1.2）。⁸

Box 1.2 如何报告钓鱼诈骗

将钓鱼邮件转发到 spam@uce.gov ——以及在邮件中被冒名顶替的组织。当您包含完整的电子邮件头部时，您的报告效果最佳，但大多数电子邮件程序隐藏了此信息。为了确保包含头部，请在您喜欢的搜索引擎中搜索您的电子邮件服务的名称和“完整电子邮件头部”。

向联邦贸易委员会在 FTC.gov/complaint 报告。

访问 Identitytheft.gov。钓鱼受害者可能成为身份盗窃的受害者；有一些措施可以减少您的风险。

您还可以将钓鱼邮件报告给 reportphishing@apwg.org。反钓鱼工作组 —— 包括 ISP、安全厂商、金融机构和执法机构 —— 使用这些报告来打击钓鱼。

联邦贸易委员会（FTC）和全国房地产经纪人协会^® 在 2016 年警告房屋买家有一封电子邮件和电汇诈骗。黑客曾经入侵一些消费者和房地产专业人员的电子邮件账户，以获取有关即将进行的房地产交易的信息。在确定了交易的结束日期后，黑客会发送一封电子邮件给买家，冒充房地产专业人员或地契公司。虚假的电子邮件会说有关汇款说明的最后一分钟更改，并告诉买家将收盘成本汇款至其他账户。但这将是骗子的账户。如果买家上钩，他们的银行账户可能在几分钟内被清空。通常，这将是买家永远看不到的钱。

当时，FTC 重申其建议，不要通过电子邮件发送财务信息，如果用户在网上提供财务信息，请确保网站是安全的。搜索以 https 开头的 URL（“s” 代表安全）。而且，不要点击电子邮件中的链接以访问组织的网站，自己查找真实的 URL 并输入网址。此外，无论发送者是谁，都要小心打开电子邮件中的附件和下载文件。这些文件可能包含可以削弱计算机安全性的恶意软件。⁹

钓鱼攻击也可以试图利用最近的事件，比如健康保险公司安泰的数据泄露影响了超过 8000 万客户。骗子发送了假冒的安泰邮件，假装帮助客户，但实际上是钓鱼获取他们的个人信息。这封假邮件被设计成看起来像是来自安泰，并要求客户点击一个链接以获取免费的信用监控或信用卡账户保护。安泰报告称，将通过邮件联系现有和以前的客户，提供有关如何参加信用监控的具体信息。安泰还表示，不会因数据泄露而致电客户，也不会通过电话索要信用卡信息或社会安全号码。 ¹⁰

## 1.6 21 世纪的新社会工程水平

俄罗斯以迄今大多数美国人不熟悉的方式部署了信息和网络战的混合形式。通过武器化窃取的信息和传播虚假信息，俄罗斯情报机构努力贬低美国在国内外的形象，破坏其外交政策，并在国内制造分裂。最近最引人注目的例子当然是俄罗斯干预 2016 年美国总统选举，情报界确认这是为了帮助特朗普总统当选并破坏美国人对选举制度的信心。

俄罗斯干预外国选举以推进其利益并非新现象，也不仅限于美国。德国和法国政府已经发出警报，称俄罗斯目前正在进行类似的操作，以在 2019 年全国选举前瞄准被认为对俄罗斯利益不友好的候选人。

俄罗斯还在庞大的宣传网络上投入了大量资源，包括在美国的俄罗斯之声（RT），以传播削弱民主共识、加强政治边缘的虚假信息。据报道，RT 仅在华盛顿分部就花费了 4 亿美元；它拥有比任何其他广播公司都多的 YouTube 订阅者，包括 BBC。俄罗斯与 RT 一起监督数十家其他新闻来源，通过一个网站播种丑闻故事，然后通过其他网站传播和放大。俄罗斯在幕后雇佣了数百名精通英语的年轻人，操作一个庞大的虚假在线身份网络，撰写博客文章和评论。

俄罗斯进行信息战的能力得到了极大的帮助，这主要得益于其在网络空间的大量投资，而美国在对抗或威慑其侵略性探测方面仍然缺乏准备。俄罗斯在这一领域的活动反映了一项更新的国家安全战略，强调利用对手的脆弱性来采取不对称战术，同时削弱他们对抗俄罗斯政策的能力和决心。在最近的公开报告中，美国情报界确定俄罗斯是网络空间中最复杂的国家行为者之一。¹¹

俄罗斯的干预既是隐蔽的，也是公开的，积极措施多样化、规模更大、技术更复杂。他们不断根据技术和环境的变化进行调整和变形。通过同时打击欧洲和美国，这种干预似乎旨在破坏西方联盟的有效性和凝聚力，以及西方作为一个基于普遍规则而不仅仅是权力的全球秩序的合法性。¹²

2007 年，Facebook 平台扩展了更多应用程序，使用户的日历能够显示朋友的生日，地图显示用户的朋友居住地，通讯录显示他们的照片。为了实现这一目标，Facebook 让人们可以登录应用程序并分享他们的朋友是谁以及一些关于他们的信息。然后，在 2013 年，剑桥大学的研究人员亚历山大·科根创建了一个人格测验应用程序。大约有 30 万人安装了这个应用程序，他们同意分享一些他们的 Facebook 信息以及一些来自他们的朋友的信息，这些信息是根据他们的隐私设置允许的。考虑到当时平台的运作方式，科根能够访问数千万朋友的一些信息。2014 年，为了防止滥用应用程序，Facebook 宣布他们正在改变整个平台，大大限制应用程序可以访问的 Facebook 信息。最重要的是，像科根这样的应用程序再也不能要求关于一个人的朋友的信息，除非他们的朋友也授权了该应用程序。Facebook 还要求开发人员在请求用户的公共资料、好友列表和电子邮件地址之外的任何数据之前获得 Facebook 的批准。这些举措将阻止任何像科根这样的应用程序今天能够访问到如此多的 Facebook 数据。

2015 年，Facebook 从《卫报》的记者那里得知，Kogan 向剑桥分析公司分享了他的应用程序的数据，尽管开发者未经人们同意分享数据违反了 Facebook 的政策。Facebook 立即禁止了 Kogan 的应用程序，并要求 Kogan 和其他他提供数据的实体（包括剑桥分析公司）正式证明他们已删除了所有不当获取的数据。后来，Facebook 从《卫报》、《纽约时报》和 Channel 4 得知，剑桥分析公司可能并没有删除他们已经认证的数据。Facebook 立即禁止他们使用任何 Facebook 服务。

Facebook 安全团队多年来一直注意到传统的俄罗斯网络威胁，如黑客攻击和恶意软件。在 2016 年 11 月的选举日前夕，Facebook 发现并处理了几个与俄罗斯有关的威胁。其中包括一个名为 APT28 的组织的活动，美国政府公开将其与俄罗斯军事情报服务联系起来。但在主要关注传统威胁的同时，Facebook 在 2016 年夏天还注意到了一些新的行为，当时 APT28 相关账户在 DC Leaks 的旗帜下创建了虚假身份，用于向记者传播窃取的信息。Facebook 因违反政策而关闭了这些账户。选举结束后，Facebook 继续调查并了解了这些新威胁，并发现恶意行为者利用协调一致的虚假账户网络干扰选举：推广或攻击特定候选人和事业，制造对政治机构的不信任，或者简单地传播混乱。其中一些恶意行为者还使用 Facebook 广告工具作为钓鱼工具，将人们引入信息错误和虚假信息的迷宫。Facebook 还了解到了由俄罗斯互联网研究机构（IRA）发起的一场虚假信息运动，该机构一再采取欺骗手段，试图操纵美国、欧洲和俄罗斯的人民。Facebook 发现大约有 470 个与 IRA 相关的账户和页面，它们在约两年的时间里发布了约 8 万条 Facebook 帖子。据估计，在该期间大约有 1.26 亿人可能曾经在某个时候看到与 IRA 相关的 Facebook 页面的内容。在 Instagram 上，由于关于覆盖率的数据不够完整，发现了约 12 万条内容，估计还有额外的 2,000 万人可能看到了它。在同一时期，IRA 还在 Facebook 和 Instagram 上花费了约 10 万美元发布了 3,000 多则广告，估计在美国有 1,100 万人看到了。Facebook 在 2017 年 8 月关闭了这些 IRA 账户。 ¹³

根据 2018 年美国参议员马克·R·沃纳发布的一份白皮书草案，他认为，在美国国会调查俄罗斯干预 2016 年选举的过程中，许多互联网技术被滥用以及它们的提供者一再出错的程度是不言而喻的。2018 年的揭示不仅揭示了这些技术被恶意操作的能力，还揭示了整个生态系统的阴暗面。这些产品迅速增长并几乎在我们的社交、政治和经济生活的方方面面占据主导地位的速度，在很多方面掩盖了它们的创造者未能预料到其使用的有害影响的缺点。政府未能适应，并且无法或不愿充分解决这些趋势对隐私、竞争和公共话语的影响。

沃纳进一步认为，这些平台的规模和影响力要求我们确保对技术的适当监督、透明度和有效管理，这些技术在很大程度上支撑着我们的社交生活、经济和政治。我们有很多机会与这些公司、其他利益相关者和政策制定者合作，确保我们正在采取适当的保障措施，以确保这个生态系统不再存在“荒野西部”的状态——没有管理，也不向用户或更广泛的社会负责，而是更广泛地为社会、竞争和广泛的创新服务。

这仅仅是发现社交媒体工具如何被用于社会工程活动的开端。这也仅仅是对社交媒体提供商进行监管并要求他们保护公众免受利用这些工具操纵行为和影响选举结果以及社会机构运行的社会工程师的长期努力的开端。 ¹⁴

## 1.7 本书的组织

本书旨在节省管理者和网络安全步兵研究社会工程方法和缓解方法的时间。因此，本书将更好地为制定目标的规划者提供信息，并介绍针对社会工程攻击采取的可捍卫行动，并使管理者更好地应对可能使他们受害的行为。

为了使本书对研究生或专业级研讨班课程有所帮助，每章都提供了研讨班讨论主题，以及可能需要不超过 30 分钟的研讨小组项目，以便小组呈现他们的成果。这些章节的安排方式旨在分析不同类型的罪犯、合法组织和特殊利益集团对社会工程使用的情况。

第一章 简要介绍了社会工程、钓鱼以及美国执法部门打击电子犯罪的努力。

第二章 探讨了社会工程方法和策略的连续性，这些方法和策略被骗子用来利用或窃取受害者（包括个人和组织）的信息或金钱。这包括假电子邮件和网站的演变，使其在普通调查中更具技术性欺骗性，以及钓鱼的定义已经扩大到涵盖更广泛的电子金融犯罪。

第三章 探讨了近年来由于良好的经济和技术条件而蓬勃发展的钓鱼诈骗的几种方法。本章还概述了打击诈骗、身份盗窃和在线欺诈的政府机构，包括互联网犯罪投诉中心（IC3）、联邦贸易委员会（FTC，致力于防止市场上的欺诈、欺骗和不公平商业行为）、以及证券交易委员会（SEC，保护证券市场免受欺诈）。SEC 处理的许多投诉涉及通过社会工程手段实施的犯罪行为。此外，本章审查了几起利用社会工程手段实施犯罪行为的案例。

第四章 着重于保护组织和个人免受社会工程攻击。组织的安全文化对其信息安全计划的有效性起到贡献作用。管理团队应了解和支持信息安全，并为开发、实施和维护信息安全计划提供适当的资源。这种理解和支持的结果是一个计划，其中管理和员工致力于将该计划整合到业务线、支持功能和第三方管理计划中。

第五章 分析了社会工程攻击在诱饵或手法符合收件人日常或正常情境时更有效。互联网上大量的个人可识别信息（PII）使得社会工程攻击者能够找到与个人私生活或企业员工业务活动相关的话题。因此，公开可获取的个人或组织信息已成为安全问题，因为它可以在系统的社会工程攻击或诈骗或身份窃取尝试中使用。本章审查了围绕 PII 的许多安全问题。

第六章 探讨了关于外国组织利用社交媒体影响近期美国和欧洲选举结果的辩论和猜测。毫无疑问，俄罗斯组织，或许还有其他国家的组织，确实利用社交媒体试图影响选举结果。本章简要介绍了社交媒体执行官在美国国会作证的情况。随后，本章继续审视了外国参与者的活动以及国内组织和个人利用社交工程来影响选举结果的情况。

第七章 探讨了内部人员发动的社交工程攻击，这些内部人员以进入政府机构的方式，最终带走了敏感和机密材料。面临风险的不仅仅是政府——所有组织都面临着来自内部人员的某种程度的威胁，以及内部人员可能会与外部人员合谋，合作，偷窃，破坏或羞辱其员工的可能性。在本章中，我们将注意力转向了内部人员的社交工程攻击的努力。

第八章 探讨了关于社交工程的更大规模教育工作的必要性。在防范社交工程攻击方面，主要的防御手段包括防止垃圾邮件、要求对代码运行系统进行权限控制，以及由各种组织和政府机构提供的一系列提示手册。这些努力确实有助于减少社交工程攻击的数量和效果，但攻击仍在继续，每年造成数十亿美元的损失。教育工作需要得到极大扩展，最重要的是，它需要包含准确的信息，以对抗错误信息、错误信息和假新闻，并且需要能够使用公正和准确的信息来做到这一点。这将需要利用所有的传播工具，向人们提供关于通过已确认的所有媒体传播的不准确或恶意材料的信息。

第九章 从前几章中得出结论，并且着眼于反击社交工程攻击努力的未来。

## 1.8 结论

对网络犯罪分子来说，没有什么是禁区的，无论他们使用的是钓鱼、勒索软件、冒充还是其他形式的欺骗。社会保障局(SSA)的总检察长特别警告公众，特别是社会保障受益人，要注意以个人信息为目标的欺诈诈骗行为。诈骗分子使用电话、电子邮件和其他方法获取个人信息，然后利用这些信息进行身份盗窃。在最近的骗局中，身份盗窃者冒充政府官员，试图说服人们提供个人和财务信息。他们可能声称自己是 SSA 的员工，或者是(在桑迪飓风之后)联邦紧急管理局(FEMA)的员工，并要求提供社会保障号码和银行信息，以确保人们能够领取福利。诈骗分子还可能声称某人中了彩票或其他奖品，但必须支付费用、税款或其他费用，才能领取奖金。 ¹⁵

税收身份盗窃是指某人使用被盗的社会安全号码(SSN)来获得税款退还或工作的情况。人们可能会在收到美国国内税务局(IRS)的来信时发现这种情况，信中提到他们的 SSN 被用于多次纳税申报，或者 IRS 的记录显示他们从一个陌生的雇主那里获得了收入。或者，IRS 可能会拒绝接受他们的电子纳税申报，因为已经有重复的申报。税务诈骗者还会通过电子邮件和电话尝试说服纳税人提供个人信息。IRS 已多次警告人们注意这些诈骗行为。 ¹⁶

## 1.9 关键点

本章涵盖的关键点包括：

+   社会工程是一种非常有效的攻击过程，超过 80%的网络攻击和超过 70%的国家级攻击都是通过利用人类而不是计算机或网络安全漏洞发起的。

+   社会工程攻击通常要求人们执行攻击者希望从受害者那里诱导出的所需行为。

+   社会工程学是指任何依赖欺骗人们采取行动或泄露信息的攻击的广义术语。

+   钓鱼计划试图让互联网用户相信他们收到了来自可信来源的电子邮件，而实际情况并非如此。

+   社交网络网站以及一般的公司网站为罪犯提供了大量信息，以发送官方文件，并将它们发送给对特定主题表现出兴趣的个人目标。

+   在恶意软件感染的第一阶段，用户可能会收到一封“钓鱼”电子邮件，该邮件获取了用户的信息或使用用户的凭据进入系统。

+   网络犯罪对个人和商业的影响可能很大，后果从仅仅是不便到财务破产不等。

+   网络犯罪在跨国性质上越来越普遍，世界各地生活在不同国家的个人一起合作进行相同的阴谋。

+   计算机用户应采取几种做法来防止网络钓鱼成功，尽管这些做法被广泛宣传，但每天仍然有社会工程阴谋的受害者。

+   社交媒体平台已成为社会工程师利用其进行犯罪活动的新工具。

+   社交媒体产品的迅速增长并几乎在我们的社会、政治和经济生活的各个方面占主导地位，在很大程度上掩盖了它们的创作者在预见其使用的有害影响方面的缺陷。

## 1.10 研讨会讨论话题

研究生或专业级别研讨会的讨论话题：

+   研讨会参与者在使用社会工程策略或战术的情况下有何经验？

+   讨论参与者或其亲戚或朋友在网络钓鱼攻击方面的任何经历。

+   讨论参与者、他们的家人或朋友在收到网络钓鱼邮件或电话时所采取的行动。他们为什么会采取这种方式？

+   讨论参与者如何看待执法部门应对网络犯罪的努力。

## 1.11 研讨会小组项目

将参与者分成多个小组，每个小组花 10 至 15 分钟时间开发社会工程策略的清单及其所适用的各种环境。完成后，让各小组交换他们的社会工程策略清单，然后各小组花 10 至 15 分钟时间制定快速识别社会工程策略及其所涉及任务的清单。作为一个小组讨论所选的策略及其如何被识别。

### 关键词

+   虚假信息：是为了欺骗而提供的虚假和无关信息。

+   身份盗窃犯罪：身份盗窃和身份欺诈是指以欺诈或欺骗手段获得并使用他人个人数据的所有犯罪行为的术语，通常是为了经济利益。

+   恶意链接：是将用户引导至包含恶意代码（如间谍软件、病毒或特洛伊木马）的网站的超链接，这些代码可能会感染访问这些网站的计算机。

+   恶意软件：包括病毒、间谍软件和未经您同意安装在计算机或移动设备上的其他不受欢迎的软件。这些程序可能导致设备崩溃，并可用于监视和控制您的在线活动。它们还可能使您的计算机容易受到病毒的侵害，并传送不需要或不合适的广告。犯罪分子利用恶意软件窃取个人信息、发送垃圾邮件和进行欺诈活动。

+   个人身份可识别信息（PII）：是指可以用于区分或追踪个人身份的信息，无论是单独使用还是与其他个人或识别信息结合使用，这些信息与特定个人相关联或可链接到特定个人。

+   网络钓鱼：网络钓鱼是指骗子使用欺诈性的电子邮件或短信，或者模仿网站，以获取您分享有价值的个人信息，例如帐户号码、社会安全号码或登录 ID 和密码。骗子利用您的信息来窃取您的钱财，或者您的身份，或者两者兼而有之。

+   庞氏骗局：是一种投资欺诈，通过从新投资者那里收集的资金支付现有投资者。庞氏骗局的组织者经常承诺投资你的钱，并以几乎没有风险的方式产生高回报。但在许多庞氏骗局中，骗子并不会投资这笔钱。相反，他们用它来支付先前投资的人，并可能留下一些给自己。

+   勒索软件诈骗：利用一种恶意软件感染计算机并限制用户访问其文件或威胁永久破坏其信息，除非支付赎金，通常要求以比特币支付。

+   鱼叉式网络钓鱼：鱼叉式网络钓鱼攻击不同于常规网络钓鱼尝试，因为它们针对特定收件人并似乎来自可信任的来源。

## 参考资料

+   1\. Netflix 钓鱼骗局：不要上当。美国联邦贸易委员会。Tressler，Colleen。2018 年 12 月 26 日访问于 2019 年 2 月 2 日。[`www.consumer.ftc.gov/blog/2018/12/netflix-phishing-scam-dont-take-bait`](https://www.consumer.ftc.gov)

+   2\. 主动社会工程防御（ASED）。沈伟。国防高级研究计划局项目信息。2019 年 2 月 1 日访问于。[`www.darpa.mil/program/active-social-engineering-defense`](https://www.darpa.mil)

+   3\. 在犯罪、恐怖主义和国土安全众议院司法小组委员会之前的声明。联邦调查局助理局长 Gordon M. Snow。2010 年 7 月 28 日，华盛顿特区。2019 年 2 月 1 日访问于。[`archives.fbi.gov/archives/news/testimony/the-fbis-efforts-to-combat-cyber-crime-on-social-networking-sites`](https://archives.fbi.gov)

+   4\. 网络钓鱼攻击中的技术趋势。美国网络安全与基础设施安全局（US-CERT）。Milletary，Jason。访问于 2019 年 2 月 2 日。[www.us-cert.gov/sites/default/files/publications/phishing_trends0511.pdf](http://www.us-cert.gov)

+   5\. 桌面和笔记本计算机恶意软件事件预防和处理指南。国家标准与技术研究所，信息技术实验室（ITL），计算机安全部门（CSD），应用网络安全部门（ACD）。Murugiah Souppaya（NIST）和 Karen Scarfone（Scarfone Cybersecurity）。2013 年 7 月访问于 2019 年 2 月 2 日。[`csrc.nist.gov/publications/detail/sp/800-83/rev-1/final`](https://csrc.nist.gov)

+   6\. 勒索软件横行：FBI 及合作伙伴努力打击这一网络威胁。美国联邦调查局。2015 年 1 月 20 日访问。[`www.fbi.gov/news/stories/ransomware-on-the-rise`](https://www.fbi.gov)

+   7\. 钓鱼。美国联邦贸易委员会。2017 年 7 月访问。[`www.consumer.ftc.gov/articles/0003-phishing`](https://www.consumer.ftc.gov)

+   8\. 安全提示（ST04-014）避免社会工程和钓鱼攻击。美国-证书。原始发布日期：2009 年 10 月 22 日。最后修订日期：2018 年 11 月 21 日。访问日期：2019 年 2 月 2 日。[`www.us-cert.gov/ncas/tips/ST04-014`](https://www.us-cert.gov)

+   9\. 骗子钓鱼骗取抵押贷款结算费用。美国联邦贸易委员会。Tressler，Colleen。2016 年 3 月 18 日访问。[`www.consumer.ftc.gov/blog/2016/03/scammers-phish-mortgage-closing-costs`](https://www.consumer.ftc.gov)

+   10\. 安 them 黑客攻击，第 2 部分：钓鱼诈骗。美国联邦贸易委员会。Tressler，Colleen。2015 年 2 月 10 日访问。[`www.consumer.ftc.gov/blog/2015/02/anthem-hack-attack-part-2-phishing-scams`](https://www.consumer.ftc.gov)

+   11\. 在参议院外交关系委员会作证。朱利安·史密斯，新美国安全中心战略与国家构想项目高级研究员和主任。2017 年 2 月 9 日访问。[www.foreign.senate.gov/download/smith-testimony-020917&amp;download=1](http://www.foreign.senate.gov)

+   12\. 俄罗斯干预对德国 2017 年选举的影响。康斯坦茨·斯特尔岑穆勒博士和罗伯特·博世，布鲁金斯学会美国和欧洲中心高级研究员。在美国参议院情报特别委员会之前作证。2017 年 6 月 28 日访问。[www.intelligence.senate.gov/sites/default/files/documents/sfr-cstelzenmuller-062817b.pdf](http://www.intelligence.senate.gov)

+   13\. 在美国众议院能源和商业委员会听证会上的 Facebook。马克·扎克伯格，董事长兼首席执行官作证。2018 年 4 月 11 日访问。[www.docs.house.gov/meetings/IF/IF00/20180411/108090/HHRG-115-IF00-20180411-SD002.pdf](http://www.docs.house.gov)

+   14\. 社交媒体和科技公司监管的潜在政策建议（草案）。美国参议员马克·R·沃纳。访问日期：2019 年 2 月 3 日。[`www.warner.senate.gov/public/_cache/files/d/3/d32c2f17-cc76-4e11-8aa9-897eb3c90d16/65A7C5D983F899DAAE5AA21F57BAD944.social-media-regulation-proposals.pdf`](https://www.warner.senate.gov)

+   15\. 欺诈警示：社会工程诈骗瞄准社会保障受益人。社会保障局。2012 年 11 月 21 日访问。[`oig.ssa.gov/newsroom/news-releases/advisory21`](https://oig.ssa.gov)

+   16\. 反击税务身份盗窃。联邦贸易委员会。格雷辛，西娜。2019 年 1 月 30 日访问。[`www.consumer.ftc.gov/blog/2019/01/fight-back-against-tax-identity-theft`](https://www.consumer.ftc.gov)
