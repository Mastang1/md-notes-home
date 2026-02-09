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
