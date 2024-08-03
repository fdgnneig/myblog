### 系统解答fastbin合并问题
- [原创]关于fastbin合并问题的研究-Pwn-看雪论坛-安全社区_安全招聘_bbs.pediy.com (2023_3_20 11_22_58).html
- malloc_consolidate既可以作为fastbin的初始化函数，也可以作为fastbin的合并函数。fastbin的合并是在满足了指定条件后，调用malloc_consolidate实现的
- fastbin会在以下情况下进行合并（合并是对所有fastbin中的chunk而言）。
  - malloc：
    - 1、在申请large chunk时。
    - 2、当申请的chunk需要申请新的top chunk时。
  - free：
    - 3、free的堆块大小大于fastbin中的最大size。（注意这里并不是指当前fastbin中最大chunk的size，而是指fastbin中所定义的最大chunk的size，是一个固定值。）