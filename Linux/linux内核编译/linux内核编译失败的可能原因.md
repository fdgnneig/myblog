# 资料收集
https://blog.csdn.net/Luckiers/article/details/124544179


# 为什么内核编译过程中会报错？
- 可能原因
  - 缺少本地依赖库：通过apt安装指定程序可以解0
  - config设置：通过设置或取消设置指定的内核配置选项可以解决
  - gcc版本与内核版本不匹配：使用对应版本gdb解决
  - 内核源文件版本问题：尽量保证要编译的内核和编译环境使用的内核版本相互匹配
    - 当编译环境使用的是较旧的内核版本，尝试编译较新的版本内核时，新版本内核中部分依赖的函数在编译环境的旧版本内核中不存在

# 内核编译排错思路
- https://blog.csdn.net/weixin_36432129/article/details/130668648
  - linux内核编译不通过问题的两种排查方法（非正式）-CSDN博客 (2024_7_24 23_58_06).html

# 漏洞复现为目标的linux内核编译最佳实践
- https://blog.csdn.net/u014679440/article/details/135659333
  - 制作基于ubuntu-base的文件系统，用于sudo,网络内核cve调试-CSDN博客 (2024_7_24 17_41_28).html
- 文中作者通过docker搭建不同版本的ubunto系统，之后将下载到的linux内核代码在对应docker中编译
- 编译环境的linux内核版本与要编译的linux内核版本是否匹配会影响内核编译过程中是否报错
  - 文中在1604ubuntu版本的docker中编译 linux-4.8.13.tar.xz 版本的内核

# 编译环境中linux内核版本对编译linux内核的影响 form chatgpt4o
- 选择与 Linux 内核版本相近的 Ubuntu 发行版作为编译环境可以提高内核编译的成功率
- 工具链的兼容性
  - 每个 Ubuntu 发行版提供的编译器（如 GCC）、链接器、构建工具和库版本都与发布时的内核版本相匹配。使用与内核版本相近的 Ubuntu 版本，可以确保这些工具链版本的兼容性，从而减少编译过程中出现的错误。
- 内核依赖的库和头文件
  - 内核编译依赖于系统上的一些库和头文件（如 libc、glibc 等）。选择相近版本的 Ubuntu 发行版确保这些库和头文件的版本与内核源码期望的版本相匹配，避免由于版本不一致导致的编译错误或不兼容问题。
- 配置和默认选项
  - 不同版本的内核和发行版可能会有不同的配置选项和默认参数。相近版本的 Ubuntu 发行版会提供更接近的默认配置选项和参数，使得内核编译配置步骤（如 make menuconfig）更加顺利，减少人为配置错误的概率。
- 依赖包的可用性
  - 较新的内核版本可能需要一些新的工具或库，这些工具和库在较旧的 Ubuntu 版本中可能不可用或版本过低。使用相近版本的 Ubuntu 发行版可以确保这些依赖包在官方仓库中是可用且版本合适的，避免了手动下载和安装第三方依赖包的麻烦。

# ubuntu不同发行版和linux内核的对应关系
## from 豆包
Ubuntu 23.10：代号为 Mantic Minotaur 的 Ubuntu 23.10发行版本已升级基于 Linux Kernel 6.3内核。
Ubuntu 23.04：被称为“Lunar Lobster”的 Ubuntu 23.04搭载了最新的 Linux 6.2内核系列。
Ubuntu 22.10：预计将使用 Linux 5.19内核。
Ubuntu 22.04 LTS：默认内核版本为 Linux 5.15 LTS。
Ubuntu 21.10：内核版本为5.13.0-20-generic。
Ubuntu 21.04：将 Linux 内核升级至了5.11。
Ubuntu 20.10：包含5.8版 Linux 内核。
Ubuntu 20.04 LTS：系统自带的内核版本为5.4.0-67-generic。
Ubuntu 19.10：装备了 Linux 5.2kernel 内核。
Ubuntu 19.04：使用 Linux 5.0内核。
Ubuntu 18.10：基于 Linux 4.17内核。
Ubuntu 18.04 LTS：内核版本为4.15.7。
Ubuntu 17.10：内核同样基于 Linux Kernel 4.13。
Ubuntu 17.04：包括 Linux 内核4.10。
Ubuntu 16.10：基于 Linux Kernel 4.8内核。

## from gpt4o
- https://blog.csdn.net/SHELLCODE_8BIT/article/details/132266380 中给出了相同信息，证明信息具有一定可信性
Ubuntu 4.10 (Warty Warthog)
内核版本：2.6.8
Ubuntu 5.04 (Hoary Hedgehog)
内核版本：2.6.10
Ubuntu 5.10 (Breezy Badger)
内核版本：2.6.12
Ubuntu 6.06 LTS (Dapper Drake)
内核版本：2.6.15
Ubuntu 6.10 (Edgy Eft)
内核版本：2.6.17
Ubuntu 7.04 (Feisty Fawn)
内核版本：2.6.20
Ubuntu 7.10 (Gutsy Gibbon)
内核版本：2.6.22
Ubuntu 8.04 LTS (Hardy Heron)
内核版本：2.6.24
Ubuntu 8.10 (Intrepid Ibex)
内核版本：2.6.27
Ubuntu 9.04 (Jaunty Jackalope)
内核版本：2.6.28
Ubuntu 9.10 (Karmic Koala)
内核版本：2.6.31
Ubuntu 10.04 LTS (Lucid Lynx)
内核版本：2.6.32
Ubuntu 10.10 (Maverick Meerkat)
内核版本：2.6.35
Ubuntu 11.04 (Natty Narwhal)
内核版本：2.6.38
Ubuntu 11.10 (Oneiric Ocelot)
内核版本：3.0
Ubuntu 12.04 LTS (Precise Pangolin)
内核版本：3.2
Ubuntu 12.10 (Quantal Quetzal)
内核版本：3.5
Ubuntu 13.04 (Raring Ringtail)
内核版本：3.8
Ubuntu 13.10 (Saucy Salamander)
内核版本：3.11
Ubuntu 14.04 LTS (Trusty Tahr)
内核版本：3.13
Ubuntu 14.10 (Utopic Unicorn)
内核版本：3.16
Ubuntu 15.04 (Vivid Vervet)
内核版本：3.19
Ubuntu 15.10 (Wily Werewolf)
内核版本：4.2
Ubuntu 16.04 LTS (Xenial Xerus)
内核版本：4.4
Ubuntu 16.10 (Yakkety Yak)
内核版本：4.8
Ubuntu 17.04 (Zesty Zapus)
内核版本：4.10
Ubuntu 17.10 (Artful Aardvark)
内核版本：4.13
Ubuntu 18.04 LTS (Bionic Beaver)
内核版本：4.15
Ubuntu 18.10 (Cosmic Cuttlefish)
内核版本：4.18
Ubuntu 19.04 (Disco Dingo)
内核版本：5.0
Ubuntu 19.10 (Eoan Ermine)
内核版本：5.3
Ubuntu 20.04 LTS (Focal Fossa)
内核版本：5.4
Ubuntu 20.10 (Groovy Gorilla)
内核版本：5.8
Ubuntu 21.04 (Hirsute Hippo)
内核版本：5.11
Ubuntu 21.10 (Impish Indri)
内核版本：5.13
Ubuntu 22.04 LTS (Jammy Jellyfish)
内核版本：5.15
Ubuntu 22.10 (Kinetic Kudu)
内核版本：5.19
Ubuntu 23.04 (Lunar Lobster)
内核版本：6.2
Ubuntu 23.10 (Mantic Minotaur)
内核版本：6.5