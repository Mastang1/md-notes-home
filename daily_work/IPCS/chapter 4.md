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
