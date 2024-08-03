- https://vul.360.net/archives/645
- 攻击DSP：揭开高通Hexagon的神秘面纱 (2024_1_1 16_31_20).html

# 该研究中提到的参考资，先依次学习此类资料
- Attacking Hexagon: Security Analysis of Qualcomm’s aDSP简单介绍了Hexagon架构、如何与AP进行通信、攻击面分析以及盲测DSP应用；
  - attacking-hexagon-recon19.pdf
  - 分析见 Attacking Hexagon Security Analysis of Qualcomms aDSP.md

- Pwn2Own Qualcomm DSP介绍了通过QEMU模拟的方式Fuzz DSP、内核攻击面、发现的漏洞（降级攻击、数据序列化、应用和内核）；
  - Pwn2Own Qualcomm DSP - Check Point Research (2024_1_1 16_32_40).html

- In-Depth Analyzing and Fuzzing for Qualcomm Hexagon Processor 介绍了一种基于路径覆盖的Fuzz方法
  - AS21-In-Depth-Analyzing-Fuzzing-Qualcomm-Hexagon-Processor.pdf
  - 视频 https://www.youtube.com/watch?v=V5B9zDHU2Do

确定ida8.3是否原生支持hexagon 不支持

qemu是支持hexagon架构的

可以使用hexagon SDK中的objdump工具反汇编 hexagon程序

# 关键点摘抄
- 目前常见的是第六代Hexagon芯片（V6x），它有三个PD（Protection Domains）：supervisor、Guest OS和User。supervisor拥有最高权限。supervisor有一个开源实现hexagonMVM，我认为它有一定的参考性，但由于代码年久失修，它可能无法反映近几年硬件情况。我其实更愿意称Guest OS为kernel，这样也许更容易理解（不一定对）。高通实现了名为QuRT的内核，但它并未开源。Linux内核同样支持Hexagon架构，你可以在arch/hexagon下找到相关代码。应用通常在User mode下运行，它的权限最
- 你可以使用SDK中hexagon-llvm-objdump工具反编译二进制文件，从而获取相关指令，也可以通过IDA插件idp_hexagon反编译二进制文件。目前IDA插件并不能将指令翻译为伪代码，因此只能通过阅读汇编指令来了解程序行为。
- SDK提供了一些开源库，比如用于神经网络的hexagon_nn库。源码分析要比阅读汇编指令轻松的多，因此我优先选择分析这些开源库。
- hexagon_readelf工具可以读取hexagon_nn_skel.so各个段信息：
- 一种adb中输出log信息的方法 adb logcat -s adsprpc


- 在应用漏洞挖掘层面
  - dsp应用位于手机特定目录，与so形式存在，应用加载时首先加载shell文件
  - sdk提供了开源库，优先分析开源库 神经网络的hexagon_nn库
  - 通过分析开源代码，实现了信息泄露漏洞 任意地址写漏洞
  - 漏洞利用中，通过上述两漏洞，调用qurt_mem_mprotect()修改内存属性为可写可执行，之后将shellcode写入，并执行


- 在内核漏洞挖掘层面
  - 通过设备名称+qurt_qdi_obj_t结构体 找到驱动处理函数入口
  - 找到驱动处理函数的传参方式
  - 在i2c驱动中找到多个漏洞，包括任意地址写0和任意地址读，任意地址写，内存赋值的目的地址和源数据均用户可控


- 内核漏洞利用层面
  - 总体思路：建起从AP到Hexagon内核攻击的桥梁。要在hexagon_nn应用中编写shellcode，它可以将AP发来的请求转发给上述读写原语，从而实现在AP侧直接读写Hexagon内核。
    - hexagon_nn项目中编写读写原语，将使用sdk编译，得到读写原语的shellcode，此时可以劫持dsp中的函数指针，利用该shellcode，可以在dsp中执行任意命令
    - ***自己分析来看，此时的驱动漏洞实现了hexagon内核的任意地址读写，所以有潜力基于此实现向dsp注入代码，实现调试器，推测gongxiling的思路就是通过驱动代码漏洞实现的***
  - 是能否借助Hexagon攻击AP内核？例如能否将内存映射到Hexagon中（Hexagon芯片很高级：它有自己的MMU），通过Hexagon来攻击Android内核？
    - 该想法最终没实现，但具体思路是需要研究Hexagon的页表格式并识别出页表寄存器
    - hexagonMVM代码中创建页表的代码可以参考，从而获得有关页表的知识
    - 《Hexagon Virtual Machine Specification》中描述了页表相关格式
  - 上述想法未能实现的原因是 
    - dsp中存在mmu以及页表、页寄存器相关概念，但是dsp所操作的内存可能不是物理内存，在ap给dsp分配内存时，有时分配的是不连续的内存，有的设备会在DSP芯片前面加上SMMU，SMMU可以将零散的（不连续的）物理地址映射到连续的虚拟地址上，此时DSP看到的是连续的“物理地址”（实际是SMMU给它营造的虚拟地址）
    - SMMU的加入显然会限制DSP的访存能力：DSP看到的不再是物理地址，而是SMMU导出的虚拟地址。这意味着DSP每次访存都要经过SMMU的翻译。此时，即便我可以篡改DSP的页表，也无法逃离SMMU的牢笼（它们之间的关系类似hypervisor和kernel）

- fuzz
  - 需要基于谷歌Pixel 4工程机，工程机的Secure boot是关闭的，，它存在着一种隐秘的攻击：钉枪攻击。简单来说这种攻击方式利用了ARM架构的调试特点：它支持核间调试。我可以在一台Pixel 4手机上完成自我调试，例如使用CPU1调试CPU2。有意思的是：我（位于EL1）可以使被调试的CPU进入EL3，以侵占的方式篡改EL3内存。实现这种攻击只需要能够加载AP侧内核module即可

- 上述概念中，llm进行解释

在ARM架构中，EL（Exception Level）是一个用来区分处理器运行模式的层级。不同的EL代表了处理器的不同特权级别，每个级别都有其特定的权限和访问资源的能力。以下是EL的各个级别及其含义：

EL0（Application Level）：这是最低的特权级别，通常用于运行应用程序。在这个级别，处理器只能访问有限的系统资源，并且受到严格的安全限制。

EL1（Hypervisor Level）：在虚拟化环境中，EL1通常用于运行虚拟机监视器（Hypervisor），它负责管理虚拟机并提供虚拟机与硬件之间的接口。

EL2（Secure World Level）：这个级别在ARM的TrustZone技术中使用，用于运行安全操作系统（Secure OS），它提供了一个隔离的环境来执行安全敏感的任务。

EL3（Secure Hypervisor Level）：这是最高的特权级别，通常用于运行安全监视器（Secure Hypervisor），它提供了最严格的安全控制，并且可以访问所有的系统资源。

在您提到的“钉枪攻击”中，EL1和EL3的上下文是这样的：

EL1：攻击者位于这个级别，通常是在虚拟机监视器（Hypervisor）或操作系统内核中运行。

EL3：这是安全监视器（Secure Hypervisor）运行的级别，它负责管理整个系统的安全策略。

钉枪攻击（Spearphishing）通常是指一种社会工程学攻击，攻击者通过欺骗手段诱使目标泄露敏感信息或执行恶意操作。在您提供的上下文中，钉枪攻击可能是指利用ARM架构的调试功能，攻击者在EL1级别上通过某种方式（可能是利用漏洞或调试接口）影响或控制位于EL3级别的安全监视器，从而实现对系统安全策略的绕过或破坏。

这种攻击的关键在于，攻击者能够在较低的特权级别（EL1）上执行操作，这些操作最终影响到了更高的特权级别（EL3），这通常是非常困难的，因为每个EL都有严格的安全隔离措施。在ARM架构中，这种攻击可能涉及到利用调试接口、硬件漏洞或其他安全漏洞来实现跨特权级别的操作。