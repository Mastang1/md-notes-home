
根据您的要求，我基于《IPCF User Manual》第一章中的插图内容，为您生成了对应的 Mermaid 代码。这些代码可以在支持 Mermaid 的编辑器中渲染为图表，并导出为 SVG 格式插入 Word 文档。

为了确保 Word 兼容性，我使用了标准的流程图和时序图语法，未添加复杂的交互脚本。

### 1. IPCF 系统架构图 (对应 1.4 Architecture)

此图展示了 IPCF 驱动程序的层级结构，从顶层的客户应用到底层的硬件平台。

```mermaid
graph TD
    %% 节点定义
    App[Customer Application]

    subgraph IPCF_Driver [IPCF Driver Package]
        direction TB
        IpcShm[ipc-shm<br/>Shared Memory Generic Implementation]
        IpcOs[ipc-os<br/>OS Abstraction]
        IpcHw[ipc-hw<br/>HW Abstraction]
    end

    Hw[NXP HW Platform]

    %% 连接关系
    App <--> IpcShm
    IpcShm <--> IpcHw
    IpcShm <--> IpcOs
    IpcOs <--> IpcHw
    IpcHw <--> Hw

    %% 样式调整 (可选，用于增强黑白打印对比度)
    classDef box fill:#fff,stroke:#000,stroke-width:1px;
    class App,IpcShm,IpcOs,IpcHw,Hw box
```

### 2. 非托管通道数据流 (对应 1.5 Unmanaged Channel Data Flow)

此时序图展示了禁用缓冲区管理时（类似 POSIX 共享内存），应用程序直接拥有通道内存的通信流程。

```mermaid
sequenceDiagram
    participant App1 as OS1:App
    participant IPC1 as OS1:IPC
    participant IPC2 as OS2:IPC
    participant App2 as OS2:App

    Note over App1, IPC1: 初始化阶段
    App1->>IPC1: shm_acquire_buf(chan_id)
    IPC1-->>App1: return channel shm ptr

    Note over App1, App2: 数据传输阶段
    App1->>App1: Write data in SHM
    App1->>IPC1: shm_tx_unmanaged(chan_id)
    IPC1--)IPC2: Interrupt (Notify)
    IPC2->>App2: rx_notify_cb(chan_id, shm)
    App2->>App2: Read data from SHM

    Note over App1, App2: 下一次传输
    App1->>App1: Write data in SHM
    App1->>IPC1: shm_tx_unmanaged(chan_id)
    IPC1--)IPC2: Interrupt (Notify)
    IPC2->>App2: rx_notify_cb(chan_id, shm)
    App2->>App2: Read data from SHM
```

### 3. 托管通道数据流 (对应 1.5 Managed Channel Data Flow)

此时序图展示了启用缓冲区管理时，驱动程序控制缓冲池和队列的通信流程（零拷贝机制）。

```mermaid
sequenceDiagram
    participant App1 as OS1:App
    participant IPC1 as OS1:IPC
    participant IPC2 as OS2:IPC
    participant App2 as OS2:App

    %% 发送端流程
    App1->>IPC1: shm_acquire_buf(chan_id)
    IPC1->>IPC1: Get buf from SRAM buf pool
    IPC1-->>App1: return buf
    App1->>IPC1: shm_tx(chan_id, buf, size)
    IPC1->>IPC1: Push buf ptr in TX queue

    %% 跨核心中断
    IPC1--)IPC2: Interrupt

    %% 接收端流程
    IPC2->>IPC2: Pop buf ptr from RX queue
    IPC2->>App2: rx_notify_cb(chan_id, buf, size)
    App2->>App2: Save buf ptr

    %% 应用处理与释放
    App2->>App2: Process buf data
    App2->>IPC2: shm_release_buf(chan_id, buf)
    IPC2->>IPC2: Put buf in SRAM buf pool
```

### 4. 接收中断缓解技术 (对应 1.5 Mitigation Techniques)

此流程图展示了 IPCF 如何通过禁用中断并使用 SoftIRQ/Task 批处理描述符来避免中断风暴。

```mermaid
graph TD
    %% 定义节点
    Start((Rx ISR))
    DisableIRQ[Disable IRQ]
    ClearIRQ[Clear IRQ]
    SchedSoft[Schedule softirq]
    Reti((Return))

    StartSoft((softirq))
    PopBD[Pop BD from BD ring]
    CallCB[Call app Rx CB]
    CheckEmpty{BD ring empty?}
    CheckBudget{Budget finished?}
    EnableIRQ[Enable IRQ]
    Yield[Yield CPU]
    EndSoft((Return))

    %% ISR 流程
    Start --> DisableIRQ
    DisableIRQ --> ClearIRQ
    ClearIRQ --> SchedSoft
    SchedSoft --> Reti

    %% SoftIRQ 流程
    StartSoft --> PopBD
    PopBD --> CallCB
    CallCB --> CheckEmpty

    CheckEmpty -- No --> CheckBudget
    CheckBudget -- No --> PopBD
    CheckBudget -- Yes --> Yield
    Yield -.-> StartSoft

    CheckEmpty -- Yes --> EnableIRQ
    EnableIRQ --> EndSoft

    %% 样式
    classDef process fill:#fff,stroke:#000,stroke-width:1px;
    classDef decision fill:#fff,stroke:#000,stroke-width:1px,stroke-dasharray: 5 5;
    class DisableIRQ,ClearIRQ,SchedSoft,PopBD,CallCB,EnableIRQ,Yield process
    class CheckEmpty,CheckBudget decision
```

### 5. 示例应用程序逻辑 (对应 1.6 Sample Application)

展示了 Ping-Pong 通信及核心崩溃恢复（Core Crash）的处理逻辑。

```mermaid
sequenceDiagram
    participant App1 as OS1 Application
    participant Drv1 as OS1 ipcf-shm
    participant Drv2 as OS2 ipcf-shm
    participant App2 as OS2 Application

    Note over App1, App2: === 初始化阶段 (Initialization) ===
    
    App1->>Drv1: ipc_shm_init()
    App1->>Drv1: ipc_shm_is_remote_ready()
    
    App2->>Drv2: ipc_shm_init()
    App2->>Drv2: ipc_shm_is_remote_ready()

    Note over App1, App2: === 正常通信 (Normal Operation) ===
    
    App1->>App2: MSG exchange (Ping)
    App2-->>App1: MSG exchange (Pong)

    Note over App1, App2: === 崩溃与恢复 (Crash & Recovery) ===

    rect rgb(240, 240, 240)
        Note over App1, Drv1: 核心崩溃 (CORE CRASH)
        
        Note right of App2: 1. 检测到远程丢失
        App2->>Drv2: ipc_shm_is_remote_ready()
        Drv2-->>App2: FALSE

        Note right of App2: 2. 本地重新初始化
        App2->>Drv2: ipc_shm_free()
        App2->>Drv2: ipc_shm_init()
        
        loop 等待远程恢复 (Wait for Remote)
            App2->>Drv2: ipc_shm_is_remote_ready()
            Drv2-->>App2: FALSE
        end
    end

    Note left of App1: 3. 远程核心重启
    App1->>Drv1: ipc_shm_init()
    
    Note right of App2: 4. 连接恢复
    App2->>Drv2: ipc_shm_is_remote_ready()
    Drv2-->>App2: TRUE

    Note over App1, App2: === 通信恢复 (Resumed) ===
    App1->>App2: MSG exchange (Ping)
    App2-->>App1: MSG exchange (Pong)
```