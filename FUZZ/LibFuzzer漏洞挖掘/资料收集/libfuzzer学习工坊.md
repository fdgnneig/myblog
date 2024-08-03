https://github.com/Dor1s/libfuzzer-workshop

工坊学习之路
https://www.anquanke.com/member/154755
# LibFuzzer workshop学习之路 （第一部分）
# 环境搭建
跟着https://github.com/Dor1s/libfuzzer-workshop 搭建就好了，主要是build llvm的环境可能要make一会儿。编译好后拿到libfuzzer.a(静态链接库文件)，就可以开始上手实践了。
# 第一个fuzzer
```bash
// fuzz_target.cc
extern "C" int LLVMFuzzerTestOneInput(const uint8_t *Data, size_t Size) {
  DoSomethingInterestingWithMyAPI(Data, Size);
  return 0;  // Non-zero return values are reserved for future use.
}
```
通过将libfuzzer静态库连接到可执行文件的方法构建fuzzer可执行文件
```bash
clang++ -g -O1 -fsanitize=fuzzer,address \
fuzz_target.cc ../../libFuzzer/Fuzzer/libFuzzer.a \
-o fuzzer_target
```
# 第二个fuzzer
```bash
clang++ -g -std=c++11 -fsanitize=fuzzer,address first_fuzzer.cc ../../libFuzzer/Fuzzer/libFuzzer.a -o first_fuzzer
```
# 针对大型开源项目的fuzzing
- 对应工坊中的lesson5
- 注意针对大型开源项目的fuzzing行动，开源项目的编译flag会极大影响fuzzer的执行效率，需要选择有效的编译选项，从而提升fuzzing效率
- 这次的目标和lesson04有所不同，lesson04中我们针对.h函数库中的函数进行fuzz，而libfuzzer的威力远不止于此，它还可以对大型开源库进行模糊测试。在源码编译开源库时选择合适的选项以及将libfuzzer与开源库链接在一起以进行fuzz，这些细节将在该lesson05体现。
## 编译被fuzzing的目标程序，得到.a文件
- 首先解包tar xzf openssl1.0.1f.tgz。接着执行./config生成makefile。之后：
```bash
make clean
make CC="clang -O2 -fno-omit-frame-pointer -g -fsanitize=address" -j$(nproc)
```
- 首先，上面命令不规范，CC环境变量用于设置c语言的编译器，故仅使用CC="clang"时可行，而clang相关编译选项应当由CFLAGS或CXXFLAGS指定的
- 上述命令完成了fuzz目标的源码编译，生成了被fuzzing对象的.a文件。关键在于在编译过程中使用了-O2、-fno-omit-frame-pointer两个flag用于优化编译效果，使用-g生成了调试信息，使用-fsanitize=address在目标程序中引入了ASAN，并在使用make的-j参数实现了并行编译
- 通过以上编译优化，提升了fuzzer的运行效率
## 编译fuzzer，链接被fuzzing程序的.a文件
- 选择合适的编译器和编译选项，完成对该库的源码编译，生成.a文件。接下来就要研究编写fuzzer接口函数了。workshop提供了openssl_fuzzer.cc：
```bash
#include <openssl/ssl.h>
#include <openssl/err.h>
#include <assert.h>
#include <stdint.h>
#include <stddef.h>

#ifndef CERT_PATH
# define CERT_PATH
#endif

SSL_CTX *Init() {
  SSL_library_init();
  SSL_load_error_strings();
  ERR_load_BIO_strings();
  OpenSSL_add_all_algorithms();
  SSL_CTX *sctx;
  assert (sctx = SSL_CTX_new(TLSv1_method()));
  /* These two file were created with this command:
      openssl req -x509 -newkey rsa:512 -keyout server.key \
     -out server.pem -days 9999 -nodes -subj /CN=a/
  */
  assert(SSL_CTX_use_certificate_file(sctx, CERT_PATH "server.pem",
                                      SSL_FILETYPE_PEM));
  assert(SSL_CTX_use_PrivateKey_file(sctx, CERT_PATH "server.key",
                                     SSL_FILETYPE_PEM));
  return sctx;
}

extern "C" int LLVMFuzzerTestOneInput(const uint8_t *Data, size_t Size) {
  static SSL_CTX *sctx = Init();
  SSL *server = SSL_new(sctx);
  BIO *sinbio = BIO_new(BIO_s_mem());
  BIO *soutbio = BIO_new(BIO_s_mem());
  SSL_set_bio(server, sinbio, soutbio);
  SSL_set_accept_state(server);
  BIO_write(sinbio, Data, Size);
  SSL_do_handshake(server);
  SSL_free(server);
  return 0;
}
```
这里就涉及到openssl库提供的相关方法了，本篇主要讲解fuzz相关，就不细讲openssl了。总之就是要先搞清楚openssl的用法，再通过include openssl提供的函数来对openssl进行fuzz。编译如下：
```bash
clang++ -g openssl_fuzzer.cc -O2 -fno-omit-frame-pointer -fsanitize=address,fuzzer \
    -I openssl1.0.1f/include openssl1.0.1f/libssl.a openssl1.0.1f/libcrypto.a \
    ../../libFuzzer/Fuzz/libFuzzer.a -o openssl_fuzzer
```
- 上述命令使用了-fsanitize=address,fuzzer，从而生成最终的fuzzer二进制文件
- -I指定inlcude的搜索路径，同时链接静态库libcrypto.a和libFuzzer.a以使用库中的函数。
- 运行fuzzer，爆出crash，溯源，发现心脏滴血漏洞
# LibFuzzer workshop学习之路 （第二部分）
# 使用clang的采样分析器提升fuzzing效率
- 对应lesson 08，针对libxml2进行fuzz
## 编译libxml2
```bash
tar xzf libxml2.tgz
cd libxml2

./autogen.sh
export FUZZ_CXXFLAGS="-O2 -fno-omit-frame-pointer -gline-tables-only -fsanitize=address,fuzzer-no-link"
CXX="clang++ $FUZZ_CXXFLAGS" CC="clang $FUZZ_CXXFLAGS" \
    CCLD="clang++ $FUZZ_CXXFLAGS"  ./configure
make -j$(nproc)
```
- 使用flag：-fno-omit-frame-pointer，该flag用于生成行号表相关调试信息，其中包含函数名、文件名、行号信息，且不包含其他数据，同时该flag也是使用clang中采样分析器的必备flag
## clang中的采样分析器（可以提高程序运行效率）
- 采样分析器用于在应用程序执行时收集运行时信息，例如硬件计数等。 它们通常非常高效，并且不会产生较大的运行时开销。 由分析器收集的样本数据可在编译期间用于确定代码中执行最多的区域，从而在程序再次编译到过程中有效提高程序运行效率
- 该思路可以运用于编译被fuzzing项目以及fuzzer本身，从而提高fuzzing运行效率
- 使用样本分析器中的数据需要对程序的构建方式进行一些更改。 在编译器可以使用分析信息之前，代码需要在事件探查器下执行。以下是使用采样分析器进行优化时的通常构建周期：  
1. 使用源码行表信息构建代码。 您可以使用所有通常用来构建应用程序的常规构建flag。 唯一的要求是将-gline-tables-only或-g添加到命令行。 这对于分析器能够将指令映射回源代码行位置非常重要。
```bash
$ clang++ -O2 -gline-tables-only code.cc -o code
```
2. 在采样分析器下运行可执行文件。 您使用的特定分析器并不重要，只要它的输出可以转换为LLVM优化器可以理解的格式即可。 当前，存在用于Linux Perf探查器（https://perf.wiki.kernel.org/）的转换工具，因此这些示例假定您正在使用Linux Perf探查代码。
- 注意使用-b标志。 这告诉Perf使用最后分支记录（LBR）记录函数调用链。 尽管这不是严格要求的，但它提供了更好的函数调用信息，从而提高了配置文件数据的准确性。
```bash
$ perf record -b ./code
```
3. 将收集的配置文件数据转换为LLVM的样本配置文件格式。 当前通过AutoFDO转换器create_llvm_prof支持此功能。 可从https://github.com/google/autofdo获得。 构建并安装后，您可以使用以下命令将perf.data文件转换为LLVM样本配置文件：
- 这将读取perf.data和二进制文件./code并将配置文件数据保存在code.prof中。请注意，如果上一步中在没有-b标志的情况下运行perf，则在调用create_llvm_prof时需要使用--use_lbr = false。
```bash
$ create_llvm_prof --binary=./code --out=code.prof
```
4. 使用收集到的配置文件再次构建代码。 此步骤将上一步得到的配置文件文件反馈给优化器。 这将导致二进制文件比原始二进制文件执行得更快。 请注意，您不需要使用与第一步中使用的参数完全相同的参数来构建代码。 唯一的要求是使用-gline-tables-only和-fprofile-sample-use来构建代码。
```bash
$ clang++ -O2 -gline-tables-only -fprofile-sample-use=code.prof code.cc -o code
```
## 编译fuzzer
- fuzzer源码
```bash
// Copyright 2015 The Chromium Authors. All rights reserved.
// Use of this source code is governed by a BSD-style license that can be
// found in the LICENSE file.

#include <stddef.h>
#include <stdint.h>

#include "libxml/parser.h"

void ignore (void* ctx, const char* msg, ...) {
  // Error handler to avoid spam of error messages from libxml parser.
}

extern "C" int LLVMFuzzerTestOneInput(const uint8_t* data, size_t size) {
  xmlSetGenericErrorFunc(NULL, &ignore);

  if (auto doc = xmlReadMemory(reinterpret_cast<const char*>(data),
                               static_cast<int>(size), "noname.xml", NULL, 0)) {
    xmlFreeDoc(doc);
  }

  return 0;
}
```
- 编译fuzzer命令
- 可能的编译指令1(libfuzzer工坊项目提供)
```bash
cd ..
clang++ -std=c++11 xml_read_memory_fuzzer.cc $FUZZ_CXXFLAGS -I libxml2/include \
    libxml2/.libs/libxml2.a ../../libFuzzer/libFuzzer.a -lz \
    -o xml_read_memory_fuzzer
```
- 可能的编译指令2（分析文章提供）
```bash
clang++ -O2 -fno-omit-frame-pointer -gline-tables-only -fsanitize=address,fuzzer-no-link -std=c++11 xml_read_memory_fuzzer.cc -I libxml2/include libxml2/.libs/libxml2.a -fsanitize=fuzzer -lz -o libxml2-v2.9.2-fsanitize_fuzzer1
```
## sanitizer设置
除了ASAN等常用sanitizer，clang还对更具体的漏洞类型提供了sanitizer检测
- -fsanitize-address-field-padding=<value>
  - Level of field padding for AddressSanitizer
- -fsanitize-address-globals-dead-stripping
  - Enable linker dead stripping of globals inAddressSanitizer
- -fsanitize-address-poison-custom-array-cookie
  - Enable poisoning array cookies when using customoperator new[] in AddressSanitizer
- -fsanitize-address-use-after-scope
  - Enable use-after-scope detection in AddressSanitizer
- -fsanitize-address-use-odr-indicator
  - Enable ODR indicator globals to avoid false ODR violation reports in partially sanitized programs at the cost of an increase in binary size
- 可以将-fsanitize-address-use-after-scope加入到编译选项中，从而在ASAN基础上加上针对uaf漏洞的检测
```bash
export FUZZ_CXXFLAGS="-O2 -fno-omit-frame-pointer -gline-tables-only -fsanitize=address,fuzzer-no-link -fsanitize-address-use-after-scope"

CXX="clang++ $FUZZ_CXXFLAGS" CC="clang $FUZZ_CXXFLAGS" \
    CCLD="clang++ $FUZZ_CXXFLAGS"  ./configure

make -j$(nproc)
clang++ -O2 -fno-omit-frame-pointer -gline-tables-only -fsanitize=address,fuzzer-no-link -fsanitize-address-use-after-scope -std=c++11 xml_read_memory_fuzzer.cc -I libxml2/include libxml2/.libs/libxml2.a -fsanitize=fuzzer -lz -o libxml2-v2.9.2-fsanitize_fuzzer1
```
# 使用字典
- ./libxml2-v2.9.2-fsanitize_fuzzer1 -max_total_time=60 -print_final_stats=1 -dict=./xml.dict corpus1
- 由于使用了-print_final_stats=1 ，所以fuzzing执行完毕后会输出统计信息,输出信息中统计了字典中不同字段的命中频率，同时扩充了字典中的字段，可以将此时的到的字段保存到字典文件中，方便之后使用
- 统计信息中stat::new_units_added:5007给出了本次fuzzing触发的代码路径数量
```bash
#468074    REDUCE cov: 2521 ft: 8272 corp: 2493/82Kb lim: 135 exec/s: 7801 rss: 452Mb L: 105/135 MS: 4 InsertRepeatedBytes-CMP-CopyPart-CrossOver- DE: "\x01\x00\x00\x00"-
#468322    REDUCE cov: 2521 ft: 8272 corp: 2493/82Kb lim: 135 exec/s: 7805 rss: 452Mb L: 85/135 MS: 3 CopyPart-CopyPart-EraseBytes-
#468381    REDUCE cov: 2521 ft: 8273 corp: 2494/82Kb lim: 135 exec/s: 7806 rss: 452Mb L: 74/135 MS: 1 InsertByte-
#468390    REDUCE cov: 2521 ft: 8273 corp: 2494/82Kb lim: 135 exec/s: 7806 rss: 452Mb L: 66/135 MS: 2 ChangeASCIIInt-EraseBytes-
#468391    REDUCE cov: 2521 ft: 8273 corp: 2494/82Kb lim: 135 exec/s: 7806 rss: 452Mb L: 89/135 MS: 1 EraseBytes-
#468575    DONE   cov: 2521 ft: 8273 corp: 2494/82Kb lim: 135 exec/s: 7681 rss: 452Mb
###### Recommended dictionary. ######
"\x08\x00" # Uses: 366
"Q\x00" # Uses: 370
"\x00:" # Uses: 325
"\x97\x00" # Uses: 301
"\x0d\x00" # Uses: 335
"\xfe\xff\xff\xff" # Uses: 273
"UCS-" # Uses: 294
"\x15\x00" # Uses: 277
"\x00\x00" # Uses: 289
"\xff\xff\xff\x1c" # Uses: 258
"\xff\xff\xff!" # Uses: 257
"\xff\xff\xff\x01" # Uses: 250
"UTF-1" # Uses: 250
"\xff\xff\xffN" # Uses: 236
"UTF-16LE" # Uses: 223
"ISO-10" # Uses: 228
"ISO-1064" # Uses: 256
"\x0a\x00\x00\x00" # Uses: 246
"Q\x00\x00\x00" # Uses: 247
"\xf1\x1f\x00\x00\x00\x00\x00\x00" # Uses: 212
"$\x00\x00\x00\x00\x00\x00\x00" # Uses: 194
"\xff\xff\xff\x0e" # Uses: 211
"\x09\x00" # Uses: 226
"\x01\x00\x00\xfa" # Uses: 212
"\x01\x00\x00\x02" # Uses: 239
"\xac\x0f\x00\x00\x00\x00\x00\x00" # Uses: 206
"\xffO" # Uses: 263
"\xff\x03" # Uses: 235
"\xff\xff\xff\xff\xff\xff\xff\x10" # Uses: 200
"\xf4\x01\x00\x00\x00\x00\x00\x00" # Uses: 203
"UTF-16BE" # Uses: 188
"\x00\x00\x00P" # Uses: 207
"\x0a\x00" # Uses: 196
"\xff\xff" # Uses: 203
"\xff\xff\xff\xff\xff\x97\x96\x80" # Uses: 186
"\x01 \x00\x00\x00\x00\x00\x00" # Uses: 187
"\x00\x00\x00\x00\x00\x00\x00$" # Uses: 156
"P\x00" # Uses: 186
"\xff\xff\xff\xff" # Uses: 197
"\xff\xff\xff\x09" # Uses: 202
"\x12\x00\x00\x00\x00\x00\x00\x00" # Uses: 204
"\x01\x01" # Uses: 170
"\x01\x00\x00\x00\x00\x00\x00\x10" # Uses: 197
"\xff\xff\xff\xff\xff\xff\xff\x03" # Uses: 183
"\x00\x00\x00\x00\x00\x00\x00\x00" # Uses: 174
"\xff\x05" # Uses: 198
"US-ASCII" # Uses: 214
"\x01\x00" # Uses: 201
"xlmns" # Uses: 189
"\xff\xff\xff\x14" # Uses: 191
"xmlsn" # Uses: 179
"\x00\x00\x00\x03" # Uses: 201
"xmlns" # Uses: 182
"\xaf\x0f\x00\x00\x00\x00\x00\x00" # Uses: 186
"\xff\xff\xff\xff\xff\xff\x0e\xb9" # Uses: 176
"\xff\x09" # Uses: 178
"ISO-1" # Uses: 191
"la" # Uses: 157
"\x01\x00\x00\x00" # Uses: 173
"\x01\x00\x00\x00\x00\x00\x00\x14" # Uses: 172
"\xff\xff\xff\x7f\x00\x00\x00\x00" # Uses: 164
"\x00\x00\x00\x04" # Uses: 154
"\x01\x00\x00\x00\x00\x00\x00\x00" # Uses: 153
"\x00\x00\x00\x02" # Uses: 136
"\x04\x00\x00\x00\x00\x00\x00\x00" # Uses: 140
"ISO-10646-" # Uses: 146
"id" # Uses: 155
"\x00\x01" # Uses: 145
"\x00\x02" # Uses: 140
"\x01\x00\x00\x08" # Uses: 165
"\x00\x00\x00\x00\x00\x00\x00\x1e" # Uses: 136
"\xff\xff\xff\xff~\xff\xff\xff" # Uses: 130
"\x81\x96\x98\x00\x00\x00\x00\x00" # Uses: 150
"\x03\x00\x00\x00" # Uses: 116
"\x18\x00\x00\x00" # Uses: 176
"\xff\xff\xff\xff\xff\xff\xff\xf9" # Uses: 124
"%\x17\x8f[" # Uses: 130
"\x0e\x00\x00\x00" # Uses: 142
"\x01\x00\x00\x00\x00\x00\x00\xfa" # Uses: 96
"\x06\x00\x00\x00\x00\x00\x00\x00" # Uses: 116
"\x00\x04" # Uses: 161
"\x00\x00\x00\x0b" # Uses: 119
"\x00\x00\x00\x06" # Uses: 141
"annnn\xd4nnnnnn" # Uses: 100
"\x1f\x00\x00\x00\x00\x00\x00\x00" # Uses: 106
"\x00\x00\x00\x00\x00\x00\x00\x17" # Uses: 114
"\x16\x00\x00\x00\x00\x00\x00\x00" # Uses: 122
"\x0f\x00" # Uses: 121
"inlc0a" # Uses: 128
"\x01\x00\x00O" # Uses: 101
"\x01\x04" # Uses: 117
"\x01P" # Uses: 122
"\xfb\x00\x00\x00\x00\x00\x00\x00" # Uses: 95
"\x03\x00\x00\x00\x00\x00\x00\x00" # Uses: 107
"\x00\x00\x00\x00\x00\x00\x00\x03" # Uses: 98
"\x00#" # Uses: 102
"\x00\x00\x00\x0d" # Uses: 92
"ISO-8859-1" # Uses: 100
"\xff\xf9" # Uses: 77
"\xf7\x0f\x00\x00\x00\x00\x00\x00" # Uses: 79
"\xff\xff\xff\xff\xff\xff\xff\xfb" # Uses: 82
"\x01\x0b" # Uses: 101
"\xff\xff\xff\xff\xff\xff\x0e\xff" # Uses: 78
"><>\xb7" # Uses: 81
"<b" # Uses: 81
"UTF-8\x00" # Uses: 76
"\xff\xff\xff\xff\xff\xff\xff\x09" # Uses: 63
"#\x00\x00\x00" # Uses: 75
"S\x00\x00\x00\x00\x00\x00\x00" # Uses: 73
"a\xff" # Uses: 70
"TIONb" # Uses: 46
"\x01\x00\x00?" # Uses: 69
"!\x00\x00\x00" # Uses: 67
"\x00\x00\x00\x01" # Uses: 74
"\xff\xff\xff\xff\xff\xff\x1f\x02" # Uses: 57
"\x01\x00\x00\x00\x00\x00\x00\x16" # Uses: 57
"-\x00\x00\x00\x00\x00\x00\x00" # Uses: 53
"\x01\x00\x00\x05" # Uses: 65
":b" # Uses: 67
"\x17\x1c_>" # Uses: 63
"\xff\xff\xff\xff\xff\xff\xff\x18" # Uses: 64
"\x00\x00\x00'" # Uses: 45
"\x00\x00\x00\x05" # Uses: 52
"\xff\xff\xff\x0d" # Uses: 51
"US-AS" # Uses: 48
"a>" # Uses: 53
"C\x00\x00\x00\x00\x00\x00\x00" # Uses: 44
"\xff\xff\x00\x00" # Uses: 36
"\x01\x07" # Uses: 49
"@\x00\x00\x00" # Uses: 46
"\x02\x00" # Uses: 32
"+\x00" # Uses: 37
"\x00\x00\x00\x00\x00\x00 \x02" # Uses: 42
"\x00\x0f" # Uses: 37
"\xff\xff\xff\xff\xff\xff\xff$" # Uses: 49
"ASCII" # Uses: 40
"\x00\x00\x00\x00\x00\x00\x01\x00" # Uses: 30
"a\xff:-\xec" # Uses: 27
"\xff\x1a" # Uses: 30
"'''''''''&''" # Uses: 23
"\x01\x00\x00\x00\x00\x00\x01\x1d" # Uses: 34
"TIOIb" # Uses: 19
"J\x00\x00\x00\x00\x00\x00\x00" # Uses: 15
"N\x00\x00\x00" # Uses: 10
"\x01O" # Uses: 8
"\xff\xff\xff\x02" # Uses: 6
"HTML" # Uses: 8
"\x00P" # Uses: 9
"\xff\xff\xff\x00" # Uses: 9
"\xff\x06" # Uses: 9
"\x7f\x96\x98\x00\x00\x00\x00\x00" # Uses: 4
"^><b>" # Uses: 5
"\x01\x0a" # Uses: 5
"\x13\x00" # Uses: 1
###### End of recommended dictionary. ######
Done 479244 runs in 61 second(s)
stat::number_of_executed_units: 479244
stat::average_exec_per_sec:     7856
stat::new_units_added:          5007
stat::slowest_unit_time_sec:    0
stat::peak_rss_mb:              467
```
# 语料库蒸馏 
./libxml2-v2.9.2-fsanitize_fuzzer1 -merge=1 corpus1_min corpus1
# LibFuzzer workshop学习之路（第三部分）
## 编译过程中指定configure脚本参数，提升被fuzzing程序效率
- 对应lesson11
- 被fuzzing目标程序编译
```bash
tar xzf pcre2-10.00.tgz
cd pcre2-10.00

./autogen.sh
export FUZZ_CXXFLAGS="-O2 -fno-omit-frame-pointer -gline-tables-only -fsanitize=address,fuzzer-no-link -fsanitize-address-use-after-scope"
CXX="clang++ $FUZZ_CXXFLAGS" CC="clang $FUZZ_CXXFLAGS" \
    CCLD="clang++ $FUZZ_CXXFLAGS"  ./configure --enable-never-backslash-C \
    --with-match-limit=1000 --with-match-limit-recursion=1000
make -j
```
- 程序编译过程中configure脚本执行使用了相关参数，从而提升程序执行效率，间接提升fuzzing效率
- 相关configure参数
  - --with-match-limit=1000:限制一次匹配时使用的资源数为1000,默认值为10000000
  - --with-match-limit-recursion=1000:限制一次匹配时的递归深度为1000,默认为10000000(几乎可以说是无限)
  - --enable-never-backslash-C:禁用在字符串中，将反斜线作为转义序列接受。
  - 通过以上设置提升了被编译的正则表达式库的运行效率，从而提升其在fuzzing过程中的表现
## 编译过程中指定链接器相关参数从而提升fuzzer效率
承接上文，编译fuzzer的指令
```bash
clang++ -O2 -fno-omit-frame-pointer -gline-tables-only -fsanitize=address,fuzzer-no-link -fsanitize-address-use-after-scope pcre2_fuzzer.cc -I pcre2-10.00/src -Wl,--whole-archive pcre2-10.00/.libs/libpcre2-8.a pcre2-10.00/.libs/libpcre2-posix.a -Wl,-no-whole-archive -fsanitize=fuzzer -o pcre2-10.00-fsanitize_fuzzer
```
- clang中-Wl参数可以指定ld链接器的参数
  - -Wl,<arg>,<arg2>...
  - 将<arg>中逗号分隔的参数传递给链接器
  - https://clang.llvm.org/docs/ClangCommandLineReference.html
- ld链接器相关参数
  - 通过查询GNU官网，从而查询属于GNU软件的ld软件的使用手册
  - https://sourceware.org/binutils/docs/ld/
  - https://sourceware.org/binutils/docs/ld/Options.html#Options
  - --whole-archive可以把 在其后面出现的静态库包含的函数和变量输出到动态库
  - --no-whole-archive则关掉这个特性，因此这里将两个静态库libpcre2-8.a和libpcre2-posix.a里的符号输出到动态库里，使得程序可以在运行时动态链接使用到的函数，也使得fuzz效率得到了提升。
  - 在上述命令中，-Wl,--whole-archive pcre2-10.00/.libs/libpcre2-8.a pcre2-10.00/.libs/libpcre2-posix.a将libpcre2-8.a、libpcre2-posix.a两个静态链接库连接到最终的fuzzer中，如果将目标项目中所有的静态链接库连接到fuzze中，则可以触发更多地代码，提升发现漏洞的可能性，下面指令中将目标项目中所有静态链接库链接到fuzzer中
```bash
clang++ -O2 -fno-omit-frame-pointer -gline-tables-only -fsanitize=address,fuzzer-no-link -fsanitize-address-use-after-scope pcre2_fuzzer.cc -I pcre2-10.00/src -Wl,--whole-archive pcre2-10.00/.libs/*.a -Wl,-no-whole-archive -fsanitize=fuzzer -o pcre2-10.00-fsanitize_fuzzer
``` 
  - 起作用的关键指令部分-Wl,--whole-archive pcre2-10.00/.libs/*.a，用于将项目中所有静态链接库链接到fuzze中
## fuzzer运行时使用-max_len指定随机数据长度，提升fuzzer效率
- 编译项目
```bash
tar xzf re2.tgz
cd re2
export FUZZ_CXXFLAGS="-O2 -fno-omit-frame-pointer -gline-tables-only -fsanitize=address,fuzzer-no-link -fsanitize-address-use-after-scope"
make clean
CXX=clang++ CXXFLAGS="$FUZZ_CXXFLAGS"  make -j
```
- 编译fuzzer，使用-std的flag指定项目所使用的c语言标准
```bash
clang++ -O2 -fno-omit-frame-pointer -gline-tables-only -fsanitize=address,fuzzer-no-link -fsanitize-address-use-after-scope -std=gnu++98 target.cc -I re2/ re2/obj/libre2.a -fsanitize=fuzzer -o re2_fuzzer
```
- 通过设置-print_final_stats= -max_len= -max_total_time=，在相同时间段内以不同的输入长度最大值运行fuzzer，判断输入最大值对fuzzer代码覆盖率的影响
- max_len为10，执行时间为100秒,-print_final_stats=1打印最后的结果，corpus1作为语料库的存放处，发现覆盖代码块36个
```bash
➜  10 git:(master) ✗ ./re2_fuzzer ./corpus1 -print_final_stats=1 -max_len=10 -max_total_time=100

Done 643760 runs in 101 second(s)
stat::number_of_executed_units: 643760
stat::average_exec_per_sec:     6373
stat::new_units_added:          36
stat::slowest_unit_time_sec:    0
stat::peak_rss_mb:              456
```
- max_len为500，执行时间为100秒,-print_final_stats=1打印最后的结果，corpus4作为语料库的存放处，发现覆盖代码块117个
```bash
./re2_fuzzer ./corpus4 -print_final_stats=1 -max_len=500 -max_total_time=100

Done 119361 runs in 101 second(s)
stat::number_of_executed_units: 119361
stat::average_exec_per_sec:     1181
stat::new_units_added:          117
stat::slowest_unit_time_sec:    0
stat::peak_rss_mb:              827
```
  - 设置max_len为1000，执行时间为100秒,-print_final_stats=1打印最后的结果，corpus3作为语料库的存放处,发现覆盖代码块97个
```bash
./re2_fuzzer ./corpus3 -print_final_stats=1 -max_len=1000 -max_total_time=100

Done 105935 runs in 101 second(s)
stat::number_of_executed_units: 105935
stat::average_exec_per_sec:     1048
stat::new_units_added:          97
stat::slowest_unit_time_sec:    0
stat::peak_rss_mb:              830
```

# 提高fuzzing效果的关键
1. 使用合理的编译选项，针对被fuzzing程序以及fuzzer自身进行编译优化
   - 通过设置configure运行时参数，从而提升被fuzzing程序运行效率
     1. --with-match-limit=1000:限制一次匹配时使用的资源数为1000,默认值为10000000
     2. --with-match-limit-recursion=1000:限制一次匹配时的递归深度为1000,默认为10000000(几乎可以说是无限)
     3. --enable-never-backslash-C:禁用在字符串中，将反斜线作为转义序列接受。
   - 通过设置clang编译参数，从而提升fuzzer运行效率
     1. -O2
     2. -fno-omit-frame-pointer
     3. -fno-omit-frame-pointer （涉及采样分析器）
     4. -Wl （在clang中设置链接器ld的参数）
2. 使用字典
   1. -print_final_stats=1使用后会在fuzzer运行完毕后输出字典中各字段的使用频率以及字典的扩充，可以用于有效扩展字典大小
3. 使用优良的种子文件
4. 使用并行fuzzing
5. fuzzer运行时使用-max_len等参数
   - 结合-max_total_time、-print_final_stats=1参数，可以通过使用不同的max_len值运行相同时间的fuzzer，从而判断不同长度的输入对fuzzer代码覆盖率的影响
6. 将目标项目中更多的静态链接库链接到fuzzer中，从而让fuzzer触发更多的代码
   1. -Wl,--whole-archive pcre2-10.00/.libs/*.a
   
