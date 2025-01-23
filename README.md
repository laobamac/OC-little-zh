# OC-Little-zh: 适用于OpenCore的ACPI热补丁指南

<div align="center">
             <img src="logo.png" alt="OCLP-Mod Logo" width="256" />
             <h1>OC-little-zh</h1>
</div>

<div align=center>

[![OpenCore Version](https://img.shields.io/badge/支持的OpenCore版本:-≤1.0.4-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/支持的macOS:-≤15.3-white.svg) ![Last Update](https://img.shields.io/badge/最后更新:-25.01.23-blueviolet.svg)</br>

</div>

**目录**

## 前言

* [**免责声明**](#免责声明)
* [**关于OC-little-zh**](#关于)
* [**关于修改信息**](#关于修改)

## 主要

* [**ACPI基础知识和指南**](/00_ACPI/README.md)
* [**通过SSDT启用、添加设备/功能**](/01_Adding_missing_Devices_and_enabling_Features/README.md)
* [**禁用设备**](/02_Disabling_Devices/README.md)
* [**修复显卡**](/11_Graphics/README.md) (核显/独显)
* [**修复USB**](/03_USB_Fixes/README.md)
* [**修复重启、睡眠、唤醒**](04_Fixing_Sleep_and_Wake_Issues/README.md)
* [**修复外设问题**](/13_Peripherals/README.md)
* [**笔记本特殊补丁**](/05_Laptop-specific_Patches/README.md)
* [**修复CMOS相关**](/06_CMOS-related_Fixes/README.md)
* [**启动文件夹**](/07_BOOT_Folder/README.md)
* [**内核扩展加载顺序**](/10_Kexts_Loading_Sequence_Examples/README.md)
* [**MMIO白名单**](/12_MMIO_Whitelist/README.md)
* [**修复不正确的内存速度**](/15_RAM/README.md)

## 特殊
* [**在黑苹果使用OCLP补丁**](/14_OCLP_Wintel/README.md)
* [**跳过Board-ID验证、使用VMM覆写**](/09_Board-ID_VMM-Spoof/README.md)
* [**工具&资源**](/C_Utilities_and_Resources/README.md)

## 额外

### 配置相关
* [**配置提示和技巧**](/A_Config_Tips_and_Tricks/README.md)
* [**启动参数解读**](/H_Boot-args/README.md)
* [**OC怪癖**](/08_Quirks/README.md)
* [**分享你的EFI**](/M_EFI_Upload_Chklst/README.md)
* [**一些奇怪的内容**](/P_Else/README.md)

### OpenCore Auxiliary Tools (OCAT) 使用指南
* [**同步OpenCore**](/D_Updating_OpenCore/README.md)
* [**OpenCore部分值的计算**](/B_OC_Calculators/README.md)
* [**切换到不向其他系统注入ACPI的OpenCore Mod**](/O_OC_NO_ACPI/README.md)
* [**通过OCAT创建EFI**](/F_Desktop_EFIs/README.md)

### macOS相关
* [**终端命令**](/Terminal_Commands.md#readme)
* [**兼容性图表**](/E_Compatibility_Charts/README.md)
* [**修复系统更新通知**](/S_System_Updates/README.md)
* [**macOS 14.4 +安装解决方案**](/W_Workarounds/README.md)
* [**创建macOS多版本安装器**](/U_USB_Multi_installer/README.md)

### 其他系统
* [**Windows相关**](/I_Windows/README.md)
* [**启用Linux引导项**](/G_Linux/README.md)
* [**macOS 虚拟化**](/V_Virtualization/README.md)

### 杂项
* [**OpenCanopy主题**](/T_Themes/README.md)
* [**为AppleALC创建、修改layout-id**](/L_ALC_Layout-ID/README.md)
* [**编译精简后的Kext**](/J_Compiling_Kexts/README.md)
* [**通过系统报告调试**](/K_Debugging/README.md)
* [**创建多合一补丁 (`SSDT-ALL`)**](/N_SSDT-ALL/README.md)
* [**同时使用Clover与OpenCore**](/R_BootloaderChooser/README.md)

___

## 免责声明
1. OC-Little-zh 并非教你如何配置OpenCore来安装macOS，需要黑苹果相关教程请移步 Dortania 的 [**OpenCore Install Guide**](https://dortania.github.io/OpenCore-Install-Guide/) 或其他教程文档！相反，它是一个补充资源，为与黑苹果提供ACPI修复指南，以及有关的技巧、方法。
2. 此仓库中撰写的文档目的是让用户可以通过修复ACPI等操作使 macOS 的 **正常工作** ，并且不会违反 ACPI 规范。因此，**OC-Little-zh** 不提倡使用破坏ACPI规范的方法——比如直接修补“DSDT”，因为它不是获得“真正原生的macOS”体验的**正确**措施。事实上，情况恰恰相反，[**正如 insanelymac的一篇帖子**](https://www.insanelymac.com/forum/topic/352881-when-is-rebaseregions-necessary/#comment-2790870)，某些情况下仍需对DSDT进行重定位，以使macOS正常启动。
	
## 关于
用于 OpenCore Boot Manager 的指南、ACPI 热补丁和重命名集合，基于 [**OC-Little by Daliansky**](https://github.com/daliansky/OC-little)和[**OC-Little-Translated by 5T33Z0**](https://github.com/5T33Z0/OC-Little-Translated)，并加以汉化、补充、修正。

除了ACPI指南外，此存储库还包含与 OpenCore Package 和 Dortania 的 OpenCore Guides 提供的指南相辅相成的其他指南。它涵盖了大量的黑苹果相关内容，包括：

- ACPI 基础知识
- 启用设备、功能
- 修复睡眠/唤醒和其他电源相关问题
- 定制USB端口
- 修复笔记本电脑的特殊问题
- 修复显卡相关问题
- 配置相关提示
- 在黑苹果上使用OCLP补丁

以上内容在目录均提到。本文主要对象是OpenCore，但大部分内容页适用于Clover，可以加以参考。

### 关于修改
- 为了保证文档教学的准确性，所有文字均为我人工翻译，并尽量让文档适合中文的语言习惯。这是一个工作量巨大的进程，如果有不正确的地方，请联系我。
- 根据问题、组件、方法等的类型，将存储库重构为更合理的（子）部分和类别。
- 重写了令人困惑/误导的整个部分（例如“ACPI”和“定制USB端口”）
- 增加了缺失的描述
- 我并没有非常充裕的时间，高中的生活繁忙且杂乱。我会在必要时对文档进行更新来确保其适用于最新版本的OpenCore、macOS。如果我没有立刻回复你的请求，请耐心等待。


## 贡献
 如果您想为本文档提供的信息做出贡献，以改进/扩展它，请创建issue或加入QQ群来联系我，注明到章节/部分并描述您想要添加，更改，更正或扩展的内容。

## STAR 历史

<a href="https://star-history.com/#laoamac/OC-little-zh&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=laobamac/OC-little-zh&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=laobamac/OC-little-zh&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=laobamac/OC-little-zh&type=Date" />
 </picture>
</a>

## 鸣谢
- [**OC-Little by Daliansky**](https://github.com/daliansky/OC-little) 和 [**OC-Little-Translated by 5T33Z0**](https://github.com/5T33Z0/OC-Little-Translated)

<details>
<summary><strong>5T33Z0's 5H0UT 0UT5</strong></summary>

> - Thanks to al the [**contributors**](https://github.com/5T33Z0/OC-Little-Translated/graphs/contributors) for improving and expanding the repo! Additional credits for contributors outside of the github realm are listed in the respective chapters/sections.
> - sascha_77 for Kext Updater, ANYmacOS and helping me to unbrick my Lenovo T530 BIOS!
> - Apfelnico for introducing me to ASL/AML Basics
> - Bluebyte for having good conversations
> - Cyberdevs from insanelymac for his Z170X Gaming 5 Guide and for being a prime example of a Moderator who treats others with respect!
</details>

<details>
<summary><strong>Daliansky's original Credits</strong></summary>

> - Special credit to:
> 	- @XianWu write these ACPI component patches that useable to OpenCore
> 	- @Bat.bat, @DalianSky, @athlonreg, @iStar丶Forever their proofreading and finalization.
> - Credits and thanks to：
> 	- @冬瓜-X1C5th
> 	- @OC-xlivans
> 	- @Air 13 IWL-GZ-Big Orange (OC perfect)
> 	- @子骏oc IWL
> 	- @大勇-小新air13-OC-划水小白
> 	- @xjn819
> 	- Acidanthera for maintaining OpenCorePkg
</details>
