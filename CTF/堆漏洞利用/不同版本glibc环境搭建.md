https://bbs.kanxue.com/thread-269145.htm

前言
众所周知，pwn的Glibc环境向来是一个难解题，很多大佬在编译不同版本的Glibc都很头疼，一个不注意就容易出错。像Github上的glibc-all-in-one搭配patchelf

 
glibc-all-in-one:matrix1001/glibc-all-in-one: A convenient glibc binary and debug file downloader and source code auto builder (github.com)

 
patchelf:NixOS/patchelf: A small utility to modify the dynamic linker and RPATH of ELF executables (github.com)

 
对于新手来说搞个虚拟机编译环境一个包没装好就容易挂掉，然后就GG，这实在是很浪费生命的一件事情。而patchelf其实有时候也不太顶用，还有Docker里的

 
pwnDocker:skysider/pwndocker - Docker Image | Docker Hub

 
其实有时候也感觉不太好用，而且需要依靠作者更新，自己编译也容易出错。但是这倒是激发了我一个想法，为每个Libc版本搭建个docker容器，然后通过映射关系将题目映射进容器中，相当于只需要容器中的libc环境，这样就不需要考虑这些东西了。

项目
经过大量测试，自己写了一个小项目，适合所有Libc版本，只要docker hub中有对应libc版本的ubuntu容器，该容器对应的apt源还有在更新，就能用，跟自己本身环境没啥关系。实测所有版本都行，一键搭建，一键使用：

 
Github:PIG-007/pwnDockerAll (github.com)

 
Gitee:PIG-007/pwnDockerAll (gitee.com)

 
详情看项目里。