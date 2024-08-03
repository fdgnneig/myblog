- 学习内容来自afl源码以及sakuraのAFL源码全注释
# afl-gcc.c
- 该程序用于代替gcc，从而编译待fuzz的程序源码，编译过程中会进行插桩，即在程序指定位置插特定汇编代码
## find_as  
- 该函数最终目的是找到afl实现的as程序（即汇编器）的路径，将该路径保存到as_path全局变量中，若未找到，则报异常，as_path后面会被用于调用afl-as，从而完成对程序的插桩和汇编
## edit_params
- 该函数的参数为调用afl-gcc时main函数接收到的argv和argc，即调用afl-gcc时传入的参数
- 函数目的是根据argv和argc，完成cc_params[]的填充，即cc_params中的内容将用于触发指定程序执行，从而完成afl-gcc的编译功能
- 解析argv[0]，即传入的可执行文件名，若argv[0]为afl-clang++，则将cc_params[0]赋值为clang++，即将afl开头的编译器名转为实际的编译器名称，说明当执行afl-clang++时，实际完成编译任务的是clang++ （afl-gcc 与afl-g++ 与之类似，分别对应gcc与g++编译器）
- 从argv[1]开始往后遍历每个传入的参数，将对应参数写入cc_params[] 该过程中根据环境变量的不同，也会将不同的出参数设置到cc_params[] 中，***find_as函数中获得的as_path也会被设置到cc_params[] 中，说明此时使用的汇编器as是afl自己实现的，即afl-as，这也是插桩的关键*** ，cc_params[]最后一个元素设置为null，函数结束
## main
- 首先调用 find_as
- 其次调用 edit_params 
- 最后execvp(cc_params[0], (char **) cc_params); 即根据cc_params[]中的内容调用对应编译器(例如clang clang++ gcc g++)，完成编译过程
## 综上afl-gcc即一个wrapper，最终会使用clang clang++ gcc g++等编译器完成编译过程，但此过程中使用到的汇编器as是afl自己实现的，这也是插桩的关键，从而在该过程中实现插桩

# afl-as.c
- 该程序接受gcc传入的参数，从而完成实际的汇编，函数参数为gcc传入的argv和argc
## edit_params
- 核心功能是构造as_params[] ,将as文件的路径设置为as_params[0] ，再将argv和argc中其他参数赋值到as_params[]中
- 将其他参数设置到as_params[]中，其中的关键包括input_file变量和modified_file变量，两者均为文件路径，前者表示gcc传入的输入文件的路径，后者表示经as处理过后的输出文件的路径(即完成插桩后的文件路径)
- as_params[]最后一个元素设置为null
## add_instrumentation
- 关键的插桩函数，处理input_file文件，生成modified_file文件，将instrumentation插入所有适当的位置。注意此时input_file文件中的内容是汇编指令，说明在此之前源文件已经经历过编译
- 在while循环中，通过fgets从input_file中逐行读取内容保存到line中，再将line逐行保存到modified_file对应的文件中，该循环过程中判断每一个line中的内容，此时line为一条汇编指令，判断汇编指令前序特征，若该汇编指令为分支或函数，则会在modified_file中插入instrumentation trampoline，即完成插桩过程
- 关于instrumentation trampoline，后文叙述
## main
- 初始随机种子
- 根据环境变量设置对应flag变量
- edit_params(argc, argv) 根据argc, argv构造as_params[]
- add_instrumentation()，进行插桩
- fork出一个子进程，让子进程来执行execvp(as_params[0], (char **) as_params);从而触发系统的as程序完成汇编
  - 这其实是因为我们的execvp执行的时候，会用as_params[0]来完全替换掉当前进程空间中的程序，如果不通过子进程来执行实际的as，那么后续就无法在执行完实际的as之后，还能unlink掉modified_file
- waitpid(pid, &status, 0)等待子进程结束


# afl-fast-clang.c
- 因为AFL对于上述通过afl-gcc来插桩这种做法已经属于不建议，并提供了更好的工具afl-clang-fast，通过llvm pass来插桩。
- 该文件位于源码/llvm_mode目录下
- afl-clang-fast.c这个文件其实是clang的一层wrapper，和之前的afl-gcc一样，只是定义了一些宏，和传递了一些参数给真正的clang。
## find_obj
- 该函数与afl-gcc.c中的find_as函数类似，从环境变量或argv0中获取路径信息，并依据该路径确定afl-llvm-rt.o文件的路径信息，并将afl-llvm-rt.o文件路径保存到obj_path全局变量中
## edit_params
- 函数参数为argc和argv
- 与上述两文件中的edit_params函数一样，主要作用是根据用户传入的argc和argv赋值cc_params[] ，并根据cc_params[] 中的内容调用实际clang或clang++完成编译
- cc_params[0]被赋值为clang或clang++，说明实际编译工作由该两者完成
- 依次将输入的参数（argc和argv）设置到cc_params[]中，该过程中会根据环境变量的不同设置不同的值
- ***代码注释：有两种方法完成插桩，通过afl-llvm-pass.so来注入instrumentation，但是现在也支持llvm的trace-pc-guard模式实现插桩***
## main
- 寻找obj_path路径
- 编辑参数cc_params
- 替换进程空间，执行要调用的clang和为其传递参数
  - execvp(cc_params[0], (char**)cc_params);
  
# afl-llvm-pass.so.cc
- 此处需要补充llvm编译优化相关知识，此部分内容会在后期集中补充，故此时暂列相关资料
- cscd70
  - https://github.com/UofT-EcoSystem/CSCD70
- cscd70 sakura笔记
  - https://github.com/eternalsakura/sakura_llvm_opt
- LLVM开发者手册
  - https://blog.csdn.net/qq_23599965/article/details/88538590
## runOnModule
- 该函数是本文件中实现功能的关键函数
- 函数中遍历每个基本块，向其中插入实现了如下伪代码功能的instruction ir来进行插桩。
- cur_location = <COMPILE_TIME_RANDOM>; 
- shared_mem[cur_location ^ prev_location]++; 
- prev_location = cur_location >> 1;
- 函数具体实现需要学习llvm相关内容后再分析

# afl-llvm-rt.o.c
- AFL LLVM_Mode中存在着三个特殊的功能。这三个功能的源码位于afl-llvm-rt.o.c中。
## deferred instrumentation功能 延迟注入
- 该功能可以让被fuzz程序在执行到某处代码时暂停，之后通过执行fork函数生成子进程执行之后的代码，通过持续生成子进程，该功能可以让程序中指定一段代码被反复执行，从而对该段代码进行fuzz。该功能可以减少操作系统、链接与libc内部执行程序的成本，该机制也被称为fork-server
  - forkserver即被fuzzing的程序，afl会将被fuzzing的程序读取输入的时候将程序断下，之后通知被fuzzing的程序生成子进程并处理afl输入的数据，子进程将执行结果通过管道通知afl，通过以上的方法，被fuzzing的代码是被fork出来的，无需重新执行程序，故提升了fuzzing效率
- 为实现该功能，需要在选定位置上添加如下代码，并使用afl-clang-fast重新编译代码
```c
#ifdef __AFL_HAVE_MANUAL_CONTROL
  __AFL_INIT();
#endif
```
### __AFL_INIT()
- 该函数内部调用__afl_manual_init
### __afl_manual_init
```c
void __afl_manual_init(void) {
  static u8 init_done;

  if (!init_done) {

    __afl_map_shm();
    __afl_start_forkserver();
    init_done = 1;

  }

}
```
- __afl_map_shm()函数读取环境变量获取共享内存地址，并将其保存到__afl_area_ptr
-  __afl_start_forkserver();中实现forkserver机制
###  __afl_start_forkserver()
- 函数向状态管道写入4字节，提示aflfuzz 准备就绪
- 阻塞接收控制管道数据，当收到数据，则执行一次fuzz
- fork一个子进程执行fuzz
- 向状态管道写入子进程pid，等待子进程结束
- 子进程结束后，向状态管道写入4字节，通知afl 此次fuzz执行完毕

## persistent mode功能 持久模式
- 上面我们其实已经介绍过persistent mode的一些特点了，那就是它并不是通过fork出子进程去进行fuzz的，而是认为当前我们正在fuzz的API是无状态的，当API重置后，一个长期活跃的进程就可以被重复使用，这样可以消除重复执行fork函数以及OS相关所需要的开销。
所以的使用方法如下：
```bash
while (__AFL_LOOP(1000)) {
  /* Read input data. */
  /* Call library code to be fuzzed. */
  /* Reset state. */
}
/* Exit normally */
```
- 循环次数不能设置过大，因为较小的循环次数可以将内存泄漏和类似故障的影响降到最低。所以循环次数设置成1000是个不错的选择。

- 接下来我们来解读一下源码，首先介绍一个__attribute__ constructor，demo如下，代表被此修饰的函数将在main执行之前自动运行
```bash
__attribute__((constructor(1))) void before_main1(){
    printf("before_main1\n");
}
__attribute__((constructor(2))) void before_main2(){
    printf("before_main2\n");
}
__attribute__((destructor(1))) void after_main1(){
    printf("after_main1\n");
}
__attribute__((destructor(2))) void after_main2(){
    printf("after_main2\n");
}
int main(){
    printf("main\n");
}
...
...
before_main1
before_main2
main
after_main2
after_main1
```
### __attribute__((constructor(CONST_PRIO))) void __afl_auto_init(void) 函数
- 该函数获取环境变量PERSIST_ENV_VAR
- 根据环境变量DEFER_ENV_VAR的值决定是否直接返回
- 若未直接返回，则调用__afl_manual_init，该函数之前讨论过
### __afl_persistent_loop 函数
- 宏定义__AFL_LOOP内部调用__afl_persistent_loop函数
- 该函数逻辑较为简单是，以下引用sakura原文，该函数与其他机制联合起来理解
  - 如果是第一次执行loop
    - 如果is_persistent为1
      - 清空__afl_area_ptr，设置__afl_area_ptr[0]为1，__afl_prev_loc为0
    - 设置cycle_cnt的值为传入的max_cnt参数，然后直接返回1
  - 如果不是第一次执行loop
    - 如果cycle_cnt减一（代表需要执行的循环次数减一）后大于0
      - 发出信号SIGSTOP来让当前进程暂停
      - 设置__afl_area_ptr[0]为1，__afl_prev_loc为0，然后直接返回1
    - 如果cycle_cnt为0
      - 设置__afl_area_ptr指向一个无关数组__afl_area_initial。
### 联系起来看持久模式的使用
- sakura原文逻辑梳理得较为清晰，此处整体理解下逻辑
- persistent mode功能的实现建立在deferred instrumentation之上，与后者不同的是当 persistent mode中通过发送信号的方式让被fuzzing的进程多次执行，这就避免了使用fork生成多个子进程进行fuzz的资源开销，提高了fuzz效率
- 具体逻辑可以查看sakura原文


## trace-pc-guard mode功能
- sakura讲的较为清楚是，直接复制
- 该功能是实现对程序每个edge进行插桩，原本仅仅对程序每个基本块进行插桩
- 要使用这个功能，需要先通过AFL_TRACE_PC=1来定义DUSE_TRACE_PC宏，从而在执行afl-clang-fast的时候传入-fsanitize-coverage=trace-pc-guard参数，来开启这个功能，和之前我们的插桩不同，开启了这个功能之后，我们不再是仅仅只对每个基本块插桩，而是对每条edge都进行了插桩。
```bash
ifdef AFL_TRACE_PC
  CFLAGS    += -DUSE_TRACE_PC=1
endif
```
- __sanitizer_cov_trace_pc_guard这个函数将在每个edge调用，该函数利用函数参数guard指针所指向的uint32值来确定共享内存上所对应的地址。
- 每个edge上都有应该有其不同(但其实可能相同，原因下述)的guard值
```bash
void __sanitizer_cov_trace_pc_guard(uint32_t* guard) {
  __afl_area_ptr[*guard]++;
}
```
- 而这个guard指针的初始化在__sanitizer_cov_trace_pc_guard_init函数里，llvm会设置guard其首末分别为start和stop。
- 它会从第一个guard开始向后遍历，设置guard指向的值，这个值是通过R(MAP_SIZE)设置的，定义如下，所以如果我们的edge足够多，而MAP_SIZE不够大，就有可能重复，而这个加一是因为我们会把0当成一个特殊的值，其代表对这个edge不进行插桩。
这个init其实很有趣，我们可以打印输出一下stop-start的值，就代表了llvm发现的程序里总计的edge数。
```bash
#  define R(x) (random() % (x))
...
...
  *(start++) = R(MAP_SIZE - 1) + 1;

  while (start < stop) {

    if (R(100) < inst_ratio) *start = R(MAP_SIZE - 1) + 1;
    else *start = 0;

    start++;

  }
```



# afl-fuzz.c
## setup_signal_handlers函数
- 该函数中调用sigaction函数设置不同linux信号的处理函数
## check_asan_opts
- check asan选项
- 读取环境变量ASAN_OPTIONS和MSAN_OPTIONS，做一些检查
## fix_up_sync
- 如果通过-M或者-S指定了sync_id，则更新out_dir和sync_dir的值
  - 设置sync_dir的值为out_dir
  - 设置out_dir的值为out_dir/sync_id
## save_cmdline
- 拷贝当前的命令行参数
## fix_up_banner
- 修剪并且创建一个运行横幅
## check_if_tty
- 检查是否在tty终端上面运行。
  - 读取环境变量AFL_NO_UI的值，如果为真，则设置not_on_tty为1，并返回
  - ioctl(1, TIOCGWINSZ, &ws)通过ioctl来读取window size，如果报错为ENOTTY，则代表当前不在一个tty终端运行，设置not_on_tty
## get_core_count
- 计数logical CPU cores
# check_crash_handling
# check_cpu_governor
# setup_post
# setup_shm
- 创建共享内存，将共享内存指针保存到trace_bits，共享内存中保存fuzz过程中生成的程序覆盖率信息
- 初始化virgin_bits等数组
## init_count_class16
- 该函数用于为数组count_class_lookup16[]中填充内容，在计算是否发现新路径之前，该数组会被用于规整共享内存中的路径命中数。以下部分来自sakura原文
- 因为trace_bits是用一个字节来记录是否到达这个路径，和这个路径被命中了多少次的，而这个次数在0-255之间，但比如一个循环，它循环5次和循环6次可能是完全一样的效果，为了避免被当成不同的路径，或者说尽可能减少因为命中次数导致的区别。
- 在每次去计算是否发现了新路径之前，先把这个路径命中数进行规整，比如把命中5次和6次都统一认为是命中了8次，见下面这个。
```c
static const u8 count_class_lookup8[256] = {
        [0]           = 0,
        [1]           = 1,
        [2]           = 2,
        [3]           = 4,
        [4 ... 7]     = 8,
        [8 ... 15]    = 16,
        [16 ... 31]   = 32,
        [32 ... 127]  = 64,
        [128 ... 255] = 128
};
```
- 而为什么又需要用一个count_class_lookup16呢，是因为AFL在后面实际进行规整的时候，是一次读两个字节去处理的，为了提高效率，这只是出于效率的考量，实际效果还是上面这种效果。
## setup_dirs_fds
- 函数中创建输出文件夹及其子目录， 包括queue、crashes、hangs等目录
- 函数中还准备相关fd变量sdf
- sakura笔记逻辑详细，可供参考
## read_testcases
- 该函数读取输入文件夹中的文件，扫描输入文件夹中所有文件，判断文件是否存在，过滤特殊文件，检查文件大小，最后将文件路径作为参数调用add_to_queue，从而将文件加入到queue中
- sakura笔记逻辑详细，可供参考
## add_to_queue(u8 *fname, u32 len, u8 passed_det)
- 为链表结构申请堆空间，将文件名、文件大小等数据写入链表结构体中，最终形成的链表结构中记录了queue中待fuzz的文件的详细信息
## load_auto
- 循环遍历尝试打开指定文件（文件名：alloc_printf("%s/.state/auto_extras/auto_%06u", in_dir, i)，根据注释，该文件中的内容是自动生成的），尝试读取文件内容，以读取到的内容和长度调用maybe_add_auto函数
## ***maybe_add_auto(u8 *mem, u32 len)***
- 没有太能理解具体操作的意义，主要是不清楚函数中所操作数据的内涵
- 该函数两个参数分别为从指定文件中读取到的数据及其长度，函数中将读取到的数据进行判断比较，同时会对其进行赋值
- Sakura有详细记录，弄懂具体数据含义后需要再看
- 根据注释，猜测该函数中处理的相关数据与fuzz预料相关
## pivot_inputs
- 逻辑上说这个函数就是为inputdir里的testcase，在output dir里创建hard link
- 该过程中会根据全局变量的值进行相应设置
## load_extras
- 如果定义了extras_dir，则从extras_dir读取extras到extras数组里，并按size排序。
## find_timeout
- 从指定文件中读取timeout相关数据，当该数据满足一定要求时，则将其设置到全局变量exec_tmout中
## detect_file_args
- 这个函数其实就是识别参数里面有没有@@，如果有就替换为out_dir/.cur_input，如果没有就返回
# setup_stdio_file
- 该函数用于在特殊情况下将输出文件夹的fd句柄保存到变量out_fd中，
- 如果out_file为NULL，如果没有使用-f，就删除原本的out_dir/.cur_input，创建一个新的out_dir/.cur_input，保存其文件描述符在out_fd中
# check_binary
check指定路径处要执行的程序是否存在，且它不能是一个shell script
# perform_dry_run
- 该函数会遍历queue中所有测试用例，调用函数 calibrate_case 让被fuzz程序将所有测试用例执行一遍，根据执行结果返回相应信息，从而评估测试用例的有效性
# u8 calibrate_case(char **argv, struct queue_entry *q, u8 *use_mem, u32 handicap, u8 from_queue)
- 这个函数评估input文件夹下的case，来发现这些testcase的行为是否异常；以及在发现新的路径时，用以评估这个新发现的testcase的行为是否是可变（这里的可变是指多次执行这个case，发现的路径不同）等等