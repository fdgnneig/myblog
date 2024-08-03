# libfuzzer编译链接
https://i-m.dev/posts/20190831-143715.html
# libfuzzer测试执行
https://i-m.dev/posts/20190901-210526.html
# 进程、作业、并行fuzz
- 测试代码：
```c++
// process.cpp
#include <iostream>

int global_count = 0;

extern "C" int LLVMFuzzerTestOneInput(const uint8_t *Data, size_t Size) {
    if (global_count < 5) {
        std::cout << "global count: " << ++global_count << std::endl;
    }
    return 0;
}
```
- 编译：clang -fsanitize=fuzzer process.cpp -o fuzzer
- 当使用默认选项启动模糊测试程序时：./fuzzer
- 只会有一个作业，当然也只有一个测试进程。
- 此时，libFuzzer的日志信息会输出到终端的stderr，程序本身的stdout会被输出到终端的stdout。
- 而其中所有测试样例其实都是运行在同一个进程内，也就是说函数LLVMFuzzerTestOneInput被循环调用了，因此全局变量将会在每个样例运行时共享。如上述示例程序的stdout为：
    global count: 1
    global count: 2
    global count: 3
    global count: 4
    global count: 5
- 这也就是libFuzzer为什么建议不要去修改全局变量，因为如果此做，同一个测试样例在两次被修改得不同的全局变量环境下运行，可能会产生不一致的结果，这违反了输入-行为确定性的规则。
- 然后如果使用jobs选项指定作业数量：./fuzzer -jobs=2
- 此时日志和程序输出不会输出到终端，而是重定向到fuzz-N.log文件，查看其内容，发现二中中打印的global_count都是1、2、3、4、5，这说明二者运行在不同的进程之中，全局变量也是独立的。
- 但是多任务也会随之引发协调问题，每个任务并不知道其他任务发现了哪些样例，这时候需要指定reload等选项应对这些问题。
- 然后是进程的并发数量问题，这个数量由workers参数指定。
- 当指定了多个任务时，程序启动时会先产生一个master进程，同时并发启动相应数量个worker进程，master会把jobs分配到workers上去执行，当某个任务结束后，相应进程终止，同时master会启动一个新的任务进程分配到对应的worker上，平均每个worker上会分配jobs/workers个任务。
- ./fuzzer -jobs=16
- 对于8核电脑，其进程树如下：
```bash
fuzzer -jobs=16
  ├─sh -c ./fuzzer >fuzz-0.log 2>&1
  │   └─fuzzer
  │       └─{fuzzer}
  ├─sh -c ./fuzzer >fuzz-1.log 2>&1
  │   └─fuzzer
  │       └─{fuzzer}
  ├─sh -c ./fuzzer >fuzz-2.log 2>&1
  │   └─fuzzer
  │       └─{fuzzer}
  ├─sh -c ./fuzzer >fuzz-3.log 2>&1
  │   └─fuzzer
  │       └─{fuzzer}
  └─5*[{fuzzer}]
```
- 如果电脑不准备干其他活，可以把workers提升到CPU核心数量，满载并行。但是再往上加worker数量大多就没什么用了，反正也达不到并行效果，总有些进程会被挂起。但是也是有例外的，比如如果程序里有一步需要网络IO，这时候CPU不能真正并行也不是什么问题。
- 额外说一下线程吧，libFuzzer不管理线程，但你可以在自己的逻辑代码中自己开线程，自己记得关上就行。