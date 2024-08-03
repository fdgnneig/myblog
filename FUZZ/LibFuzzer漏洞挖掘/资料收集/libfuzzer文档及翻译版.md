- libfuzzer文档
https://llvm.org/docs/LibFuzzer.html
- libfuzzer文档翻译
https://blog.csdn.net/zhang14916/article/details/89208540
- libfuzzer实例
http://tutorial.libfuzzer.info/
# libfuzzer中LLVM的SanitizerCoverage提供代码覆盖率统计功能
https://clang.llvm.org/docs/SanitizerCoverage.html
编译过程中编译选项-fsanitize-coverage=可选参数可以在该文档中查到
# fuzzer的入口函数
```c++
// fuzz_target.cc
extern "C" int LLVMFuzzerTestOneInput(const uint8_t *Data, size_t Size) {
  DoSomethingInterestingWithMyAPI(Data, Size);
  return 0;  // Non-zero return values are reserved for future use.
}   
```
# 编译fuzzer二进制程序
- 需要在编译和链接阶段使用 -fsanitize=fuzzer，该flag会将libfuzzer的mian函数作为程序入口函数，从而执行fuzzer，还可以结合以下sanitizer使用
```bash
clang -g -O1 -fsanitize=fuzzer                         mytarget.c # Builds the fuzz target w/o sanitizers
clang -g -O1 -fsanitize=fuzzer,address                 mytarget.c # Builds the fuzz target with ASAN
clang -g -O1 -fsanitize=fuzzer,signed-integer-overflow mytarget.c # Builds the fuzz target with a part of UBSAN
clang -g -O1 -fsanitize=fuzzer,memory                  mytarget.c # Builds the fuzz target with MSAN
```
- 如果修改大型项目的CFLAGS（它在编译时需要使用自己的main函数），则可能希望只请求检测而不链接：
```bash
clang -fsanitize=fuzzer-no-link mytarget.c
```
然后可以通过在链接阶段传递-fsanitize = fuzzer将libFuzzer链接到所需的程序。
# 语料库蒸馏
- 如果您拥有庞大的语料库（无论是通过模糊处理生成还是通过其他方式获取），您都可能希望将其最小化，同时又要保留完整的覆盖范围。 一种方法是使用-merge = 1标志：
```bash
mkdir NEW_CORPUS_DIR  # Store minimized corpus here.
	./my_fuzzer -merge=1 NEW_CORPUS_DIR FULL_CORPUS_DIR
```
- 您可以使用相同的标志将更多有趣的项目添加到现有语料库中。仅触发新覆盖率的输入将被添加到第一个语料库。
```bash
./my_fuzzer -merge=1 CURRENT_CORPUS_DIR NEW_POTENTIALLY_INTERESTING_INPUTS_DIR
```
# fuzzer停止时会将触发错误的输入写入磁盘，根据不同类型得错误，对文件进行不同命名
```bash
crash-<sha1> leakle-<sha1> timeout-<sha1> 
```
# 并行fuzzer
- 每个libFuzzer进程都是单线程的，除非要测试的库启动其自己的线程。 但是，也可以并行运行多个libFuzzer进程同时共享语料库目录。 这样做的好处是，一个模糊器进程发现的任何新输入都可用于其他模糊器进程（除非您使用-reload = 0选项将其禁用）。
- 这主要由-jobs = N选项控制，该选项指示应将N个模糊测试作业运行完成（即直到找到错误或达到时间/迭代限制）。这些作业将在一组工作进程中运行，默认情况下使用一半的可用CPU内核来运行工作进程
- -workers = N选项可以修改工作进程的数量。 例如，在12核计算机上以-jobs = 30运行时，默认情况下将运行6个工作程序，每个工作程序在完成整个过程后平均会出现5个错误，即一个工作进程中包含了5个libfuzzer得fuzzing线程
# 附加模式
- 实验模式-fork = N（其中N是并行作业的数量）通过单独的进程（使用fork-exec，而不仅仅是fork）启用oom-，timeout-和crash-resistant的模糊测试。
- 顶部的libFuzzer进程本身不会进行任何模糊处理，但是会生成N个并发子进程，从而为它们提供语料库的随机子集。 子进程退出后，顶层进程将由子进程生成的语料库合并回主语料库。
- 相关标志：
  - -ignore_ooms：默认情况下为True。 如果在子进程之一的模糊处理过程中发生OOM，则将复制器保存在磁盘上，并且模糊处理将继续。
  - -ignore_timeouts：默认情况下为True，与-ignore_ooms相同，但用于处理超时情况
  - -ignore_crashes：默认情况下为False，与-ignore_ooms相同，但对于所有其他崩溃。
- 计划最终将-jobs = N和-workers = N替换为-fork = N。
- 即使用-fork = N可以作为另一种并发fuzzing的方案
# 恢复合并（主要用于解决语料库蒸馏过程中报错导致中断的问题）
- 合并大型语料库可能会很耗时，并且通常需要在可抢占的VM上进行，这可能会随时终止进程。为了无缝地恢复合并，请使用-merge_control_file标志并使用killall -SIGUSR1 /path/to/fuzzer/binary优雅地停止合并。 例子：
```bash
% rm -f SomeLocalPath
% ./my_fuzzer CORPUS1 CORPUS2 -merge=1 -merge_control_file=SomeLocalPath
...
MERGE-INNER: using the control file 'SomeLocalPath'
...
# While this is running, do `killall -SIGUSR1 my_fuzzer` in another console
==9015== INFO: libFuzzer: exiting as requested

# This will leave the file SomeLocalPath with the partial state of the merge.
# Now, you can continue the merge by executing the same command. The merge
# will continue from where it has been interrupted.
% ./my_fuzzer CORPUS1 CORPUS2 -merge=1 -merge_control_file=SomeLocalPath
...
MERGE-OUTER: non-empty control file provided: 'SomeLocalPath'
MERGE-OUTER: control file ok, 32 files total, first not processed file 20
...
```
# fuzzer运行选项
## 语料库选项
- 要运行模糊器，请传递零个或多个语料库目录作为命令行参数。 模糊器将从这些语料库目录中的每一个中读取测试输入，并且所生成的任何新测试输入将被写回到第一个语料库目录中：
```bash
./fuzzer [-flag1=val1 [-flag2=val2 ...] ] [dir1 [dir2 ...] ]
```
- 如果将文件列表（而不是目录）传递到模糊器程序，则它将重新运行这些文件作为测试输入，但不会执行任何模糊测试。 在这种模式下，模糊器可以用作回归测试（例如在连续集成系统上），以检查目标函数并且保存的输入仍然有效。
## 其他重要运行选项
- -help
  - 打印帮助消息（-help = 1）。
- -seed
  - 随机种子。 如果为0（默认值），则生成种子。
- -runs
  - 单个测试用例运行的次数，-1（默认）为无限期运行。
- -max_len
  - 测试输入的最大长度。 如果为0（默认值），则libFuzzer会尝试根据语料库猜测一个好的值（并报告该值）。
- -len_control
  - 首先尝试生成小的输入，然后随着时间的推移尝试更大的输入。 指定增加长度限制的速率（更小==更快）。 默认值为100。如果为0，fuzzer会立即尝试输入大小最大为max_len的输入。
- -timeout
  - 超时（以秒为单位），默认为1200。如果输入的时间长于此超时时间，则该过程将视为失败情况。
- -rss_limit_mb
  - 内存使用限制，以Mb为单位，默认为2048。使用0禁用限制。 如果输入需要执行的RSS存储器数量超过此数量，则该过程将视为失败情况。 每秒在一个单独的线程中检查该限制。 如果不使用ASAN / MSAN，则可以使用“ ulimit -v”代替。
- -malloc_limit_mb
  - 如果非零，则当目标尝试通过一个malloc调用分配此Mb数量时，模糊器将退出。 如果为零（默认），则应用与rss_limit_mb相同的限制。
- -timeout_exitcode
  - 如果libFuzzer报告超时，则使用退出代码（默认为77）。
- -error_exitcode
  - 如果libFuzzer本身（不是消毒剂）报告错误（泄漏，OOM等），则使用退出代码（默认值为77）
- -max_total_time
  - 如果为正，则表示运行模糊器的最大总时间（以秒为单位）。 如果为0（默认值），则无限期运行。
- -merge
  - 如果设置为1，则来自第二，第三等语料库目录的任何语料库输入会触发新的代码覆盖，这些语料库输入将合并到第一个语料库目录中。 默认值为0。此标志可用于最小化语料库。
- -merge_control_file
  - 指定用于合并过程的控制文件。 如果合并进程被杀死，fuzzer将尝试在该文件中保存合并状态，并以此恢复合并过程，默认情况下，将使用一个临时文件。
- -minimize_crash
  - 如果为1，则最小化攻击者提供的崩溃输入。 与-runs = N或-max_total_time = N一起使用以限制尝试次数。
- -reload
  - 如果设置为1（默认值），则将定期重新读取语料库目录以检查是否有新输入；否则，请重新输入。 这允许检测其他模糊处理过程发现的新输入。
- -jobs
  - 要完成的模糊测试作业数。 默认值为0，它将运行一个模糊测试过程，直到完成为止。如果值> = 1，则执行对应数量个模糊测试作业，且此类模糊测试作业运行在并行且相互分离work进程集合中，每个这样的work进程都将其stdout/stderr重定向到fuzz-<JOB>.log中
- -workers
  - 运行模糊测试作业以完成操作的并发work进程数。如果为0（默认值），则使用后两者的最小值（jobs，NumberOfCpuCores（）/2）。
- -dict
  - 提供输入关键字的字典； 请参阅字典一节。
- -use_counters
  - 使用覆盖率计数器来生成命中代码块的频率的近似计数； 默认为1。覆盖率计数器：https://clang.llvm.org/docs/SanitizerCoverage.html#coverage-counters
- -模糊器发现了一个更好的（较小的）输入，可以触发先前发现的功能（将-reduce_inputs = 0设置为禁用）。_inputs
  - 尝试减小输入的大小，同时保留其全部功能集； 默认为1。
- -use_value_profile
  - 使用value profile来指导语料库的扩展； 默认为0。
- -only_ascii
  - 如果为1，则仅生成ASCII（isprint`` +''isspace）输入。预设为0。
- -artifact_prefix
  - 提供将模糊工件（崩溃，超时或缓慢的输入）另存为$（artifact_prefix）文件时要使用的前缀。 默认为空。
- -exact_artifact_path
  - 如果为空则忽略（默认）。如果为非空，则将失败（崩溃，超时）时的单个工件写为$（exact_artifact_path）。 这将覆盖-artifact_prefix，并且不会在文件名中使用校验和。请勿将相同的路径用于多个并行进程。
- -print_pcs
  - 如果为1，则打印出新覆盖的PC。 预设为0。
- -print_final_stats
  - 如果为1，则在fuzzing结束时打印统计信息。预设为0。
- -detect_leaks
  - 如果为1（默认值）并且启用了LeakSanitizer，请尝试在模糊测试期间检测内存泄漏（即，不仅在关机时）。
- -close_fd_mask
  - 指示输出流在fuzzer启动时关闭。请注意，这将从目标代码中删除诊断输出（例如，断言失败时的消息）。
  - 0（默认）：不关闭stdout或stderr 
  - 1：关闭stdout 
  - 2：关闭stderr 
  - 3：关闭stdout和stderr。
- 有关标志的完整列表，请使用-help = 1运行fuzzer二进制文件。
# 输出
- fuzzer运行过程中的输出信息
```bash
INFO: Seed: 1523017872
INFO: Loaded 1 modules (16 guards): [0x744e60, 0x744ea0),
INFO: -max_len is not provided, using 64
INFO: A corpus is not provided, starting from an empty corpus
#0    READ units: 1
#1    INITED cov: 3 ft: 2 corp: 1/1b exec/s: 0 rss: 24Mb
#3811 NEW    cov: 4 ft: 3 corp: 2/2b exec/s: 0 rss: 25Mb L: 1 MS: 5 ChangeBit-ChangeByte-ChangeBit-ShuffleBytes-ChangeByte-
#3827 NEW    cov: 5 ft: 4 corp: 3/4b exec/s: 0 rss: 25Mb L: 2 MS: 1 CopyPart-
#3963 NEW    cov: 6 ft: 5 corp: 4/6b exec/s: 0 rss: 25Mb L: 2 MS: 2 ShuffleBytes-ChangeBit-
#4167 NEW    cov: 7 ft: 6 corp: 5/9b exec/s: 0 rss: 25Mb L: 3 MS: 1 InsertByte-
```
- 输出的早期部分包括有关模糊器选项和配置的信息，包括当前的随机种子（在Seed：行中；可以用-seed = N标志覆盖）。
- 其他输出行具有事件代码和统计信息的形式。 可能的事件代码是：
  - READ
    - 模糊器已从语料库目录中读取了所有提供的输入样本。
  - INITED
    - 模糊器已完成初始化，包括通过被测代码运行每个初始输入样本.
  - NEW
    - 模糊测试器创建了一个测试输入，该输入涵盖了被测代码的新区域。 此输入将保存到主要语料库目录。
  - REDUCE
    - 模糊器发现了一个更好的（较小的）输入，可以触发先前发现的功能（将-reduce_inputs = 0设置为禁用）。
  - pulse
    - 模糊器已生成2n个输入（定期生成以向用户保证模糊器仍在工作）。
  - DONE
    - 模糊器已完成操作，因为它已达到指定的迭代限制（-运行）或时间限制（-max_total_time）
  - RELOAD
    - 模糊器正在定期从语料库目录中重新加载输入； 这样，它就可以发现其他模糊器进程发现的任何输入（请参阅并行模糊处理）。
- 每个输出行还报告以下统计信息（非零时）：
  - cov
    - 执行当前语料库所覆盖的代码块或边的总数。
  - ft
    - libFuzzer使用不同的信号来评估代码覆盖率：边缘覆盖率，边缘计数器，值配置文件，间接调用方/被调用方对等。这些组合在一起的信号称为功能（ft :)。
  - corp
    - 当前内存中测试语料库中的条目数及其大小（以字节为单位）。
  - lim
    - 语料库中新条目长度的当前限制。 随着时间增加，直到达到最大长度（-max_len）。
  - exec/s
    - 每秒模糊器迭代执行的次数。
  - rss
    - 当前的内存消耗。 
- 对于NEW和REDUCE事件，输出行还包含有关产生新输入的变异操作的信息：
  - L:
    - 新输入的大小（以字节为单位）
  - MS: <n> <operations>
    - 用于生成输入的变异操作的计数和列表。
# 高级特性
## 字典
LibFuzzer支持用户提供的带有输入语言关键字或其他有趣字节序列（例如多字节魔术值）的词典。 使用-dict = DICTIONARY_FILE。 对于某些输入语言，使用字典可能会大大提高搜索速度。 字典语法类似于AFL的-x选项所使用的语法：
```bash
# Lines starting with '#' and empty lines are ignored.

# Adds "blah" (w/o quotes) to the dictionary.
kw1="blah"
# Use \\ for backslash and \" for quotes.
kw2="\"ac\\dc\""
# Use \xAB for hex values
kw3="\xF7\xF8"
# the name of the keyword followed by '=' may be omitted:
"foo\x0Abar"
```
注意字典文件中关键字必须单独占一行，后面不能跟注释，注释必须另起一行，例如如下字典文件解析时会报错
```bash
"\xff\x00" # Uses: 84
"\x00\x00" # Uses: 77
"\x16\x00" # Uses: 103
"\x0f\x00" # Uses: 81
"\x12\x00" # Uses: 75
"\x05\x00" # Uses: 66
```
## 跟踪CMP指令
带有附加的编译器标志-fsanitize-coverage = trace-cmp（默认情况下是-fsanitize = fuzzer的一部分，请参见https://clang.llvm.org/docs/SanitizerCoverage.html#tracing-data-flow，它处于打开状态）libFuzzer将拦截CMP指令并根据截获的CMP指令的参数引导突变。 这可能会减缓模糊的速度，但很有可能会改善结果。
## Value Profile
- 使用-fsanitize-coverage = trace-cmp（默认值为-fsanitize = fuzzer）和额外的运行时标志-use_value_profile = 1时，模糊器将收集比较指令参数的value profiles并将某些新值视为新覆盖范围。
- 当前的实现大致执行以下操作：
  - 编译器通过接收两个CMP参数的回调对所有CMP指令进行检测。
  - 回调计算（caller_pc＆4095）|  （popcnt（Arg1 ^ Arg2）<< 12），并使用此值在位集中设置一位。
  - 比特集合中的每个新观察到的比特都被视为新的覆盖范围。
- 此功能可能会发现许多有趣的输入，但是有两个缺点。 首先，额外的工具可能会导致额外的速度降低2倍。 其次，语料库可能增长数倍。
## Fuzzer友好的构建模式
- 有时所测试的代码不是模糊测试友好的。 示例：
  - 目标代码使用PRNG种子，例如 因此，即使最终结果相同，两次后续调用也可能会执行不同的代码路径。这将使模糊器将两个相似的输入视为显着不同，并且会炸毁测试语料库。 例如。  libxml在其哈希表中使用rand（）。
  - 目标代码使用校验和来防止输入无效。 例如。  png检查每个块的CRC。
- 在许多情况下，构建禁用某些模糊不友好功能的特殊模糊友好构建是有意义的。 我们建议针对所有此类情况使用通用的构建宏，以实现一致性：FUZZING_BUILD_MODE_UNSAFE_FOR_PRODUCTION。
```c++
void MyInitPRNG() {
#ifdef FUZZING_BUILD_MODE_UNSAFE_FOR_PRODUCTION
  // In fuzzing mode the behavior of the code should be deterministic.
  srand(0);
#else
  srand(time(0));
#endif
}
```
## AFL相容性
- LibFuzzer可以与AFL在同一个测试语料库上一起使用。 两个模糊测试者都希望测试语料库位于目录中，每个输入一个文件。 您可以在同一个语料库上同时运行两个模糊器：
```bash
./afl-fuzz -i testcase_dir -o findings_dir /path/to/program @@
./llvm-fuzz testcase_dir findings_dir  # Will write new tests to testcase_dir
```
- 定期重新启动两个模糊器，以便它们可以使用彼此的发现。当前，没有一种简单的方法可以在共享同一个语料库目录的同时并行运行两个模糊引擎
- 您还可以在目标函数LLVMFuzzerTestOneInput上使用AFL：在此处以下示例
- https://github.com/llvm/llvm-project/tree/main/compiler-rt/lib/fuzzer/afl
## fuzzing效果提升
- 旦实现了目标函数LLVMFuzzerTestOneInput并将其模糊化，您将想知道该函数或语料是否可以进一步改进。 当然，一种易于使用的指标是代码覆盖率。
- 我们建议使用Clang Coverage来可视化和研究您的代码覆盖率（示例）。
- https://clang.llvm.org/docs/SourceBasedCodeCoverage.html
- https://github.com/google/fuzzer-test-suite/blob/master/tutorial/libFuzzerTutorial.md#visualizing-coverage
## 用户自定义变异器
- LibFuzzer允许使用自定义（用户提供）的变异器，有关更多详细信息，请参见结构感知的模糊化。
- https://github.com/google/fuzzing/blob/master/docs/structure-aware-fuzzing.md
## 启动初始化
- 如果需要初始化要测试的库，则有几个选项。
- 最简单的方法是在LLVMFuzzerTestOneInput（或对您有用的全局范围内）中具有一个静态初始化的全局对象：
```c++
extern "C" int LLVMFuzzerTestOneInput(const uint8_t *Data, size_t Size) {
  static bool Initialized = DoInitialization();
  ...
```
另外，您可以定义一个可选的init函数，它将接收您可以读取和修改的程序参数。 仅在确实需要访问argv / argc时才执行此操作。
```c++
extern "C" int LLVMFuzzerInitialize(int *argc, char ***argv) {
 ReadAndMaybeModify(argc, argv);
 return 0;
}
```
## 将libFuzzer用作库
- 如果被模糊处理的代码必须使用自己的mian函数，则可以将libFuzzer作为库来调用。 确保在编译过程中传递-fsanitize = fuzzer-no-link，并将您的二进制文件链接到libFuzzer的无main函数版本。 在Linux安装中，该位置通常位于如下位置：
- /usr/lib/llvm-version/lib/clang/clang-version/lib/linux/libclang_rt.fuzzer_no_main-architecture.a
- 如果从源代码构建libFuzzer，则该文件位于build输出目录中的以下路径中：
- lib/linux/libclang_rt.fuzzer_no_main-architecture.a
- 从这里开始，代码可以执行所需的任何设置，并且在准备开始进行模糊测试时，可以调用LLVMFuzzerRunDriver，并传入程序参数和回调。 就像LLVMFuzzerTestOneInput一样调用此回调，并且具有相同的签名。
```c++
extern "C" int LLVMFuzzerRunDriver(int *argc, char ***argv,
                  int (*UserCb)(const uint8_t *Data, size_t Size));
```
## 内存泄露
- 使用AddressSanitizer或LeakSanitizer构建的二进制文件将尝试在进程关闭时检测内存泄漏。 对于过程中的模糊处理，这很不方便，因为一旦发现泄漏突变，模糊器就需要向复制者报告泄漏。 但是，在每个突变之后进行完整的泄漏检测非常昂贵。
- 默认情况下（-detect_leaks = 1），libFuzzer将在执行每个变异时计算malloc和free调用的数量。 如果数字不匹配（本身并不意味着有泄漏），libFuzzer将调用价格更高的LeakSanitizer传递，并且如果发现实际泄漏，则将向复制者报告，并且过程将退出。
- 如果目标存在大量泄漏，并且禁用了泄漏检测，则最终将耗尽RAM（请参见-rss_limit_mb标志）。
## 不同平台编译libfuzzer
- 默认情况下，在MacOS和Linux上，LibFuzzer作为LLVM项目的一部分而构建。其他操作系统的用户可以使用-DLIBFUZZER_ENABLE = YES标志明确请求编译。 可以使用check-fuzzer对被fuzz的目标进行测试，check-fuzzer位于build目录中，该目录通过使用-DLIBFUZZER_ENABLE_TESTS = ON标志进行配置。
- ninja check-fuzzer
# windows下使用libfuzzer
- 是的，libFuzzer现在支持Windows。r341082中添加了初始支持。任何Clang 9版本都支持它。您可以从LLVM Snapshot Builds下载具有libFuzzer的Windows版Clang。
- https://llvm.org/builds/
- 不支持在没有ASAN的Windows上使用libFuzzer。 不支持使用/ MD（动态运行时库）编译选项构建模糊器。 将来可能会增加对这些功能的支持。 也不支持使用/ INCREMENTAL链接选项（或暗示它的/ DEBUG选项）链接模糊器。
## libfuzzer能够具有较好效果的目标程序
正则表达式匹配器，文本或二进制格式的解析器，压缩，网络，加密。
