在 VectorCAST 中，如果您希望在执行用例时将某个变量的值打印并显示在 UI 或最终的测试执行报告（Execution Report）中，主要可以通过以下三种标准方式来实现：

### 方法一：使用 `<<ANY>>` 关键字（最简单快捷，推荐用于常规参数）

如果您只想在报告中看到某个入参、返回值或全局变量的实际运行值，而不需要对其进行严格的 Pass/Fail 判断对比，可以直接在参数树（Parameter Tree）中使用 `<<ANY>>`。

- **操作步骤**：
    1. 在用例的 Parameter Tree 中找到您想打印的变量。
    2. 在该变量的 **Expected Values（期望值）** 列中，右键或通过下拉菜单选择（或手动输入） **`<<ANY>>`**。
- **效果**：当测试执行时，框架会自动捕获该变量的实际值并打印在 Execution Report 中，且由于期望值是“任意值”，所以该项比对永远会显示绿色的 `<match>`（通过），不会导致用例失败。

### 方法二：在 User Code 中使用 `vCAST_UC_WRITE_EXPECTED` 接口（适用于复杂自定义打印）

如果您在编写 **Expected User Code（预期用户代码）**，想要打印计算过程中的中间变量、或者附加自定义的文本标签和实际值到报告的 "User Code Expected Values" 区域，可以使用 VectorCAST 提供的专有内置函数 `vCAST_UC_WRITE_EXPECTED`。

- **函数原型**： `void vCAST_UC_WRITE_EXPECTED (const char *param, const char *name, int match, const char *actual);` _(参数说明：param为变量的标签名称；name为在报告中显示的自定义描述文本；match为布尔判断结果(真或假)；actual为格式化后的实际值字符串)_。
- **代码示例**： 双击参数打开 User Code 面板，在 Expected User Code 中编写如下代码：
    
    ```
    char actual_str;
    double return_value = <<uut.func.return>>;
    /* 将实际值格式化为字符串 */
    snprintf( actual_str, 20, "%lg", return_value );
    /* 强行输出到报告中 */
    vCAST_UC_WRITE_EXPECTED( "uut.func.return",
                             "自定义打印：验证返回值",
                             ((1.1 <= return_value) && (return_value <= 1.3)),
                             actual_str );
    ```
    

### 方法三：使用标准输出或 Probe Print API（直接追加到报告末尾）

如果您是在任意阶段的 User Code（如 Test Case Init、Input User Code）或 Probe Points（探针）中，您可以使用标准 C 语言的 `printf` 或者 VectorCAST 提供的专用打印宏，将信息直接输出。

- **专有打印宏**：为了保证在各种嵌入式目标板上都能安全捕获数据，推荐使用以下 VectorCAST 内置的探针打印函数：
    - `vcast_probe_print(char* message);`
    - `vcast_probe_print_int(int value);`
    - `vcast_probe_print_float(float value);`
    - `vcast_probe_print_unsigned(unsigned int value);`
- **⚠️ 关键配置要求**： 为了让这些通过 `printf` 或 `vcast_probe_print` 打印的内容能够显示在 Execution Report 中，您**必须**开启标准输出重定向：
    1. 点击菜单栏 **Tools => Options**。
    2. 切换到 **Execute（执行）** 选项卡。
    3. 勾选 **Redirect standard output（重定向标准输出）** 或 `STANDARD_OUTPUT` 设置为 Redirect。
- **效果**：测试执行后，所有被打印的内容（包括变量值和自定义字符串）都会被收集，并集中追加显示在 Execution Report 的最底部（Standard Output from Driver 区域）。