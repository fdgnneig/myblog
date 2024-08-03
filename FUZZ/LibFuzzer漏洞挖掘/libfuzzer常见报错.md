fuzz运行时内存超过限制，从而报错

OOMs
Out-of-memory (OOM) bugs slowdown in-process fuzzing immensely. By default libFuzzer limits the amount of RAM per process by 2Gb.
Try fuzzing the woff benchmark with an empty seed corpus:

cd ~/woff
mkdir NEW_CORPUS
./woff2-2016-05-06-fsanitize_fuzzer NEW_CORPUS -jobs=8 -workers=8

Pretty soon you will hit an OOM bug:

==30135== ERROR: libFuzzer: out-of-memory (used: 2349Mb; limit: 2048Mb)
   To change the out-of-memory limit use -rss_limit_mb=<N>

   Live Heap Allocations: 3749936468 bytes from 2254 allocations; showing top 95%
   3747609600 byte(s) (99%) in 1 allocation(s)
   ...
   #6 0x62e8f6 in woff2::ConvertWOFF2ToTTF src/woff2_dec.cc:1274
   #7 0x660731 in LLVMFuzzerTestOneInput FTS/woff2-2016-05-06/target.cc:13:3

The benchmark directory also contains a reproducer for the OOM bug. Find it. Can you reproduce the OOM?

Sometimes using 2Gb per one target invocation is not a bug, and so you can use -rss_limit_mb=N to set another limit.`