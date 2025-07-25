#  eSIM漏洞危机-几十亿手机可被监听  
原创 子午猫  网络侦查研究院   2025-07-13 00:29  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/4kCmTUe2v2bujwd3M0M1ICStsbhAHWtth8dQwoBBFoNDafDAzGbm1sCA8bqVWIjs40A8lu9rtuD4yeOOwDNadg/640?wx_fmt=png "")  
  
  
  
在科技飞速发展的当下，通信技术的革新日新月异。过去几年，全球手机用户都明显感受到了一种变化，那就是传统SIM卡正在逐渐被新一代eSIM（嵌入式用户身份模块）技术所取代。这种小小的、无法物理插拔的芯片，仿佛是通信领域的一颗新星，以其安全、便利的特性，迅速在智能手机、平板、可穿戴设备乃至车联网终端等众多领域占据了一席之地。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ok4fxxCpBb4ZlVOZhMsM6ibGRbzvicWk5tGbmHDHfhtPDNrgh7JPwYiacMNoCYVvpcGdXMHKo9gdXt9mB8oG3NZGw/640?wx_fmt=png&from=appmsg "")  
  
当人们都在为eSIM带来的便捷欢呼时，一项犹如晴天霹雳的最新研究结果却给整个通信行业浇了一盆冷水。Security Explorations公司创始人Adam Gowdiak近期披露的消息，让人们看到了这一“安全”通信核心背后隐藏的巨大危机。全球数十亿eSIM卡中使用的关键组件Java Card，竟然存在严重的安全漏洞。这可不是一个小问题，攻击者利用这个漏洞，不仅能够在物理接触设备后轻易提取关键凭据，甚至还能通过远程OTA（空中下载）方式在eSIM芯片中植入恶意代码，进而实现监听、SIM克隆、通信劫持等一系列令人胆寒的攻击手段。  
### 01 漏洞源头：六年前被忽视的安全缺陷  
  
要说清楚这个漏洞，就不得不提到Oracle Java Card虚拟机中的“类型混淆”漏洞。Java Card是Oracle专为资源受限设备打造的Java应用执行平台，它的身影广泛出现在eSIM芯片、银行卡、身份认证设备等诸多领域。早在六年前，Gowdiak就发现了它在执行字节码时缺乏关键的类型校验，这一缺失可能会导致权限绕过。按常理来说，这样的问题应该引起高度重视并及时修复，可让人意想不到的是，Oracle竟然以“问题不适用”为由，对这个漏洞未予修复。  
  
近期，Gowdiak带着对这个漏洞的担忧，对Kigen生产的eUICC（嵌入式通用集成电路卡）进行了攻击测试。这一测试的结果令人震惊，他先是通过物理方式成功提取了设备私钥，接着下载的运营商配置文件（profile）竟然是以明文传输的，而这些文件中还包含着关键密钥。再加上Java Card字节码验证的缺失，攻击者就像拿到了一把万能钥匙，能够安装任意恶意applet，从而完全控制eSIM，并且整个过程没有任何安全告警。  
  
其实，在Java Card的发展历程中，也有过一些其他小的安全问题出现，但都没有像这次的“类型混淆”漏洞这样，具有如此大的潜在危害。曾经也有开发者反馈过一些关于Java Card在某些特殊环境下的兼容性问题，但都没有涉及到像权限绕过、完全控制eSIM这样严重的情况。  
### 02 eSIM多配置共存：一处破防，全局失守  
  
eSIM与传统SIM卡相比，确实有着明显的优势，其中之一就是它支持多个配置文件并存。用户可以同时保有主运营商与本地临时运营商的服务，这使得出境漫游、切换资费等操作变得更加灵活，给用户带来了极大的便利。然而，事物总是具有两面性，这种“共处一芯”的机制，在带来便利的同时，也带来了新的攻击面。  
  
Hacker News论坛用户就曾敏锐地指出：“假如你在某国旅行时安装当地运营商的eSIM作为副配置，若该国政府与运营商合作，可能借此提取你主配置中的关键密钥，监听所有通信。”这可不是危言耸听，Gowdiak也认可这一推测，他表示：“一旦某个恶意配置被植入，它可以突破芯片边界，影响上面存储的所有运营商配置安全。”这意味着攻击者不需要费尽心机去攻破每一个运营商的网络，只要利用其中一个“合作”或“被攻陷”的节点，就可能达成系统级的控制。  
  
此次被测试的Kigen芯片，可不是一个小角色，该公司为全球数十亿设备提供eSIM服务。虽然Kigen宣称已经向“数百万”受影响设备推送了补丁，但考虑到其庞大的服务覆盖规模，仍然有大量设备处于暴露状态。而且，Java Card并非Kigen所独有，根据GSMA规范，它是被全球所有eSIM供应商广泛采用的标准组件。这就意味着，这个漏洞的影响范围可能远远超出了一个厂商、一个型号，而是会波及整个eSIM产业生态。  
  
在实际应用中，已经有一些用户反馈，在使用多配置eSIM的设备进行漫游时，出现了一些不明原因的数据流量异常情况。虽然目前还不能完全确定这些情况就是由这个漏洞引起的，但也足以让人们对eSIM的安全问题产生更多的担忧。  
### 03 攻击路径明确，国家级威胁尤为严峻  
  
从攻击路径来看，利用本次漏洞需要具备一些特定的能力。首先，初期需要获取目标设备的物理接触权限，当然，通过供应链实现这一目的也是一种途径。其次，攻击者要熟悉eSIM配置机制与运营商密钥管理流程，这可不是一件简单的事情，需要对通信技术有深入的了解。最后，还需要编写并植入Java Card Applet，完成远程OTA部署。  
  
对于普通网络犯罪团伙来说，要在短期内大规模复现这些攻击行为确实存在很大难度。但是，我们不能忽视的是，对于那些具备丰富资源、充足时间以及强大供应链渗透能力的国家级攻击者而言，这或许就是又一个可利用的“通信根基漏洞”。他们可以通过这个漏洞实施一系列恶意行为，比如克隆eSIM，实现短信、电话的中转与监听；植入持久恶意代码，让恶意程序长期驻留于设备通信模块；拦截2FA认证信息，发起账户接管与横向攻击；甚至还能破坏或封锁通信能力，制造信息断网或瘫痪效果。  
  
在当今，零信任架构尚未完全覆盖通信芯片，安全审计也还存在盲区，这样的“芯片级后门”很难被传统防御工具发现。就像在一个看似坚固的城堡中，突然出现了一条不为人知的密道，而城堡中的守卫却对此一无所知。  
  
值得一提的是，在一些对信息安全要求极高的行业，如金融、政府等，已经开始对eSIM的使用进行重新评估。一些金融机构表示，虽然eSIM带来了便利，但在这个漏洞问题没有得到彻底解决之前，他们会谨慎考虑是否继续在相关设备中采用eSIM技术。  
  
eSIM的便利性和普及确实是通信未来的趋势，这一点不可否认。但如果其安全机制无法与技术演进保持同步，那么带来的可能不是人们所期望的自由和便捷，而是通信安全的失控。此次事件给整个产业界敲响了警钟，安全绝不是在上线之后发现问题再去补救，而应该从架构选型开始就介入。从Java Card规范，到eSIM多配置设计，再到OTA信任模型，每一个环节的“合规”技术决策，都有可能成为系统性风险的诱因。面对新一代通信技术，我们唯有从“底层信任”开始重塑，才能筑牢真正安全的通信生态，让通信技术在安全的轨道上健康发展。否则，未来我们可能会面临更多类似的安全危机，通信领域的发展也将受到严重的阻碍。  
  
  
  
**END**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/4kCmTUe2v2bujwd3M0M1ICStsbhAHWtt0VVqCfFLOVnpmeNJ3R59doWtI0AmqLn4Qkic8aAS06l0pATjcYx10zw/640?wx_fmt=png "")  
  
  
