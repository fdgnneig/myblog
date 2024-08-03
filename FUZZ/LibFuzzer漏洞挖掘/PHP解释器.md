# php运行生命周期
https://blog.csdn.net/onlymayao/article/details/104867952
- cli模式下的生命周期
  - php_module_startup //模块初始化阶段
  - php_request_startup //请求初始化阶段
  - php_execute_script //脚本执行阶段（读取你的代码，进行解析）
  - php_request_shutdown //请求关闭阶段
  - php_module_shutdown //模块关闭阶段
# 项目中已有的fuzzer
- 项目中还提供了fuzzer种子文件生成程序，详细见下面链接
- https://github.com/php/php-src/blob/5b01c4863fe9e4bc2702b2bbf66d292d23001a18/sapi/fuzzer/README.md
- php-fuzz-parser: Fuzzing language parser and compiler
- php-fuzz-unserialize: Fuzzing unserialize() function
- php-fuzz-unserializehash: Fuzzing unserialize() for HashContext objects
- php-fuzz-json: Fuzzing JSON parser (requires --enable-json)
- php-fuzz-exif: Fuzzing exif_read_data() function (requires --enable-exif)
- php-fuzz-mbstring: Fuzzing mb_ereg[i]() (requires --enable-mbstring)
- php-fuzz-execute: Fuzzing the executor
# 项目中已有的fuzzer：fuzzer-mbstring.c，针对mb_ereg等正则表达式匹配函数进行fuzzing
## fuzzer_init_php
- 该函数调用sapi_startup和fuzzer_module.startup函数，
- sapi_startup定义在php-src/main/SAPI.c中，用于启动sapi相关功能
- fuzzer_module.startup后者中调用了php_module_startup，完成了php运行生命周期中的第一步
## onig_set_parse_depth_limit
- 该函数来自php项目中所引用的外部库Oniguruma，该库用于提供正则表达式功能，此时该函数用于设置正则表达式解析深度，从而允许堆栈溢出
## fuzzer_request_startup
其内部调用php_request_startup()函数，从而启动php引擎，将php请求初始化
- php_request_startup()函数执行过程
https://cloud.tencent.com/developer/article/1693227
## fuzzer_setup_dummy_frame
- 该函数用于设置虚拟堆栈框架，从而便于触发异常
## fuzzer_call_php_func
- 使用ZVAL_STRING将c字符串转为php字符串
- 调用fuzzer_call_php_func_zval
  - fuzzer_call_php_func_zval调用call_user_function用于执行对应的php函数，call_user_function是php提供的函数，该函数可以根据函数名调用其他函数
    - call_user_function调用_call_user_function_impl
      - _call_user_function_impl调用zend_call_function
## fuzzer_shutdown_php
- 其内部调用了php_module_shutdown和sapi_shutdown，用于结束php函数的调用
- sapi_shutdown定义在php-src/main/SAPI.c中，用于结束sapi相关功能
# 项目中已有的fuzzer-parser.c，针对.php文件的编译过程进行fuzzing
## fuzzer_do_request_from_buffer
- 将随机数据作一个php文件，供php进行解析
- fuzzer_do_request_from_buffer("fuzzer.php", (const char *) Data, Size, /* execute */ 0);
  - fuzzer_do_request_from_buffer内部调用zend_compile_file和zend_execute
    - fuzzer_do_request_from_buffer函数最后一个参数为0时，仅调用zend_compile_file编译php文件，输出opcode
    - fuzzer_do_request_from_buffer函数最后一个参数为1时，会调用zend_compile_file将php文件编译为opcode，之后zend_execute调用虚拟机功能，执行opcode
  - zend目录中保存Z-engine的实现，后者是php语言的核心执行引擎
# php脚本实际执行流程
1. 在php的cli命令行模式下，调用php可执行文件执行php脚本
   - /usr/local/bin/php -f test.php
2. /sapi/cli/php_cli.c文件中的main函数开始运行
   - 调用php_execute_script，该函数定义在{PHPSRC}/main/main.c
     - 调用zend_execute_scripts，该函数定义在{PHPSRC}/Zend/zend.c
       - 调用zend_compile_file，该函数声明在{PHPSRC}/Zend/zend_compile.c，该函数用于将php文件编译为opcode，返回opcode的数组，
         - 在引擎初始化时，zend_compile_file被赋值为compile_file（来自{PHPSRC}/Zend/zend_language_scanner.c），即实际调用的是compile_file函数
       - 调用zend_execute，其声明在{PHPSRC}/Zend/zend_execute.c，将zend_compile_file返回的opcode数组作为参数，逐条执行
         - 在引擎初始化时，zend_execute被赋值为execute函数（来自PHPSRC}/Zend/zend_vm_execute.h），即实际调用的是execute函数
# 项目中已有的fuzzer-execute.c，针对.php文件的编译、执行过程进行fuzzing
## fuzzer_do_request_from_buffer
- 核心函数仍然是## fuzzer_do_request_from_buffer，但该fuzzer中该函数第四个参数设置为1，从而允许zend_execute执行，从而将随机数据生成的作为.php文件编译运行
### 关于fuzzer中的zend_execute_ex
- 根据参考资料《PHP 内核分析：Zend 虚拟机（优质资料）》当php中自定义函数执行时，会生成并先后执行INIT_FCALL、DO_FCALL两条opcode指令
  - INIT_FCALL指令用于准备执行函数时所需要的上下文数据
  - DO_FCALL负责执行函数
    - DO_FCALL内部会调用zend_execute_ex函数
      - 默认情况下zend_execute_ex函数就是execute_ex（来自PHPSRC}/Zend/zend_vm_execute.h），即上文中zend_execute实际执行的函数，execute_ex中通过创建循环逐一执行opcode（通过调用opcode结构体中的handler成员）
      - php提供机制（类似于钩子机制）可以使用自定义函数指针覆盖zend_execute_ex，从而自定义opcode的解析过程，在本fuuzer中，自定义了fuzzer_execute_ex函数，将其赋值给zend_execute_ex，从而完成了自定义opcode处理过程
### 关于fuzzer中的zend_compile_string
- 根据资料《看我如何玩转PHP代码加密与解密》，php中存在eval函数，该函数将字符串作为参数，将参数字符串作为php指令进行执行（前提是字符串格式符合php代码要求），例如eval('echo 1;');将会输出1,eval函数常被运用于php代码加密，即将加密后的字符串解密后作为参数传递给eval函数，从而执行php代码
- eval在zend engine中会调用zend_compile_string函数，该函数与zend_compile_file类似，只不过后者是将文件内容编译为php的opcode，前者将字符串内容编译为opcode
- php中提供了hook机制，可以替换zend_compile_string函数，仅需以下两条指令，即可将zend_compile_string函数替换为自定义函数fuzzer_compile_string
  - orig_compile_string = zend_compile_string;//用于保存原始的complie_string函数
  - zend_compile_string = fuzzer_compile_string;//替换complie处理函数
- 本fuzzer中通过将zend_compile_string赋值为fuzzer_compile_string，而fuzzer_compile_string内部又调用了orig_compile_string函数，从而在原始的orig_compile_string函数基础增加了字符串长度判断的功能，从而避免输入数据过长，影响fuzzer效率（具体可见benfuzzer源码）
# 项目中已有的fuzzer-exif.c，针对图片格式解析进行fuzzing
- exif头：Exif是一种图像文件格式，它的数据存储与JPEG格式是完全相同的。实际上Exif格式就是在JPEG格式头部插入了数码照片的信息，从而称此类信息为exif头
- exif_read_data 用于从 JPEG 或 TIFF 文件中读取 EXIF 头信息
- 该fuzzer的核心操作是将libfuzzer提供的随机数据转为php流
  - stream = php_stream_memory_create(TEMP_STREAM_DEFAULT);
  - php_stream_write(stream, (const char *) Data, Size);
- 之后将php流转为zval结构体
  - php_stream_to_zval(stream, &stream_zv);
- 进一步将zval结构体作为图片数据调用Exif处理
  - fuzzer_call_php_func_zval("exif_read_data", 1, &stream_zv);
- 调用完毕后将zval结构体引用计数-1，若引用计数为0，则zval结构体被释放
  - zval_ptr_dtor(&stream_zv);
- 其他exif函数：https://www.php.net/manual/zh/book.exif.php
- 其他图像处理函数：https://www.php.net/manual/zh/refs.utilspec.image.php
# 项目中已有的fuzzer-json.c，针对json数据解析进行fuzz ing
- 关键函数：初始化 PHP 的json解析器 php_json_parser_init
  - php_json_parser_init(&parser, &result, data, Size, option, 10);
- 解析 json 字符串 php_json_yyparse
  - php_json_yyparse(&parser) == SUCCESS
# 项目中已有的fuzzer-unserialize.c，针对php反序列化进行fuzzing
- unserialize,即反序列化
- 序列化：将对象转换为便于传输的格式，例如java语言中将java对象转为字节序列，从而便于网络传输，在进行浏览器访问的时候，我们看到的文本、图片、音频、视频等都是通过二进制序列进行传输的，均需要对应的序列化
- 反序列化：序列化的逆过程，当字节序列传输完毕后，需要将字节序列还原为对应对象，该过程即反序列化
```java
//java对象序列化实例
  Student student = new Student(1,"1234","小明");
  //序列化
  byte[] bytes = toBytes(student);
  System.out.println(bytes.length);
  //反序列化
  Student student0 = (Student) toObj(bytes);
  System.out.println(student0);
```
-
# 项目中自己增加的fuzzer-pregmatch.c，用于fuzzing字符串匹配函数
- 关键在于使用字符串匹配函数对由随机数据生成的两个字符串进行匹配
  - fuzzer_call_php_func("preg_match", 2, args);
# 参考资料
- PHP解释器引擎执行流程：https://blog.csdn.net/phpkernel/article/details/5716342
- php代码的一生（系列文章）：https://www.notee.cc/PHP/engine_general_lifetime_of_php_code_1/
- PHP内核探索之ZEND虚拟机2-方法调用：https://cloud.tencent.com/developer/news/191272
- PHP 内核分析：Zend 虚拟机（优质资料）：http://joshuais.me/php-zend-vm/
- 看我如何玩转PHP代码加密与解密：https://xz.aliyun.com/t/2403
- php手册：https://www.php.net/docs.php
# 当前思路
- protobuf的使用
- 使用php文件模板作为种子调用fuzzer
- fuzzer_mbstring中如何调用php中普通函数（是否可以用于调用其他函数）
  - 可能被fuzzing的字符串相关函数
    - 正则表达式相关（有实例）：https://www.php.net/manual/zh/book.pcre.php
    - 多字节字符串函数（有实例）：https://www.php.net/manual/zh/ref.mbstring.php
    - 字符串处理函数：https://www.php.net/manual/zh/ref.strings.php
    - exif相关函数：https://www.php.net/manual/zh/book.exif.php

# 扩展php解释器fuzzer
## 基于fuzzer-mbstring.c扩展
- 该fuzze中fuzzing的两个函数mb_ereg以及mb_eregi用于接受多字节字符串以及正则表达式，从而进行模式匹配，两者区别在于后者不区分大小写
- 目前思路
  1. 将更多多字节字符串处理函数引入fuzzing
  2. 创建正则表达式字典提高fuzzing效率
     1. 自创字典
     2. 参考其他已有正则表达式相关字典：
        1. 谷歌chromium：https://chromium.googlesource.com/chromium/src/+/master/testing/libfuzzer/fuzzers/dicts/regexp.dict#1
        2. AFL：https://github.com/rc0r/afl-fuzz/tree/master/dictionaries
     3. 将libfuzzer生成的推荐字典纳入fuzzing过程
  3. 使用protobuf_mutation生成符合正则表达式的随机输入
     1. 定义正则表达式语法：https://www.fuzzingbook.org/html/Grammars.html
  4. 在项目中使用更多的sanitizer包括 address sanitizer、undefined behaviorsanitizer、memory sanitizer

## 不使用字典的情况下
./php-fuzz-mbstring -max_total_time=60 -print_final_stats=1

#115794	DONE   cov: 3487 ft: 9394 corp: 1659/16Kb lim: 33 exec/s: 1898 rss: 150Mb
###### Recommended dictionary. ######
"\x00\x00" # Uses: 657
"\xff\xff" # Uses: 597
"\x04\x00" # Uses: 647
"8bit" # Uses: 549
"\x01\x00\x00\x00" # Uses: 511
"\x01\x00" # Uses: 611
"\x07\x00" # Uses: 618
"\x00\x00\x00\x00" # Uses: 449
"UTF-" # Uses: 449
"\x0c\x00" # Uses: 546
"\x02\x00" # Uses: 526
"\x01\x0a" # Uses: 505
"\x1b\x00\x00\x00" # Uses: 336
"\x00w" # Uses: 384
"\x01\x13" # Uses: 362
"X-Powered-By" # Uses: 152
"\x01\x18" # Uses: 261
"UTF-1" # Uses: 229
"UTF-8" # Uses: 186
"\x1e\x00" # Uses: 170
"\xff\xff\xff\xff\xff\xff\xff\xd7" # Uses: 137
"\x00\x00\x00\x05" # Uses: 93
"Content-Type" # Uses: 69
"X-Pow" # Uses: 79
"\x1f\x00" # Uses: 73
"\xff\xff\xff\xff\xff\xff\xff\x00" # Uses: 57
"\x02\x00\x00\x00\x00\x00\x00\x00" # Uses: 47
"\x01\x00\x00\x00\x00\x00\x00\x00" # Uses: 23
"\x00\x00\x00\x00\x00\x00\x00\x00" # Uses: 30
"\x01\x00\x00\x00\x00\x00\x00\x05" # Uses: 2
###### End of recommended dictionary. ######
Done 115794 runs in 61 second(s)
stat::number_of_executed_units: 115794
stat::average_exec_per_sec:     1898
stat::new_units_added:          2173
stat::slowest_unit_time_sec:    0
stat::peak_rss_mb:              150

## 使用自创字典的情况下
./php-fuzz-mbstring -max_total_time=60 -print_final_stats=1 -dict=./dict/my_regularexpression 

  #107978	DONE   cov: 3594 ft: 9721 corp: 1796/17Kb lim: 43 exec/s: 1770 rss: 156Mb
###### Recommended dictionary. ######
"\x00\x00" # Uses: 125
"\x01\x09" # Uses: 114
"\x12\x00\x00\x00" # Uses: 129
"\x01\x14" # Uses: 102
"\x0b\x00" # Uses: 103
"UTF-8" # Uses: 87
"\x00\x1d" # Uses: 125
"\x0a\x00" # Uses: 100
"\xff\xff\xff\xff" # Uses: 73
"\x1b\x00" # Uses: 108
"\x0c\x00" # Uses: 106
"\xff\xff\xff\xff\xff\xff\xff\xd7" # Uses: 53
"\x08\x00" # Uses: 83
"\x07\x00" # Uses: 105
"UTF-" # Uses: 83
"\x17\x00" # Uses: 93
"\x14\x00" # Uses: 77
"\x01\x00\x00\x00" # Uses: 67
"\x0f\x00\x00\x00" # Uses: 82
"\x00\x00\x00\x00" # Uses: 57
"\x00\x00\x00\x00\x00\x00\x00\x00" # Uses: 47
"\x00\x00\x00\x00\x00\x00\x00\x1b" # Uses: 42
"\xff\xff" # Uses: 71
"HTTP/" # Uses: 33
"X-Powered-By" # Uses: 33
"\xff\x0d" # Uses: 43
"\x01\x00\x00\x00\x00\x00\x00\x00" # Uses: 27
"WWW-Authenti" # Uses: 18
"\x01\x00" # Uses: 32
"\xfe\x00\x00\x00\x00\x00\x00\x00" # Uses: 28
"Location" # Uses: 27
"UTF-1" # Uses: 29
"\x00\x00\x00\x02" # Uses: 33
"X-Powere" # Uses: 25
"\xff\xff\xff\xff\xff\xff\xff\x00" # Uses: 18
"\x00\x00\x00\x1d" # Uses: 26
"\x01\x00\x00\x00\x00\x00\x00\x01" # Uses: 13
"\x0a\x00\x00\x00" # Uses: 11
"\xdc\x00\x00\x00\x00\x00\x00\x00" # Uses: 1
"\x00\x00\x00\x00\x00\x00\x00\x0c" # Uses: 0
"\x00\x00\x00\x00\x00\x00\x00\xde" # Uses: 0
###### End of recommended dictionary. ######
Done 107978 runs in 61 second(s)
stat::number_of_executed_units: 107978
stat::average_exec_per_sec:     1770
stat::new_units_added:          2188
stat::slowest_unit_time_sec:    0
stat::peak_rss_mb:              156
root@ubuntu:/work/php-src/sapi/fuzzer# 

## 使用chromium字典的情况下 
综上使用自己的正则表达式相关字典，fuzzing效果并无明显提升，尝试使用其他fuzzing项目中正则表达式字典，结果来看，fuzzer的cov指标有所提升，print_final_stats输出中的数据有所提升
 ./php-fuzz-mbstring -max_total_time=60 -print_final_stats=1 -dict=./dict/chromium_regularexpression 

#99238	DONE   cov: 4115 ft: 10260 corp: 1838/14995b lim: 21 exec/s: 1626 rss: 151Mb
###### Recommended dictionary. ######
"\x00\x00\x00\x1c" # Uses: 54
"\x01\x00" # Uses: 97
"\x01\x00\x00\x12" # Uses: 67
"\x01\x0d" # Uses: 71
"\x00\x00\x00\x0a" # Uses: 53
"\x07\x00" # Uses: 77
"\x05\x00" # Uses: 74
"8bit" # Uses: 48
"\xff\xff\xff\xff" # Uses: 56
"\x1d\x00" # Uses: 71
"\xff\xff\xff\x0d" # Uses: 51
"\xff\xff" # Uses: 54
"UTF-" # Uses: 49
"\x0d\x00" # Uses: 40
"\x06\x00" # Uses: 40
"Location" # Uses: 25
"\x01\x1b" # Uses: 37
"\x01\x00\x00\x01" # Uses: 31
"\x00\x00\x00\x00" # Uses: 36
"\x0f\x00" # Uses: 39
"\x00\x06" # Uses: 29
"\x00\x00\x00\x00\x00\x00\x00\xdf" # Uses: 14
"\x07\x00\x00\x00\x00\x00\x00\x00" # Uses: 16
"\xff\xff\xff\xff\xff\xff\xff\x00" # Uses: 20
"\x08\x00\x00\x00" # Uses: 16
"UTF-1" # Uses: 16
"\x00\x19" # Uses: 15
"X-Powered-By" # Uses: 15
"\xff\xff\xff\x15" # Uses: 16
"\x19\x00\x00\x00" # Uses: 15
"\x00\x05" # Uses: 14
"UTF-8" # Uses: 7
"\x00\x00\x00\x00\x00\x00\x00\x01" # Uses: 9
"Content-Leng" # Uses: 4
"HTTP/" # Uses: 4
"\x01\x00\x00\x00" # Uses: 3
"\x01\x00\x00\x00\x00\x00\x00\xd9" # Uses: 6
"\x01\x00\x00\x00\x00\x00\x00\x1b" # Uses: 3
"\x01\x00\x00\x00\x00\x00\x00\x00" # Uses: 5
"\x0c\x00\x00\x00\x00\x00\x00\x00" # Uses: 0
###### End of recommended dictionary. ######
Done 99238 runs in 61 second(s)
stat::number_of_executed_units: 99238
stat::average_exec_per_sec:     1626
stat::new_units_added:          2138
stat::slowest_unit_time_sec:    0
stat::peak_rss_mb:              151

## 使用libfuzzer推荐的字典
./php-fuzz-mbstring -max_total_time=60 -print_final_stats=1 -dict=./dict/libfuzzer_recommend 

#116470	DONE   cov: 3594 ft: 9211 corp: 1575/15905b lim: 29 exec/s: 1909 rss: 148Mb
###### Recommended dictionary. ######
"\x00\x18" # Uses: 160
"\x01\x07" # Uses: 155
"\x01\x1c" # Uses: 133
"\x01\x06" # Uses: 145
"\xff\xff\xff\xff\xff\xff\xff\x03" # Uses: 84
"\x18\x00\x00\x00" # Uses: 110
"\x01\x00\x00\x00\x00\x00\x005" # Uses: 73
"\x00\x00\x00\x00\x00\x00\x00\x04" # Uses: 75
"\x01w" # Uses: 77
"\x00\x09" # Uses: 15
"\xffv" # Uses: 15
"\x01\x00\x00\x0c" # Uses: 2
###### End of recommended dictionary. ######
Done 116470 runs in 61 second(s)
stat::number_of_executed_units: 116470
stat::average_exec_per_sec:     1909
stat::new_units_added:          2124
stat::slowest_unit_time_sec:    0
stat::peak_rss_mb:              148

## 自己创建字典+chromium字典+libfuzzer推荐字典
混合字典的效率的确更强
./php-fuzz-mbstring -max_total_time=60 -print_final_stats=1 -dict=./dict/mix_my_chromium_libfzzer 

#111634	DONE   cov: 4229 ft: 10910 corp: 2014/18Kb lim: 29 exec/s: 1830 rss: 152Mb
###### Recommended dictionary. ######
"\x00\x0b" # Uses: 68
"\x04\x00\x00\x00" # Uses: 56
"v\x00\x00\x00" # Uses: 64
"\x00\x07" # Uses: 57
"\x00\x00\x00\x00\x00\x00\x00\x10" # Uses: 18
"\xffv" # Uses: 38
"\x0e\x00" # Uses: 40
"v\x00" # Uses: 28
"\x01\x00\x00\x00\x00\x00\x00\xdb" # Uses: 11
"\x19\x00" # Uses: 11
"\xff\x0f" # Uses: 8
"\xff\xff\xff\xff\xff\xff\xff\xff" # Uses: 3
"\x01\x02" # Uses: 13
"\x00\x00\x00\x0e" # Uses: 6
"\x11\x00\x00\x00" # Uses: 1
###### End of recommended dictionary. ######
Done 111634 runs in 61 second(s)
stat::number_of_executed_units: 111634
stat::average_exec_per_sec:     1830
stat::new_units_added:          2449
stat::slowest_unit_time_sec:    0
stat::peak_rss_mb:              152

## 使用正则表达式种子文件提升fuzzzing效率
常用正则表达式积累
/(\w+):\/\/([^/:]+)(:\d*)?([^# ]*)/

/[-a-z]/

/ter\b/

/\d{2}-\d{5}/

/<\s*(\S+)(\s[^>]*)?>[\s\S]*<\s*\/\1\s*>/

^\d{n}$

^\d{n,}$

^\d{m,n}$ 

^[0-9]*$

^[0-9]+(.[0-9]{n})?$

^[0-9]+(.[0-9]{m,n})?$

\b[^\Wa-z0-9_][^\WA-Z0-9_]*\b

^[\u4e00-\u9fa5]{0,}$

^http:\/\/([\w-]+(\.[\w-]+)+(\/[\w-     .\/\?%&=\u4e00-\u9fa5]*)?)?$

\b[^\Wa-z0-9_][^\WA-Z0-9_]*\b

^.[a-zA-Z]\w{m,n}$

^.[A-Za-z0-9]+$

\w+([-+.']\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*

^[1-9]([0-9]{16}|[0-9]{13})[xX0-9]$

^13[0-9]{1}[0-9]{8}|^15[9]{1}[0-9]{8}

(^[0-9]{3,4}\-[0-9]{3,8}$)|(^[0-9]{3,8}$)|(^\([0-9]{3,4}\)[0-9]{3,8}$)|(^0{0,1}13[0-9]{9}$)

((\(\d{3}\)|\d{3}-)|(\(\d{4}\)|\d{4}-))?(\d{8}|\d{7})

^(25[0-5]|2[0-4][0-9]|[0-1]{1}[0-9]{2}|[1-9]{1}[0-9]{1}|[1-9])\.(25[0-5]|2[0-4][0-9]|[0-1]{1}[0-9]{2}|[1-9]{1}[0-9]{1}|[1-9]|0)\.(25[0-5]|2[0-4][0-9]|[0-1]{1}[0-9]{2}|[1-9]{1}[0-9]{1}|[1-9]|0)\.(25[0-5]|2[0-4][0-9]|[0-1]{1}[0-9]{2}|[1-9]{1}[0-9]{1}|[0-9])$

(P\d{7})|G\d{8})

^[a-zA-Z0-9]+([a-zA-Z0-9\-\.]+)?\.s|)$

^((?:4\d{3})|(?:5[1-5]\d{2})|(?:6011)|(?:3[68]\d{2})|(?:30[012345]\d))[ -]?(\d{4})[ -]?(\d{4})[ -]?(\d{4}|3[4,7]\d{13})$

^(\d[- ]*){9}[\dxX]$

^[A-Z0-9]{8}-[A-Z0-9]{4}-[A-Z0-9]{4}-[A-Z0-9]{4}-[A-Z0-9]{12}$

^([a-zA-Z]\:|\\)\\([^\\]+\\)*[^\/:*?"<>|]+\.txt(l)?$

^#?([a-f]|[A-F]|[0-9]){3}(([a-f]|[A-F]|[0-9]){3})?$

^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$

^http://([\w-]+\.)+[\w-]+(/[\w-./?%&=]*)?$

^(\(\d{3,4}-)|\d{3.4}-)?\d{7,8}$

^\d{15}|\d{18}$

^(0?[1-9]|1[0-2])$

^((0?[1-9])|((1|2)[0-9])|30|31)$






