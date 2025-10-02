# 十九、 法律事项(版权、许可证、担保(缺)、出口限制)

## 十九、 法律事项(版权、许可证、担保(缺)、出口限制)

### Unofficial Translation Disclaimer / 非官方翻译声明

This is an unnofficial translation of the [Nmap license details](http://www.insecure.org/nmap/man/man-legal.html) into [Finnish]. It was not written by Insecure.Com LLC, and does not legally state the distribution terms for Nmap -- only the original English text does that. However, we hope that this translation helps [Finish] speakers understand the Nmap license better.

这是 Nmap 许可证明细 的非官方的中文译本。它并非由 Insecure.Com LLC 编写，不是有法律效 力的 Nmap 发布条款－－只有原英文版具有此法律效力。然而，我们希望该译本帮助中文用户更 好地理解 Nmap 许可证。

在 [`www.insecure.org/nmap/`](http://www.insecure.org/nmap/)可获得 Nmap 的最新版本。

Nmap 安全扫描器(C) 1996-2005 是 Insecure.Com LLC 的产品。 Nmap 也是 Insecure.Com LLC 的注册商标。这是一个免费软件，按照免费软件联盟 V2.0 的 GNU 普通公共许可证的规定，可以 重新发布和/或修改。这保证用户在一定的条件下可以使用、修改和重新发布此软件。如果需要 将 Nmap 嵌入专用软件，我们会销售相应的许可证(联系 sales@insecure.com)。很多安全扫 描器厂商已经获得了 Nmap 技术的许可证，如主机发现、端口扫描、OS 系统检测以及服务/版本 检测等技术。

注意，GPL 对“衍生工作”有重要的限制，但未给出有关的详细描述。为避免误解，我们认为如 果某个应用进行以下工作时被认为是该许可证的 “衍生工作”：

*   集成 Nmap 的源代码
*   读取或包含 Nmap 拷贝权的数据文件，如 nmap-os-fingerprints 或 nmap-service-probes.
*   执行 Nmap 并解析结果(与之相反的是，shell 执行或执行菜单应用，仅仅显示原始 Nmap 输出，则不是衍生工作。)
*   将 Nmap 集成/包含/组合至一个专用的可执行安装程序 如由 InstallShield 生成的安装 程序。
*   将 Namp 链接到进行上述工作的库或程序中。

名词“Nmap”包含了 Nmap 的任何部分以及衍生工作，因此不仅限于上述内容。上述内容使用了 一些常见的例子来说明衍生工作。这些限制仅适用于真正重新发布 Nmap 时。例如，不禁止开发 和销售 Nmap 的前端，可以任意发布，但必须说明从 [`www.insecure.org/nmap/`](http://www.insecure.org/nmap/)下载 Nmap

这些条款并不是在 GPL 之上增加限制，只是为了明确说明与 Nmap(有 GPL 许可证) 产品有关的 “衍生工作”。这类似于 Linus Torvalds 对 Linux 内核模块“衍生工作”的解释。我们的解释 仅适用于 Nmap - 不适合其它 GPL 产品。

如果对有 GPL 许可证限制的 Nmap 应用于非 GPL 工作有任何问题，我们将提供相关的帮助。如上 述条款所述，我们提供了可选的许可证以集成 Nmap 到专用应用和设备，这些合同已被销售给多 个安全厂商，通常都包含了永久的许可证以及提供优先支持、更新，并资助可持续的 Nmap 技术 开发。请发送邮件至 sales@insecure.com 以获得更多信息。

作为 GPL 的一个特殊例外 Insecure.Com LLC 授权许可链接该程序的代码至任何版本的 OpenSSL 库，这个库的发布符合等同于 Copying.OpenSSL 文件中包含的许可证，它的发布同时包含了两 个内容。除 OpenSSL 外，在任何方面都必须遵守 GNU GPL。如果改变这个文件，就可能超出该文 件的版本，但并不对此负责。

如果收到书面的许可证协定或合同文件，与上述条款不同时，该可选的许可证协定具有优先权 Nmap 软件提供源代码，这是因为我们认为用户有权在运行之前知道程序的工作内容，同时也使

用户可以检查软件的漏洞(目前还未发现)。

源代码还允许被移植到新的平台 修改 Bug 以及增加新功能 鼓励用户向 fyodor@insecure.org 发送更新，以并入正式的发布中。发送这些更新至 Fyodor 或 Insecure.Org 开发列表，表明允 许 Fyodor 和 Insecure.Com LLC 具有无限止的、非独有的权限来使用、修改和重新定义这些代 码的许可证。 Nmap 将一直以开放代码的方式提供，由于无法进行代码的许可证重新定义从而可 能导致了对其它开放软件项目的破坏问题(如 KDEt NASM)，这一点很重要。偶尔我们会向第三方 提供重新定义的代码许可证。如果希望为自己的贡献说明一个特定的许可证条件，在发送时指 明。

本程序的发布只是为了应用，不提供任何担保，也不隐含任何有关特定目的的商业或适用性担 保。在 [`www.gnu.org/copyleft/gpl.html`](http://www.gnu.org/copyleft/gpl.html) 的 GNU 通用公共许可证或 Nmap 中包含的 COPYING 文件中可查看更多细节。

还需要注意，Nmap 已经导致了一些编写拙劣的应用程序、TCP/IP 栈甚至操作系统的崩溃。Nmap 不能用于破坏关键系统，除非准备好系统受到崩溃的影响。Nmap 可能会造成系统或网络的崩溃 但我们不对 Nmap 可能产生的破坏和问题负任何责任。

由于崩溃的风险以及一些攻击者在攻击系统前使用 Nmap 进行检测，因此一些管理员对此感到不 安，报怨他们的系统受到扫描。因此，建议在扫描之间获得允许，哪怕是对一个网络很轻微的 扫描。

为安全起见，不要将 Nmap 安装在有特殊权限的环境(如 suid root)。

这个产品包含了由 [Apache Software Foundation](http://www.apache.org/) 开发的软件，发布时还包含了 [Libpcap](http://www.tcpdump.org/) 可移植 包捕获器的改进版本。Windows 版的 Nmap 使用了源于 [WinPcap library](http://www.winpcap.org/) 的 libpcap。正常的描 述支持由 [PCRE library](http://www.pcre.org/) 提供，这也是一个开放源码软件，由 Philip Hazel 编写。特定的原网 络功能使用了 [Libdnet](http://libdnet.sourceforge.net/) 网络库，由 Dug Song 编写，Nmap 发布时带有一个改进版本。Nmap 可以 与 [OpenSSL 加密工具库](http://www.openssl.org/)链接用于 SSL 版本检测。所有本段描述的第三方软件在遵守 BSD 风格软 件许可证下均可以免费发布。

美国出口控制：Insecure.Com LLC 认为 Nmap 归入美国 ECCN(出口控制分类编码) 5D922。这个 分类称为“不受 5D002 控制的信息安全软件”。这个分类的唯一限制是 AT(反恐怖主义)，这个 限制几乎适用于所有的产品，禁止出口到一些国家，如伊朗和朝鲜。因此，出口 Nmap 不需要任 何特殊的许可证、允许或其它政府授权。