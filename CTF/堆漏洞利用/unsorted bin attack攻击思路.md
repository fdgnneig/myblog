
- 该漏洞利用效果是在任意地址处写入一个较大固定值（即arena结构体中 bin[]成员的内存地址）
- 写入该值可以将指定变量变得极大，或获取该值本身就可以作为信息泄露是，即通过arena 中成员的内存地址推测libc加载基址

# https://xz.aliyun.com/t/7251#toc-12
- 深入理解unsorted bin attack - 先知社区 (2023_3_30 14_41_36).html
- unsorted bin attack作为一种久远的攻击方式常常作为其他攻击方式的辅助手段，
- 比如修改global_max_fast为一个较大的值使得几乎所有大小的chunk都用fast bin的管理方式进行分配和释放，
- 又或者修改_IO_list_all来伪造_IO_FILE进行攻击。
- unsorted bin attack不仅仅是起到将一个地址写入一个libc地址的目的，在修改_IO_list_all时，我们可以根据成员变量推断出伪造的文件结构体的_chain为small bin[0x60]进而伪造vtable；
- 在构造得当时，即使没有UAF也可以实现House-of-Roman；在能够编辑的情况下劫持top_chunk控制堆块分配也是一种新奇的利用方式。

# https://ciphersaw.me/ctf-wiki/pwn/linux/heap/unsorted_bin_attack/
- ctf-wiki对应文件
- 我们通过修改循环的次数来使得程序可以执行多次循环。
- 我们可以修改 heap 中的 global_max_fast 来使得更大的 chunk 可以被视为 fast bin，这样我们就可以去执行一些 fast bin attack了。
