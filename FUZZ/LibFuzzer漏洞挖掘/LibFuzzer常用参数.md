# libfuzzer官方参数手册
https://llvm.org/docs/LibFuzzer.html
# 第一个参数为语料库目录，主要用于保存fuzzing过程中由fuzzer生成的能够触发不同代码路径的测试用例，一般在fuzzer执行前，该目录为空
```bash
$ mkdir PDF_SEEDS
$ cd PDF_SEEDS
$ wget http://www.africau.edu/images/default/sample.pdf
$ cd ../
$ ./fuzz_pdf ./CORPUS_no_seed/
INFO: Seed: 1378167629
INFO: Loaded 1 modules   (19936 inline 8-bit counters): 19936 [0xafc700, 0xb014e0),
INFO: Loaded 1 PC tables (19936 PCs): 19936 [0x98b550,0x9d9350),
INFO:        0 files found in ./CORPUS_no_seed/
INFO: -max_len is not provided; libFuzzer will not generate inputs larger than 4096 bytes
INFO: A corpus is not provided, starting from an empty corpus
#2      INITED cov: 312 ft: 313 corp: 1/1b lim: 4 exec/s: 0 rss: 36Mb
        NEW_FUNC[1/1]: 0x6a1320 in Lexer::isSpace(int) (/work/xpdf-4.02/build/fuzz_pdf+0x6a1320)
#3      NEW    cov: 318 ft: 322 corp: 2/2b lim: 4 exec/s: 0 rss: 39Mb L: 1/1 MS: 1 ChangeBit-
#15     NEW    cov: 318 ft: 325 corp: 3/4b lim: 4 exec/s: 0 rss: 51Mb L: 2/2
```
当然语料库目录在fuzzer运行前也可以不为空，可以将之前fuzzer执行得到的语料库重新利用，从而达到快速提升覆盖率的目的
```bash
cp -r sapi/fuzzer/corpus/exif ./my-exif-corpus
sapi/fuzzer/php-fuzz-exif ./my-exif-corpus
```
# 第二个参数为种子文件的目录，用于保存攻击者提供给fuzzer的种子文件，fuzzer可以以该文件为基础进行变异，在fuzzer执行前，该目录不为空
```bash
$ ./fuzz_pdf ./CORPUS_with_seed/ ./PDF_SEEDS/
INFO: Seed: 1708208234
INFO: Loaded 1 modules   (19936 inline 8-bit counters): 19936 [0xafc700, 0xb014e0),
INFO: Loaded 1 PC tables (19936 PCs): 19936 [0x98b550,0x9d9350),
INFO:        0 files found in ./CORPUS_with_seed/
INFO:        1 files found in ./PDF_SEEDS/
INFO: -max_len is not provided; libFuzzer will not generate inputs larger than 4096 bytes
INFO: seed corpus: files: 1 min: 3028b max: 3028b total: 3028b rss: 33Mb
#2      INITED cov: 750 ft: 751 corp: 1/3028b lim: 4 exec/s: 0 rss: 37Mb
#4      NEW    cov: 752 ft: 760 corp: 2/6056b lim: 4 exec/s: 0 rss: 41Mb L: 3028/3028 MS: 2 ChangeBinInt-ChangeByte-
```
- 以上两个参数均可以看作语料库目录，模糊器将从这些语料库目录中的每一个中读取测试输入，区别仅仅在于libfuzzer会将新生成的测试输入将被写回到第一个语料库目录中
- 所以fuzzer运行前，一般保证第一个参数目录为空，将相关种子文件保存在第二个参数目录，从而将生成的样本保存在第一个目录
# 相关flag
## -max_total_time=90  即fuzzer运行时常最多为90s
## -dict=字典目录
## -dump_coverage = 1 保存覆盖率报告到文件系统
## LLVM_PROFILE_FILE环境变量指定覆盖率报告的名称
## -timeout
- 超时（以秒为单位），默认值为1200。如果输入的时间长于此超时时间，则该过程将视为失败情况。
## -close_fd_mask	
- 指示输出流在启动时关闭。 请注意，这将从目标代码中删除诊断输出（例如，断言失败时的消息）。
  - 0（默认）：不关闭stdout或stderr 
  - 1：关闭stdout 
  - 2：关闭stderr 
  - 3：关闭stdout和stderr
## -runs
- 单个测试运行的次数，-1（默认）为无限期运行。
## -merge=1 
- 语料库蒸馏
- 如果您拥有庞大的语料库（无论是通过模糊处理生成还是通过其他方式获取），您都可能希望将其最小化，同时又要保留完整的覆盖范围。 一种方法是使用-merge = 1标志：
```bash
mkdir NEW_CORPUS_DIR  # Store minimized corpus here.
	./my_fuzzer -merge=1 NEW_CORPUS_DIR FULL_CORPUS_DIR
```
- 您可以使用相同的标志将更多有趣的项目添加到现有语料库中。 仅触发新覆盖率的输入将被添加到第一个语料库。
```bash
./my_fuzzer -merge=1 CURRENT_CORPUS_DIR NEW_POTENTIALLY_INTERESTING_INPUTS_DIR
```
## -only_ascii
- 如果为1，则仅生成ASCII（isprint`` +''isspace）输入。 预设为0。
```bash
sapi/cli/php sapi/fuzzer/generate_parser_corpus.php
mkdir ./my-parser-corpus
sapi/fuzzer/php-fuzz-parser -merge=1 ./my-parser-corpus sapi/fuzzer/corpus/parser
sapi/fuzzer/php-fuzz-parser -only_ascii=1 ./my-parser-corpus
```
## -print_final_stats=1
- fuzzing完成后，打印输出统计结果，常与-max_total_time联合使用，后者用于限制fuzzer执行时间