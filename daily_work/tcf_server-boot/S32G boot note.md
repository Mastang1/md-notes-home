# 技术笔记：NXP S32G3 启动流程与安全构建机制解析

## 一、 核心概念解析

在深入探讨 S32G3 的启动流水线之前，需明确以下核心组件与术语。S32G3 是一款面向汽车网络和域控制器的多核异构芯片，其启动流程深度依赖于**硬件安全引擎（HSE, Hardware Security Engine）**。

- **BootROM（固化引导代码）**：
    
    芯片上电后首个执行的程序。在 S32G3 中，BootROM 运行在 **HSE_M7 核心**（HSE 专属的 Cortex-M7 核）上，而非用户的应用核。它由 NXP 出厂时固化在内部 ROM 中，不可篡改，构成了整个系统的**信任根（Root of Trust）**。
    
- **IVT (Image Vector Table, 镜像向量表)**：
    
    BootROM 从外部存储介质固定偏移地址读取的首个数据结构。它相当于启动过程的“导航图”，包含指向 DCD、HSE 固件和用户 Bootloader 等组件的物理地址指针及配置字（如是否开启安全启动）。
    
- **DCD (Device Configuration Data, 设备配置数据)**：
    
    （注：有时被误拼为 CDCD）。由于 BootROM 运行时外部大容量 RAM（如 LPDDR4）尚未初始化，DCD 充当了早期硬件初始化的配置脚本。BootROM 会解析并执行 DCD 中的寄存器写操作指令，主要用于**初始化外部 DDR 内存、基础时钟和引脚**。
    
- **Blob / HSE FW (HSE 固件/二进制块)**：
    
    指 HSE 硬件安全引擎的固件。HSE 内部有受保护的安全存储区，初始阶段需将 HSE 固件（NXP 提供的加密包，通常称为 Pink Image）打包在启动镜像中。BootROM 会将其解密并安装/更新到 HSE 的内部安全 Flash 中。
    
- **Bootloader (用户引导加载程序)**：
    
    用户提供的引导代码。在多核系统中通常分级（如 TF-A BL2/BL31 -> U-Boot，或 Cortex-M7 的 Autosar 启动代码）。
    

## 二、 启动镜像所在的 Memory (存储介质)

芯片上电复位后，BootROM 会读取启动模式引脚（BMODE）或内部熔丝（Fuses）的状态，决定从何处加载启动镜像。支持的介质包括：

1. **QSPI NOR Flash**：最常见的启动介质。BootROM 初始以 1-bit SDR 模式读取，后续由 Bootloader 切换至高速模式（如 8-bit DDR/Octal）。
    
2. **eMMC / SD Card**：通过 uSDHC 接口启动。
    
3. **串行启动 (Serial Boot)**：通过 UART 或 CAN 接口将程序下载到内部 SRAM，通常用于工厂裸板烧录、调试或“救砖”。
    

## 三、 S32G3 详细启动流程（信任链构建）

整个启动过程是一个逐级验证的**信任链（Chain of Trust）**构建过程，涵盖五个主要阶段：

### 阶段 1：硬件复位与 BootROM 启动

1. 芯片上电，硬件复位解除。
    
2. 仅 **HSE_M7 核心**苏醒，开始执行内部的 **BootROM**。
    
3. BootROM 根据 BMODE/Fuses 初始化对应的外部存储控制器（如 QSPI）。
    

### 阶段 2：解析 IVT 与执行 DCD

4. BootROM 从 Flash 预设的首地址读取 **IVT**。
    
5. 解析 IVT 结构，验证其合法性（若开启安全启动，还会验证 IVT 的 MAC/签名）。
    
6. BootROM 根据 IVT 指针找到 **DCD 数据块**。
    
7. 逐条执行 DCD 脚本，完成 LPDDR4 内存和 PLL 等基础硬件的初始化。
    

### 阶段 3：HSE 固件加载与安全验证（核心加密环节）

8. BootROM 检查 IVT 中的 `BOOT_SEQ` 配置字，确认系统是否要求 **Secure Boot（安全启动）**。
    
9. BootROM 根据 IVT 找到 **HSE Blob (HSE FW)**。
    
10. BootROM 使用芯片内部自带的根密钥对 HSE 固件进行**解密和验签 (Authentication)**。若验证失败，启动流程终止。
    
11. 验证通过后，BootROM 将系统控制权移交给运行在 HSE_M7 上的 **HSE 固件**。
    

### 阶段 4：验证并加载 Customer Bootloader

12. HSE 固件接管后，根据配置从外部 Flash 读取用户的 **Bootloader**（如 TF-A BL2）。
    
13. HSE 利用其内部安全存储的密钥目录（Key Catalogs）对 Bootloader 进行**非对称加密签名验证**（如 RSA 或 ECC 算法）。
    
14. 认证通过后，HSE 将 Bootloader 拷贝到已初始化的 LPDDR4 或内部 SRAM 中。
    
15. HSE 释放对应应用核心（如 Cortex-A53_0 或应用 Cortex-M7）的复位信号，应用核开始执行 Bootloader。
    

### 阶段 5：操作系统启动

16. A53 核上的 Bootloader（如 U-Boot）启动后，继续延续信任链。
    
17. U-Boot 读取 Linux Kernel 镜像，并通过 MU（消息单元）调用底层 HSE 的加密 API 对 Kernel 进行身份验证。
    
18. 验证无误，正式拉起操作系统或业务代码。
    

---

## 四、 补充：安全启动模式下的镜像打包过程 (Image Packaging)

在 Secure Boot 模式下，最终烧录到 Flash 中的镜像（通常是一个 `.bin` 文件）并非单一代码，而是多个组件按特定内存布局组装，并附加数字签名的数据包。

### 1. 镜像打包所需的组件

- **用户 Bootloader 二进制文件**（如编译好的 `bl2.bin` 或 `u-boot.bin`）。
    
- **DCD 配置表**（通常由配置工具生成的二进制格式数据）。
    
- **HSE FW (Pink Image)**：NXP 官方提供的加密 HSE 固件包。
    
- **IVT 与 Boot Data**：描述整个镜像布局的头部信息。
    
- **数字证书/签名数据**：用于验证 Bootloader 的签名块。
    

### 2. 打包与签名工具链

NXP 提供了配套工具来完成这一过程：

- **S32 Design Studio (S32DS) - IVT Configuration Tool**：用于图形化配置 IVT 参数、DCD 寄存器，并生成二进制的 IVT/DCD 块。
    
- **HSE 签名工具 (如 NXP 提供的 Python 脚本或 CST - Code Signing Tool)**：用于对用户的 Bootloader 生成哈希并使用私钥进行签名。
    

### 3. 具体打包步骤

- **步骤一：生成并提取基础二进制**
    
    编译应用代码生成 Bootloader 二进制文件；确认所需的 HSE FW 版本文件。
    
- **步骤二：配置与生成 DCD**
    
    通过芯片的引脚/时钟配置工具（如 S32 Configuration Tool）生成 DDR 和时钟的初始化寄存器序列，导出为 DCD 二进制文件。
    
- **步骤三：签名应用 Bootloader**
    
    1. 准备一对非对称密钥（公钥和私钥）。公钥后续需要被预配（Provision）到 S32G3 芯片的 HSE 安全存储区中。
        
    2. 使用私钥和 NXP 签名工具，对 Bootloader 二进制文件进行签名，生成**签名块（Signature Block）**。
        
- **步骤四：配置 IVT 并生成总镜像**
    
    1. 打开 IVT Configuration Tool。
        
    2. 导入 Bootloader 二进制、生成的签名块、DCD 二进制文件以及 HSE FW。
        
    3. 配置工具会自动分配这些组件在 Flash 中的相对偏移地址（例如：IVT 通常在偏移 0x0 处，Bootloader 在 0x1000 处，等等）。
        
    4. 勾选启用 Secure Boot 选项，配置 Boot 启动目标核（如 A53_0）。
        
    5. 工具执行合成操作，最终拼接生成一个完整的、包含签名数据的 `flash_image.bin`。
        

### 4. 典型 Flash 内存布局示例 (Secure Boot)

合成后的镜像烧录至 QSPI Flash 时，其典型的物理结构如下：

Plaintext

```
[ Flash 起始地址 (例如 0x0000_0000) ]
-----------------------------------------
| IVT (镜像向量表) + Boot Data            |  <- 包含指向后续组件的指针
-----------------------------------------
| DCD (设备配置数据)                      |  <- 用于 BootROM 初始化 LPDDR4
-----------------------------------------
| HSE Firmware (Pink Image)               |  <- 供 BootROM 解密并安装到 HSE
-----------------------------------------
| Bootloader Signature Block (签名块)     |  <- 包含公钥证书和数字签名
-----------------------------------------
| Application Bootloader (如 TF-A/U-Boot) |  <- 被签名保护的用户引导代码
-----------------------------------------
| Application Data / OS Image             |  <- (可选) OS 镜像，后续由 U-boot 校验
-----------------------------------------
```

完成打包并烧录至 Flash 后，芯片上电即可按照前文所述的“阶段1至阶段5”流程，由硬件自动执行密码学校验和安全启动。