# GNU make官网
https://www.gnu.org/software/make/
# make用户手册
https://www.gnu.org/software/make/manual/
# make用户手册web页面
https://www.gnu.org/software/make/manual/make.html
# 常用参数
## -j
- 用于并行执行编译命令
- 用make -j带一个参数，可以把项目在进行并行编译，比如在一台双核的机器上，完全可以用
- make -j4，让make最多允许4个编译命令同时执行，这样可以更有效的利用CPU资源。
- 还是用Kernel来测试：
  - 用make： 40分16秒
  - 用make -j4：23分16秒
  - 用make -j8：22分59秒
- 由此看来，在多核CPU上，适当的进行并行编译还是可以明显提高编译速度的。但并行的任务不宜太多，一般是以CPU的核心数目的两倍为宜。
- 不过这个方案不是完全没有cost的，如果项目的Makefile不规范，没有正确的设置好依赖关系，并行编译的结果就是编译不能正常进行。如果依赖关系设置过于保守，则可能本身编译的可并行度就下降了，也不能取得最佳的效果。
- 实际使用过程中，-j参数常与nproc命令一起使用，该命令返回可用的处理器单元数量，从而实现代码的高效编译
```bash
make CC="clang -O2 -fno-omit-frame-pointer -g -fsanitize=address" -j$(nproc)
```