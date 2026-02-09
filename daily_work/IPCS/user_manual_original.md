# 第一章 跨平台通信框架 (IPCF)

## 1.1 软件产品概述

跨平台通信框架 (IPCF) 是一个子系统，它使应用程序能够通过各种传输接口（如共享内存等）进行通信。这些应用程序可以运行在位于同一芯片或不同芯片上的多个同构或异构处理核心上，并运行在不同的操作系统（AUTOSAR, Linux, FreeRTOS, Zephyr 等）上。

IPCF 专为 NXP 嵌入式系统设计，具有低延迟和极小占用的特点。它提供了零拷贝 API (zero-copy API)，客户可以直接使用以获得最大性能、最小开销和低 CPU 负载。该驱动程序通过仅在本地内存域中执行所有写入操作，确保了本地和远程共享内存之间的免干扰 (freedom from interference)。客户可以使用 XRDC/SMPU 外设对其软件实施内存保护。

客户可以根据硬件、操作系统和传输接口的需求选择构建所需的内容。

## 1.2 用例

下图展示了 IPCF 解决的一些用例：

- 在多个同构或异构处理核心上的 IPCF 用例。
- 具有多个实例的 IPCF 用例。
- Linux 内核的 IPCF 用例。
- Linux 用户空间的 IPCF 用例。

## 1.3 软件内容

IPCF 软件包包含支持 AUTOSAR、FreeRTOS、Zephyr 和裸机 (baremetal) 的共享内存通信驱动程序。对于 Linux 支持，IPCF 软件包发布在 GitHub 上。

驱动程序附带了示例应用程序，演示了 Ping-Pong 消息通信（更多详细信息请参见示例自述文件）。

IPCF 软件包包含一个用于 NXP S32 Design Studio 的配置工具组件，该组件使用快捷简便。此组件安装在 NXP S32 Real Time Drivers (RTD) 发布版本之上，并可与所有 RTD 组件一起使用。

适用于 Linux 的 IPCF 共享内存驱动程序和示例应用程序作为树外 (out-of-tree) 内核模块集成在 NXP Auto Linux BSP 中。IPCF 驱动程序在 NXP Auto Linux BSP (fsl-image-auto) 的 Yocto 镜像中提供，但也可以手动构建。

如果客户应用程序需要从用户空间使用 IPCF，适用于 Linux 的 IPCF 共享内存驱动程序还提供了一个用户空间静态库。

Linux 驱动程序的 IPCF 实现源代码和示例发布在： `https://github.com/nxp-auto-linux/ipc-shm`

IPCF 软件包还包含：

**发布说明 (Release Notes)（关于发布的信息）：**

1. 支持的平台
2. 软件依赖项
3. 经过验证的编译器
4. 关于安装步骤的说明
5. 新功能
6. 已知限制
7. 许可和支持

**IPCF 驱动程序用户手册 (User Manual)：**

1. 针对不同集成的安装说明
2. 针对不同操作系统和平台的驱动程序使用、编译和配置
3. 编译、构建和运行示例应用程序的说明
4. 驱动程序 API 的描述

**质量包 (Quality Package)** - 提供给 RTM 发布版本的客户。

## 1.4 架构

IPCF 驱动程序包含以下层：

- **共享内存通用实现**：与硬件和操作系统无关。
- **硬件抽象组件**：对各种硬件 IP 模块（MSCM, INTC...）的抽象。
- **操作系统抽象组件**：用于通用操作系统服务的操作系统无关 API。

## 1.5 驱动程序详情

IPCF 驱动程序使用以下方式进行缓冲区管理：

- **非托管通道数据流 (Unmanaged channel data flow)**：（类似于 POSIX 共享内存）禁用缓冲区管理，应用程序拥有整个通道内存；用例示例：视频流或非关键数据交换。每个应用程序拥有通道内存的一半。
- **托管通道数据流 (Managed channel data flow)**：内存被分为缓冲池 (buffer pools)，缓冲区管理由驱动程序控制；用例示例：CAN 转发或 Flash 更新。可以同时处理多个数据流。

IPCF 驱动程序支持以下核间通知方法：

- **核间中断**（硬件通知）
- **轮询方法**（发送和接收由用户管理）

IPCF 驱动程序使用中断合并 (interrupt coalescing) 技术减少接收中断的开销，从而避免中断风暴。

当触发接收中断时，代码会禁用中断，处理接收 FIFO 中的所有缓冲区描述符，然后重新启用中断。

## 1.6 IPCF 示例应用程序

这是 IPCF 共享内存驱动程序示例应用程序。它演示了使用共享内存驱动程序进行的 Ping-Pong 消息通信。

一个应用程序初始化驱动程序，等待远程应用程序准备就绪，并对接收到的每条消息进行回显回复。另一个应用程序发送若干消息并等待回复。如果一个核心崩溃或应用程序崩溃，另一个应用程序会检测到此情况并重新初始化所有实例。

# 第二章 IPCF Linux 共享内存驱动程序

## 2.1 概述

Linux IPCF 共享内存驱动程序实现了与运行在同一处理器不同核心上的 RTOS 应用程序之间的共享内存通信。

该驱动程序附带一个示例应用程序，演示了与 RTOS 应用程序进行的 Ping-Pong 消息通信（更多详细信息请参见示例目录中的自述文件）。

该驱动程序作为树外 (out-of-tree) 内核模块集成在 NXP Auto Linux BSP 中。

此 Linux 驱动程序的源代码发布在 github.com 上。

## 2.2 硬件平台

支持的处理器列在示例应用程序文档中。

## 2.3 配置说明

在驱动程序 API 级别可以配置五个与硬件相关的参数：TX 和 RX 核间中断 ID、本地核心 ID、远程核心 ID 和受信任核心。

中断 ID 是 MSCM 核对核定向中断 ID。对于 S32xx 有效值范围是 [0..2]，对于 S32G3xx 是 [0..11]，对于 S32V234 是 [0..3]。TX 和 RX 中断 ID 必须不同。

**注意**：驱动程序配置中预期的中断 ID 与相应的处理器异常编号（用于注册中断处理程序）不同。具体信息请参见各平台的参考手册。

可以通过将其 ID 设置为 `IPC_IRQ_NONE` 来禁用 TX 中断。禁用时，远程应用程序必须通过调用函数 `ipc_shm_poll_channels()` 来检查传入消息。

本地和远程核心 ID 配置分为核心类型和核心索引。核心类型和索引的支持值定义在 `ipc_shm_core_type` 和 `ipc_shm_core_index` 枚举中。

对于 ARM 平台，可以通过选择 `IPC_CORE_DEFAULT` 作为核心类型，为本地和/或远程核心 ID 分配默认值。使用此默认值时，驱动程序会自动选择核心索引。

**注意**：有关支持的核心类型和索引，请参见各平台的参考手册。

远程核心 ID 指定要中断的远程核心，本地核心 ID 指定被远程核心中断作为目标的的核心。本地核心 ID 配置仅适用于 Linux 可能运行 SMP 且中断可能由与目标核心不同的核心处理（例如中断负载平衡）的平台。

受信任核心掩码指定哪些核心（与本地核心类型相同）有权访问被远程作为目标的本地核心的核间中断状态寄存器。掩码可以由 `ipc_shm_core_index` 枚举中定义的有效值组成。

**注意**：本地核心 ID 和受信任核心配置仅适用于运行 Linux 的 S32xx 平台。对于其他平台或操作系统，不使用本地核心 ID 配置。

如果使用 Linux IPCF 共享内存用户空间驱动程序，用户空间静态库 (`libipc-shm`) 将在初始化时自动插入 IPCF UIO/CDEV 内核模块。可以通过调用者设置 `IPC_UIO_MODULE_DIR`（用于使用 uio 驱动程序）或 `IPC_CDEV_MODULE_DIR`（用于使用字符驱动程序）变量，在编译时覆盖目标板 rootfs 中内核模块的路径。

## 2.4 注意事项

此驱动程序提供对在内核空间或用户空间中映射为非缓存 (non-cachable) 的物理内存的直接访问。因此，应用程序应在共享内存缓冲区中仅进行对齐访问。在使用可能进行非对齐访问的函数（例如字符串处理函数）时应谨慎。

该驱动程序通过仅在本地内存中执行所有写入操作，确保本地和远程内存域之间免受干扰 (freedom from interference)。

只要只有一个线程进行推送 (pushing) 且只有一个线程进行弹出 (popping)，该驱动程序就是线程安全的：单生产者-单消费者 (Single-Producer -Single-Consumer)。

这种线程安全性是无锁的，并且在环形缓冲区中的写入和读取索引之间需要一个额外的哨兵 (sentinel) 元素，该元素永远不会被写入。

该驱动程序对于不同的实例是线程安全的，但对于同一个实例则不是。

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

# 第四章 IPCF 共享内存安装程序

## 4.1 概述

IPCF 软件包包含一个独立安装程序和一个用于 S32 Design Studio 的更新站点。

## 4.2 独立安装程序

要安装 IPCF 驱动程序，请按照以下步骤操作：

1. 启动独立安装程序。
2. 在欢迎窗口中点击“Next”（下一步）。
3. 阅读并接受许可条款。点击“Next”以完成安装。
4. 选择要安装的软件包并点击“Next”。
5. 选择目标文件夹并点击“Install”（安装）。 默认路径为 `C:/NXP/IPCF_<version>`
6. 安装完成后点击“Finish”（完成）。
7. 安装后，可以将 IPCF 添加到新的 Tresos 项目中，或者可以使用软件包中提供的 IPCF 示例。

## 4.3 S32 Design Studio 安装程序

IPCF 作为 S32 Design Studio 的更新站点 (Update Site) 交付。在这种情况下，必须在安装 RTD 软件包之后，按照以下步骤进行安装：

1. 启动 S32 Design Studio 并选择一个工作空间 (workspace)。
2. 选择 Help -> Install New Software . . . （帮助 -> 安装新软件...）
3. 点击 Add（添加），然后点击 Archive . . . （归档...）并选择 IPCF 发布版本中包含的更新站点文件。
4. 勾选要安装的 IPCF 软件包并继续安装过程。
5. 安装后，可以将 IPCF 添加到新的 S32DS 项目中，或者可以使用软件包中提供的 IPCF 示例。

# 第五章 IPCF 共享内存示例应用程序

## 5.1 概述

IPCF 软件包包含演示驱动程序功能的示例。

关于构建和运行的详细信息，请参阅每个示例中提供的 `description.txt` 文件。

# 第六章 IPCF 共享内存集成

## 6.1 配置

IPCF 共享内存驱动程序可以在 S32 Design Studio 和 Tresos 中进行配置。

用户可以修改 `local_shm_addr`、`remote_shm_addr` 和 `shm_size` 的值。这些更改也必须反映在链接器文件 (linker file) 中。共享 SRAM 必须是**可共享 (SHAREABLE)** 和**不可缓存 (NON-CACHEABLE)** 的。如果共享内存是可缓存的，那么用户必须在使用 IPC 发送函数之前使缓存无效。

如果用户为 `inter_core_tx_irq` 或 `inter_core_rx_irq` 选择 `IRQ_NONE-POLLING`，则使用轮询方法（参见 `ipc_shm_poll_channels` API 函数）。为了获得更好的性能，建议选择核间中断。

用户必须选择远程核心类型和索引。这些值必须在另一个核心的配置中反向设置。

用户可以配置 IPCF 通信通道（**非托管 (UNMANAGED)** 或 **托管 (MANAGED)**：具有 `num_bufs` 和 `buf_size` 值的缓冲池）。

实例的最大数量由 `IPC_SHM_MAX_INSTANCES` 定义（最大 255）。共享内存通道的最大数量由 `IPC_SHM_MAX_CHANNELS` 定义（最大 255）。为托管通道配置的缓冲池的最大数量由 `IPC_SHM_MAX_POOLS` 定义（最大 255）。每个池的最大缓冲区数量由 `IPC_SHM_MAX_BUFS_PER_POOL` 定义（最大 65535）。

用户可以添加一个新的 IPCF 实例以与另一个核心通信。

在 Tresos 中，为“Inter Core Rx IRQ”参数配置的值应与为“Local Core”参数配置的值相关联。例如：如果“Local Core”选择为 `IPC_CORE_M33`，则在“Inter Core Rx IRQ”中必须选择专用于 M33 核心的中断。

有关参数、值和结构类型的更多信息，请参见 **IPCF 驱动程序 API** 章节。有关 IPCF 驱动程序集成的更多详细信息，请参见软件包中提供的示例应用程序的说明。

### 6.1.1 S32DS 配置

_(此部分在原文档中主要包含 S32DS 配置界面的示例截图，展示了常规模式配置、非托管通道配置及托管通道配置)_

### 6.1.2 Tresos 配置

_(此部分在原文档中主要包含 Tresos 配置界面的示例截图，展示了常规配置、实例配置、非托管通道配置及托管通道缓冲区配置)_

# 第七章 IPCF 共享内存布局

## 7.1 内存布局和大小 [7.1]

用户可以使用以下公式计算可共享内存的大小：

**IPCF 非托管通道内存布局和大小**

- `非托管通道总大小 (TOTAL UNMANAGE CHANNEL SIZE) = 16 + buffer_size`

**IPCF 托管通道内存布局和大小**

- `TX 环形缓冲区 (RING for TX) = 16 + (total_buffers_no + 1) * 8`
- `pool0 环形缓冲区 (RING for pool0) = 16 + (pool0.buffers_no + 1) * 8`
- `Pool0 = ring_for_pool + pool0.buf_size * num_bufs`
- `Channel0 = RING for TX + pool0 + ...`
- `托管通道 (MANAGE CHANNEL) = channel0 + ...`

# 第八章 IPCF 共享内存驱动程序 API

## 8.1 IPCF 共享内存驱动程序 API

### 8.1.1 枚举 ipc_shm_channel_type

`enum ipc_shm_channel_type` 通道类型

**定义**

```
enum ipc_shm_channel_type {
    IPC_SHM_MANAGED,
    IPC_SHM_UNMANAGED
};
```

**常量**

- `IPC_SHM_MANAGED`：启用了缓冲区管理的通道
- `IPC_SHM_UNMANAGED`：禁用缓冲区管理，应用程序拥有整个通道内存

**描述** 对于非托管通道，应用程序对通道内存拥有完全控制权，且 ipc-shm 设备不进行缓冲区管理。

### 8.1.2 枚举 ipc_shm_queue_type

`enum ipc_shm_queue_type` 队列类型

**定义**

```
enum ipc_shm_queue_type {
    IPC_SHM_CHANNEL_QUEUE,
    IPC_SHM_POOL_QUEUE
};
```

**常量**

- `IPC_SHM_CHANNEL_QUEUE`：通道队列
- `IPC_SHM_POOL_QUEUE`：缓冲池队列

### 8.1.3 枚举 ipc_shm_core_type

`enum ipc_shm_core_type` 核心类型

**定义**

```
enum ipc_shm_core_type {
    IPC_CORE_DEFAULT,
    IPC_CORE_A53,
    IPC_CORE_M7,
    IPC_CORE_M4,
    IPC_CORE_Z7,
    IPC_CORE_Z4,
    IPC_CORE_Z2,
    IPC_CORE_R52,
    IPC_CORE_M33,
    IPC_CORE_BBE32
};
```

**常量**

- `IPC_CORE_DEFAULT`：用于让驱动程序自动选择远程核心类型
- `IPC_CORE_A53`：ARM Cortex-A53 核心
- `IPC_CORE_M7`：ARM Cortex-M7 核心
- `IPC_CORE_M4`：ARM Cortex-M4 核心
- `IPC_CORE_Z7`：PowerPC e200z7 核心
- `IPC_CORE_Z4`：PowerPC e200z4 核心
- `IPC_CORE_Z2`：PowerPC e200z2 核心
- `IPC_CORE_R52`：ARM Cortex-R52 核心
- `IPC_CORE_M33`：ARM Cortex-M33 核心
- `IPC_CORE_BBE32`：Tensilica ConnX BBE32EP 核心

### 8.1.4 枚举 ipc_shm_core_index

`enum ipc_shm_core_index` 核心索引

**定义**

```
enum ipc_shm_core_index {
    IPC_CORE_INDEX_0,
    IPC_CORE_INDEX_1,
    IPC_CORE_INDEX_2,
    IPC_CORE_INDEX_3,
    IPC_CORE_INDEX_4,
    IPC_CORE_INDEX_5,
    IPC_CORE_INDEX_6,
    IPC_CORE_INDEX_7
};
```

**常量**

- `IPC_CORE_INDEX_0` ... `IPC_CORE_INDEX_7`：处理器索引 0 到 7

### 8.1.5 结构体 ipc_shm_pool_cfg

`struct ipc_shm_pool_cfg` 内存缓冲池参数

**定义**

```
struct ipc_shm_pool_cfg {
    uint16 num_bufs;
    uint32 buf_size;
};
```

**成员**

- `num_bufs`：缓冲区数量
- `buf_size`：缓冲区大小

### 8.1.6 结构体 ipc_shm_managed_cfg

`struct ipc_shm_managed_cfg` 托管通道参数

**定义**

```
struct ipc_shm_managed_cfg {
    uint8 num_pools;
    struct ipc_shm_pool_cfg *pools;
    void (*rx_cb)(void *cb_arg, const uint8 instance, uint8 chan_id, void *buf, uint32 size);
    void *cb_arg;
};
```

**成员**

- `num_pools`：缓冲池数量
- `pools`：内存缓冲池参数
- `rx_cb`：接收回调函数
- `cb_arg`：可选的接收回调参数

### 8.1.7 结构体 ipc_shm_unmanaged_cfg

`struct ipc_shm_unmanaged_cfg` 非托管通道参数

**定义**

```
struct ipc_shm_unmanaged_cfg {
    uint32 size;
    void (*rx_cb)(void *cb_arg, const uint8 instance, uint8 chan_id, void *mem);
    void *cb_arg;
};
```

**成员**

- `size`：非托管通道内存大小
- `rx_cb`：接收回调函数
- `cb_arg`：可选的接收回调参数

### 8.1.8 结构体 ipc_shm_channel_cfg

`struct ipc_shm_channel_cfg` 通道参数

**定义**

```
struct ipc_shm_channel_cfg {
    enum ipc_shm_channel_type type;
    union {
        struct ipc_shm_managed_cfg managed;
        struct ipc_shm_unmanaged_cfg unmanaged;
    } ch;
};
```

**成员**

- `type`：来自 `enum ipc_shm_channel_type` 的通道类型
- `ch.managed`：托管通道参数
- `ch.unmanaged`：非托管通道参数

### 8.1.9 结构体 ipc_shm_remote_core

`struct ipc_shm_remote_core` 远程核心类型和索引

**定义**

```
struct ipc_shm_remote_core {
    enum ipc_shm_core_type type;
    enum ipc_shm_core_index index;
};
```

**成员**

- `type`：来自 `enum ipc_shm_core_type` 的核心类型
- `index`：核心编号

**描述** 核心类型可以是 `IPC_CORE_DEFAULT`，在这种情况下，核心索引并不重要，因为它由驱动程序自动选择。

### 8.1.10 结构体 ipc_shm_local_core

`struct ipc_shm_local_core` 本地核心类型、索引和受信任核心

**定义**

```
struct ipc_shm_local_core {
    enum ipc_shm_core_type type;
    enum ipc_shm_core_index index;
    uint32 trusted;
};
```

**成员**

- `type`：来自 `enum ipc_shm_core_type` 的核心类型
- `index`：被远程核心中断作为目标的核心编号
- `trusted`：受信任核心掩码

**描述** 核心类型可以是 `IPC_CORE_DEFAULT`，在这种情况下，核心索引并不重要，因为它由驱动程序自动选择。 受信任核心掩码指定哪些核心（与本地核心类型相同）有权访问被作为目标的本地核心的核间中断状态寄存器。掩码可以由 `enum ipc_shm_core_index` 组成。

### 8.1.11 结构体 ipc_shm_cfg

`struct ipc_shm_cfg` IPC 共享内存参数

**定义**

```
struct ipc_shm_cfg {
    uintptr local_shm_addr;
    uintptr remote_shm_addr;
    uint32 shm_size;
    sint16 inter_core_tx_irq;
    sint16 inter_core_rx_irq;
    uint8 mru_tx_channel_id;
    uint8 mru_rx_channel_id;
    struct ipc_shm_local_core local_core;
    struct ipc_shm_remote_core remote_core;
    uint8 num_channels;
    struct ipc_shm_channel_cfg *channels;
#ifdef USING_OS_AUTOSAROS
    ISRType isr_id_handler;
#endif
};
```

**成员**

- `local_shm_addr`：本地共享内存物理地址
- `remote_shm_addr`：远程共享内存物理地址
- `shm_size`：本地/远程共享内存大小
- `inter_core_tx_irq`：为共享内存驱动程序 Tx 预留的核间中断
- `inter_core_rx_irq`：为共享内存驱动程序 Rx 预留的核间中断
- `mru_tx_channel_id`：用于共享内存驱动程序 Tx 的 MRU 通道索引
- `mru_rx_channel_id`：用于共享内存驱动程序 Rx 的 MRU 通道索引
- `local_core`：被远程核心中断作为目标的本地核心
- `remote_core`：触发中断的远程核心
- `num_channels`：共享内存通道数量
- `channels`：IPC 通道参数数组
- `isr_id_handler`：定义用于处理中断的 OsIsr 名称（仅在使用 AutosarOS 时）

**描述** 使用的 TX 和 RX 中断必须不同。对于 ARM 平台，可以使用 `IPC_CORE_DEFAULT` 为本地和远程核心分配默认值。本地核心仅用于 Linux 可能运行在多个核心上的平台，对于 RTOS 和裸机实现将被忽略。 本地和远程通道及缓冲池配置必须对称。

### 8.1.12 结构体 ipc_shm_instances_cfg

`struct ipc_shm_instances_cfg` IPC 共享内存参数

**定义**

```
struct ipc_shm_instances_cfg {
    uint8 num_instances;
    struct ipc_shm_cfg *shm_cfg;
};
```

**成员**

- `num_instances`：共享内存实例数量
- `shm_cfg`：IPC 共享内存参数数组

### 8.1.13 函数 ipc_shm_init_instance

`sint8 ipc_shm_init_instance(uint8 instance, const struct ipc_shm_cfg *cfg)` 初始化 IPC-Shm 驱动程序的指定实例

**参数**

- `instance (uint8)` – 实例 id
- `cfg (const struct ipc_shm_cfg*)` – ipc-shm 实例配置

**描述** 成功时返回 `IPC_SHM_E_OK`，否则返回错误代码。

### 8.1.14 函数 ipc_shm_init

`sint8 ipc_shm_init(const struct ipc_shm_instances_cfg *cfg)` 初始化共享内存设备

**参数**

- `cfg (const struct ipc_shm_instances_cfg*)` – 配置参数

**描述** 函数不可重入。

**返回** 成功时返回 0，否则返回错误代码。

### 8.1.15 函数 ipc_shm_free_instance

`void ipc_shm_free_instance(const uint8 instance)` 取消初始化 IPC-Sh