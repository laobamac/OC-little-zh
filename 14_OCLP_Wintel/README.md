# 在遗留Wintel系统上安装较新版本的macOS

## 关于
正如你所知道的，Dortania开发了[**OpenCore Legacy Patcher**](https://github.com/dortania/OpenCore-Legacy-Patcher)（OCLP），用于在老旧的Intel Core CPU（从第1代到第6代）Mac上安装并运行macOS 12及更新版本（Kaby Lake到Comet Lake的CPU仍被macOS 15支持）。它通过在目标系统上安装OpenCore引导加载器来实现，注入所需的设置和[附加kexts](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts)，使其能够：

- 在不受支持的Board-ID上引导更新的macOS版本，利用最适合所用CPU的原生SMBIOS（[更多详情](/09_Board-ID_VMM-Spoof/README.md)）。
- 重新启用macOS 13+（第1代到第3代Intel Core CPU）中的传统SMC CPU电源管理
- 修复由于禁用`SecureBootModel`、系统完整性保护（`SIP`）和Apple移动文件完整性（`AMFI`）而导致的系统更新问题

此外，OCLP在安装后应用[磁盘补丁](https://dortania.github.io/OpenCore-Legacy-Patcher/PATCHEXPLAIN.html#on-disk-patches)（“root-patches”），重新启用关键功能，如硬件图形加速（iGPU/GPU）以及WiFi/BT，以延长这些过时机器的生命周期。

你可能不知道的是：一些设置、kext和root-patches也可以在Wintel系统上使用。但是，`config.plist`的必要调整以及哪些kext需要注入（其中一些仅在Apple系统上需要）没有得到Dortania的官方文档支持，也不会在Discord上提供帮助。因此，我创建了详细的配置指南，以便你的老Hackintosh能够运行macOS 12及更新版本。

**对我们相关的补丁是**：

- iGPU驱动（[恢复图形加速和Metal图形API支持](https://khronokernel.github.io/macos/2022/11/01/LEGACY-METAL-PART-1.html)）
- 适用于传统（非metal）AMD和NVIDIA Kepler卡的GPU驱动
- 用于重新启用以前支持的Wi-Fi/Bluetooth卡的框架

## 最新OCLP状态更新
- [**macOS Sequoia OCLP更新说明**](/14_OCLP_Wintel/Sequoia_Notes.md)
- [**macOS Sonoma OCLP更新说明**](/14_OCLP_Wintel/Sonoma_Notes.md)

## 配置指南
以下是配置指南，用于更新你的OpenCore EFI和`config.plist`，使其能够运行macOS 13及更新版本：

- [**在第1代Intel Core系统上安装macOS 13+**](/14_OCLP_Wintel/Guides/Nehalem-Westmere-Lynnfield.md)
- [**在Sandy Bridge系统上安装macOS 13+**](/14_OCLP_Wintel/Guides/Sandy_Bridge.md)
- [**在Ivy Bridge系统上安装macOS 13+**](/14_OCLP_Wintel/Guides/Ivy_Bridge.md)
- [**在Haswell/Broadwell系统上安装macOS 13+**](/14_OCLP_Wintel/Guides/Haswell-Broadwell.md)
- [**在Skylake系统上安装macOS 13+**](/14_OCLP_Wintel/Guides/Skylake.md)
- [**通用CPU和SMBIOS指南**](/14_OCLP_Wintel/Guides/CPU_to_SMBIOS.md)

> [!重要]
>
> 从macOS 14.3.x更新到14.4.x及更高版本时，安装程序可能会早期崩溃。这与`SecureBootModel`有关，因此在安装期间应将其设置为`Disabled`（→详见[**解决方法**](/W_Workarounds/macOS14.4.md)部分）。

## （重新）启用功能
- [**修复macOS Sonoma+中的WiFi和蓝牙问题**](/14_OCLP_Wintel/Enable_Features/WiFi_Sonoma.md)
- [**在macOS Sequoia中启用`AirportItlwm.kext`**](/14_OCLP_Wintel/Enable_Features/AirportItllwm_Sequoia.md)
- [**如何在macOS Sequoia中禁用Gatekeeper**](/14_OCLP_Wintel/Guides/Disable_Gatekeeper.md)
- [**如何在macOS安装期间启用自动根补丁**](/14_OCLP_Wintel/Guides/Auto-Patching.md)
- [**强制启用OCLP中的根补丁**](/14_OCLP_Wintel/Enable_Features/Force-enable_Root-Patches.md)

## 故障排除
- [**运行macOS beta版本的注意事项**](/14_OCLP_Wintel/Beta_dos_donts.md)
- [**从失败的根补丁尝试中恢复**](/14_OCLP_Wintel/Guides/Reverting_Root_Patches.md)
- [**OCLP和macOS兼容性差距**](/14_OCLP_Wintel/Bridging_the_gap.md)
- [**通过终端触发macOS更新**](/14_OCLP_Wintel/macOS_Update_Terminal.md)
- [**解决macOS Sequoia中的睡眠问题**](https://www.insanelymac.com/forum/topic/360040-macos-15-sequoia-does-not-enter-sleep-mode-properly/#comment-2826474)（InsanelyMac上的讨论）

## 获取macOS安装器

有几种方法可以直接从Apple获取和下载macOS安装器。以下是其中一些：

1. **OpenCore Legacy Patcher**。它可以下载macOS 11+并创建USB安装器。
2. [**MIST**](https://github.com/ninxsoft/Mist)：GUI应用，用于下载macOS安装器和Apple Silicon固件。
3. **终端**。打开终端并输入以下命令：<br>
	`softwareupdate  --fetch-full-installer --list-full-installers`（获取安装器列表）<br>
	`softwareupdate  --fetch-full-installer --list-full-installer-version xx.xx`（替换xx.xx为你想下载的版本）

更多选项，请查看[**实用工具**](/C_Utilities_and_Resources#getting-macos)部分。

## 其他
- [**OCLP常见问题解答**](https://dortania.github.io/OpenCore-Legacy-Patcher/FAQ.html#application-requirements)
- [**OCLP变更日志**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/CHANGELOG.md)
- [**OCLP故障排除**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/docs/TROUBLESHOOTING.md)
- [**在macOS中安装Windows，无需Bootcamp**](/I_Windows/Install_Windows_NoBootcamp.md)
- [**非Metal版Apple应用程序合集**](https://archive.org/details/apple-apps-for-non-metal-macs)（Archive.org）
- [**macOS发布说明**](https://developer.apple.com/documentation/macos-release-notes)

## 贡献
##### 这段内容来自5T33Z0原话翻译
虽然我创建了这些指南，并在细节上非常用心，但总有改进的空间。在验证这些指南时，我只有iMac11,3（Lynnfield）、iMac12,2（Sandy Bridge）和一台Ivy Bridge笔记本进行测试。所以，如果你有任何建议或更新的指令来改进这些指南或工作流程，欢迎创建问题并告知我！
