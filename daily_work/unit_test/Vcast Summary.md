> [!abstract] VectorCAST 核心知识库 本笔记基于VectorCAST的底层运行机制、环境配置逻辑以及实际工程开发流程整理而成。旨在帮助开发者快速建立对VectorCAST单元测试与系统测试的全局认知，并掌握从工程创建到报告生成的标准操作规范。

---

### 一、 核心原理与类比说明

#### 1. 单元测试环境 vs 系统测试环境

为了更直观地理解两种测试环境的定位，我们可以用**“造车与试车”**来进行类比：

- **单元测试环境（Unit Test Environment）**：**“把汽车的方向盘、刹车单独拆下来，连上测试仪单独测。”**
    - **原理**：利用自动生成的测试驱动（Driver）和桩函数（Stubs）作为“假壳子”，将目标代码（单个或几个文件）与整个软件系统**完全剥离并隔离**。
    - **目的**：纯粹地验证局部代码的逻辑处理、边界条件是否正确。
- **系统测试环境（System Test Environment）**：**“把所有的零件拼成一辆完整的汽车，开到马路上实际跑跑看。”**
    - **原理**：不再生成驱动和桩代码，而是通过专门的Python脚本（如 `system_tests.py`）接管整个应用程序，将真实软件编译为整体可执行程序。
    - **目的**：针对集成后的完整应用进行自动化测试、收集系统级代码覆盖率，并利用基于变更的测试（CBT）进行光速回归验证。

#### 2. 用例执行与局部编译机制（ELF/EXE）

- **统一打包构建**：VectorCAST并不会为每个用例单独编译可执行文件。在局部环境构建（Build）时，它会将该环境下选定的被测源码（UUT）、测试驱动（Driver）和桩函数（Stubs）一次性编译并链接成**唯一的一个测试可执行文件（ELF/EXE）**。
- **数据与逻辑分离**：所有的测试用例数据（输入值、期望值）被统一保存在底层的 `.tst` 测试脚本文件中。
- **批处理执行（Batch Execute）**：执行测试时，工具调用该唯一的ELF文件，并通过批处理模式连续加载 `.tst` 文件中的用例数据进行批量运行。
- **增量优势**：**如果仅修改了用例的输入或期望值，完全不需要重新编译C代码**，VectorCAST只需增量执行受影响的脚本即可，极大节省时间。只有修改了C源码，才会重新插桩并编译这个ELF文件。

#### 3. 外部依赖与自动打桩原理（Auto-Stubbing）

- **原理**：面对嵌套极深的外部依赖，**不需要**将所有底层相关源文件（`.c/.cpp`）加入测试环境。只要将依赖的**头文件（`.h`）路径**加入工程的搜索目录（Search Directories），VectorCAST就能提取外部函数的“原型（Prototypes）”，自动生成桩函数接管并隔离外部调用。
- **干预**：开发者直接在界面上对生成的桩函数配置模拟的返回值，即可人为控制外部依赖行为，实现隔离测试。

---

### 二、 工程组成架构 (Composition)

VectorCAST工程在物理与逻辑上具有极强的结构性，整体工程信息保存在一个项目工作区中。

#### 1. 逻辑节点树 (Project Tree)

工程在界面左侧呈现为自上而下的五层树状节点：

1. **Project Node（项目根节点）**：代表整个工程体系。
2. **Compiler Node（编译器节点）**：挂载特定的编译器和目标板配置选项（如GCC、MinGW等）。
3. **Test Suite Node（测试套件节点）**：用于对拥有相同配置（如覆盖率类型、宏定义）的测试环境进行封装和分组。
4. **Group Node（分组节点）**：概念上的文件夹，纯粹用于组织底层环境，本身不带独立配置。
5. **Environment Node（环境节点）**：最底层的实际测试执行单元（如包含具体C文件的测试实例）。

#### 2. 物理文件结构

核心文件存放在工程目录及同名的 `Environment Directory` 下：

- **`project.vcm`**：工程XML配置文件，记录整体逻辑树和结构。
- **`.cfg`**：局部环境的编译器和工具具体配置选项文件。
- **`.env`**：测试环境构建脚本，记录选中的源文件等信息。
- **`.mfg`**：VectorCAST工程管理层的配置选项文件。
- **`.tst`**：测试用例脚本文件，保存所有测试数据的输入与期望值。

---

### 三、 核心名词解释 (Glossary)

- **UUT (Unit Under Test)**：被测单元。即当前测试环境中，你真正需要测试其逻辑的核心C/C++源文件。
- **Driver (测试驱动)**：VectorCAST自动生成的包含 `main()` 的代码，负责调用UUT并传递 `.tst` 中的输入数据。
- **Stubs (桩函数)**：替代UUT调用的外部缺失函数的“假函数”。 VectorCAST根据头文件自动生成，用于隔离外部依赖。
- **CBT (Change-Based Testing, 基于变更的测试)**：一种极其节省时间的机制。当源码发生微小修改时，工具能自动计算并识别出受影响的测试用例子集，并**仅重新运行这部分测试**。
- **Configuration Inheritance (配置继承)**：子节点（如环境节点）会自动继承父节点（如编译器、Test Suite）的属性（如搜索路径、覆盖率模式），避免重复配置。

---

### 四、 开发与操作全步骤（从创建到报告）

以下是C代码单元测试的标准落地流程：

#### Step 1: 创建空工程 (Create Empty Project)

- 点击 `File => New => VectorCAST Project => Empty Project`。
- 输入工程名（Project Name），选择底层编译器（如 VectorCAST_MinGW_C），并设置基础工作目录（Base Directory），点击 `Create`。

#### Step 2: 配置编译器与复用 (Configure & Reuse)

- **配置搜索路径**：右键编译器或Test Suite节点 -> `Open Configuration`，展开 `Manage`，在 `Source Directories` 中添加源码路径和**依赖的头文件路径**（必须添加头文件路径以支持自动打桩）。
- **配置复用（若需）**：若需复用外部已有配置，可右键工程根节点选择 `Add Compiler => From Configuration File` 导入现成的 `.CFG` 文件。若需在同工程内复用，可直接对配置节点进行 `Copy/Paste` 或 `Duplicate` 操作，随后通过 `Clear Single Option` 单独修改其源文件目录。

#### Step 3: 创建测试环境 (Create Unit Test Environment)

- 在 `Group` 节点上右键 -> `Create Unit Test Environment => Interactive` 启动向导。
- 输入环境名称（Environment Name）。（注：如果编译器配置无误，向导步骤1的选择编译器会自动继承并完成）。
- 进入 `Choose UUTs & Stubs` 步骤，仅选择你需要测试的目标C文件作为 **UUT**。

#### Step 4: 构建环境 (Build)

- 点击向导右下角的 `Build`。
- 此时VectorCAST会在后台解析被测代码与头文件，生成驱动代码和Stubs，并调用编译器打包成一个测试可执行文件（ELF/EXE）。完成后自动打开环境视图。

#### Step 5: 开发测试用例 (Add Test Cases)

- 在环境视图（Environment View）中，展开目标函数，右键选择 `Insert Test Case`。
- 在界面表格中，针对入参填入**输入值（Input Values）**，针对输出或被触发的桩函数填入**期望值（Expected Values）**。

#### Step 6: 增量编译与执行 (Incremental Build/Execute)

- 右键对应的节点（可以是单用例、环境节点或整个工程），选择 `Build/Execute => Incremental` 或直接点击 `Execute`。
- 系统会进行批处理执行。如果仅修改了用例，工具直接执行ELF文件与数据比对；若修改了C源码，则仅重新插桩并编译受影响的部分（CBT机制）。

#### Step 7: 查看覆盖率与生成报告 (Generate Reports)

- **状态查看**：执行完毕后，主界面的状态面板（Status Panel）将更新对应节点的通过/失败状态（绿/红/黄）及覆盖率百分比（Statement, Branch, MC/DC）。
- **生成报告**：右键相关节点 -> `Reporting => Full Status Report` 生成完整的包含配置数据及度量指标的HTML总结报告（Manage Report）。也可通过 `Files` 选项卡右键特定代码文件，生成代码级别的 `Metrics Report` 或 `Original Source Coverage Report`。