
以下是根据提供的 "UserManual_S32GXXXa.pdf" 文档内容，针对 **第六章：IPCF Shared Memory Sample Applications** 的中文翻译。

---

# 第六章 IPCF 共享内存示例应用程序

## 6.1 Linux IPCF 共享内存示例应用程序

### 6.1.1 概述

本示例应用程序是一个内核模块，旨在演示使用共享内存驱动程序与 RTOS 应用程序进行的 Ping-Pong（往返）消息通信。

应用程序初始化共享内存驱动程序并向远程应用程序发送消息，在发送每条消息后等待回复。当收到远程应用程序的回复时，它会被唤醒并发送下一条消息。

该应用程序既可以构建为使用核间中断（默认行为）通知远程应用程序，也可以在不通知远程应用程序的情况下进行传输。如果使用后者，远程应用程序将轮询可用消息。该应用程序通过 sysfs 文件从控制台进行控制（参见“运行应用程序”一节）。

### 6.1.2先决条件

- 支持的处理器的 EVB 板
- NXP Automotive Linux BSP

### 6.1.3 构建应用程序

本模块使用 NXP Auto Linux BSP 中的 Yocto 构建，但在需要时也可以手动构建。

> **注意**：模块也包含在 NXP Auto Linux BSP 预构建镜像中，可从 NXP Auto Linux BSP Flexera 目录下载。

**使用 Yocto 构建**

1. 按照使用 Yocto 构建 NXP Auto Linux BSP 的步骤操作（参见 Flexera 目录中的 Linux BSP 用户手册）。
2. 使用分支 `release/IPCF_RELEASE_NAME` 并修改 `build/sources/meta-alb/recipes-kernel/ipc-shm/ipc-shm.bb` 文件：
    - 将 `BRANCH ?= "${RELEASE_BASE}"` 修改为 `BRANCH ?= "release/IPCF_RELEASE_NAME"`
    - 将 `SRCREV = "xxxxxxxxxx"` 修改为 `SRCREV = "${AUTOREV}"`
    - 其中 `IPCF_RELEASE_NAME` 是 Flexera 目录中 Inter-Platform Communication Framework (IPCF) 发布版本的名称。
3. **注意**：使用 `fsl-image-auto` 镜像配合任何支持的机器，或在 `conf/local.conf` 文件中添加以下行：`IMAGE_INSTALL_append_pn-fsl-image-auto = " ipc-shm"`。

**手动构建**

1. 从 Code Aurora 获取 NXP Auto Linux 内核和 IPCF 驱动程序：
    
    ```
    git clone https://source.codeaurora.org/external/autobsps32/linux/
    git clone https://source.codeaurora.org/external/autobsps32/ipcf/ipc-shm/
    git -C ipc-shm checkout --track origin/release/IPCF_RELEASE_NAME
    ```
    
    > **注意**：请使用发布分支 `release/IPCF_RELEASE_NAME`，其中 `IPCF_RELEASE_NAME` 为 Flexera 目录中的版本名称。
    
2. 导出 `CROSS_COMPILE` 和 `ARCH` 变量，并提供所需的配置来构建 Linux 内核：
    
    ```
    export CROSS_COMPILE=/<toolchain-path>/aarch64-linux-gnu-
    export ARCH=arm64
    make -C ./linux s32gen1_defconfig
    make -C ./linux
    ```
    
3. 提供内核源码位置以构建 IPCF 驱动程序和示例模块，例如：
    
    ```
    make -C ./ipc-shm/sample KERNELDIR=$PWD/linux modules
    ```
    
    > **注意**：对于 S32G3xx，必须添加 `PLATFORM_FLAVOR`： `make -C ./ipc-shm/sample PLATFORM_FLAVOR=s32g3 KERNELDIR=$PWD/linux modules`。
    

### 6.1.4 运行应用程序

1. 如果示例是手动构建的，请将 `ipc-shm-dev.ko` 和 `ipc-shm-sample.ko` 复制到 rootfs 中。
2. 启动 Linux：请参阅 Auto Linux BSP 用户手册中的“如何启动（How to boot）”部分。
3. Linux 启动后插入 IPCF 内核模块：
    
    ```
    insmod /lib/modules/`uname -r`/extra/ipc-shm-dev.ko
    insmod /lib/modules/`uname -r`/extra/ipc-shm-sample.ko
    ```
    
4. 清除内核日志：
    
    ```
    dmesg -c > /dev/null
    ```
    
5. 向远程 OS 发送 10 条 Ping 消息并显示内核日志输出：
    
    ```
    echo 10 > /sys/kernel/ipc-shm-sample/ping
    dmesg -c
    ```
    
6. 使用不同的消息数量重复上一步。
7. 卸载模块：
    
    ```
    rmmod ipc-shm-sample ipc-shm-dev
    ```
    

。

### 6.1.5 配置说明

**轮询 (Polling)** 为了编译支持轮询的共享内存示例应用程序，必须将 makefile 参数 `POLLING` 设置为 `yes`，例如：

```
make -C ./ipc-shm/sample POLLING=yes KERNELDIR=$PWD/linux modules
```

> **注意**：远程示例应用程序也必须构建为支持轮询。更多详细信息请参考远程示例的构建说明。

该示例演示了如何使用共享内存轮询 API 来轮询传入消息，以代替使用核间中断通知。

---

## 6.2 Linux IPCF 共享内存示例应用程序（多实例）

### 6.2.1 概述

本示例应用程序是一个内核模块，演示了使用共享内存驱动程序与 RTOS 应用程序进行的 Ping-Pong 消息通信。

应用程序使用**两个实例**初始化共享内存驱动程序，并向远程应用程序发送消息，在发送每条消息后等待回复。当收到远程应用程序的回复时，它会被唤醒并发送下一条消息。

该应用程序既可以构建为使用核间中断（默认行为）通知远程应用程序，也可以在不通知远程应用程序的情况下进行传输。如果使用后者，远程应用程序将轮询可用消息。该应用程序通过 sysfs 文件从控制台进行控制。

### 6.2.2 先决条件

- 支持的处理器的 EVB 板
- NXP Automotive Linux BSP

### 6.2.3 构建应用程序

本模块使用 NXP Auto Linux BSP 中的 Yocto 构建，但在需要时也可以手动构建。

> **注意**：模块也包含在 NXP Auto Linux BSP 预构建镜像中。

**使用 Yocto 构建**

1. 按照使用 Yocto 构建 NXP Auto Linux BSP 的步骤操作。
2. 使用分支 `release/IPCF_RELEASE_NAME` 并修改 `build/sources/meta-alb/recipes-kernel/ipc-shm/ipc-shm.bb`（修改方法同 6.1.3 节）。

**手动构建**

1. 从 Code Aurora 获取代码（步骤同 6.1.3 节）。
2. 导出变量并构建 Linux 内核（步骤同 6.1.3 节）。
3. 构建 IPCF 驱动程序和示例模块（注意路径差异）：
    
    ```
    make -C ./ipc-shm/sample_multi_instance KERNELDIR=$PWD/linux modules
    ```
    
    > **注意**：对于 S32G3xx，必须添加 `PLATFORM_FLAVOR=s32g3`。
    

### 6.2.4 运行应用程序

1. 如果手动构建，复制 `ipc-shm-dev.ko` 和 `ipc-shm-sample_multi-instance.ko` 到 rootfs。
2. 启动 Linux。
3. 插入内核模块：
    
    ```
    insmod /lib/modules/`uname -r`/extra/ipc-shm-dev.ko
    insmod /lib/modules/`uname -r`/extra/ipc-shm-sample_multi-instance.ko
    ```
    
4. 清除内核日志。
5. 在**实例 0** 上向远程 OS 发送 10 条 Ping 消息并显示日志：
    
    ```
    echo 10 > /sys/kernel/ipc-shm-sample-instance0/ping
    dmesg -c
    ```
    
6. 在**实例 1** 上向远程 OS 发送 10 条 Ping 消息并显示日志：
    
    ```
    echo 10 > /sys/kernel/ipc-shm-sample-instance1/ping
    dmesg -c
    ```
    
7. 卸载模块：
    
    ```
    rmmod ipc-shm-sample-instance ipc-shm-dev
    ```
    

。

---

## 6.3 Linux IPCF 共享内存用户空间示例应用程序

### 6.3.1 概述

本示例应用程序演示了使用**用户空间**共享内存驱动程序与 RTOS 应用程序进行的 Ping-Pong 消息通信。

应用程序初始化共享内存驱动程序并向远程示例应用程序发送消息，在发送每条消息后等待回复。本应用可构建为使用核间中断或轮询模式。

### 6.3.2 先决条件

- 支持的处理器的 EVB 板：S32V234, S32GEN1
- NXP Automotive Linux BSP

### 6.3.3 构建应用程序

**使用 Yocto 构建**

1. 遵循 Yocto 构建步骤。
    
    - 对于 **S32V234**：使用 `release/bsp23.0` 分支，修改 `ipc-shm.bb` 中的 SRCREV，并配置 `ipc-shm-uio` 黑名单。
    - 对于 **S32GEN1**：使用 `release/IPCF_RELEASE_NAME` 分支并修改 SRCREV。
    - 启用用户空间 I/O 驱动程序（`User-space I/O drivers`）。
    - 使用 `fsl-image-auto` 镜像。
2. 从 Code Aurora 获取 IPCF-ShM 用户空间驱动程序：
    
    ```
    git clone https://source.codeaurora.org/external/autobsps32/ipcf/ipc-shm-us/
    git -C ipc-shm-us submodule update --init --remote
    ```
    
3. 使用 IPCF-ShM 库构建示例应用程序，提供目标板 rootfs 中 IPC UIO 内核模块的位置和平台名称：
    
    ```
    make -C ./ipc-shm-us/sample PLATFORM=S32V234 IPC_UIO_MODULE_DIR="/lib/modules/<kernel-release>/extra"
    ```
    
    其中 `PLATFORM` 可以是 `S32V234` 或 `S32GEN1`。
    

**手动构建**

1. 获取内核和 IPCF 驱动程序源码（同上）。
2. 配置 Linux 内核以启用用户空间 I/O 驱动程序 (`make -C ./linux menuconfig` -> `Userspace I/O drivers`)。
3. 构建 Linux 内核。
4. 构建 IPCF-ShM 驱动程序模块：
    
    ```
    make -C ./ipc-shm-us/common KERNELDIR=$PWD/linux modules
    ```
    
5. 构建示例应用程序（同 Yocto 步骤 3）。

### 6.3.4 运行应用程序

1. 将 `ipc-shm-sample.elf` 复制到目标板 rootfs。如果是手动构建，还需复制 `ipc-shm-uio.ko` 到编译时指定的目录。
2. 启动 Linux。
3. 运行示例并在提示时指定要交换的 Ping 消息数量：
    
    ```
    ./ipc-shm-sample.elf
    Input number of messages to send:
    ```
    
    > **注意**：输入 0 或发送中断信号（如 Ctrl + C）可退出示例。
    

### 6.3.5 配置说明

**轮询 (Polling)** 编译时设置 `POLLING=yes`：

```
make -C ./ipc-shm-us/sample POLLING=yes PLATFORM=S32GEN1
```

---

## 6.4 NXP S32 ARM AUTOSAR OS 的 IPCF 共享内存示例应用程序

### 6.4.1 概述

这是运行在 NXP S32 ARM 平台上的 AUTOSAR OS 的 IPCF 共享内存驱动程序示例应用程序。它演示了与 Linux 内核应用程序使用共享内存驱动程序进行的 Ping-Pong 消息通信。应用程序会对从 Linux 应用程序接收到的每条消息进行回显回复。支持中断通知或轮询模式。

### 6.4.2 先决条件

1. 支持的处理器的 EVB 板：S32V234, S32G274A, S32R45, S32K3xx, S32G399A
2. Cygwin 2.8.0 或更高版本
3. 目标平台的 NXP AUTOSAR OS
4. 目标平台的编译器（GCC, GHS, IAR, Diab）
5. IPCF 软件包。

### 6.4.3 构建应用程序

发布包包含预构建的 AUTOSAR OS 演示二进制文件。

1. 打开 Cygwin 控制台并进入示例应用位置 `c:/NXP/IPCF_<version>/sample/s32_autosar`。
2. 根据目标平台和编译器设置环境变量（如 `OS_PATH`, `GCCDIR`, `GHSDIR` 等）。
3. 为目标平台构建，例如：
    
    ```
    make platform=s32g274a compiler=gccarm clean all
    ```
    

### 6.4.4 运行应用程序

**在 S32V234 上运行**

1. 将 `sample.bin` 复制到 SD 卡。
2. 在 u-boot 控制台停止。
3. 将两个示例应用使用的 SRAM 共享内存置零：`initsram 0x3e900000 0x200000`。
4. 加载二进制文件：`fatload mmc 0:1 0x3eb00000 sample.bin`。
5. 启动 M4 核心：`startm4 0x3eb00200`。
6. 启动 Linux 并运行 Linux 示例应用。

**在 S32GEN1 上运行**

1. 将 `sample.bin` 复制到 SD 卡。
2. 在 u-boot 控制台停止。
3. 禁用数据缓存：`dcache off`。
4. 将 SRAM 共享内存置零：`mw.q 0x34000000 0x0 0x100000`。
5. 加载二进制文件到 RAM 然后复制到 SRAM：
    
    ```
    fatload mmc 0:1 0x80000000 sample.bin
    cp 0x80000000 0x34300000 0x100000
    ```
    
6. 启动 M7 核心：`startm7 0x34300000`。
7. 启动 Linux 并运行 Linux 示例应用。

**在 S32K3xx 上运行**

1. 使用 Trace32 PowerView。
2. 执行脚本加载二进制文件：`DO s32k3xx.cmm output/sample.elf IPCF_Example_S32K3xx_M7_1.elf`。

### 6.4.5 配置说明

**轮询 (Polling)** 编译时设置 `polling=yes`。 **内存保护** 示例可以使用 RTD 中的 XRDC 驱动程序强制执行内存保护。

---

## 6.5 NXP S32 ARM RTD 的 IPCF 共享内存示例应用程序

### 6.5.1 概述

这是运行在 NXP S32 平台 Cortex-M7 核心上的 NXP S32 RTD 的 IPCF 共享内存驱动程序示例应用程序。它演示了与 Linux（A53 核心）或嵌入式应用程序（M7 核心）的 Ping-Pong 通信。支持裸机或 FreeRTOS 构建。

### 6.5.2 先决条件

支持处理器：S32G274A, S32R45, S32G399A。需要 Cygwin 或 MSYS Linux Shell。

### 6.5.3 构建应用程序

1. 打开 Shell 并进入 `C:/NXP/IPCF_<version>/sample/s32_rtd`。
2. 在 `project_parameters.mk` 中设置环境变量。
3. 构建示例：
    
    ```
    make platform=s32g274a os_target=freertos compiler=gccarm clean build
    ```
    
    > **注意**：FreeRTOS 仅移植到了 gccarm。
    

### 6.5.4 运行应用程序

1. 将生成的二进制文件（如 `IPCF_Example_S32G274A_M7_0.bin`）复制到 SD 卡。
2. 在 u-boot 中停止。
3. 禁用数据缓存：`dcache off`。
4. SRAM 置零：`mw.q 0x34000000 0x0 0x100000`。
5. 加载二进制文件到 SRAM：
    
    ```
    fatload mmc 0:1 0x80000000 IPCF_Example_<PLATFORM>_M7_0.bin
    cp.q 0x80000000 0x34300000 0x600000
    ```
    
6. 启动 M7 核心：`startm7 0x34501000` (地址参见 sample.map)。
7. 启动 Linux (`boot`) 并运行 Linux 示例应用。
8. 发送消息并观察结果：`echo 10 > /sys/kernel/ipc-shm-sample-instance/ping`。

---

## 6.6 NXP S32 ARM RTD 的 IPCF 多实例示例应用程序

### 6.6.1 概述

此示例演示了 NXP S32 RTD 在 Cortex-M7 核心上运行的多实例共享内存通信。如果一个核心或应用程序崩溃，另一个示例会检测到并重新初始化所有实例。

### 6.6.3 构建应用程序

构建命令需指定核心，例如：

```
make platform=s32g274a core=m7_0 clean build
make platform=s32g274a core=m7_1 clean build
```

### 6.6.4 运行应用程序

1. 将二进制文件复制到 SD 卡。
2. 在 u-boot 中禁用缓存并置零 SRAM。
3. 分别为两个实例加载二进制文件到 SRAM（具体地址见手册）。
4. 启动 M7 核心（例如 S32G274A 使用 `startm7 0x34401000`）。
5. 启动 Linux 并加载多实例内核模块。
6. 分别向 instance0 和 instance1 发送消息并验证结果。