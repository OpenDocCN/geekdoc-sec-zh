# 十二、基于风险的网络安全和堆积风险

从所有案例研究中可以明显看出，任何因网络安全遭到重大破坏而陷入困境的组织，都没有清楚地了解他们所承担的风险。不言而喻，这些组织缺乏对其现行风险的相关和知情的看法。

有效的网络安全管理依赖于对优先风险的准确捕捉和升级。如果问题或难题没有在个人层面得到一致的捕捉，并在重大时得到适当的上报，管理层将在不知情的环境中运作，不知道真正的差距及其相对优先级。

在本章中，我们将介绍:

*   什么是网络安全风险？
*   你如何捕捉和管理个体风险？
*   您如何使用以下工具来衡量、监控和管理风险集群:
*   ο风险登记簿
*   ο风险评估
*   如何应用基于风险的网络安全管理。

单独管理风险虽然重要，但如果无法看到全局，仍然会产生问题。

当组织因入侵和数据丢失而遭受重大经济损失时，通常会出现一系列独立且未解决的风险。我们将此称为堆积风险，并将在本章的网络风险登记簿部分更详细地讨论这一主题。

堆积的风险–一系列相关的问题加在一起可能会造成比单个信息更大的财务影响。

在我们了解什么是风险之前，考虑一下人们在理解任何类型的风险时存在的普遍问题和偏见是有益的。

考虑以下事项，根据它们每年在北美造成的死亡人数，您认为它们会对生命构成何种威胁:

*   自动售货机
*   棕熊袭击。
*   毛绒玩具。
*   是左撇子。

没有任何度量或分析，我们很容易对现实产生扭曲的印象。

事实上，在美国，自动售货机每年杀死的人比棕熊还多(当人们摇动自动售货机以回收松散的物品时，自动售货机会砸到人)。毛绒玩具比棕熊或自动售货机造成更多的死亡。左撇子被认为是最大的杀手，因为左撇子使用为右撇子设计的设备会引发事故。

(Halpern 和 Coren 在 1991 年进行的一项备受争议的研究显示，左撇子的预期寿命有显著差异。这项研究后来被许多人认为可能包含一些统计异常而不予考虑。然而，有一个共识是，左撇子使用右撇子的设备确实会给他们带来更多的事故。)

*   根据美国消费者产品安全委员会的数据，2012 年美国有 11 人因毛绒玩具死亡。
*   美国每年有 2 到 3 人因自动售货机而死亡。
*   北美每年平均有一人因棕熊袭击而丧生。
*   因左撇子导致的死亡人数没有记录。

这与网络安全有什么关系？

我们在网络安全方面也有同样的问题，如果不能准确理解隐藏在风险背后的数字，我们可能会在安全工作和预算的重点上犯错误。

在没有全面了解风险的情况下，作为一名网络安全经理，我可能会倾向于优先考虑在加密数据上的支出，因为这涵盖了许多潜在的攻击面。然而，如果我对这些问题以及相对的对策成本和收益有全面的了解，我可以很容易地发现有 20 个或更多更高优先级、更高影响和更低成本的项目需要首先解决。

造成最大损失的是最大的、未解决的风险。您需要对您的整体风险有一个全面和关联的视图，以便能够准确地了解网络安全的优先事项。

当风险孤立出现时，不可能理解它们的相对优先级。

在我们进入大的、联合起来的视图之前，我们仍然需要理解捕捉和管理个体风险的基础。

什么是网络安全 风险？

任何有可能对我们使用的电子设备或者它们存储或处理的信息造成有害影响的事情，都可以被认为是网络安全风险。请记住，这可能包括直接影响我们安全状态的流程和其他非技术性项目。

例如，如果存在没有定期提供安全意识培训的问题，这仍然可能是网络安全的一个风险，因为它很有可能导致工作人员的不良使用做法，从而造成更多成功的恶意软件攻击。

在本书的前面，我们已经讨论了威胁、漏洞和其他差距。当它们(I)有足够的发生概率，并且(ii)如果发生，有潜在的有害影响，也可以被认为是风险源时，也可以考虑其中的每一个。

这是因为风险的两个关键因素是:

1.  1)问题发生的概率(也称为潜在性、可能性或偶然性)。
2.  足以引起重大关注的重大影响。

有许多方法可以测量概率。最有效的方法是确保所有可能性或某件事情发生的机会的表达式都被转换成百分比值。分配给风险的初始概率百分比不可能也不一定完全正确。这是因为分配给每个风险的百分比值将随着风险信息的增加而提高。

还有许多不同的方法来衡量影响。最有效的方法是将潜在中断的成本转化为财务金额，反映(I)在问题发生后修复或恢复问题的成本，以及(ii)问题可能给组织带来的成本。

请记住，由于业务中断、收入损失或品牌受损而给组织带来的成本通常是财务影响评估的最高值。

在这两种情况下，如果没有数值，就不可能以任何有意义的方式评估风险。这是因为记录和监控风险的人没有能力在一个共同的尺度上比较风险。

例如，如果我有一个风险(风险 A )会造成一百万美元的损失，另一个风险(风险 B )会造成一千万美元的损失，这两个风险都可能被认为是“非常高”的影响，这取决于我们组织的规模和预算。只有有了具体的数字，我才能确定其中一个风险比另一个风险大十倍，因此更有可能需要优先解决。

风险通常使用简单的数学公式来帮助确定优先级。只需将风险的概率(%)乘以潜在影响($)，我就可以得到一个调整后的风险数字，它可以帮助我对风险进行优先级排序。

*   风险 A 有 75%的几率发生，如果发生，将会造成 100 万美元的损失。

0.75 x 万= 75 万美元

*   风险 B 有 5%的几率发生，如果发生，将会造成一千万美元的损失。

0.05 x 10，000，000 = 50 万美元

虽然风险 B 具有更高的潜在财务影响，但是通过使用概率，我们可以确定风险 A 发生的可能性更高，这意味着我们应该首先解决风险 A 。

然而，还有第三个关键参数需要考虑，proxiT2。

接近度是一种时间度量，有助于评估我们预期风险多快会变得活跃和有问题。

例如，我们可能在六个月后推出一项新服务，该服务与风险 A 相关联，但风险 B 可能是已经存在的差距或问题。在这种情况下，我们可能会合理地选择比尚未成为活跃问题的风险更早地解决眼前的风险。

获取有关风险的基本数字信息是管理风险的关键一步。

构成风险的基本要素是存在足够的可能性和影响，使项目具有足够的重要性来跟踪。这就是所谓的物质 品质。

重要性–达到一个重要或重要的水平。

一般来说，组织越大，财务影响必须越大，才能认为某件事具有足够的重要性，可以作为风险进行记录和管理。

如果我在单个应用程序中发现了一个单独的、严重的漏洞，只有当它(就其本身而言)会对我的组织产生重大影响时，它才可能被视为风险。我仍然需要确保通过正常流程设法关闭它，但我不需要要求将其作为风险进行跟踪。

但是，我也可以确定同样的严重漏洞可能存在于大量其他应用程序中，需要紧急调查。在这种情况下，如果我认为集体影响重大，我会将其升级为风险。

每个组织都定义了自己的实质性阈值，通常是一个财务金额。如果差距或问题有可能给组织带来超过 x 美元的财务风险(其中 x 是组织确定的实质性阈值)，则应将其纳入风险管理流程。

本章的风险登记簿一节中有更多关于实质性的内容。

您如何捕获和管理单个 RISK？

当风险被报告时，您需要有效地管理它。

为了有效地管理单个风险，需要对如何捕获和管理每个风险采取一致的方法。如果您使用一致的流程，您创建的风险信息可以更容易地进行比较、关联、上报(如有必要)和优先排序。

有几个可用的风险框架，包括 COSO(国际标准化组织的风险管理方法)、ISO 31000(特雷德韦委员会的发起组织、的 Co 委员会、 O 组织)企业风险管理框架和伊萨卡·CRISC(信息系统控制风险认证)。

以上所有框架都是有效的，可以根据您的个人或组织偏好进一步探索。出于我们的目的，我们将着眼于所有风险框架共有的核心概念。

3 个关键成分是:

*   所有权:确保每个活动风险都有一个明确负责的所有者。
*   生命周期:定义并使用一致的风险生命周期，让你知道风险处于什么状态。
*   风险信息:确保获取关于风险的足够信息，包括概率和影响。

船主 船:

每个单独的风险必须有一个明确指定的负责人，他接受管理风险的责任。单点责任制在这里和在整个安全框架中一样重要。其他人可以也愿意帮助管理和控制风险，但必须有一个特定的人来控制和负责管理风险，直到最终关闭。

终身 循环:

所有的风险都有一个生命周期。它们被发现、调查、分析、处理并在必要时关闭。最简单的风险生命周期可能认为风险只是“未解决”(仍是潜在威胁)或“已解决”(不再是潜在问题)。

您的风险生命周期阶段越精细，以后就越容易将仍在调查的新风险与风险管理过程中的其他风险区分开来。

一组良好的基本生命周期阶段是:

*   已确定
*   调查
*   分析
*   治疗
*   监控
*   关闭

这些不需要按顺序进行。例如，会有一些风险被报告(识别)、调查、发现不是风险并关闭。

风险信息

除了对风险的简要描述之外，获取关于风险的其他关键信息也很重要。如前一节所述，确保获取关于概率和影响的信息至关重要。如果一开始不准确也没关系，因为关于风险的信息应该在其生命周期中得到改进(或阐述)。

当风险信息的获取方式使风险影响在以后更容易分析时，它会更有效。例如，当选择列表可以被利用时，比仅在自由文本中记录影响信息更有用。例如，您可以为可能受影响的内容选择多个值:

*   失去关键服务
*   关键产品的损失
*   品牌/组织形象
*   客户数据
*   公司数据
*   员工数据
*   业务流程
*   财务流程
*   内部应用
*   外部应用
*   法规或法律合规性
*   知识产权

其他标准风险信息可以包括邻近性(风险可能多久发生)和可管理性(风险所有者认为我们有能力控制风险，如果我们选择这样做的话)。

通常还会记录是谁报告的以及何时报告的(日期和时间)。

每个单独的风险也将(在分析过程中)有一种或多种识别的管理方法。这些被称为风险对策或风险处理方案。通常有多达 5 种处理风险的主要方法:

*   预防。这意味着你停止了风险的起因，因此阻止了风险的出现。例如，不要采用导致风险的特定技术组件。这有时被认为是规避风险的一种形式。
*   还原。少做一些事情来减少潜在的影响。例如，减少系统存储的记录数量，以减少潜在的丢失风险。
*   验收。什么都不做，如果你认为潜在的概率和影响低到足以吸收，而追求其他风险处理方法又过于昂贵。
*   权变。如果风险真的发生，创建一个后备计划来帮助减少风险的影响。例如，如果有风险的系统发生故障，有一个替代系统或流程可以接管。
*   转移。把风险变成别人的责任。例如，你可以选择为潜在损失投保。

在许多情况下，会选择一个以上的风险处理方案。例如，你可以选择少做一些事情(减少)并投保(转移)剩余风险。

在如何记录关于个体风险的附加信息方面有很大的灵活性。为了使风险信息可用，关键步骤是以一致的方式捕获上述关键信息，然后将其共享到适当的风险列表中，称为风险区域 斯特。

网络风险区 斯特:

风险登记簿–一个中央存储库，通常采用一致的电子格式，包含每个潜在、重大损失或损害风险的条目。通常有一个最低实质性阈值，例如，在存储库中的条目被请求 终止之前，必须达到或超过的最低潜在财务损失值。

尽管风险登记簿只是一个风险列表，但如果它包含使用一致格式捕获的每个风险的关键信息，它可以提供简单的方法来识别:

*   被报告的相似或相同的风险。
*   风险结合在一起(叠加风险)会给部分安全领域带来比单个风险更多的整体潜在问题。
*   要解决的相对风险优先级。

网络风险登记册的重要之处在于，它能捕捉到任何可能对数字世界造成重大损害或破坏的重大风险。这意味着任何可能对网络安全方法产生重大影响的东西都应该在这里得到管理和监控。

在大型组织中，通常需要由不同的人管理不同程度的风险。这完全没问题，只要有一个将超过某个确定的实质性级别的风险上报给网络安全经理的流程。

在这种情况下，最好是每个人都使用一个单一的风险登记册，并根据每个人的权限级别限制登记册的访问或可见性。这将有助于确保更容易识别风险的总体趋势和模式，并且上报的风险已经有了标准格式。

使用单一风险登记簿的另一个原因是帮助识别堆积的风险。

请记住，当不同的个体风险可能组合在一起，以一种意想不到的方式影响我们组织的部分数字景观时，就会出现堆叠风险。例如，可能有三种独立的风险，由组织的三个不同部分报告，所有这些风险都表明它们影响攻击面的同一部分或同一业务流程、应用程序或同一物理位置。我能建立联系的唯一方法是，如果这些信息在同一个地方并被记录下来，就能报告这些模式。

在获取和管理风险登记册的方式上，您掌握的情报越多，您在管理网络安全优先事项和使组织免受攻击、入侵或其他故障方面就越明智和有效。

例如，先进的网络风险登记册将允许风险所有者从应用程序、关键业务流程、资产、网站和其他关键参数的选择列表中进行选择。这使得风险登记簿能够通过这些相同的变量来显示风险。

网络风险登记册是一种反应性的持续运作机制，有助于了解任何时间点的整体风险状况。

登记册的有用性取决于它掌握信息的情况如何，以及它的风险信息选择设计得如何。

一个主要的好处是，由于每个风险的重要性(影响和概率)和邻近性都已到位，我现在可以对需要最大和最快关注的优先级做出明智的决策。

有非常全面的风险管理软件解决方案，我设计了其中的一个(AdaptiveGRC ),它们不需要花费太多就可以部署到位，特别是如果它们可以开箱即用并在以后进行改进的话。

仅仅依靠被动的风险管理技术是不合适的，也是不可靠的。现在，我们还应该看看什么是风险评估，以及如何使用它来更主动地确定差距。

风险评估 测试人员:

风险评估–对现有或计划活动、资产、服务、应用、系统或产品的潜在危险或差距进行主动检测的系统流程。

一旦风险管理流程到位，风险登记册是被动检测风险的好方法。

风险评估是一种积极主动的方法，可确保对任何高价值目标的风险进行常规分析和考虑。

风险评估旨在识别任何可能损害他们正在审查的特定目标的主要风险。他们捕获的问题和信息将取决于被评估的目标。例如，如果我正在对一个应用程序进行网络风险评估，我的问题可能包括了解该应用程序是否可以从互联网访问，它保存了多少记录，它处理或存储的信息是否包含个人或财务数据，它是否有预期的关键安全控制措施，它过去是否遇到过问题或损失，它使用了什么技术等等。

这些信息有助于我了解该应用程序作为目标的吸引力有多大，如果它遭到破坏可能会造成多大的损失，以及是否已经采取了足够的安全措施。

尽管风险评估过程因目标而异，但它们的目标始终是相同的:

*   目标的价值和敏感程度如何？
*   是否已经考虑并解决了正确的风险？
*   还有哪些差距(如果有的话)需要解决？

在投入使用之前，应定期进行风险评估(例如，每次更新或每年，以先发生的为准)。

对于网络安全，我们总是特别关注确保对数字领域至关重要的目标项目进行风险评估。具体来说:

*   技术服务(内部或外部)
*   新硬件，尤其是网络连接设备。
*   软件应用。
*   新的数据交换连接(入站或出站)。
*   任何数据存储位置。
*   网络接入点和其他网关。

风险评估的结果有助于积极了解适用于特定流程或组件的风险集合，这些流程或组件会影响我们组织的网络安全状况。

如果在评估过程中发现任何风险超过了我们的实质性阈值，我们也应该将其报告到风险登记簿中。

为了防止风险组形成并破坏我们的深度防御，我们可以使用风险登记簿来被动监控性能和风险评估，以主动检查关键目标中的重大风险链。

如何应用基于风险的网络安全管理 措施:

许多组织不太可能负担得起充分保护一切的费用。这意味着，有效地将精力集中在确保能够创造最高业务收入的项目上是很重要的，如果受到损害，将会对业务收益产生最大的影响。

这被称为采取基于风险的方法，因为我使用每个被损害的项目的潜在影响和价值来帮助确定其优先级。

基于风险的–一种考虑失败的财务影响及其概率以确定其相对重要性和优先处理 的方法。

在预算和资源有限的情况下，有必要采取循序渐进的方法来(I)了解真正的业务优先级是什么，(ii)优化环境以最大化安全将提供的价值和覆盖范围，以及(iii)提供适当的安全控制。

如果我试图保护一个新的或以前不安全的环境，我很可能会使用一种更快的、基于风险的方法来完成以下任务，而不是尝试在第一遍中保护所有内容:

*   首先确定价值最高的信息目标。
*   确定信息需要流经和流向的数字资产。
*   核实需求和业务案例，了解如何以及在哪里需要信息。
*   考虑我的组织面临的威胁及其发生的概率。
*   根据业务案例，尽量减少任何敏感数据的影响。
*   然后添加适当的安全控制。

如果已经建立了一个强大的风险登记册和一套风险评估流程，我可以利用这些资源来帮助实现这些目标。

在风险捕获和评估流程尚不成熟的情况下，新的网络安全经理通常会首先对组织进行高级风险评估。下一章将介绍这个过程的一个简单版本。