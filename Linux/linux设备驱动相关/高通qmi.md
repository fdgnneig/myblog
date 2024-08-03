    - 关于qmi
      - https://osmocom.org/projects/quectel-modems/wiki/QMI
      - https://en.wikipedia.org/wiki/Qualcomm_MSM_Interface
    - 关于qmi可能的对应的驱动程序
      - 在高通msm内核中，支持QMI（Qualcomm MSM Interface）的驱动程序主要有以下几种123：
        - qmi_wwan：这是一个在Linux内核中用于实现QMI协议的驱动程序，它允许主机系统与基于Qualcomm的无线设备进行通信3
        - GobiNet：这是另一种支持QMI的驱动程序2
        - USB Serial：这是一种usb串口驱动，有两种适配方式，修改option.c或qcserial.c文件2。
        - qcmap_msgr：这是9X07模块提供的服务来进行拨号，通过与dsi_netctrl对比，qcmap_msgr接口更清晰，而且操作也简单1。
        - 以上驱动程序都可以用于支持QMI，具体选择哪种驱动程序取决于你的具体需求和设备配置。在实际使用中，可能需要对驱动程序进行一些定制化的修改以适应特定的硬件和软件环境。
      - 实际安卓源码中，定位到 linux设备驱动\代码保存\源码\drivers\net\usb\qmi_wwan.c 可以后续分析
    - qmi相关学习资料
      - https://zhuanlan.zhihu.com/p/62245921
        - 该资料给出了qmi 大致基础

- qmi源码文档
- https://android.googlesource.com/kernel/msm/+/android-7.1.0_r0.2/Documentation/arm/msm/msm_qmi.txt

- copilot
Qualcomm MSM 内核中的 QTI 服务与 Qualcomm MSM 接口 (QMI) 相关，QMI 是用于与 Qualcomm 基带处理器交互的专有接口1。 在Linux内核中，可以通过两个互斥的驱动程序来使用QMI：GobiNet和qmi_wwan1。这两个驱动程序采用完全不同的方法来处理协议。 GobiNet 是一个复杂的驱动程序，它在内核中实现大部分核心协议逻辑，而 qmi_wwan 将所有这些任务留给用户空间进程1。 QMI 客户端驱动程序向 QMI 服务发送请求并接收响应。 QMI 客户端驱动程序向 QMI 服务注册以接收有关系统事件的指示，并且当系统中发生事件时，QMI 服务将指示发送到客户端2。 请注意，使用内核代码需要对 Linux 和编程有很好的了解。如果您对这些主题感到不舒服，您可能需要向熟悉这些主题的人寻求帮助。