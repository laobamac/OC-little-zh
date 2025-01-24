# 如何创建MMIO白名单
## 此文章暂时搁置，可以直接看我的视频学习！
## [「技巧三分钟」黑苹果定制MMIO白名单教程，让你的锐龙7000+和酷睿10th+通过EarlyBoot](https://www.bilibili.com/video/BV1zwwCeEERQ/?spm_id_from=333.1387.homepage.video_card.click)
- [关于](#about)
- [操作说明](#instructions)
	- [1. 生成启动日志](#1-generating-a-bootlog)
	- [2. 分析启动日志](#2-analyzing-the-bootlog)
	- [3. 将十六进制转换为十进制](#3-converting-hex-to-decimal)
	- [4. 填充并验证MMIO白名单](#4-populating-and-verifying-the-mmio-whitelist)
	- [5. 完成步骤](#5-finishing-touches)
- [资源](#resources)

---

## 关于
MMIO代表内存映射输入/输出（Memory-Mapped Input/Output）。它是一种在CPU与计算机外设之间执行I/O操作的方法。I/O设备的内存和寄存器被映射到（并与）地址值相关联。

MMIO白名单是一种安全功能，用于控制计算机系统中某些内存地址的访问权限，仅允许特定进程或设备访问，而拒绝所有其他访问。

在一些系统（如AMD Threadripper和Intel Ice Lake）中，这些MMIO区域需要在OpenCore的`config.plist`中被列入白名单，以便成功引导macOS。还有一个启动器特性叫做`DevirtualiseMmmio`，需要启用，以便找到需要列入白名单的MMIO区域。

本指南帮助你找到系统中的特定MMIO区域，并将其添加到MMIO白名单中。

## 操作说明

### 1. 生成启动日志
使用OCAT将OpenCore从RELEASE版本切换到DEBUG版本：

- 在OCAT中，从菜单栏选择“编辑 > OpenCore DEBUG”（勾选）
- 挂载EFI并打开`config.plist`
- 将当前的EFI文件夹备份到一个FAT32格式的USB闪存驱动器！
- 在`config.plist`中，做如下更改：
	- 启用`DevirtualiseMmio`（在`Booter/Quirks`中）
	- 将`Misc/Debug/Target`设置为：`67`
- 更新OpenCore文件和驱动程序
- 保存并重启

启动日志将以.txt文件存储在`EFI`文件夹中（即使启动失败也会生成日志）

### 2. 分析启动日志
- 挂载EFI
- 打开`OpenCore-xxxx-xx-xx.txt`文件
- 按 <kbd>CMD</kbd> + <kbd>F</kbd> 打开查找对话框
- 搜索“**MMIO devirt**”或仅搜索“**devirt**”
- 你可能会看到如下内容： </br>
	`MMIO devirt 0xF80F8000 (0x1 pages, 0x8000000000000001) skip 0` </br>
	`MMIO devirt 0xFED1C000 (0x4 pages, 0x8000000000000001) skip 0`

> [!注意]
> 
> 另外，你可以使用Corpnewt的[**MmioDevirt**](https://github.com/corpnewt/MmioDevirt)脚本来分析日志并生成MMIO白名单。在这种情况下，你可以跳过步骤3，直接将`MmioWhitelist`数组复制到`config.plist`中，保存并重启即可完成。

### 3. 将十六进制转换为十进制
为了将启动日志中找到的地址（没有被跳过的，即`skip 0`）添加到MMIO白名单中，我们首先需要将其转换为十进制：

- 运行Hackintool(没有macOS的话使用Windows的计算器-科学型即可，HEX->DEC)
- 点击“Calc”选项卡
- 使用“Value”部分将十六进制转换为十进制值。

**示例**：

十六进制 | 十进制
------------|----------
0xF80F8000 | 4161765376
0xFED1C000 | 4275159040

### 4. 填充并验证MMIO白名单
- 打开`config.plist`
- 将从启动日志中找到的十进制地址添加到`Booter/MmioWhitelist`中，如下所示：</br>![MMIOWhitelist01](https://user-images.githubusercontent.com/76865553/205931912-fee2d569-3265-47fb-a493-4c9556658805.png)
- 保存并重启
- 在macOS中检查新创建的启动日志：</br>
	`MMIO devirt 0xF80F8000 (0x1 pages, 0x8000000000000001) skip 1`</br>
	`MMIO devirt 0xFED1C000 (0x4 pages, 0x8000000000000001) skip 1`
- 如你所见，这两个MMIO区域现在被跳过了（`skip 1`），这意味着这些区域的内存现在再次可供UEFI使用。

### 5. 完成步骤
完成MMIO白名单创建和测试后，执行以下操作：

- 在OCAT中，重新选择“编辑 > OpenCore DEBUG”以取消勾选
- 挂载EFI并打开`config.plist`
- 禁用日志功能。将`Misc/Debug/Target`设置为`3`（默认）或`0`（完全禁用日志）
- 更新OpenCore文件和驱动程序，以切换回RELEASE版本
- 保存并重启

## 资源
- [**使用`DevirtuliseMmio`**](https://caizhiyuan.gitee.io/opencore-install-guide/extras/kaslr-fix.html#using-devirtualisemmio)
- [**DevirtualiseMmio和MMIO白名单**](https://www.macos86.it/topic/5511-let-talk-aboutdevirtualise-mmio-quirk-and-mmio-whitelist/)
