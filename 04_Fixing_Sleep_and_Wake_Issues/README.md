# 修复睡眠和唤醒问题

本节包含了解决与睡眠和唤醒相关的常见问题的修复方法，特别是在笔记本电脑上，但不限于此。以下是涵盖的内容：

## [`PTSWAKTTS`](/04_Fixing_Sleep_and_Wake_Issues/PTSWAK_Sleep_and_Wake_Fix/README.md)：全面的睡眠和唤醒修复

这个SSDT是修复大多数睡眠和唤醒问题的核心，通常与本节中的其他补丁一起使用。它由二进制重命名和ACPI热修复(SSDT)组成。

### 补丁如何工作

- 方法`_PTS`(准备睡眠)，`_Wak`(唤醒)和`_TTS`(转换到状态)通过二进制重命名进行修改。
- ***SSDT-PTSWAKTTS*** 重新定义了原始方法，如果这些方法中的任何一个被触发(无论是由睡眠计时器自动触发，按下睡眠/电源按钮，还是通过菜单)，它会捕获这些事件并处理睡眠/唤醒(与其他额外的SSDT一起使用)。

> [!WARNING]
> 
> ***SSDT-PTSWAKTTS.aml*** 必须在本节的其他一些热修复之前加载。关于每个补丁的详细信息，请查看相应修复的 `README` 文件。

## 修复 [`PNP0C0E 睡眠问题`](/04_Fixing_Sleep_and_Wake_Issues/PNP0C0E_Sleep_Correction_Method/README.md)

如果按下电源或睡眠按钮导致即时重启或关机，则需要此补丁。为了使其生效，必须与 ***SSDT-PTSWAKTTS*** 一起使用。

## 修复即时唤醒问题：[`0D/6D 补丁`](/04_Fixing_Sleep_and_Wake_Issues/060D_Instant_Wake_Fix/README.md)

这个补丁用于修复即时唤醒问题，即系统在进入睡眠状态后立刻被唤醒。通常是由于某些组件/设备阻止系统进入睡眠/休眠状态。

## 修复 [`AOAC 睡眠问题`](/04_Fixing_Sleep_and_Wake_Issues/Fixing_AOAC_Machines/README.md)

这些补丁用于修复最近使用**始终开启，始终连接**(`AOAC`)技术的笔记本电脑的睡眠和待机问题。

## 配置 [`ASPM`](/04_Fixing_Sleep_and_Wake_Issues/Setting_ASPM_Operating_Mode/README.md)

**ASPM**(活动状态电源管理)是一种在系统级别支持的电源链接管理方案。在ASPM管理下，**PCI设备**尝试在空闲时进入省电模式。如果它们打断了睡眠，你可以修改蓝牙/WiFi或其他设备的活动电源状态。

## 更改 [休眠模式](/04_Fixing_Sleep_and_Wake_Issues/Changing_Hibernation_Modes/README.md)

用于更改与**系统电源管理**相关的设置的`pmset`命令，例如睡眠/休眠模式。

## 注意事项和其他资源
- 在应用任何这些热修复之前，请确保你没有使用Dortania或OpenCore包中提供的通用ACPI表，因为它们通常包含额外的设备、设备名称和路径，以覆盖多种场景(例如`SSDT-PLUG`)。相反，应该根据你的系统的特定需求定制它们，或者使用[**SSDTTime**](https://github.com/corpnewt/SSDTTime)生成自己的表。仅此一项就可以防止睡眠和唤醒问题。
- 查看Dortania的后安装指南，了解更多关于[**修复睡眠问题**](https://github.com/dortania/OpenCore-Post-Install/blob/master/universal/sleep.md)的信息。
- [**修复电源管理、睡眠和休眠问题**](https://github.com/zx0r/HackintoshBible/blob/main/PowerManagement/README.md)(针对macOS Sonoma和Sequoia，由zx0r提供)。
- 深入了解[**Darkwake**](https://www.insanelymac.com/forum/topic/342002-darkwake-on-macos-catalina-boot-args-darkwake8-darkwake10-are-obsolete/)，它的作用和误区。
- 在研究这些修复如何工作时，我发现本节中使用的SSDT和二进制重命名基本上是“逆向工程”`DSDT`补丁，这些补丁由RehabMan在maciASL的DSDT修补引擎中创建。它们也可以在他的“笔记本电脑DSDT修补”库中找到：
	- **睡眠和唤醒** 修复: https://github.com/RehabMan/Laptop-DSDT-Patch/tree/master/system
	- **0D/6D** 修复: https://github.com/RehabMan/Laptop-DSDT-Patch/tree/master/usb
	- **合盖** 修复: https://github.com/RehabMan/Laptop-DSDT-Patch/tree/master/misc
