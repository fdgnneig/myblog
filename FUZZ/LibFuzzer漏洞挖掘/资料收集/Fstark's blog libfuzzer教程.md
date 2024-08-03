- 资料来源：https://myfzy.top/2020/06/03/libfuzzer/
# libfuzzer相对于afl的优势 
LibFuzzer在概念上与American Fuzzy Lop（AFL）类似，但它是在单个进程中执行了所有模糊测试。进程内的模糊测试可能更具针对性，由于没有进程反复启动的开销，因此与AFL相比可能更快。
# 编译fuzzer可执行文件的命令
```bash
clang++ -g -O1 -fsanitize=fuzzer,address -fsanitize-coverage=trace-pc-guard \
fuzz_target.cc ../../libFuzzer/Fuzzer/libFuzzer.a \
-o mytarget_fuzzer
```
- -g和-O1是gcc/clang的通用选项，前者保留调试信息，使错误消息更易于阅读；后者指定优化等级为1（保守地少量优化），但这两个选项不是必须的。
- -fsanitize-coverage=trace-pc-guard: 为 libfuzzer 提供代码覆盖率信息
- -fsanitize-coverage=trace-pc-guard选项在高版本的clang中已不再适用，代码的覆盖率情况默认自动开启。
- libFuzzer.a: 为libfuzzer项目中执行build.sh 编译好生成的 libFuzzer.a
# libfuzzer命令行格式
./your-fuzzer -flag1=val1 -flag2=val2 ... dir1 dir2 ...
# libfuzzer重运行模式
./your-fuzzer -flag1=val1 -flag2=val2 ... file1 file2 ...
- 与上面一样，但是选项后面接的是文件列表而非文件夹列表，这些输入样例将会重新读取并输入运行，不会产生新样例，在回归测试时十分有用。
# 常用运行选项
- 选项  默认   效用
- verbosity	1	运行时输出详细日志
- seed	0	随机种子。如果为0，则自动生成
- runs	-1	单个测试运行的次数（-1表示无限）
- max_len	0	测试输入的最大长度。若为0，libFuzzer会自行猜测
- minimize_crash	0	如果为1，则最小化提供的崩溃输入。与-runs = N- -max_total_time   = N一起使用以限制尝试次数。
- reduce_inputs	1	尝试减小输入的大小，同时保留其全部功能集
- fork	0	在子过程中发生fuzzing的实验模式
- ignore_timeouts	1	在fork模式下忽略超时
- ignore_crashes	0	在fork模式下忽略崩溃
- ignore_ooms	1	在fork模式下忽略OOM
- cross_over	1	交叉输入
- rss_limit_mb	2048	内存使用限制，以Mb为单位。使用0则禁用限制。
- mutate_depth	5	每个输入连续突变的数量
- shuffle	1	启动时打乱初始语料库
- prefer_small	1	打乱语料库时将小输入置于优先位置
- timeout	1200	若为正，表示单元运行最大秒数。超时会被提前中止
- error_exitcode	77	libFuzzer本身出错时的退出码
- timeout_exitcode	77	libFuzzer超时退出码
- max_total_time	0	若为正，表示整个模糊测试运行最大秒数
- dict	0	提供输入关键字的字典
- use_counters	1	使用覆盖率计数器生成命中代码块的频率的近似计数；默认为1。
- help	0	打印帮助并退出
- merge	0	不损失覆盖率前提下，将第2/3/4/…个语料库合并到第一个中去
- merge_control_file	0	指定合并进程的控制文件，用于恢复合并状态
- jobs	0	运行的作业数量。所有作业的输出会被重定向到fuzz-JOB.log。
- workers	0	运行作业的并发进程数。若为0，实验CPU核心数一半
- reload	1	每N秒载主语料库，以知悉其他进程发现的单元
- only_ascii	0	仅生成ASCII（isprint + isspace）输入
- artifact_prefix	0	提供将模糊处理工件（崩溃，超时或缓慢的输入）另存为$- （artifact_prefix）file时要使用的前缀。默认为空。
- exact_artifact_path	0	如果为空则忽略（默认）。如果为非空，则将失败（崩溃，- 时）时的单个工件写为$（exact_artifact_path）。这将覆盖 -artifact_prefix，并- 不会在文件名 中使用校验和。请勿将相同的路径用于多个并行进程。
- detect_leaks	1	如果为1（默认值）并且启用了LeakSanitizer，则尝试在模糊测- 期间  检测内存泄漏（即，不仅在关闭时）。
- print_final_stats	0	退出时打印统计信息
- print_corpus_stats	0	退出时打印语料库元素统计信息
- print_coverage	0	退出时打印覆盖率信息
- close_fd_mask	0	为1则在关闭stdout，为2则关闭stderr，为3则关闭二者
# 全部命令行选项
  - Flags:	value	strictly in form -flag=value
  - verbosity	1	Verbosity level.
  - seed	0	Random seed. If 0, seed is generated.
  - runs	-1	Number of individual test runs (-1 for infinite runs).
  - max_len	0	Maximum length of the test input. If 0, libFuzzer tries to- guess a  good value based on the corpus and reports it.
  - lea_control	100	Try generating small inputs first, then try larger inputs- over  time. Specifies the rate at which the length limit is increased- (smaller ==  faster). If 0, immediately try inputs with size up to- max_len. Default value     is 0, if LLVMFuzzerCustomMutator is used.
  - seed_inputs	0	A comma-separated list of input files to use as an- additional   seed corpus. Alternatively, an “@” followed by the name of a- file containing  the comma-seperated list.
  - cross_over	1	If 1, cross over inputs.
  - mutate_depth	5	Apply this number of consecutive mutations to each- input.
  - reduce_depth	0	Experimental/internal. Reduce depth if mutations- lose   unique features
  - shuffle	1	Shuffle inputs at startup
  - prefer_small	1	If 1, always prefer smaller inputs during the- corpus    shuffle.
  - timeout	1200	Timeout in seconds (if positive). If one unit runs more- than    this number of seconds the process will abort.
  - error_exitcode	77	When libFuzzer itself reports a bug this exit code- will be  used.
  - timeout_exitcode	70	When libFuzzer reports a timeout this exit code- will    be used.
  - max_total_time	0	If positive, indicates the maximal total time in- seconds    to run the fuzzer.
  - help	0	Print help.
  - fork	0	Experimental mode where fuzzing happens in a subprocess
  - ignore_timeouts	1	Ignore timeouts in fork mode
  - ignore_ooms	1	Ignore OOMs in fork mode
  - ignore_crashes	0	Ignore crashes in fork mode
  - merge	0	If 1, the 2-nd, 3-rd, etc corpora will be merged into the- 1-st  corpus. Only interesting units will be taken. This flag can be used- to   minimize a corpus.
  - stop_file	0	Stop fuzzing ASAP if this file exists
  - merge_control_file	0	Specify a control file used for the merge process.- If   a merge process gets killed it tries to leave this file in a state- suitable   for resuming the merge. By default a temporary file will be- used.
  - minimize_crash	0	If 1, minimizes the provided crash input. Use with- -runs=N  or -max_total_time=N to limit the number attempts. Use with - -exact_artifact_path to specify the output. Combine with    - ASAN_OPTIONS=dedup_token_length=3 (or similar) to ensure that the- minimized     input triggers the same crash.
  - cleanse_crash	0	If 1, tries to cleanse the provided crash input to- make it  contain fewer original bytes. Use with -exact_artifact_path to- specify the   output.
  - use_counters	1	Use coverage counters
  - use_memmem	1	Use hints from intercepting memmem, strstr, etc
  - use_value_profile	0	Experimental. Use value profile to guide fuzzing.
  - use_cmp	1	Use CMP traces to guide mutations
  - shrink	0	Experimental. Try to shrink corpus inputs.
  - reduce_inputs	1	Try to reduce the size of inputs while preserving- their     full feature sets
  - jobs	0	Number of jobs to run. If jobs >= 1 we spawn this number of- jobs    in separate worker processes with stdout/stderr redirected to- fuzz-JOB.log.
  - workers	0	Number of simultaneous worker processes to run the jobs. If- zero,   “min(jobs,NumberOfCpuCores()/2)” is used.
  - reload	1	Reload the main corpus every seconds to get new units- discovered    by other processes. If 0, disabled
  - report_slow_units	10	Report slowest units if they run for more than- this     number of seconds.
  - only_ascii	0	If 1, generate only ASCII (isprint+isspace) inputs.
  - dict	0	Experimental. Use the dictionary file.
  - artifact_prefix	0	Write fuzzing artifacts (crash, timeout, or slow- inputs)    as $(artifact_prefix)file
  - exact_artifact_path	0	Write the single artifact on failure (crash,- timeout)   as $(exact_artifact_path). This overrides -artifact_prefix and- will not use   checksum in the file name. Do not use the same path for- several parallel  processes.
  - print_pcs	0	If 1, print out newly covered PCs.
  - print_funcs	2	If >=1, print out at most this number of newly covered - functions.
  - print_final_stats	0	If 1, print statistics at exit.
  - print_corpus_stats	0	If 1, print statistics on corpus elements at exit.
  - print_coverage	0	If 1, print coverage information as text at exit.
  - dump_coverage	0	Deprecated.
  - handle_segv	1	If 1, try to intercept SIGSEGV.
  - handle_bus	1	If 1, try to intercept SIGBUS.
  - handle_abrt	1	If 1, try to intercept SIGABRT.
  - handle_ill	1	If 1, try to intercept SIGILL.
  - handle_fpe	1	If 1, try to intercept SIGFPE.
  - handle_int	1	If 1, try to intercept SIGINT.
  - handle_term	1	If 1, try to intercept SIGTERM.
  - handle_xfsz	1	If 1, try to intercept SIGXFSZ.
  - handle_usr1	1	If 1, try to intercept SIGUSR1.
  - handle_usr2	1	If 1, try to intercept SIGUSR2.
  - lazy_counters	0	If 1, a performance optimization isenabled for the- 8bit     inline counters. Requires that libFuzzer successfully installs- its SEGV handler
  - close_fd_mask	0	If 1, close stdout at startup; if 2, close stderr; if- 3,    close both. Be careful, this will also close e.g. stderr of asan.
  - detect_leaks	1	If 1, and if LeakSanitizer is enabled try to detect- memory  leaks during fuzzing (i.e. not only at shut down).
  - purge_allocator_interval	1	Purge allocator caches and quarantines- every    seconds. When rss_limit_mb is specified (>0), purging starts when- RSS exceeds  50% of rss_limit_mb. Pass purge_allocator_interval=-1 to- disable this    functionality.
  - trace_malloc	0	If >= 1 will print all mallocs/frees. If >= 2 will- also     print stack traces.
  - rss_limit_mb	2048	If non-zero, the fuzzer will exit uponreaching- this     limit of RSS memory usage.
  - malloc_limit_mb	0	If non-zero, the fuzzer will exit if the target tries- to    allocate this number of Mb with one malloc call. If zero (default)- same limit  as rss_limit_mb is applied.
  - exit_on_src_pos	0	Exit if a newly found PC originates from the given- source   location. Example: -exit_on_src_pos=foo.cc:123. Used primarily- for testing    libFuzzer itself.
  - exit_on_item	0	Exit if an item with a given sha1 sum was added to- the  corpus. Used primarily for testing libFuzzer itself.
  - ignore_remaining_args	0	If 1, ignore all arguments passed after this- one.   Useful for fuzzers that need to do their own argument parsing.
  - focus_function	0	Experimental. Fuzzing will focus on inputs that- trigger     calls to this function. If -focus_function=auto and- -data_flow_trace is used,   libFuzzer will choose the focus functions- automatically.
  - analyze_dict	0	Experimental
  - use_clang_coverage	0	Deprecated; don’t use
  - data_flow_trace	0	Experimental: use the data flow trace
  - collect_data_flow	0	Experimental: collect the data flow trace
# 并行fuzzer
- 提高模糊测试效率的另一种方法是使用更多的CPU。如果您使用-jobs=N它运行模糊器，它将产生N个独立fuzzing的作业
- -workers=M设置并行的work进程的数量，work进程数量最多不超过CPU拥有的内核数的一半
- 当指定了多个任务时，程序启动时会先产生一个master进程，同时并发启动相应数量个worker进程，master会把jobs分配到workers上去执行，当某个任务结束后，相应进程终止，同时master会启动一个新的任务进程分配到对应的worker上，平均每个worker上会分配jobs/workers个任务。
- ./your-fuzzer MY_CORPUS/ seeds/ -jobs=8
- 在8核计算机上，这将产生4个work进程，每个work进程包含两个fuzzing作业。如果其中一个被退出，将自动创建另一个，最多8个。
```bash
fuzzer -jobs=8
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
```
# 字典
- libfuzzer 和 afl 使用的 dictionary 文件的语法是一样的， 所以可以直接拿 afl 里面的 dictionary 文件来给 libfuzzer 使用。
- 官方字典示例
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
- #开头的行 和 空行会被忽略
- kw1= 这些就类似于注释， 没有意义
- 真正有用的是由 " 包裹的字串，这些 字串 就会作为一个个的关键字， libfuzzer 会用它们进行组合来生成样本。
- libfuzzer 使用 -dict 指定 dict 文件，下面使用 xml.dict 为 dictionary 文件，进行 fuzz。
    - ./your-fuzzer -dict=DICTIONARY_FILE
# 重现crash时显示栈的符号信息
- ASAN_OPTIONS=symbolize=1 用于显示栈的符号信息
- ASAN_OPTIONS=symbolize=1 ./first_fuzzer ./crash-0eb8e4ed029b774d80f2b66408203801
# 使用-dict和-print_final_stats选项的结果
- 如果我们在fuzzer运行的选项里有使用字典 -dict和-print_final_stats执行完打印统计信息，最后的输出可能还会多出两块，形如下面
```bash
###### Recommended dictionary. ######
"X\x00\x00\x00\x00\x00\x00\x00" # Uses: 1228
"prin" # Uses: 1353
...........................
...........................
...........................
"U</UTrri\x09</UTD" # Uses: 61
###### End of recommended dictionary. ######
Done 1464491 runs in 301 second(s)
stat::number_of_executed_units: 1464491
stat::average_exec_per_sec:     4865
stat::new_units_added:          1407
stat::slowest_unit_time_sec:    0
stat::peak_rss_mb:              407
```
- 开始由 #### 夹着的是 libfuzzer 在 fuzz 过程中挑选出来的 dictionary， 同时还给出了使用的次数，这些 dictionary 可以在以后 fuzz 同类型程序时 节省 fuzz 的时间。
- 然后以 stat: 开头的是一些 fuzz 的统计信息， 主要看 stat::new_units_added 表示整个 fuzz 过程中触发了多少个代码单元。
- 可以看到直接 fuzz , 5分钟 触发了 1407 个代码单元