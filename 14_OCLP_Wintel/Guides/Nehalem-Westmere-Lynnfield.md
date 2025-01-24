# 在老旧Intel CPU上安装macOS Ventura及更高版本

[![OpenCore版本](https://img.shields.io/badge/OpenCore版本：-0.9.4+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/支持的macOS:-≤15.2-white.svg)

<details>
<summary><b>目录</b> (点击展开)</summary><br>

- [在老旧Intel CPU上安装macOS Ventura及更高版本](#在老旧intel-cpu上安装macos-ventura及更高版本)
	- [关于](#关于)
		- [免责声明](#免责声明)
	- [CPU要求](#cpu要求)
		- [潜在的候选CPU](#潜在的候选cpu)
		- [补丁原理](#补丁原理)
	- [注意事项与限制](#注意事项与限制)
	- [准备工作](#准备工作)
		- [更新OpenCore和kexts](#更新opencore和kexts)
	- [配置修改](#配置修改)
	- [测试更改](#测试更改)
		- [调整SMBIOS](#调整smbios)
			- [从macOS Big Sur 11.3+升级时](#从macos-big-sur-113升级时)
			- [从macOS Catalina或更早版本升级时](#从macos-catalina或更早版本升级时)
	- [macOS安装](#macos安装)
		- [获取macOS](#获取macos)
		- [选项1：从macOS 11.3或更新版本升级](#选项1从macos-113或更新版本升级)
		- [选项2：从macOS Catalina或更早版本升级](#选项2从macos-catalina或更早版本升级)
	- [安装后](#安装后)
		- [安装其他GPU的驱动程序](#安装其他gpu的驱动程序)
		- [验证SMC CPU电源管理](#验证smc-cpu电源管理)
			- [优化CPU电源管理](#优化cpu电源管理)
		- [移除/禁用启动参数](#移除禁用启动参数)
		- [验证AMFI是否开启](#验证amfi是否开启)
	- [OCLP与系统更新](#oclp与系统更新)
		- [在系统更新后重新应用root补丁](#在系统更新后重新应用root补丁)
		- [OCLP 应用程序更新通知](#oclp-应用程序更新通知)
	- [注意事项](#注意事项)
	- [进一步资源](#进一步资源)
	- [致谢](#致谢)

</details>

## 关于
除了在不受支持的平台（如Sandy/Ivy Bridge，Haswell/Broadwell和Skylake）上安装macOS Ventura及更新版本外，使用OpenCore和OpenCore Legacy Patcher（OCLP）还可以在第一代Intel Core CPU上安装和运行macOS。

| ⚠️ 重要状态更新 |
|:----------------------------|
| 一切正常。

### 免责声明
本指南旨在提供有关如何调整EFI和config.plist的通用信息，以便在不受支持的Wintel系统上安装和运行macOS Monterey及更高版本。由于我只有一台使用i7-870的iMac11,3来检查它所使用的kext和设置，因此此指南应视为实验性质！我创建它是为了阐明如何尝试在古老硬件上安装macOS 13+的基本原理——并不是所有系统都能成功！因此，请避免使用“报告问题”功能来寻求个性化的配置修复帮助。此类问题报告将立即关闭！

## CPU要求
为了检查你的系统是否有可能安装并运行macOS，你需要验证你的CPU是否满足最低要求：

1. **64位**的Intel CPU型号，且该型号曾用于Apple的Mac（即不支持Atom、Celeron和Pentium）
2. CPU必须支持**SSE4.2**，以便启动macOS 10.14 Mojave及更高版本

### 潜在的候选CPU
第一代Intel Core CPU。请查看[此列表](/14_OCLP_Wintel/CPU_to_SMBIOS.md)以了解你的CPU是否被支持以及适合的SMBIOS。

### 补丁原理
在macOS Ventura及更高版本中，Kaby Lake之前的CPU系列已被弃用。对于Sandy Bridge及更早的系列，主要受影响的是：

- CPU指令集：
	- 缺少AVX2.0，无法执行加密任务
	- 缺少[rdrand指令](https://github.com/reenigneorcim/SurPlus)
- CPU电源管理（移除了`ACPI_SMC_PlatformPlugin`）
- iGPU、dGPU和Metal支持
- USB 1.1支持
- 传统以太网和Wi-Fi支持

因此，我们将准备配置，添加所需的补丁、设置和kext，以便安装和运行macOS 13+，然后在后期安装中使用OpenCore Legacy Patcher来补充iGPU/GPU驱动程序（以及其他缺失的部分）。

## 注意事项与限制
在尝试在不受支持的系统上安装macOS Monterey及更高版本之前，您需要了解以下内容：

- ⚠️ **备份**你当前工作的EFI文件夹到一个FAT32格式的USB闪存驱动器，以防万一出现问题，因为我们需要修改配置和EFI文件夹的内容。
- **USB支持**：Ventura完全弃用了USB 1.1协议，并且删除了一些USB端口映射。所以你可能需要手动重新映射它。
- **iGPU/GPU**：
	- 检查你的iGPU/GPU是否被OCLP支持。虽然可以在后期安装中为Intel、NVIDIA和AMD卡添加驱动，但[支持的卡片列表有限](https://dortania.github.io/OpenCore-Legacy-Patcher/PATCHEXPLAIN.html#on-disk-patches)
	- AMD Navi系列卡（Radeon 5xxx和6xxx）不能在Haswell之前的CPU上使用，因为它们需要AVX 2.0指令集，而只有Haswell及更新的处理器支持。
- **网络**：
	- 对于**以太网**，有为传统LAN控制器准备的kexts，[可以在这里找到](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Ethernet)
	- **WiFi和蓝牙**：
		- 对于启用Broadcom Wi-Fi/BT卡，你将需要一组不同的[补丁kexts](/10_Kexts_Loading_Sequence_Examples#example-7-broadcom-wifi-and-bluetooth)来加载，需要通过`MinKernel`和`MaxKernel`设置来控制。在macOS 12.4及更高版本中，`bluetoothd`中引入了新的地址检查，如果两个蓝牙设备有相同的地址，将触发错误。可以通过添加启动参数`-btlfxallowanyaddr`（由[BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM) kext提供）来绕过此问题。
		- 同样适用于[Intel WiFi/BT](/10_Kexts_Loading_Sequence_Examples#example-8a-intel-wifi-airportitlwm-and-bluetooth-intelbluetoothfirmware)卡，使用[OpenIntelWireless](https://github.com/OpenIntelWireless) kexts。
		- [在macOS Sonoma中启用WiFi](/14_OCLP_Wintel/Enable_Features/WiFi_Sonoma.md)需要额外的kext，并且在后期安装时需要应用根补丁！
- **安全性**：使用OCLP修改系统需要禁用SIP、Apple Secure Boot和AMFI，因此在安全性方面存在一些妥协。
- **系统更新**：
	- 在应用根补丁后，增量（或增量）更新将不可用。每次都会下载整个macOS安装程序（大约12GB）！
	- ⚠️ 不要安装**安全响应更新**（RSR），它们在Haswell之前的系统上将无法安装。更多信息[**这里**](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1019)。
- **其他**：查看下面的链接，了解有关macOS 12及更高版本中已移除的组件/功能的深入文档，以及这些变化对Kaby Lake之前的系统的影响。但请记住，这些文档是针对真实Mac编写的，因此某些问题对Wintel系统不适用。
	- [OpenCore Legacy Patcher对macOS Sonoma的支持状态](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1076)
	- [OpenCore Legacy Patcher对macOS Ventura的支持状态](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/998)
	- [Legacy Metal支持和macOS Ventura/Sonoma](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1008)

## 准备工作
我假设你已经有一个适用于你的遗留系统的OpenCore配置。如果没有，请按照Dortania的[OpenCore安装指南](https://dortania.github.io/OpenCore-Install-Guide/prerequisites.html)创建一个。下面的说明仅为额外步骤，要求你能够安装和引导macOS Monterey及更高版本。

### 更新OpenCore和kexts
更新OpenCore到0.9.2或更高版本（强制）。因为在0.9.2之前，[`AppleCpuPmCfgLock` Quirk会在macOS Ventura运行时跳过](https://github.com/acidanthera/OpenCorePkg/commit/77d02b36fa70c65c40ca2c3c2d81001cc216dc7c)，因此无法注入重新启用SMC CPU电源管理所需的kexts，系统无法启动，除非你有一个（修改过的）BIOS，能够禁用CFG锁。要检查你当前使用的OpenCore版本，请在终端运行以下命令：

```shell
nvram 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102:opencore-version
```

- 将Kexts更新到最新版本，以避免与macOS产生兼容性问题
- 如果在安装过程中遇到问题，请禁用非必要的Kexts

## 配置修改
以下是安装macOS Monterey或更新版本所需的配置修改，以准备你的config.plist和EFI文件夹。

:bulb: 如果系统（或其组件）在安装后无法正常工作，请参考OCLP的[补丁文档](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/docs/PATCHEXPLAIN.md)，查看是否需要其他设置或kext。

配置部分 | 操作 | 描述
:-------------:| ------ | ------------
**`ACPI/Add`** | 添加 [**SSDT-CPBG.aml**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/ACPI/SSDT-CPBG.aml) | 修复Arrandale、Lynnfield和Clarkdale的`CPBG`。如果你的系统的ACPI表中没有此设备，可能不需要此补丁。
**`ACPI/Quirks`** | 根据需要设置Quirks | 请参考[OpenCore安装指南](https://dortania.github.io/OpenCore-Install-Guide/)获取与你的CPU系列相关的设置。
**`Booter/Patch`**| **添加**并**启用**以下来自OCLP配置的Booter补丁：<ul> <li> [**"跳过Board-ID检查"**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist#L220-L243) | <ul><li> 跳过Board-ID检查。 <li> 结合`ResterictEvents` kext，允许： <ul> <li> 使用最适合你的CPU的本地SMBIOS启动macOS <li> 在不受支持的系统上安装系统更新 </ul> <li> 更多[细节](/09_Board-ID_VMM-Spoof#booter-patches)
**`Kernel/Add`** 和 <br>**`EFI/OC/Kexts`** |**添加以下Kexts**：<ul><li>[**AutoPkgInstaller**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera)<li>[**AMFIPass**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera) (`MinKernel`: `21.0.0`) <li> [**ASPP-Override**](https://github.com/dortania/OpenCore-Legacy-Patcher/raw/main/payloads/Kexts/Misc/ASPP-Override-v1.0.1.zip) (`MinKernel`: `21.4.0`) <li> [**CryptexFixup**](https://github.com/acidanthera/CryptexFixup) (`MinKernel`: `22.0.0`) <li> [**RestrictEvents**](https://github.com/acidanthera/RestrictEvents) (`MinKernel`: `20.4.0`) <li> [**AppleIntelCPUPowerManagement**](https://github.com/dortania/OpenCore-Legacy-Patcher/raw/main/payloads/Kexts/Misc/AppleIntelCPUPowerManagement-v1.0.0.zip) (`MinKernel`: `22.0.0`)<li> [**AppleIntelCPUPowerManagementClient**](https://github.com/dortania/OpenCore-Legacy-Patcher/raw/main/payloads/Kexts/Misc/AppleIntelCPUPowerManagementClient-v1.0.0.zip) (`MinKernel`: `22.0.0`)  <li> [**NoAVXFSCompressionTypeZlib.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Misc) (`MinKernel`: `21.5.0`; `MaxKernel`: `21.99.99`) <li> [**NoAVXFSCompressionTypeZlib-AVXpel.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Misc) (`MinKernel`: `22.0.0`) <li> [**FeatureUnlock**](https://github.com/acidanthera/FeatureUnlock)（可选） </ul> </ul> **WiFi**（可选） <ul><li>[**IOSkywalk.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/e21efa975c0cf228cb36e81a974bc6b4c27c7807/payloads/Kexts/Wifi/IOSkywalkFamily-v1.0.0.zip) (`MinKernel`: `23.0.0`)  <li>[**IO80211FamilyLegacy.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/e21efa975c0cf228cb36e81a974bc6b4c27c7807/payloads/Kexts/Wifi/IO80211FamilyLegacy-v1.0.0.zip)（包含`AirPortBrcmNIC.kext`，确保也注入此kext） (`MinKernel`: `23.0.0`) </ul> **从EFI/OC/Kexts和config中删除以下Kexts**（如果存在）： <ul><li> **CPUFriend** <li> **CPUFriendDataProvider**| <ul> <ul><li>**AutoPkgInstaller**: 用于在macOS安装过程中自动应用根补丁。需要准备安装程序（[**详情**](/14_OCLP_Wintel/Guides/Auto-Patching.md))<li> **AMFIPass**: 来自OCLP的kext。允许在不禁用AMFI的情况下启动macOS 12+。 <li> **ASPP-Override**: 优先使用插件类型0，而不是插件类型1，以便SMC CPU电源管理工作。 <li> **Cryptexfixup**: 在没有AVX 2.0支持的系统上安装和启动macOS 13+所需（→ [OCLP支持问题 #998](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/998)) <li> **NoAVXFSCompressionTypeZlib** kexts: 防止Zlib内核崩溃的解决方法 <li> **RestrictEvents**: 强制启用VMM SB模型，允许在macOS 11.3或更新版本上对不受支持的模型进行OTA更新。需要额外的NVRAM参数。 <li> **AppleIntelCPUPowerManagement** kexts: 用于重新启用SMC CPU电源管理（[更多详情](/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management/CPU_Power_Management_(Legacy)#re-enabling-acpi-power-management-in-macos-ventura))<li> **FeatureUnlock**: 可以启用/禁用夜间模式、Sidecar、Universal Control等功能。在较旧的系统上，当硬件和SMBIOS要求不满足时，用于*禁用*这些功能。 <li> **WiFi Kexts**: 用于macOS Sonoma。重新启用现代Wi-Fi：BCM94350、BCM94360、BCM43602、BCM94331和BCM943224。传统Wi-Fi：Atheros芯片组、Broadcom BCM94322、BCM94328。
**`Kernel/Block`**| 阻止 `com.apple.iokit.IOSkywalkFamily`: <br> ![](https://user-images.githubusercontent.com/76865553/256150446-54079541-ee2e-4848-bb80-9ba062363210.png)| 阻止macOS的IOSkywalk kext，以便使用注入的kext。仅在使用“现代”WiFi卡时需要（→ [Wi-Fi修补指南](/14_OCLP_Wintel/Enable_Features/WiFi_Sonoma.md)）。 
**`Kernel/Emulate`** | 禁用 `DummyPowermanagement`（如果启用） | 如果你注入了重新启用ACPI CPU电源管理的kext，而此设置仍然启用，系统将在启动后10到15秒内冻结。
**`Kernel/Patch`** | 添加并启用以下来自[**OCLP**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist)的内核补丁（同时应用`MinKernel`和`MaxKernel`设置）： <ul>  <li> **强制启用损坏的密封上的FileVault**（可选） <li> **"SurPlus v1 - 第1部分"** <li> **"SurPlus v1 - 第2部分"** <li> **"禁用库验证执行"** <li> **"禁用_vnode_check_signature中的_csr_check()"** <li> **修复PCI总线枚举（Ventura）** <li> **修复PCI总线枚举（Sonoma）** </ul> **备注**：VMMBoard-ID内核补丁现在不再需要，因为`RestrictEvents` kext可以通过`sbvmm` NVRAM条目启用VMMBoard-ID！
**`NVRAM/Add/...-4BCCA8B30102`** | **添加以下键**： <ul> <li> **键**: `OCLP-Settings`<br>**类型**: 字符串 <br>**值**: `-allow_amfi` <li> **键**: `revblock` <br> **类型**: 字符串 <br> **值**: `media`<li>  **键**: `revpatch` <br> **类型**: 字符串 <br> **值**: `sbvmm,f16c`| <ul> <li> OCLP和RestrictEvents的设置。 <li> `media`: 阻止Ventura+中的`mediabranalysisd`服务（针对Metal 1 GPU） <li>`sbvmm,f16c` &rarr; 启用OTA更新并解决macOS 13中的图形问题（查看RestrictEvents文档了解详情）|
**`Misc/Security`**| <ul> <li> **SecureBootModel**: `Disabled` <li> **Vault**: `Optional`| 在为AMD和NVIDIA GPU修补图形驱动时必需。对于Intel HD图形，设置为`Default`可能有效。请自行尝试。
**`NVRAM/Delete/...-4BCCA8B30102`** (数组) | **添加以下字符串**： <ul> <li>  `OCLP-Settings` <li> `revblock` <li> `revpatch` | 删除这些NVRAM参数的条目，避免每次更改时需要重置NVRAM。  
**`NVRAM/Add/...-FE41995C9F82`** | **更改** **`csr-active-config`** 为：<ul><li>**`03080000`** <li> 当使用NVIDIA GPU时，设置为： **`030A0000`** </ul>**添加以下**`boot-args`: <ul><li> **`amfi_get_out_of_my_way=0x1`** 或 **`amfi=0x80`**（相同）<li> **`ipc_control_port_options=0`** <li> **`-disable_sidecar_mac`** </ul>**针对GPU的可选启动参数**（根据GPU供应商选择）： <ul><li> **`-radvesa`** <li> **`nv_disable=1`** <li> **`ngfxcompat=1`** <li> **`ngfxgl=1`** <li> **`nvda_drv_vrl=1`** <li> **`agdpmod=vit9696`** <li> **`-nokcmismatchpanic`** <li> **`vsmcgen=1`** 或 **`vsmcgen=2`** |
**`EFI/OC/Drivers`**| 将以下驱动程序添加到EFI/OC/Drivers并更新配置： <ul> <li> `ExFatDxeLegacy.efi` <li> `ResetNvramEntry.efi` 和设置：<br> ![resetnvram](https://github.com/laobamac/OC-little-zh/assets/76865553/8d955605-fb27-401f-abdd-2c616b233418) | <ul><li> **ExFatDxeLegacy.efi**: 在旧系统上启用ExFat支持 <li> **ResetNvramEntry.efi**: 添加一个启动菜单条目，用于执行NVRAM重置，而不重置启动驱动器顺序。需要UEFI BIOS。


> [!CAUTION]
> 
> 不要将NVRAM参数 `OCLP-Version` 添加到你的配置中——它仅适用于真实的Mac！它会检查你的 `config.plist` 是否与OCLP提供的版本相符。如果你配置中的版本较低，会弹出窗口询问你是否想要更新OpenCore：
>
> ![oclp-version](https://github.com/user-attachments/assets/3376afa3-da56-4311-9960-a9ec90e6010f)
>
> 如果你在这种情况下点击“OK”，你的 `OC` 文件夹将被替换为为相应的Mac型号创建的文件夹，从而使你的macOS安装变得不可启动！

> [!提示]
>
> 要找出其他可能需要的Kext、设置和启动参数，你可以使用OpenCore Legacy Patcher生成EFI文件夹并检查其内容和config.plist：
>
> - 运行OpenCore Patcher应用程序。
> - 在`设置`中选择与你的CPU家族相同的`目标型号`（通过www.everymac.com查找你的CPU类型使用的Mac型号），然后按`Return`。
> - 在主窗口中，现在会显示“构建并安装OpenCore”选项。点击它。这样会构建选定Mac型号的EFI文件夹和配置。
> - **不要安装它！**而是点击“查看构建日志”。复制日志末尾列出的路径。
> - 在Finder中，按 <kbd>CMD</kbd>+<kbd>SHIFT</kbd>+<kbd>G</kbd>，粘贴地址并按 <kbd>Enter</kbd> 进入包含“构建文件夹”的临时位置，其中包含为选定Mac型号创建的OpenCore EFI和配置。将其复制到桌面并检查线索。

## 测试更改
一旦你添加了所需的kext并进行了必要的config.plist更改，保存、重启并执行NVRAM重置。你的系统必须成功启动并应用这些更改——否则不要继续！如果仍然可以启动，现在可以准备你的系统安装macOS 13。

### 调整SMBIOS
如果你的系统成功重启，我们需要再次编辑配置并根据当前安装的macOS版本调整SMBIOS。

#### 从macOS Big Sur 11.3+升级时
从macOS 11.3或更高版本升级时，我们可以利用macOS的虚拟化功能让系统“相信”它运行在虚拟机中，因此不再需要伪装兼容的SMBIOS。所以选择最适合你的CPU家族的SMBIOS（→ 参见“潜在候选”部分，或者在[everymac.com](http://www.everymac.com)查看）。


#### 从macOS Catalina或更早版本升级时
由于macOS Catalina及更早版本缺乏应用VMM Board-ID伪装所需的虚拟化功能，因此必须暂时切换到支持的SMBIOS，以便能够安装macOS Ventura及更新版本。否则，在尝试启动时，你将看到一个带有斜线的圆圈，而不是Apple logo。

**支持的SMBIOS**：

- **台式机**：
	- **`iMac18,1`** 或更新版本
	- **`MacPro7,1`** 或 **`iMacPro1,1`**（高端台式机）
- **笔记本**：
	- **`MacBookPro14,1`** 或 
	- **`MacBookAir8,1`**
- **NUC**：
	- **`Macmini8,1`** 
- 使用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 生成新的序列号

> [!注意]
> 
> 一旦安装了macOS 12或更高版本，你可以切换到适合你CPU的SMBIOS，以实现最佳的CPU电源管理。这非常重要，因为你不能在第一代Intel Core和Xeon CPU上使用`ssdtPRGen`。

## macOS安装
所有准备工作完成后，你现在可以升级到macOS Ventura或更新版本。根据你当前的macOS版本，安装过程会有所不同。

### 获取macOS
- 下载最新版本的[OpenCore Patcher GUI应用程序](https://github.com/dortania/OpenCore-Legacy-Patcher/releases)并运行它
- 点击“创建macOS安装器”
- 点击“下载macOS安装器”
- 选择你要安装的macOS版本
- 下载完成后，“安装macOS…”应用程序将位于你的“程序”文件夹中


> [!NOTE]
> 
> 如果您想执行全新安装，OCLP 还可以创建 USB 安装程序（强烈推荐）。如果您想安装较旧的操作系统，则必须创建 USB 安装程序，因为 macOS 不允许降级。

### 选项1：从macOS 11.3或更新版本升级
仅适用于从macOS 11.3+升级。如果你使用的是macOS Catalina或更早版本，请使用选项2。

- 运行“安装macOS…”应用程序
- 会进行几次重启
- 从新的“安装macOS…”分区启动，直到它不再出现在启动选择器中

一旦安装完成并且系统成功启动，如果你只有集成显卡（iGPU）或你的GPU不被更新的macOS版本支持，它将无法启用硬件图形加速。我们将在安装后处理这个问题。

### 选项2：从macOS Catalina或更早版本升级
从macOS Catalina或更早版本升级时，建议进行干净的USB闪存驱动器安装。要创建USB安装器，你可以使用OpenCore Legacy Patcher：

- 运行磁盘工具
- 在你的内置HDD/SSD上创建一个新的APFS分区，或者使用一个独立的内置硬盘（至少60GB）来安装macOS 13 —— **不要**安装在外部硬盘上 —— 它将无法启动！
- 插入一个空的USB闪存驱动器（16GB及以上），用于创建安装器
- 运行OCLP并按照[**说明**](https://dortania.github.io/OpenCore-Legacy-Patcher/INSTALLER.html#creating-the-installer)操作
- 创建完USB安装器后，执行以下操作：
	- 将OpenCore-Patcher应用程序复制到USB安装器
	- 添加可选工具（如果网络不可用）：
		- 添加Python安装器
		- 添加MountEFI
		- 添加ProperTree
- 重启
- 从BootPicker中选择“安装macOS Ventura”
- 在你之前准备的分区上安装macOS Ventura
- 安装过程中会有几次重启。从新的“安装macOS”分区启动，直到它不再出现在启动选择器中
- 安装macOS Ventura完成后，切换回最适合你CPU的SMBIOS（如前所述）。

安装完成并且系统启动后，如果你只有集成显卡或GPU不再被macOS支持，它将无法启用硬件图形加速。我们将在安装后处理这个问题。

## 安装后
OpenCore Legacy Patcher可以重新安装macOS中被移除的组件，如图形驱动程序、框架等。这被称为“根补丁”。对于Wintel系统，我们主要利用它来安装iGPU和GPU驱动程序。

### 安装其他GPU的驱动程序
- 对于Sandy Bridge及更早版本，没有iGPU驱动程序可用。你需要一个受OCLP支持的专用GPU。
- 如果你的GPU受OCLP支持，OCLP会自动检测它，并且如果有驱动程序，可以进行安装。之后，GPU硬件加速应该可以正常工作。请注意，基于你使用的GPU，OCLP可能需要额外的设置。
- 安装完驱动程序后，在重启之前禁用以下`boot-args`，以重新启用GPU图形加速：
  - `-radvesa` —— 在前面加`#`以禁用：`#-radvesa`
  - `nv_disable=1` —— 在前面加`#`以禁用：`#nv_disable=1`

> [!重要]
>
> 在安装macOS更新之前，你可能需要重新启用AMD和NVIDIA GPU的启动参数，以将它们置于VESA模式，以便你看到图像而不是黑屏！

### 验证SMC CPU电源管理
要验证SMC CPU电源管理是否正常工作，请在终端输入以下命令：

```shell
sysctl machdep.xcpm.mode
```
如果输出为 `0`，则旧版 `ACPI_SMC_PlatformPlugin` 用于 CPU 电源管理，一切正常。如果输出为`1`，则`XCPM`的`X86PlatformPlugin`处于活动状态，这并不好，因为 Sandy Bridge CPU 不支持 XCPM。在这种情况下，请检查 OpenCore 是否注入了 SMC CPU 电源管理所需的 Kext。在终端中输入：

```shell
kextstat | grep com.apple.driver.AppleIntelCPUPowerManagement
```
这应该会产生以下输出:

```
com.apple.driver.AppleIntelCPUPowerManagement (222.0.0)
com.apple.driver.AppleIntelCPUPowerManagementClient (222.0.0) 
```
如果不存在这两个kexts，则说明它们没有被注入。所以再次检查你的配置文件和文件夹。 并确保 `AppleCpuPmCfgLock` 已经启用。

#### 优化CPU电源管理
一旦确认SMC CPU电源管理（插件类型 `0`）正常工作，使用 [Intel Power Gadget](https://www.insanelymac.com/forum/files/file/1056-intel-power-gadget/) 监控CPU的行为。如果CPU没有达到最大涡轮频率，或者基础频率过高/过低，或者空闲频率过高，请 [生成一个 `SSDT-PM`](/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management/CPU_Power_Management_(Legacy)#readme) 来优化CPU电源管理。

### 移除/禁用启动参数
在安装完macOS Ventura并且应用了OCLP的root补丁后，移除或禁用以下启动参数：

- `ipc_control_port_options=0`：**仅在使用独立GPU时需要**。如果使用Intel HD 4000，仍然需要此参数以便Firefox和基于Electron的应用程序正常运行。
- `amfi_get_out_of_my_way=0x1`：**仅在通过OCLP重新应用root补丁后需要**，适用于系统更新后。
- 将 `-radvesa` 改为 `#-radvesa` → 这将禁用此启动参数，从而重新启用AMD GPU的硬件加速。
- 将 `nv_disable=1` 改为 `#nv_disable=1` → 这将禁用此启动参数，从而重新启用NVIDIA GPU的硬件加速。

> [!NOTE]
> 
> 将当前工作的EFI文件夹备份到一个fat32格式的u盘上，以防您的系统在删除/禁用这些启动参数后无法启动！

### 验证AMFI是否开启
我们可以通过在终端输入以下命令来检查是否启用了AMFI：

```shell
sudo /usr/sbin/nvram -p | /usr/bin/grep -c "amfi_get_out_of_my_way=1"
```

- 期望的输出是 `0`：这意味着 `amfi_get_out_of_my_way=1` 启动参数（用于禁用AMFI）不在NVRAM中，表示AMFI已启用，这是好的。
- 如果输出是 `1`：这意味着 `amfi_get_out_of_my_way=1` 启动参数（用于禁用AMFI）在NVRAM中，表示AMFI被禁用。

由于新的 `AMFIPass.kext` 允许在启用AMFI的情况下启动macOS，同时禁用SIP和SecureBootModel，且应用了root补丁，因此我们希望输出是 `0`！

## OCLP与系统更新

### 在系统更新后重新应用root补丁
使用OCLP的主要优势之一是，即使在安装系统更新后，它仍然留在系统中。更新后，它会检测到缺失的图形驱动程序，并询问是否希望重新打补丁，如下所示：

![Notify](https://user-images.githubusercontent.com/76865553/181934588-82703d56-1ffc-471c-ba26-e3f59bb8dec6.png)

你只需点击“确定”，驱动程序将重新安装。完成必需的重启后，一切将恢复正常。


### OCLP 应用程序更新通知

OCLP 还可以通知你有关 Patcher 应用程序本身的可用更新。但这需要将 `OCLP-Version` 键添加到 config.plist 文件的 `NVRAM/Add` 部分：

![OCLPver01](https://github.com/laobamac/OC-little-zh/assets/76865553/4a7b0fc8-2ab5-4d2c-9ab2-ea62ca7614ff)

对于黑果用户来说，这个键是可选的，因为 OCLP 应用程序在运行时会自动通知你有关更新的消息。如果你选择将其添加到配置中，你还需要在相应的 `NVRAM/Delete` 部分添加一个重置键，以便可以应用新值：

![OCLPver03](https://github.com/laobamac/OC-little-zh/assets/76865553/2ca84e3b-fb66-4d96-8fe8-d8ee94fde0a5)

之后，每当 OpenCore Patcher 有更新时，你都会收到通知：

![OCLPver02](https://github.com/laobamac/OC-little-zh/assets/76865553/dc61fb53-2a72-4523-81fe-a0df3596bc73)

请注意，这个弹窗提到的是“OpenCore”，而不是 Patcher，因为 OCLP 是针对真实 Mac 和 Mac 用户设计的。对于“普通”Mac用户来说，使用 OCLP 可能是他们唯一更新 OpenCore、配置和kexts的方式。所以在下载最新的 OCLP 更新后，他们只需要重新构建 EFI，挂载ESP，替换 EFI/OC 文件夹，应用补丁，重启，完成。

但作为黑果用户，我们只关心应用程序更新，用来应用更新或优化的根补丁，特别是针对 iGPU、Wi-Fi 等的补丁。请记住，在每次更新后，你需要手动调整 OCLP 版本号，以避免在已安装最新版本时收到可能过时的补丁应用程序通知。所以将 `OCLP-Version` 键添加到黑果系统的配置中并非必需。


> [!TIP]
> 
> 如果您的系统在使用OCLP修补后无法启动，那么您有几个选项可以 [卸载补丁](/14_OCLP_Wintel/Guides/Reverting_Root_Patches.md).

## 注意事项
- 对系统分区应用根补丁会破坏其安全封印。这会影响系统更新：每次有系统更新时，都会重新下载完整安装包（约 12GB）。对此有一个[**解决方法**](/S_System_Updates/OTA_Updates.md)，但仅适用于 Haswell/Broadwell 及更新版本。
- 每次系统更新后，iGPU/GPU 驱动程序需要重新安装。OCLP 会处理这个过程。请确保在更新/升级 macOS 前重新启用适当的启动参数，以将 AMD/NVIDIA GPU 设置为 VESA 模式。
- ⚠️ 你不能在 Haswell 之前的系统上安装 macOS 安全响应更新（RSR）。它们将无法安装（更多信息请见 [**此处**](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1019)）。

## 进一步资源
- [**Non-Metal Wiki**](https://moraea.github.io/) by Moraea
- [**SMBIOS 兼容性图表**](https://docs.google.com/spreadsheets/d/1DSxP1xmPTCv-fS1ihM6EDfTjIKIUypwW17Fss-o343U/edit#gid=483826077)
- [**在 11+ 上启用 NVIDIA WebDrivers**](https://elitemacx86.com/threads/how-to-enable-nvidia-webdrivers-on-macos-big-sur-and-monterey.926/) by elitemacx86.com

## 致谢
- Acidanthera 提供 OpenCore 和众多 Kexts
- Corpnewt 提供 MountEFI, GenSMBIOS 和 ProperTree
- dhinakg 提供 AMFIPass
- Dortania 提供 [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher/releases) 和 [指南](https://dortania.github.io/OpenCore-Legacy-Patcher/)
- Rehabman 提供笔记本显存补丁
