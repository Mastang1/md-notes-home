**一、 为所有代码增加全局宏定义的步骤**

要为工程中的所有代码添加全局宏定义，最有效的配置层级是在 **Compiler Node（编译器节点）** 中进行设置。因为VectorCAST采用自上而下的配置继承机制，设置在编译器节点（例如 `VectorCAST_MinGW_C`）的宏定义会被其下挂载的所有测试套件（Test Suite）和测试环境（Environment）自动继承，从而实现全局应用。

**具体操作步骤如下：**

1. 在项目视图（Project View）的逻辑树中，右键点击目标 **Compiler Node（编译器节点）**，选择 **Open Configuration（打开配置）**。
2. 在弹出的配置选项编辑器（Configuration Options Editor）中，展开 **C/C++ => Preprocessor/Compiler（预处理器/编译器）** 节点。
3. 找到 **Defined variables（宏定义变量）** 字段。
4. 在该字段中填入您需要全局应用的宏。例如，输入纯宏定义 `FEATURE_A` 或带值的宏 `FEATURE_A=1`。如果是多个宏，可以使用空格将其隔开。
5. 保存配置。**非常重要的一步**：由于宏定义是在代码预处理阶段决定编译分支的，您**必须**对工程或受影响的环境执行 **Rebuild Environment（重新构建环境）** 或 **Build/Execute => Full**。这样VectorCAST才会使用新宏重新解析、插桩并编译代码，使得隐藏的物理分支生效。

**二、 VectorCAST执行代码构建的流程层级与代码生成过程**

VectorCAST将测试用例、被测源码及底层工具链打包的构建过程，按先后顺序可划分为以下几个核心的流程层级：

1. **源码解析与依赖识别 (Parsing & Dependency Evaluation)** 当您触发构建时，VectorCAST首先会解析您选定的被测C/C++源文件（UUT）以及搜索目录中的头文件。它会提取出函数原型、全局变量和数据类型，并识别被测代码调用了哪些外部未包含的函数。
2. **自动生成代码 (Auto-Code Generation)** 基于上一步的解析结果，工具会在后台自动生成两部分核心 C/C++ 代码：
    - **测试驱动 (Test Driver)**：生成包含 `main()` 函数的驱动代码。它负责在执行时接管程序控制权，并读取底层 `.tst` 脚本中的测试用例数据（Input Values）传递给被测函数。
    - **桩函数 (Stubs)**：自动为被测代码调用但未在环境中提供源码的外部依赖生成“假函数”代码，实现物理级别的代码隔离。
3. **用户代码注入 (User Code Injection)** 如果您在 UI 的 User Code 标签页（如 Unit Appendix 单元附录）中编写了自定义的 C/C++ 代码或预处理指令（如 `#undef` 取消宏定义），VectorCAST 会在此时将这些代码片段直接注入到被测源文件或测试驱动的指定位置（例如文件最顶端）。
4. **代码插桩 (Instrumentation)** 如果环境配置了覆盖率指标（如 Statement, Branch, MC/DC），VectorCAST会在这一层级对被测源代码进行修改，在代码执行路径的各个关键逻辑节点自动插入覆盖率监控代码（即插桩计数器）。
5. **调用本地工具链编译 (Compilation)** VectorCAST通过其后台的 Jobs 调度器（Job Monitor），生成具体的命令行指令，调用您在工程中配置的本地 C/C++ 编译器（如 GCC, MinGW, Green Hills 等）。它将插桩后的被测源码、自动生成的测试驱动、桩函数以及注入的用户代码，统一编译为目标文件（如 `.o` 或 `.obj`）。
6. **链接生成可执行文件 (Linking)** 工具随后调用链接器，将上述所有的目标文件链接打包，形成**唯一的一个测试可执行文件 (Test Executable，例如 ELF 固件或 EXE 文件)**。在随后的执行阶段，VectorCAST 就是直接以批处理模式调用这个可执行文件来跑所有的测试用例。
7. **（可选）CBT 增量构建 (Change-Based Testing Incremental Build)** 如果是修改代码后的增量构建（Incremental Build），VectorCAST会先在第一层级评估源文件的变更影响，随后**仅对受代码修改影响的文件**执行重新插桩和重新编译，并复用其他未变更的目标文件进行重新链接，以此极大缩短构建时间。