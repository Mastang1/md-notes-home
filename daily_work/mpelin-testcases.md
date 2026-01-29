
基于对文档 **"RTD_LIN_LLCE_TS.pdf"** 的分析，以下是S32 LIN_LLCE驱动的详细测试用例列表。这些用例涵盖了初始化、状态管理、数据传输、网络管理、错误处理及集成测试等方面。

为了清晰展示，测试用例已按功能类别分类整理。

### 1. 初始化与配置测试 (Initialization & Configuration)

| 测试用例 ID               | 测试功能与目标                                                 | 关键测试点/步骤                                                                |
| :-------------------- | :------------------------------------------------------ | :---------------------------------------------------------------------- |
| **LIN_TC_0000**       | **Dummy TC**用于非执行测试序列或作为模板，仅测试编译通过性。                    | 1. 验证编译是否通过。2. 这是一个用于仅编译或EPD测试的空用例。                                     |
| **LIN_TC_0001**       | **配置指针检查**验证传递给 `Lin_43_LLCE_Init` 的配置指针有效性。            | 1. 传入非NULL指针，DET应报错 `INVALID_POINTER`。2. 传入NULL指针（针对特定Variant），验证DET行为。 |
| **LIN_TC_0022**       | **初始化后驱动状态检查**检查调用 `Lin_43_LLCE_Init` 后驱动、通道的状态机及静态变量值。 | 1. 初始化前验证状态。2. 初始化后验证Lin变量和状态机是否正确。-                                    |
| **LIN_TC_0023**       | **通道初始化检查**验证多通道配置下的寄存器值和状态机。                           | 1. 调用 `Lin_43_LLCE_Init` 后检查寄存器值是否正确。                                   |
| **LIN_TC_0024**       | **通信准备检查**验证数据通信前的通道初始化及唤醒信号处理。                         | 1. 重置驱动状态。2. 强制唤醒信号并验证唤醒标志。3. 重新初始化并验证 `EcuM_SetWakeupEvent` 是否被调用。     |
| **LIN_TC_1500_MULTI** | **配置生成检查**验证EB Tresos配置文件的生成是否成功或按预期报错。                 | 1. 验证生成的配置是否匹配预期的错误或成功状态。                                               |
| **TC_INIT**           | **Init函数功能测试**验证 `Lin_43_LLCE_Init` 在正常参数下的行为。          | 1. 传入有效配置指针，验证函数返回 `E_OK` 且无DET/DEM错误。-                                 |
| **TC_INIT_NULL_PTR**  | **Init空指针测试**验证 `Lin_43_LLCE_Init` 传入NULL指针时的错误报告。      | 1. 传入NULL指针，验证DET报告 `LIN_43_LLCE_E_INVALID_POINTER` (PB variant)。-      |
| **TC_FNC_LIN_00101**  | **版本信息检查**验证LIN模块的版本信息。                                 | 1. 调用 `GetVersionInfo` 并校验主/次版本号及厂商ID。                                  |
| **TC_FNC_LIN_00102**  | **初始化与去初始化**验证驱动初始化及通道去初始化的状态。                          | 1. 检查通道状态，去初始化后应返回 `LIN_CH_SLEEP`。                                      |

### 2. 状态管理与API行为 (State Management & API Behavior)

| 测试用例 ID              | 测试功能与目标                                | 关键测试点/步骤                                                                            |
| :------------------- | :------------------------------------- | :---------------------------------------------------------------------------------- |
| **LIN_TC_001_CE**    | **API原型检查**验证API函数原型及其在编译时的配置开关。       | 1. 调用 `Lin_43_LLCE_GetVersionInfo` 等API验证是否存在。                                      |
| **LIN_TC_0016**      | **超时机制检查**验证 `while` 循环具有超时逃生机制以避免死循环。 | 1. 禁用主机中断。2. 调用 `GoToSleep`。3. 验证是否触发 `LIN_43_LLCE_E_TIMEOUT` 错误。                   |
| **LIN_TC_0018**      | **睡眠状态下的非法调用**验证当通道处于睡眠状态时调用特定API是否报错。 | 1. 在睡眠状态下调用 `SendFrame`, `GoToSleep` 等。2. 验证DET报告 `LIN_43_LLCE_E_STATE_TRANSITION`。 |
| **TC_API_LIN_00100** | **API功能验证**综合验证LIN驱动API的功能原型和基本行为。     | 1. 获取版本、打开接口、发送帧、休眠、唤醒检测等全流程调用。                                                     |
| **LIN_TC_2000**      | **中断标志清理**验证在启用中断前，挂起的中断标志已被清除。        | 1. 验证 `Lin_43_LLCE_Init` 调用前后中断标志的状态，确保无伪中断。                                        |
| **TC_FNC_LIN_00218** | **从模式API可用性**验证在从模式下某些API是否不可用或受限。     | 1. 在从模式下调用发送帧、休眠等API并检查状态。                                                          |

### 3. 数据传输与接收 (Data Transmission & Reception)

| 测试用例 ID                  | 测试功能与目标                                                   | 关键测试点/步骤                                                 |
| :----------------------- | :-------------------------------------------------------- | :------------------------------------------------------- |
| **LIN_TC_0017**          | **快速帧间隙丢数据测试**验证当不同通道帧间隙小于6ms时是否丢失数据。                     | 1. 无延迟发送两个头帧。2. 验证响应帧是否被正确接收，中断标志是否被意外清除。                |
| **LIN_TC_0025**          | **多SDU数据收发**验证使用不同SDU（及不同校验和类型）发送/接收数据的状态机。               | 1. 转换数据并发送帧（从节点响应模式）。2. 验证发送成功及MAF接收缓冲区数据。-              |
| **LIN_TC_0034**          | **波特率设置数据测试**验证特定波特率设置下的数据收发。                             | 1. 发送LIN帧，验证传输完成及数据正确性。-                                 |
| **LIN_TC_0038**          | **帧传输中止（新帧）**验证当请求新帧时，当前正在进行的帧传输是否被中止。                    | 1. 发送第一帧后立即发送第二帧。2. 验证仅接收到第二帧的数据。                        |
| **LIN_TC_0039**          | **帧传输中止（休眠）**验证当请求 `GoToSleep` 时，当前传输是否被中止。               | 1. 发送帧时调用 `GoToSleep`。2. 验证未收到原数据帧，总线进入睡眠。               |
| **LIN_TC_0063**          | **4主控制器支持**验证LIN驱动支持4个Master控制器。                          | 1. 对多个通道调用状态获取、休眠、唤醒API。                                 |
| **LIN_TC_0125**          | **大数据量压力测试**验证大量不同ID和SDU的数据收发稳定性。                         | 1. 循环转换并发送大量不同ID的数据帧。2. 验证每次传输的成功状态及数据一致性。-              |
| **LIN_TC_1001**          | **从节点唤醒后发送**验证作为从节点在唤醒请求后发送头和响应。                          | 1. 发送唤醒脉冲。2. 进入操作状态后发送帧并验证数据。-                           |
| **LIN_TC_1007**          | **传输中休眠中止**验证调用 `GoToSleepInternal` 能中止正在进行的传输。           | 1. 传输Header期间调用休眠。2. 验证传输未完成且通道进入睡眠。                     |
| **LIN_TC_4002**          | **空帧传输检查**验证发送Null frame的处理。                              | 1. 调用 `Lin_SendFrame` 发送空帧并验证成功。                         |
| **TC_FNC_LIN_00200**     | **发送接收顺序与中止**验证 `SendHeader` 和 `SendResponse` 的调用顺序及中止功能。 | 1. 发送帧并验证接收。2. 验证新请求会中止当前传输。-                            |
| **TC_FNC_LIN_00201**     | **综合数据收发**验证主/从响应模式及不同校验和模型下的数据收发。                        | 1. 测试Master响应模式发送。2. 测试Slave响应模式接收。3. 验证数据完整性。           |
| **TC_FNC_LIN_00202**     | **传输终止功能**验证正在进行的传输被正确终止。                                 | 1. 发送帧1，随后发送帧2。2. 验证数据接收的正确性及顺序。                         |
| **TC_FNC_LIN_00207**     | **从节点响应模式状态**验证从节点响应模式下的驱动状态。                             | 1. 发送支持PID的头，验证驱动状态。2. 发送不支持PID的头，验证忽略行为。                |
| ~~**TC_FNC_LIN_00208**~~ | ~~**主节点响应模式状态**验证主节点响应模式下的驱动状态及错误干扰处理。~~                  | ~~1. 正常发送验证。2. 注入干扰（SDU损坏、停止位损坏）。3. 验证驱动状态（TX_ERROR等）。~~ |
| **TC_FNC_LIN_00215**     | **从节点接收头/发送响应**验证从节点接收Header并发送Response的功能。               | 1. 转换PDU，发送Header。2. 验证从节点发送响应并被MAF接收。                   |
| **TC_FNC_LIN_00216**     | **从节点接收头/接收响应**验证从节点接收Header并接收Response的功能。               | 1. 发送Header，验证 `LinIf` 回调及接收字节数。                         |
| **TC_FNC_LIN_00219**     | **从节点忽略Header**验证从节点收到不同PID时忽略Header和Response。            | 1. 发送不同PID的Header，验证无响应。                                 |
| **TC_FNC_LIN_00220**     | **从节点功能验证**验证从节点唤醒、休眠及响应发送全流程。                            | 1. 验证休眠/唤醒状态流转。2. 验证 `LinIf_TxConfirmation` 回调。          |

### 4. 睡眠、唤醒与网络管理 (Sleep, Wakeup & Network Management)

|测试用例 ID|测试功能与目标|关键测试点/步骤|
|:--|:--|:--|
|**LIN_TC_0021**|**睡眠中CheckWakeup**验证在睡眠状态且开启唤醒支持时调用 `CheckWakeup` 的行为。|1. 通道休眠后调用 `CheckWakeup`。2. 验证唤醒通知变量状态。|
|**LIN_TC_0026**|**主休眠从唤醒**主节点发送休眠命令，从节点发送唤醒脉冲。|1. 主节点发送GoToSleep帧。2. 等待总线休眠，从节点发送唤醒脉冲，验证通道恢复。|
|**LIN_TC_0027**|**Internal休眠后唤醒**使用 `GoToSleepInternal` 休眠后等待唤醒脉冲。|1. 调用内部休眠，发送唤醒脉冲，检查唤醒事件设置。|
|**LIN_TC_0028**|**休眠后发送唤醒帧**通道休眠后，主节点发送唤醒脉冲。|1. 休眠后，主节点发送唤醒脉冲。2. 验证通道恢复操作状态并能发送数据。|
|**LIN_TC_0029**|**集群休眠唤醒校验**LIN集群休眠后接收唤醒，调用 `CheckWakeup`。|1. 休眠 -> 接收唤醒脉冲 -> 调用 `CheckWakeup` -> 验证通知变量。|
|**LIN_TC_0030**|**通道休眠唤醒校验**通道Internal休眠后接收唤醒，调用 `CheckWakeup`。|1. 内部休眠 -> 接收唤醒 -> 调用 `CheckWakeup` -> 验证通知。|
|**LIN_TC_0032**|**休眠中WakeupInternal**通道休眠时调用 `WakeupInternal`。|1. 休眠 -> 发送唤醒脉冲 -> 调用 `WakeupInternal` -> 获取状态。|
|**LIN_TC_1008**|**休眠后CheckWakeup**通道休眠后调用 `CheckWakeup` 评估所有通道。|1. 休眠 -> 发送唤醒脉冲 -> 验证唤醒事件变量被设置。|
|**TC_FNC_LIN_00103**|**独立休眠请求**验证各通道可独立接受休眠请求。|1. 分别对通道1和通道2进行休眠和唤醒，互不影响。|
|**TC_FNC_LIN_00203**|**休眠唤醒数据交互**验证休眠唤醒后的数据收发功能。|1. 休眠 -> 从节点唤醒 -> 验证主/从响应模式的数据交互。|
|**TC_FNC_LIN_00204**|**Internal休眠唤醒交互**验证内部休眠唤醒后的数据收发。|1. 内部休眠 -> 唤醒 -> 验证数据交互。|
|**TC_FNC_LIN_00206**|**唤醒验证功能**验证 `CheckWakeup` 调用回调函数（针对唤醒检测禁用情况）。|1. 唤醒检测禁用时调用 `CheckWakeup`，验证通道不能唤醒。|
|**TC_FNC_LIN_00210**|**GoToSleep/Wakeup功能**验证API发送休眠命令及唤醒脉冲的功能。|1. 发送休眠命令并验证MAF接收。2. 发送/接收唤醒脉冲并验证状态。|
|**TC_FNC_LIN_00211**|**Internal休眠/Wakeup功能**验证内部休眠及唤醒脉冲发送。|1. 进入内部休眠，发送/接收唤醒脉冲，验证唤醒事件。|
|**TC_WAKEUP_OK**|**Wakeup功能测试**验证正常状态下的 `Lin_WakeUp` 功能。|1. 验证 `Lin_43_LLCE_Wakeup` 调用返回 `E_OK`。|
|**TC_GOTOSLEEP**|**GoToSleep功能测试**验证正常状态下的 `Lin_GoToSleep` 功能。|1. 验证 `Lin_43_LLCE_GoToSleep` 调用返回 `E_OK`。|

### 5. 错误处理与负面测试 (Error Handling & Negative Tests)

| 测试用例 ID                                   | 测试功能与目标                                            | 关键测试点/步骤                                     |     |
| :---------------------------------------- | :------------------------------------------------- | :------------------------------------------- | --- |
| **TC_GET_VERSION_INFO_NULL_PTR**          | **版本信息空指针**验证传入NULL指针时的DET报错。                      | 1. 验证DET报告 `LIN_43_LLCE_E_PARAM_POINTER`。    |     |
| **TC_CHECKWAKEUP_NI**                     | **未初始化CheckWakeup**未初始化时调用 `CheckWakeup`。          | 1. 验证DET报告 `LIN_43_LLCE_E_UNINIT`。           |     |
| **TC_SENDFRAME_NI**                       | **未初始化SendFrame**未初始化时调用 `SendFrame`。              | 1. 验证DET报告 `LIN_43_LLCE_E_UNINIT`。           |     |
| **TC_GOTOSLEEP_NI**                       | **未初始化GoToSleep**未初始化时调用 `GoToSleep`。              | 1. 验证DET报告 `LIN_43_LLCE_E_UNINIT`。           |     |
| **TC_GOTOSLEEPINTERNAL_NI**               | **未初始化Internal休眠**未初始化时调用 `GoToSleepInternal`。     | 1. 验证DET报告 `LIN_43_LLCE_E_UNINIT`。           |     |
| **TC_WAKEUP_NI**                          | **未初始化Wakeup**未初始化时调用 `Wakeup`。                    | 1. 验证DET报告 `LIN_43_LLCE_E_UNINIT`。           |     |
| **TC_WAKEUPINTERNAL_NI**                  | **未初始化Internal唤醒**未初始化时调用 `WakeupInternal`。        | 1. 验证DET报告 `LIN_43_LLCE_E_UNINIT`。           |     |
| **TC_GETSTATUS_NI**                       | **未初始化GetStatus**未初始化时调用 `GetStatus`。              | 1. 验证DET报告 `LIN_43_LLCE_E_UNINIT`。           |     |
| **TC_INIT_TRANSITION**                    | **重复初始化**在已初始化状态下再次调用 `Init`。                      | 1. 验证DET报告 `LIN_43_LLCE_E_STATE_TRANSITION`。 |     |
| **TC_GOTOSLEEP_INVALID_CHANNEL**          | **GoToSleep无效通道**调用 `GoToSleep` 使用无效通道ID。          | 1. 验证DET报告 `LIN_43_LLCE_E_INVALID_CHANNEL`。  |     |
| **TC_GOTOSLEEP_STATE_TRANSITION**         | **休眠状态转换错误**在休眠状态下调用 `GoToSleep`。                  | 1. 验证DET报告 `LIN_43_LLCE_E_STATE_TRANSITION`。 |     |
| **TC_GOTOSLEEPINTERNAL_INVALID_CHANNEL**  | **Internal休眠无效通道**调用 `GoToSleepInternal` 使用无效通道ID。 | 1. 验证DET报告 `LIN_43_LLCE_E_INVALID_CHANNEL`。  |     |
| **TC_GOTOSLEEPINTERNAL_STATE_TRANSITION** | **Internal休眠转换错误**在休眠状态下调用 `GoToSleepInternal`。    | 1. 验证DET报告 `LIN_43_LLCE_E_STATE_TRANSITION`。 |     |
| **TC_WAKEUP_INVALID_CHANNEL**             | **Wakeup无效通道**调用 `Wakeup` 使用无效通道ID。                | 1. 验证DET报告 `LIN_43_LLCE_E_INVALID_CHANNEL`。  |     |
| **TC_WAKEUPINTERNAL_INVALID_CHANNEL**     | **Internal唤醒无效通道**调用 `WakeupInternal` 使用无效通道ID。    | 1. 验证DET报告 `LIN_43_LLCE_E_INVALID_CHANNEL`。  |     |
| **TC_WAKEUP_STATE_TRANSITION**            | **Wakeup状态转换错误**在非休眠状态下调用 `Wakeup`。                | 1. 验证DET报告 `LIN_43_LLCE_E_STATE_TRANSITION`。 |     |
| **TC_WAKEUPINTERNAL_STATE_TRANSITION**    | **Internal唤醒转换错误**在非休眠状态下调用 `WakeupInternal`。      | 1. 验证DET报告 `LIN_43_LLCE_E_STATE_TRANSITION`。 |     |
| **TC_GETSTATUS_INVALID_CHANNEL**          | **GetStatus无效通道**调用 `GetStatus` 使用无效通道ID。          | 1. 验证DET报告 `LIN_43_LLCE_E_INVALID_CHANNEL`。  |     |
| **TC_GETSTATUS_NULL_PTR**                 | **GetStatus空指针**调用 `GetStatus` 传入无效指针。             | 1. 验证DET报告 `LIN_43_LLCE_E_PARAM_POINTER`。    |     |
| **TC_CHECKWAKEUP_INVALID_CHANNEL**        | **CheckWakeup无效通道**调用 `CheckWakeup` 使用无效通道ID。      | 1. 验证DET报告 `LIN_43_LLCE_E_INVALID_CHANNEL`。  |     |
| **TC_SENDFRAME_NULL_PTR**                 | **SendFrame空指针**调用 `SendFrame` 传入无效PDU指针。          | 1. 验证DET报告 `LIN_43_LLCE_E_PARAM_POINTER`。    |     |
| **TC_SENDFRAME_TRANSITION**               | **SendFrame状态转换错误**在休眠状态下调用 `SendFrame`。           | 1. 验证DET报告 `LIN_43_LLCE_E_STATE_TRANSITION`。 |     |
| **TC_SENDFRAME_INVALID_CHANNEL**          | **SendFrame无效通道**调用 `SendFrame` 使用无效通道ID。          | 1. 验证DET报告 `LIN_43_LLCE_E_INVALID_CHANNEL`。  |     |
| **TC_FLT_LIN_00101**                      | **SendFrame超时**验证发送帧超时的错误报告。                       | 1. 触发超时条件，验证 `LIN_43_LLCE_E_TIMEOUT` 生产错误。   |     |
| **TC_FLT_LIN_00102**                      | **GoToSleep超时**验证休眠超时的错误报告。                        | 1. 触发休眠超时，验证 `LIN_43_LLCE_E_TIMEOUT` 生产错误。   |     |
| **TC_FLT_LIN_00103**                      | **GoToSleepInternal超时**验证内部休眠超时的错误报告。              | 1. 触发超时，验证 `LIN_43_LLCE_E_TIMEOUT`。          |     |

### 6. 协议错误注入测试 (Protocol Error Injection)

| 测试用例 ID              | 测试功能与目标                                                         | 关键测试点/步骤                                                                                |
| :------------------- | :-------------------------------------------------------------- | :-------------------------------------------------------------------------------------- |
| **TC_FNC_LIN_00209** | **从节点响应错误/无响应**验证从节点响应时的 `LIN_RX_ERROR` 和 `LIN_RX_NO_RESPONSE`。 | 1. 发送PID，验证无响应状态。2. 验证错误状态。                                                             |
| **TC_FNC_LIN_00214** | **主节点响应错误**验证主节点响应模式下的错误处理。                                     | 1. 注入Header Error干扰。2. 注入Data Error干扰。3. 验证驱动返回 `LIN_TX_HEADER_ERROR`, `LIN_TX_ERROR`。- |
| **TC_FNC_LIN_00217** | **从节点Header错误**验证从节点接收错误Header后向LinIf报告。                        | 1. 设置ID字节干扰，验证LinIf错误类型。2. 设置停止位干扰。                                                     |
| **TC_FNC_LIN_00221** | **从节点无响应错误**验证从节点丢失主节点响应时的 `LIN_ERR_NO_RESP`。                   | 1. 启用帧超时，从节点缺失响应，验证LinIf报错。                                                             |
| **TC_FNC_LIN_00222** | ==**ID奇偶校验错误**验证ID Parity Error时的 `LIN_ERR_HEADER`。==           | 1. 注入ID干扰，验证LinIf收到 `LIN_ERR_HEADER`。                                                   |
| **TC_FNC_LIN_00223** | ==**Header停止位错误**验证Header停止位错误时的 `LIN_ERR_HEADER`。==            | 1. 注入Header StopBit干扰，验证报错。                                                             |
| **TC_FNC_LIN_00224** | ==**响应停止位错误**验证数据停止位错误时的 `LIN_ERR_RESP_STOPBIT`。==              | 1. 注入Data StopBit干扰，验证报错。                                                               |
| **TC_FNC_LIN_00225** | ==**响应不完整错误**验证响应不完整时的 `LIN_ERR_INC_RESP`。==                    | 1. 发送不完整响应，验证LinIf报错。                                                                   |
| **TC_FNC_LIN_00226** | ==**同步场错误**验证Sync Field错误时的 `LIN_ERR_HEADER`。==                 | 1. 注入Sync Byte干扰，验证报错。                                                                  |

### 7. 集成与混合测试 (Integration & Hybrid)

|测试用例 ID|测试功能与目标|关键测试点/步骤|
|:--|:--|:--|
|**LIN_TC_H0001**|**LIN/CAN混合测试**验证LIN LLCE与CAN LLCE混合运行时的功能。|1. 执行LIN数据收发。2. 执行CAN数据收发。3. 验证两者均正常工作，互不干扰。-|
|**LIN_TC_H0018**|**LIN LLCE/MCAL混合测试**验证LIN LLCE与LIN MCAL混合使用时的错误处理。|1. 混合调用MCAL和LLCE接口（如 `Lin_GoToSleep` 和 `Lin_43_LLCE_SendFrame`）。2. 验证状态转换错误报告。|
