## 1. 常用命令及参数集合（manage、clicast）
VectorCAST 提供了两套核心的命令行接口 (CLI) 工具：**`clicast`**（针对单个测试环境）和 **`manage`**（针对包含多个环境的企业级 Manage 工程）。

### 0、工程根目录生成ut environment 的reports
方案一：使用 `manage` 命令代理传递 `clicast` 参数（最推荐）

您不需要切换目录，直接在工程根目录使用专门的 `manage` 命令行工具，配合 `--clicast-args` 参数将原本的 `clicast` 指令“透传”给指定的子环境。 **命令格式：** `manage -p <您的工程名> -c <编译器节点名> -e <env> --clicast-args Reports Custom Full <outputfile>` _(注：这种方式会自动定位到该环境的实际物理子目录并在其中生成报告)_。

以下是这两套 CLI 中最常用的命令及参数说明：

### 一、 通用基础参数

在执行具体动作前，通常需要使用以下参数来限定命令的作用范围：

- **`-e <env>`**：指定当前操作的单个测试环境名称（仅限 `clicast`）。
- **`-u <unit>` / `-s <sub>` / `-t <testcase>`**：进一步将操作范围缩小到特定的被测单元 (unit)、子程序 (subprogram) 或单个测试用例 (testcase)。
- **`-lc`**：在不依赖特定环境时执行全局命令（如配置参数、利用统配符建环境等），全称为 License Checkout。
- **`-p <project>`**：指定当前操作的 VectorCAST Manage 工程名称（仅限 `manage`）。
- **`--level=<层级>`**：指定 `manage` 命令作用的工程树层级，例如 `--level=VectorCAST_MinGW_C++/Configuration_01`。

---

### 二、 `clicast` 常用命令（单环境操作）

**1. 环境构建与重构**

- **`clicast -lc ENvironment SCript Run <scriptfile>`**： 从指定的脚本文件（如 `.env`）从零开始构建一个新的测试环境。
- **`clicast -e <env> ENvironment RE_Build`**： 完全重构一个已经存在的环境（会备份旧环境、重新解析源码并重新加载用例）。
- **`clicast -e <env> ENvironment RECompile Auto`**： 自动重新编译并链接现有的测试驱动代码（适用于源码宏定义改变等情况）。

**2. 测试用例执行**

- **`clicast -e <env> EXecute Batch`**： 以批处理模式一次性执行环境（或指定单元/子程序）下的所有测试用例。
- **`clicast -e <env> -u <unit> -s <sub> -t <testcase> EXecute Run`**： 精准执行某一个特定的测试用例。

**3. 报告生成**

- **`clicast -e <env> REports Custom MAnagement <outputfile>`**： 生成测试用例管理报告 (Management Report)，包含执行状态通过率、度量指标及覆盖率汇总。
- **`clicast -e <env> REports Custom Coverage <outputfile>`**： 生成聚合覆盖率报告 (Aggregate Coverage Report)，内含带颜色高亮的源码覆盖详情。
- **`clicast -e <env> REports Custom Full <outputfile>`**： 生成全量报告 (Full Report)，将覆盖率、度量、执行结果及用例数据合并在一份报告中。

**4. 工具配置**

- **`clicast -lc option <OPTION_NAME> <Value>`**： 设置 VectorCAST 的底层配置选项。例如，使用 `clicast -lc option COVERAGE_TYPE STATEMENT+BRANCH` 可将默认覆盖率类型设为语句+分支覆盖。

---

### 三、 `manage` 常用命令（多环境工程操作）

**1. 自动化批量构建与执行**

- **`manage -p <project> --build-execute`**： 一键自动构建或重构工程下的所有环境，并执行所有用例。
- **`manage -p <project> --build-execute --incremental`**： **增量**构建与执行。基于 Change-Based Testing (CBT) 机制，系统会自动扫描源码变更，仅重新编译被影响的环境并只执行受到代码更改影响的测试用例。

**2. 工程级报告导出**

- **`manage -p <project> --full-status=<文件名.html>`**： 生成包含整个项目大盘的综合状态报告，展示每个环境的构建状态、用例执行通过率及各覆盖率类型的汇总百分比。
- **`manage -p <project> --create-report=<报告类型> --output=<输出路径>`**： 针对整个项目或特定 `--level` 生成指定类型的详细报告，支持的类型包括 `aggregate` (聚合覆盖率)、`metrics` (度量报告)、`environment` 等。

_**它支持的合法类型（枚举值）包括**_
- _**aggregate**_：聚合覆盖率报告 (Aggregate Coverage Report)
- **metrics**：度量报告 (Metrics Report)
- **csv_metrics**：CSV格式的度量报告 (CSV Metrics Report)
- **environment**：环境覆盖率报告 (Environment Coverage Report)
- **original_source**：原始源码覆盖率报告 (Original Source Coverage Report)
- _**function_call**_：函数调用报告 (Function Call Report)

**3. 批处理脚本执行**

- **`manage -p <project> --script <script_file.bat>`**： ==执行一个包含==多条 `manage` 命令的脚本文件。由于只需要加载一次庞大的测试工程，此命令能显著提升多指令连续执行（如 CI/CD 流水线中连续构建和导出报告）的运行速度。