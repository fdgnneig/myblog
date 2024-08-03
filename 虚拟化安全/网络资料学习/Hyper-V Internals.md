- Hyper-V Internals 该博客内容均关于hyper-v
- http://hvinternals.blogspot.com/2015/10/hyper-v-internals.html

# Hyper-V debugging for beginners
- 为了调试虚拟机管理程序，Microsoft 开发了 WinDBG hveexts.dll 的特殊扩展，不幸的是，该扩展不包含在分发调试器中，并且仅对合作伙伴可用（可能是因为该扩展需要 hvix64.exe 的符号，而该符号不存在） . 此外，在目录 winxp 中，位于带有 WinDBG 的文件夹中，是 nvkd.dll 的扩展，用于调试扩展虚拟交换机 Hyper-V。
- 使用在虚拟机中运行hyper-V本体（hvix64.exe），尝试调试hyper-V本身，解决方案是为运行hyper-V的虚拟机创建虚拟串口，通过读取该串口获得调试信息，并使用windbg中的组件连接该串口，最后使用ida连接后者，从而实现调试
- 查找符号信息
  - 在 IDA PRO 中加载 hvix64.exe 时，我们会得到大约 3000 个函数，其名称类似于 sub_FFFFF8000XXXXX，因为不幸的是，微软没有为管理程序提供符号信息。有利于hypervisor的研究可以先尝试识别一些功能，无需详细研究。
  - 首先，值得使用 bindiff（或 diaphora）来比较提供符号信息的文件 hvix64、hvloader 和 winload。比较显示，网络功能（e1000）、USB、密码学和其他一些功能与 winload.exe 中的功能完全相同（在 Windows Server 2016 中，调试功能已移至单独的模块）。这将有助于设置 500 个职能的任命。相同的 bindiff 允许您将匹配函数的名称从一个数据库移动到另一个 idb。但是，这种方法应该谨慎使用，不要移动所有完全匹配的函数。至少应该通过视觉比较图匹配函数（Ctrl + E）分析结果。
  - 接下来，让我们定义异常/中断函数，它们是处理器架构 x86 的标准。用python（ParseIDT.py）编写了一个小脚本来解析IDT，它必须在IDA PRO中运行，通过WinDBG的调试模块连接到管理程序。
  - 在没有找到 ISR 的情况下，检查 IDA PRO 中的问题列表选项卡，因为在 IDA 执行的自动分析代码中找不到这些程序。
  - 接下来，您可以在读取字段值 VMCS 后在 VM 中定义退出过程。这可以在程序在 hvix64.exe 中填充 VMCS 后完成，或者使用这个脚本 display-vmcs.py，它在管理程序的上下文中读取所有字段 VMCS 并打印它们的值。 
- 查找各个分区的id以及对应调用hypercall的权限
  - 每个虚拟机，以及直接与 OS 组件一起安装的 Hyper-V 都以分区（partition）的形式呈现。每个部分都有自己的标识符，该标识符对于主机服务器必须是唯一的。
  - 每个部分都被赋予创建权限（结构 HV_PARTITION_PRIVILEGE_MASK），这决定了执行特定超级调用的能力。

# Thursday, October 22, 2015 Hyper-V internals
- 根部分（父分区，根操作系统） - Windows Server 2012 R2 包含 Hyper-V 组件；
- 来宾操作系统——安装了 Windows Server 2012 R2 的虚拟机 Hyper-V； 
- TLFS – Hypervisor 顶级功能规范：Windows Server 2012 R2； 
- LIS – Linux 集成服务
- 发现错误，后来收到编号 MS13-092（错误组件 Hyper-V Windows Server 2012 允许您从客户操作系统发送 BSOD 中的管理程序或在其他客户操作系统中运行任意代码，这些操作系统在易受攻击的主机上运行服务器），这对微软工程师来说是一个非常不愉快的惊喜。在此之前，近三年来，没有人发现 Hyper-V 中的漏洞。只有 MS10-102 是在 2010 年底发现的。在这四年中，云服务的普及率大大提高，研究人员对底层云系统的安全管理程序越来越感兴趣。 （另一个错误 MS15-042 已修复，但没有对此的详细概述）。
- 然而，公开可用的工作数量很少：研究人员不愿意花时间探索这种复杂且记录不充分的架构解决方案。本文没有描述虚拟机管理程序的具体漏洞，但它应该阐明 Hyper-V 的内部工作原理，从而部分简化未来的研究。
- 本文将介绍hypervisor的一些特性，特别是vmbus消息处理机制组件（使用窃取管理程序机制）。 （在阅读本文之前，建议先了解 ERNW 的报告 (http://goo.gl/1Cvotv)，«Hyper-V 初学者调试» (http://hvinternals.blogspot.com/2015/10/hyper-v-debugging-for-beginners.html) 和 Hypervisor TLFS (http://goo.gl/9dISj7)
## VMBUS
- 