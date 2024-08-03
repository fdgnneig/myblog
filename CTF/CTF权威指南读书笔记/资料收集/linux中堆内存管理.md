# Linux堆内存管理深入分析上
- https://introspelliam.github.io/2017/09/10/pwn/Linux%E5%A0%86%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E6%B7%B1%E5%85%A5%E5%88%86%E6%9E%90%E4%B8%8A/
- 总结
  - main_arena以及thread_arena在进程虚拟内存映射方面的表现
  - arena的个数是跟系统中处理器核心个数相关的
  - 当多线程所需的arena数量大于处理器所能支持的最大arena数量时，会重复使用已经分配好的多个arena中的一个，从而支持多个线程
  - 只有一个heap segment的main arena和thread arena的内存分布图：
  - 一个thread arena中含有多个heap segments的内存布局图
# Linux堆内存管理深入分析下
- https://introspelliam.github.io/2017/09/15/pwn/Linux%E5%A0%86%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E6%B7%B1%E5%85%A5%E5%88%86%E6%9E%90%E4%B8%8B/


# Linux堆内存管理深入分析上
![](pic/2021-07-01-20-19-28.png)
Linux堆内存管理深入分析上 _ Introspelliam (2023_3_19 17_54_07).html
# Linux堆内存管理深入分析下
![](pic/2021-07-01-21-05-59.png)
Linux堆内存管理深入分析下 _ Introspelliam (2023_3_19 17_53_05).html