
# VectorCAST 用户代码 (User Code) 核心指南

在 VectorCAST 中，用户代码 (User Code) 允许测试人员编写 C/C++ 语言表达式来动态设置或检查数据对象的值，以及执行其他动态的测试用例初始化和清理工作。

以下为您整理的 Obsidian 风格学习笔记，详细对比了四种可选用户代码的区别与用途，并深度拆解了 Environment User Code 的 13 个生命周期阶段。

---
![[Pasted image 20260422112550.png]]
## 核心概念：四类可选用户代码 (Optional User Code)

VectorCAST 提供了四种主要的可选用户代码注入点，它们的作用域和应用场景有明确的区别。

### 1. User Globals (用户全局变量)

- **功能区别**：提供一种机制，将用户自定义的数据类型（如结构体）和全局对象直接包含到测试框架（Test Harness）中。
- **核心用途**：
    - 为底层函数定义测试专用的临时数据对象。
    - 用于在纯黑盒测试或抽象接口测试时，为主程序的 `void*` 型指针分配具体且合法的外部对象。
    - 让这些自定义变量暴露在测试用例的参数树 (Parameter Tree) 中，以便像普通参数一样直接在 UI 中设置 Input 和 Expected 值。
- **代码 Demo**： 为了确保在测试框架中只创建该变量的唯一实例，声明时必须使用 `VCAST_USER_GLOBALS_EXTERN` 宏前缀。
    
    ```
    /* 定义一个测试专用的全局结构体和整型变量 */
    struct MyTestConfig { int mode; };
    VCAST_USER_GLOBALS_EXTERN struct MyTestConfig g_test_config;
    VCAST_USER_GLOBALS_EXTERN int my_global_counter;
    ```
    

### 2. User Params (用户参数)

- **功能区别**：用于指示 VectorCAST 在测试中强行使用一个**用户自定义的对象**来替代工具自动生成的函数参数对象。
- **核心用途**：
    - 建立“自定义参数”与“子程序真实入参”之间的强制物理关联。
    - 如果 VectorCAST 自动生成的驱动变量类型存在编译问题或无法满足特定的复杂内存对齐需求，可用此功能进行变量替换。
- **代码 Demo**： 语法采用 `<被替换的 VectorCAST 变量标签> <自定义变量名>` 的形式。
    
    ```
    /* 强制使用自定义的 MY_TABLE 变量来替代原有的 Table 入参 */
    <<manager.Place_Order.Table>> MY_TABLE
    ```
    

### 3. Environment User Code (环境用户代码)

- **功能区别**：最适合处理与整个测试框架 (Harness) 相关的全局动态操作，而不是针对单一函数的局部逻辑。
- **核心用途**：
    - 从外部文件（如 `.csv` 或 `.txt`）动态读取环境初始化数据。
    - 调用特定硬件驱动的初始化程序或数据库连接程序。
    - 基于动态条件分配和验证复杂的数据对象。
- **代码 Demo**： 这段代码不局限于单行，而是可以在特定的生命周期阶段（例如 `Harness Init`）写入任意 C/C++ 逻辑。
    
    ```
    /* Harness Init 阶段：打开日志文件并初始化全局状态 */
    FILE *fp = fopen("test_run.log", "w");
    if(fp != NULL) {
        fprintf(fp, "VectorCAST Test Environment Started.\n");
        fclose(fp);
    }
    g_system_ready = 1;
    ```
    

### 4. Unit User Code (单元用户代码 - 包含 Prefix 和 Appendix)

- **功能区别**：直接以物理方式将代码文本插入（附加）到被测源码文件（UUT）的**最顶端 (Prefix)** 或 **最末尾 (Appendix)**，并在编译时被视为 UUT 的一部分。
- **核心用途**：
    - **Unit Prefix (前缀)**：常用于通过 `#undef` 强行取消 UUT 中某些不应在测试环境中生效的宏定义，或重定义内部私有宏。
    - **Unit Appendix (后缀)**：常用于为 UUT 中的抽象类 `#include` 一个具体的子类实现文件（Concrete Subclass），从而让抽象类可以被实例化测试。
- **代码 Demo**：
    
    ```
    /* Unit Prefix Demo：拦截死循环宏或替换宏定义 */
    #undef MAX_LOOP_LIMIT
    #define MAX_LOOP_LIMIT 10
    
    /* Unit Appendix Demo：引入测试用的具体子类实现 */
    #include "mock_subclass.hxx"
    ```
    

---

## 深度拆解：Environment User Code 的 13 个执行阶段

Environment User Code 的强大之处在于 VectorCAST 提供了 13 个极其精确的生命周期钩子（Hooks）。在执行测试时，框架会严格按照以下顺序执行注入的代码：

1. **Header (头文件声明区)**：
    - _执行时机与用途_：作为代码通常出现在源文件最顶部。主要用于编写 `#include` 外部头文件指令或声明 `#define` 宏定义。
2. **Data (数据声明区)**：
    - _执行时机与用途_：出现于源文件的顶部区域。用于提供环境用户代码所需的类型定义、全局对象声明以及附加的子程序/支撑函数定义。
3. **Harness Init (测试框架初始化)**：
    - _执行时机与用途_：在整个测试框架（Test Harness）执行的最开始，**仅运行一次**。常用于调用硬件或操作系统的全局初始化例程。
4. **Test Case Init (测试用例初始化)**：
    - _执行时机与用途_：在测试用例的数据被加载完毕后，**立即在每次调用 UUT 之前执行**。这是为特定测试用例重置动态数据（如清零计数器或复位状态机）的最佳位置。
5. **UUT Timer Start (UUT 计时器启动)**：
    - _执行时机与用途_：在刚刚要切入调用 UUT 业务代码之前执行。通常专门用于启动性能测试的计时器。
6. **UUT Timer Stop (UUT 计时器停止)**：
    - _执行时机与用途_：在刚刚从 UUT 函数返回之后立即执行。通常用于停止性能测试的计时器。
7. **Stub Entry (桩函数入口)**：
    - _执行时机与用途_：在进入被打桩的子程序时执行，位于 Configure Stubs 的 Beginning User Code 之后。
8. **Stub Processing (桩函数处理)**：
    - _执行时机与用途_：这是所有桩函数的**公共处理代码**，每次调用**任何**桩函数时都会被执行。可以用来统计所有桩函数被调用的总次数。
9. **Stub Exit (桩函数出口)**：
    - _执行时机与用途_：在退出被打桩的子程序时执行，位于 Configure Stubs 的 Ending User Code 之前。
10. **Test Case Term (测试用例终止)**：
    - _执行时机与用途_：在 UUT 将控制权返回给测试框架后立即执行。主要用于在单个测试用例结束后检查受影响的数据流或执行针对单个用例的内存清理。
11. **Harness Term (测试框架终止)**：
    - _执行时机与用途_：在整个测试框架执行完毕且即将彻底退出前，**仅执行一次**。通常用于将最终的所有测试数据保存到外部文件或执行全局内存释放清理。
12. **Additional Unit Specs (附加单元规格)**：
    - _执行时机与用途_：用于编写额外添加到测试框架中的附加独立代码单元的**规格声明部分**（例如头文件声明）。
13. **Additional Unit Bodies (附加单元实现)**：
    - _执行时机与用途_：用于编写额外添加到测试框架中的附加独立代码单元的**实体实现部分**。例如，您可以将非交付业务代码但测试必需的复杂打印工具或模拟器引擎作为独立的源文件实现在此处。