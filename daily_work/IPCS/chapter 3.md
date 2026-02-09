
# 第三章 IPCF 实时操作系统共享内存驱动程序

## 3.1 概述

IPCF 实时操作系统 (RTOS) 共享内存驱动程序实现了与同一处理器不同核心上运行的另一个应用程序之间的共享内存通信。该驱动程序是跨平台通信框架 (IPCF) 的一部分。

该驱动程序附带一个示例应用程序，演示了与另一个示例应用程序进行的 Ping-Pong 消息通信（更多详细信息请参见示例目录中的自述文件）。

### 3.1.1 硬件平台

支持的处理器列在发布说明 (Release Notes) 文档中。

### 3.1.2 软件平台

支持的软件平台列在发布说明文档中。 用于驱动程序验证的编译器列在发布说明文档中。

## 3.2 与 RTOS 集成

要将此驱动程序集成到实时应用程序中，请在应用程序 makefile 中导入 `ipc-shm-rtos.mk`。 调用者的 makefile 必须设置以下变量：

- `SHM_PLATFORM` - 构建驱动程序的目标硬件平台
- `SHM_OS_TARGET` - 构建驱动程序的目标 RTOS
- `SHM_DRIVER_PATH` - 驱动程序目录的路径

`ipc-shm-rtos.mk` makefile 将生成以下变量：

- `SHM_DRIVER_SRC_DIR` - 驱动程序源目录
- `SHM_DRIVER_INCLUDES_DIRS` - 驱动程序包含目录
- `SHM_DRIVER_INCLUDE_FILES` - 驱动程序头文件列表
- `SHM_DRIVER_SOURCE_FILES` - 驱动程序源文件列表
- `SHM_DRIVER_OUT_FILES` - 驱动程序目标文件列表

用于构建 IPCF 驱动程序的编译器、汇编器和链接器标志来自 NXP RTD。驱动程序不需要任何额外的标志。

当使用 MSCM 时，中断处理程序的名称必须是 `ipc_shm_hardirq`；当使用 MRU 时，来自 RX 通道的回调函数名称必须是 `ipc_shm_mru_notification`。

### 3.2.1 与 NXP RTOS 集成

同样的步骤适用于与任何符合 Autosar 标准的 OS 集成：

- 必须为每个实例针对配置的 RX 外部 IRQ 注册一个二类 ISR (category 2 ISR)，此外，当使用 MSMC 时，ISR 属性 IsrFunction 必须命名为 `ipc_shm_hardirq`。
- 当使用 MU 时，必须为配置的 MU RX IRQ 注册一个 ISR，处理程序名称如下：`ipc_shm_mu_notification`。
- 当使用 MRU 时，必须为接收通道设置一个名为 `ipc_shm_mru_notification` 的中断通知函数。
- 必须配置一个名为 `ipc_shm_softirq` 的扩展、非抢占式任务，该任务不自动启动且优先级高于其他使用共享内存驱动程序的任务。
- 必须配置两个事件以在 `ipc_shm_softirq` 任务中使用：
    - `IPC_EVENT_RX_IRQ`：当从远程核心接收到消息时触发。
    - `IPC_EVENT_OS_FREE`：由用户应用程序触发以调用 `ipc_shm_free()`。

**注意**：用户应用程序除配置 ISR 和任务优先级以及任务堆栈大小外，不得干扰上述任何 OS 对象。

**与 AUTOSAR 运行时环境 (RTE) 集成 - 仅适用于 S32ZE 平台** 此功能仅具有 EAR (早期访问发布) 质量级别。 文件夹 `src\rte_integration` 包含一个名为 `IpcShm` 的驱动程序包装器以及相应的 arxml 文件（`IpcShm_Bswmd.arxml`, `IpcShm_Services.arxml` 和 `IpcShm_Types.arxml`），可用于按照以下步骤将驱动程序与 Autosar RTE 集成：

- 将 `IpcShm` arxml 文件导入 Autosar 工具链（例如：Autosar Builder），以便在将使用该驱动程序的模块中创建并连接所需端口 (Required Ports)。`IpcShm` 提供以下端口接口：`IpcShm_InitInstance`, `IpcShm_FreeInstance`, `IpcShm_AcquireBuffer`, `IpcShm_ReleaseBuffer`, `IpcShm_TransmitBuffer` 和 `IpcShm_IsRemoteReady`。
- 将 `IpcShm` arxml 文件导入 RTE 生成器（例如：Elektrobit Tresos Studio）。
- 在 RTE 配置中为 `IpcShm` 添加一个 SwComponentInstance（例如：`IpcShm_Prototype`）并将其映射到所需的 Os Application。
- 在 RTE 配置中为 `IpcShm` 添加一个 BswModuleInstance（例如：`BSW_IpcShm`）并将其映射到提供的 `IpcShm` 实现 (`Impl_IpcShm`)。将两个定时事件 (`TimingEvent_MainFunction` 和 `TimingEvent_Init`) 映射到所需的任务（例如：1ms 周期性任务）。
- 生成所有 RTE 文件。
- 在应用程序中使用生成的定义进行所需端口调用以与驱动程序交互（例如：`Rte_Call_RP_IpcShm_AcquireBuffer_Acquire`）。

### 3.2.2 与 FreeRTOS 集成

对于与 FreeRTOS 的集成：

- 当使用 MSMC 时，必须为配置的 RX 外部 IRQ 注册一个 ISR，处理程序名称为：`ipc_shm_hardirq`。
- 当使用 MU 时，必须为配置的 MU RX IRQ 注册一个 ISR，处理程序名称为：`ipc_shm_mu_notification`。
- 当使用 MRU 时，必须为接收通道设置一个名为 `ipc_shm_mru_notification` 的中断通知函数。
- 必须创建一个优先级为 `IPC_SOFTIRQ_PRIORITY` 的任务，供共享内存驱动程序使用，且必须配置名称为：`ipc_shm_softirq`。
- 驱动程序支持静态和动态分配（`configSUPPORT_STATIC_ALLOCATION` 和 `configSUPPORT_DYNAMIC_ALLOCATION`）。如果两者都选中，则 `ipc_shm_softirq` 任务将使用动态内存分配创建。

### 3.2.3 与 Zephyr 集成

对于与 Zephyr 的集成：

- 当使用 MSMC 时，必须为配置的 RX 外部 IRQ 注册一个 ISR，处理程序名称为：`ipc_shm_hardirq`。
- 当使用 MU 时，必须为配置的 MU RX IRQ 注册一个 ISR，处理程序名称为：`ipc_shm_mu_notification`。
- 当使用 MRU 时，必须为接收通道设置一个名为 `ipc_shm_mru_notification` 的中断通知函数。
- 共享内存驱动程序会创建一个优先级为 `IPC_SOFTIRQ_PRIORITY`、堆栈大小为 `IPC_SOFTIRQ_STACK_SIZE` 的线程，用于延迟中断处理。

### 3.2.4 与 XOS 集成

对于与 XOS 的集成：

- 当使用 MSMC 时，必须为配置的 RX 外部 IRQ 注册一个 ISR，处理程序名称为：`ipc_shm_hardirq`。
- 当使用 MU 时，必须为配置的 MU RX IRQ 注册一个 ISR，处理程序名称为：`ipc_shm_mu_notification`。
- 当使用 MRU 时，必须为接收通道设置一个名为 `ipc_shm_mru_notification` 的中断通知函数。
- 共享内存驱动程序会创建一个优先级为 `IPC_SOFTIRQ_PRIORITY`、堆栈大小为 `IPC_SOFTIRQ_STACK_SIZE` 的线程，用于延迟中断处理。

### 3.2.5 与裸机 (Baremetal) 集成

对于在裸机环境中的集成：

- 当使用 MSMC 时，必须为配置的 RX 外部 IRQ 注册一个 ISR，处理程序名称为：`ipc_shm_hardirq`。
- 当使用 MU 时，必须为配置的 MU RX IRQ 注册一个 ISR，处理程序名称为：`ipc_shm_mu_notification`。
- 当使用 MRU 时，必须为接收通道设置一个名为 `ipc_shm_mru_notification` 的中断通知函数。

## 3.3 配置说明

在驱动程序 API 级别可以配置五个与硬件相关的参数：TX 和 RX 核间中断 ID、本地核心 ID、远程核心 ID 和受信任核心。

中断 ID 是 MSCM 核对核定向中断 ID 或 MU/MRU 中断源。用户只能选择使用 MSCM、MU 或 MRU 驱动程序进行核心间相应的中断。驱动程序不支持在同一核心上运行的多个实例使用相同的 MRU 通道。

如果使用 MSCM 核对核定向中断，每个平台的中断 ID 可以从 RTD 头文件中选择（例如：`INT0_IRQn` 或 `MSCM_INT0_IRQn`），或者选择 `IPC_IRQ_NONE` 使用轮询方法。

可以通过将其 ID 设置为 `IPC_IRQ_NONE` 来禁用 TX 和 RX 中断。当 RX（或 TX）中断被禁用时，本地（或远程）应用程序必须通过调用函数 `ipc_shm_poll_channels()` 来检查传入消息。允许同时禁用 TX 和 RX 中断。

本地和远程核心 ID 配置分为核心类型和核心索引。核心类型和索引的支持值定义在 `ipc_shm_core_type` 和 `ipc_shm_core_index` 枚举中。本地核心 ID 和受信任核心配置保留用于 Linux 共享内存驱动程序，在此实现中不起作用。当使用 MU 或 MRU 时，本地和远程核心 ID 也不起作用。

对于 ARM 平台，可以通过选择 `IPC_CORE_DEFAULT` 作为核心类型，为远程核心 ID 分配默认值。使用此默认值时，驱动程序会自动选择核心索引。

## 3.4 注意事项

用户必须在初始化驱动程序之前将共享 SRAM 内存区域置零。

该驱动程序默认提供对映射为非缓存 (non-cachable) 的物理内存的直接访问。要使用缓存内存，需要定义符号 `IPC_D_CACHE_ENABLE`。

因此，应用程序应在共享内存缓冲区中仅进行对齐访问。在使用可能进行非对齐访问的函数（例如字符串处理函数）时应谨慎。

该驱动程序通过仅在本地内存中执行所有写入操作，确保本地和远程内存域之间免受干扰 (freedom from interference)。

只要只有一个线程进行推送 (pushing) 且只有一个线程进行弹出 (popping)，该驱动程序就是线程安全的：单生产者-单消费者 (Single-Producer -Single-Consumer)。

这种线程安全性是无锁的，并且在环形缓冲区中的写入和读取索引之间需要一个额外的哨兵 (sentinel) 元素，该元素永远不会被写入。

该驱动程序对于不同的实例是线程安全的，但对于同一个实例则不是。

驱动程序确保在发生内存溢出并损坏缓冲区描述符的情况下，托管通道将不再被（双方）使用。 驱动程序确保在发生内存溢出并损坏索引的情况下，非托管通道将不再被（双方）使用。 如果长度超过配置的最大长度，驱动程序不保证数据的完整性和正确性。
