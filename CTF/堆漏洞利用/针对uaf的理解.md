- ctfwiki
- 内存块被释放后，其对应的指针被设置为 NULL ， 然后再次使用，自然程序会崩溃。
- 内存块被释放后，其对应的指针没有被设置为 NULL ，然后在它下一次被使用之前，没有代码对这块内存块进行修改，那么程序很有可能可以正常运转。
- 内存块被释放后，其对应的指针没有被设置为NULL，但是在它下一次使用之前，有代码对这块内存进行了修改，那么当程序再次使用这块内存时，就很有可能会出现奇怪的问题。
- 而我们一般所指的 Use After Free 漏洞主要是后两种。此外，我们一般称被释放后没有被设置为NULL的内存指针为dangling pointer。