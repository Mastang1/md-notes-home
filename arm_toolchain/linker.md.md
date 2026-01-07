## 1. 链接脚本中ENTRY(Reset_handler)作用
1. 决定 ELF Header 的 Entry Point 字段
==ELF 文件头部有一个字段叫 e_entry。ENTRY 就是给这个字段赋值。==
比如你写 ENTRY(Reset_Handler)，那么最终 ELF header 中 entry point 就会被设置成 Reset_Handler 的地址。
这对 MCU 启动其实没影响，因为 MCU 不读 ELF，它读 Flash 的向量表；但对调试工具和模拟器有影响。
2. 让 GDB / OpenOCD 等调试器知道程序启动点
调试器加载 ELF 时，会把 ENTRY 定义的符号作为默认起始执行位置。
所以你用 GDB 的时候能看到：
`Start address 0x080001cd <Reset_Handler>`
如果你不写 ENTRY，GDB 可能会以 _start、main 或者某个默认符号作为起点，导致断点、单步都显得混乱。
==**一句话总结**==
==**ENTRY(Reset_Handler) 的作用是：**==
==**你告诉链接器：“把 Reset_Handler 设为 ELF 文件的入口点，让调试器、模拟器、链接优化都从这里开始认我。”**==
==**它不是 MCU 的开关，而是工具链生态的“坐标原点”。**==

## 2. 你好兄弟

在我的基于html的软件单元测试报告中，case有三种测试结果：pass/fail/error(表示发生了exception)/skip/timeout(表示在预定的时间内case未完成执行)/discard（表示因测试任务总超时到达而被放弃的测试用例），现在我需要设计这几种结果展示的预定义背景和color，请为我设计出两种符合产品级的配色方案，我要在项目中直接使用；并提供html简单展示