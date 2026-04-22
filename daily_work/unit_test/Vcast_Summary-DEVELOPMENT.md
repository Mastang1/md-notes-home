
---
_**个人总结（主要是用例执行、用例开发的的规则）：
0.无论是 Environment User Code 的各个阶段代码，还是 Probe Points（探针）代码，它们的底层运行机制都是“静态代码注入 -> 重新编译链接 -> 执行二进制文件”
 1.整个测试用例的测试原理是编译为一个elf文件进行执行；
 2.执行用例的框架：类似tcf_server，会逐个调用test case，并且为可以 在每个用例执行前后执行 test fixture操作，在vcast中叫做 “Environment user code”
 3.也可以为每个UUT（把每个C文件作为一个UUT，会做修改，然后再编译）的文件前后添加前后缀，这部分在“unit user code”部分实现，可以定义函数等，==会被作为UUT的一部分，在UI中可查==
 4.可以为每个被测的函数中加入自定义代码，用于控制类似while循环不会退出等问题；这部分在vcast中用邮件函数名实现，叫做“Probe Points”**_