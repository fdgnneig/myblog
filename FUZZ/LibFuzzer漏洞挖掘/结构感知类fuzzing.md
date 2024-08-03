# libfuzzer用户自定义变异器
```bash
  if (EF->LLVMFuzzerCustomMutator)
    Mutators.push_back({&MutationDispatcher::Mutate_Custom, "Custom"});
  else
    Mutators = DefaultMutators;
```
源码中若LLVMFuzzerCustomMutator函数存在，则会将用户自定义的突变函数加入到Mutators对象中(MutationDispatcher::Mutate_Custom函数是对LLVMFuzzerCustomMutator函数的封装)，否则将默认突变函数赋值给Mutators对象
```bash
LLVMFuzzerCustomMutator(Data, Size, MaxSize,Rand.Rand<unsigned int>());
```
LLVMFuzzerCustomMutator函数有四个参数，前三个参数与默认突变函数相同，最后一个函数为一个随机种子
# fuzzing状态相关的程序
- 部分程序中，程序运行状态的不同可能导致不同的bug，而为了挖掘更多的漏洞，需要尽可能在不同状态下针对程序进行fuzzing，一般来说程序不同的状态来源于两个方面，程序中关键数据发生变化（例如hash表处理程序中，hash表数据由于数据操作而导致不同），程序中关键代码发生变化（例如部分程序运行时通过热补丁技术修改自身代码）
- 对于第一种情况，通过hash表程序举例，该程序提供了用于操作hash表数据的api接口，通过调用该接口，可以修改hash表数据，从而改变该程序运行状态，所以以随机数据作为参数，按照随机顺序调用能够操作hash表数据的api接口，即可实现fuzzing不同状态下的hash表处理程序
- 注意采用该方法实现fuzzing时，需要在每次fuzzing迭代执行前进行程序状态初始化，以防止上一次fuzzing迭代导致的程序状态变化影响本次fuzzing结构，基于同样的原因，需要在每次fuzzing迭代执行后清理环境。
- 当通过fuzzing发现了基于状态产生的crash时，要复现该crash相对困难，因为该crash的产生不仅取决于输入的数据，该取决于当时程序的状态，在hash表程序的示例中，程序状态即不同hash表操作API的调用顺序，若想复现该crash，需要同时复现程序随机输入以及API调用顺序，不过在本例中程序api调用顺序是由程序随机输入决定的，只要给出特定输入，程序即可以特定顺序调用api
- 由于程序状态相关的crash的复现较为困难，所以一般有两种分析思路
  1. 更多的手动调试
  2. 源码中插入更多log信息，从而在运行是输出参考信息
# libprotobuf-mutator
https://github.com/google/libprotobuf-mutator
# protobuf开源项目
https://github.com/protocolbuffers/protobuf
# protobuf官网
https://developers.google.com/protocol-buffers/
# protobuf官方教程
https://developers.google.com/protocol-buffers/docs/overview
https://developers.google.com/protocol-buffers/docs/tutorials
## proto2教程
https://developers.google.com/protocol-buffers/docs/proto
## proto3教程
https://developers.google.com/protocol-buffers/docs/proto3
# 中文protubf资料
## proto2版本
https://colobu.com/2015/01/07/Protobuf-language-guide/
https://www.cnblogs.com/langqi250/p/7283702.html
## proto3版本
https://blog.csdn.net/shensky711/article/details/69696392
https://zhuanlan.zhihu.com/p/348498896




