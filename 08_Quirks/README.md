# Intel/AMD 处理器的 OpenCore Quirks

OpenCore 的 ACPI、Booter、Kernel 和 UEFI Quirks 适用于 Intel 和 AMD CPUs。信息基于 Dortania 的 [**OpenCore Install Guide**](https://dortania.github.io/OpenCore-Install-Guide/) 以及我对第 11 代及更新平台的研究。这些内容以表格形式清晰呈现。

也可参考 [**列表形式**](/08_Quirks/Quirks_List.md)，其中包含更易阅读和维护的内容，包括第 11 代和第 12 代 Intel Core CPUs 的 Quirks。

💡 **提示**：以下所有 Quirks 组合都可以在 [**OpenCore Auxiliary Tools**](https://github.com/ic005k/OCAuxiliaryTools) 中作为预设选项找到。

**图例**：

- **x** = 启用 Quirk。
- **( )** = 默认禁用 Quirk，但在某些 CPU/芯片组/主板上启用（参考注释）。
- **(x)** = 默认启用 Quirk，但在某些 CPU/芯片组/主板上禁用（参考注释）。
- **空白** = 禁用 Quirk，即明确禁用，而不是保持默认值。

**适用版本**：OpenCore ≥ 0.7.5

---

## 8th 到 10th Gen Intel CPUs（桌面，高端，笔记本/NUC）

### SMBIOS 要求
- **第 10 代桌面**： [**iMac20,1**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac20,1) 和 [**iMac20,2**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac20,2) （macOS Catalina 及更新）。
- **第 10 代笔记本/NUC**： [**多种型号**](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake-plus.html#platforminfo)。
- **第 8/9/10 代高端桌面**：[**iMacPro1,1**](https://dortania.github.io/OpenCore-Install-Guide/config-HEDT/skylake-x.html#platforminfo)（macOS High Sierra 及更新）。
- **第 8/9 代桌面**：[**iMac19,1**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac19,1)（macOS Mojave+），[**iMac18,3**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac18,3)（macOS High Sierra 或更低版本）。
- **第 8/9 代笔记本/NUC**：[**多种型号**](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake.html#platforminfo)。


### ACPI 小技巧

| CPU 系列 | [Comet Lake](https://ark.intel.com/content/www/us/en/ark/products/codename/90354/products-formerly-comet-lake.html) | 第10代 | Cascade Lake-[X](https://ark.intel.com/content/www/us/en/ark/products/codename/124664/products-formerly-cascade-lake.html#@Desktop)/[W](https://ark.intel.com/content/www/us/en/ark/products/codename/124664/products-formerly-cascade-lake.html#@Workstation), Skylake-[X](https://ark.intel.com/content/www/us/en/ark/products/126699/intel-core-i97980xe-extreme-edition-processor-24-75m-cache-up-to-4-20-ghz.html)/[W](https://ark.intel.com/content/www/us/en/ark/products/126793/intel-xeon-w2195-processor-24-75m-cache-2-30-ghz.html) | [Coffee Lake](https://ark.intel.com/content/www/us/en/ark/products/codename/97787/products-formerly-coffee-lake.html) | 第8/9代 | 描述 |
|:-----------|:---------:|:--------:|:------------:|:----------:|:-----------:|:-----------------|
|**平台**|[台式机](https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html)|[笔记本/NUC](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake-plus.html#laptop-coffee-lake-plus-and-comet-lake)|[高端平台](https://dortania.github.io/OpenCore-Install-Guide/config-HEDT/skylake-x.html)|[台式机](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html)|笔记本/NUC [第8代](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake.html) / [第9代](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake-plus.html)|对应的配置指南 |
| **SMBIOS** |iMac20,x|MacBookPro16,x / Macmini8,1|iMacPro1,1|iMac19,1|MacBookPro15,x/16,x / Macmini8,1|平台信息 |
|                 |           |          ||           |             |             |
|**FadtEnableReset**  ||||||适用于老旧系统和部分新款笔记本。可以修复电源按钮快捷键问题。除非必要，不推荐启用。|
|**NormalizeHeaders** ||||||清理 ACPI 头部信息，避免 macOS 10.13 启动崩溃。|
|**RebaseRegions**    ||||||重定位 ACPI 内存区域。不推荐使用！|
|**ResetHwSig**       ||||||将 FACS 表硬件签名重置为 0。修复固件导致的休眠唤醒问题。|
|**ResetLogoStatus**°|(x)|(x)|(x)|(x)|(x)|将 `BRGT` 表中的 `Displayed` 设置为 `0`（false）。用于修复包含 `BGRT` 表但无法正确处理显示更新的固件问题。|
|**SyncTableIDs**     ||||||修复表格以兼容旧版 Windows 系统。|

`°`默认在 `sample.plist` 中启用。由于 OpenCore 安装指南编写时此 Quirk 尚未存在，因此是否必需尚不明确。但大概率不需要。

### 引导器小技巧 (Boooter Quirks)

| CPU 系列 | Comet Lake | 第10代 | Cascade Lake X | Coffee Lake | 第8/9代 | 描述 |
|:-----------|:---------:|:--------:|:------------:|:----------:|:-----------:|:-----------------|
|**平台**|台式机|笔记本/NUC|高端台式机|台式机|笔记本/NUC|
| **SMBIOS** |iMac20,X|MacBookPro16,X / Macmini8,1|iMacPro1,1|iMac19,1|MacBookPro15,1 / Macmini8,1|系统管理 BIOS|
|                 |           |          ||           |             |             |
|**AllowRelocationBlock**||||||允许通过 relocation block 启动 macOS。需要启用 ProvideCustomSlide 和 AvoidRuntimeDefrag。|
|**AvoidRuntimeDefrag**|x|x|x|x|x|防止 `boot.efi` 的运行时内存碎片化。|
|**DevirtualiseMmio**|x|x|x|x||移除某些 MMIO 区域的运行时属性。|
|**DisableSingleUser**||||||禁用单用户模式，提高安全性。|
|**DisableVariableWrite**||||||限制 macOS 对 NVRAM 的访问。|
|**DiscardHibernateMap**||||||复用原始的休眠内存映射。|
|**EnableSafeModeSlide**|x|x|x|x|x|修改引导程序以在安全模式中启用 KASLR（地址空间布局随机化）。需要启用 ProvideCustomSlide。|
|**EnableWriteUnprotector**||||||允许对 UEFI 运行时服务代码的写入访问。|
|**ForceBooterSignature**||||||将 macOS 的启动签名设置为 OpenCore 启动器。|
|**ForceExitBootServices**||||||修复固件的早期启动崩溃。如果不清楚用途，请勿启用！|
|**ProtectMemoryRegions**||||||保护内存区域免受错误访问。|
|**ProtectSecureBoot**||||||保护 UEFI 安全启动变量免于被修改。|
|**ProtectUefiServices**°|x|x||(x)°||保护 UEFI 服务免被固件覆盖。|
|**ProvideCustomSlide**|x|x|x|x|x|在低内存情况下提供自定义 KASLR 滑动地址。|
|**ProvideMaxSlide**||||||在较高滑动地址不可用时提供最大 KASLR 滑动地址。|
|**RebuildAppleMemoryMap**|x|x|x|x|x|生成兼容 macOS 的内存映射。|
|**ResizeAppleGpuBars**||||||缩小 GPU PCI BAR 大小以兼容 macOS。|
|**SetupVirtualMap**|||x|x|x|在 SetVirtualAddresses 阶段设置虚拟内存。|
|**SignalAppleOS**||||||通过操作系统信息报告 macOS 正在加载。|
|**SyncRuntimePermissions**|x|x|x|x|x|更新运行时环境的内存权限。|

`°` Z390主板需要

### 内核小技巧 (Kernel Quirks)

| CPU 系列 | Comet Lake | 第10代 | Cascade Lake X | Coffee Lake | 第8/9代 | 描述 |
|:-----------|:---------:|:--------:|:------------:|:----------:|:-----------:|:-----------------|
|**平台**|台式机|笔记本/NUC|高端台式机|台式机|笔记本/NUC|
| **SMBIOS** |iMac20,X|MacBookPro16,X / Macmini8,1|iMacPro1,1|iMac19,1|MacBookPro15,1 / Macmini8,1|系统管理 BIOS|
|                 |           |          ||           |             |             |
|**AppleCpuPmCfgLock**||||||禁用 `AppleIntelCPUPowerManagement.kext` 中的 MSR 修改。|
|**AppleXcpmCfgLock**°|(x)|(x)||(x)|(x)|启用对 XNU 内核的写入访问，以便开启 XCPM 电源管理。|
|**AppleXcpmExtraMsrs**||||||禁用多个关键 MSR 访问，适用于不支持原生 XCPM 的某些 CPU。在 macOS 12+ 中禁用此 Quirk，因为该功能不再存在。|
|**AppleXcpmForceBoost**||||||在 XCPM 模式下强制最大性能。不推荐使用。|
|**CustomSMBIOSGuid**°°|( )|( )||( )|( )|通常适用于戴尔笔记本或有 Windows 许可证问题时。|
|**DisableIoMapper**|x|x||x|x|禁用 XNU（VT-d）中的 IOMapper 支持。|
|**DisableLinkeditJettison**|x|x||x|x|在 macOS Big Sur 中提高 `Lilu.kext` 性能，无需 `keepsyms=1` 引导参数。|
|**DisableRtcChecksum**||||||禁用 `AppleRTC` 的主校验和（0x58-0x59）写入。|
|**ExtendBTFeatureFlags**||||||将 `FeatureFlags` 设置为 `0x0F`，以启用蓝牙的所有功能，包括 Continuity。|
|**ExternalDiskIcons**||||||为所有 AHCI 磁盘强制使用内部磁盘图标。如果可能，尽量避免使用！|
|**ForceSecureBootScheme**||||||强制使用 x86 方案进行 IMG4 验证。如果 `SecureBootModel` ≠ 默认值，虚拟机中需要启用此项。|
|**IncreasePciBarSize**||||||允许 `IOPCIFamily` 使用 2 GB PCI BAR 启动。尽量避免！|
|**LapicKernelPanic**°°°|( )|( )||( )|( )|禁用 LAPIC 中断引发的内核崩溃。|
|**LegacyCommpage**||||||适用于不支持 SSSE3 的旧平台。|
|**PanicNoKextDump**|x|x||x|x|防止内核在崩溃日志中打印 kext 转储。适用于 macOS 10.13 及以上版本。|
|**PowerTimeoutKernelPanic**|x|x||x|x|禁用因 `setPowerState` 超时引发的内核崩溃。|
|**ProvideCurrentCpuInfo**||||||解决与 Microsoft Hyper-V 的兼容性问题。|
|**SetApfsTrimTimeout**|-1|-1||-1|-1|设置 APFS 文件系统在 SSD 上的 TRIM 超时时间（单位：ms）。|
|**ThirdPartyDrives**||||||在 macOS 10.15 及更新版本中启用第三方 SSD 的 TRIM 和休眠支持。|
|**XhciPortLimit**°°°°|(x)|(x)||(x)|(x)|修补各种 kexts 以移除 15 个 USB 端口限制。|

`°` `AppleXcpmCfgLock`: 如果可以在 BIOS 中禁用 CFGLock，则不需要此选项。</br>
`°°` `CustomSMBIOSGuid`: 针对戴尔或索尼 VAIO 笔记本启用。</br>
`°°°` `LapicKernelPanic`: 针对 HP 系统启用。</br>
`°°°°` `XhciPortLimit`: 对于 macOS 11.3 及更高版本禁用——建议创建 USB 端口映射代替！

---

### UEFI 小技巧 (UEFI Quirks)

| CPU 系列 | Comet Lake | 第10代 | Cascade Lake X | Coffee Lake | 第8/9代 | 描述 |
|:-----------|:---------:|:--------:|:------------:|:----------:|:-----------:|:-----------------|
|**平台**|台式机|笔记本/NUC|高端台式机|台式机|笔记本/NUC|
| **SMBIOS** |iMac20,X|MacBookPro16,X / Macmini8,1|iMacPro1,1|iMac19,1|MacBookPro15,1 / Macmini8,1|系统管理 BIOS|
|                 |           |          ||           |             |             |
|**ActivateHpetSupport**||||||强制启用 HPET（高精度事件计时器），如果 BIOS 中没有相关选项。|
|**DisableSecurityPolicy**||||||禁用平台安全策略。如果使用 UEFI 安全启动，请勿启用。|
|**EnableVectorAcceleration**|x|x||||启用 SHA-512 和 SHA-384 哈希算法的 AVX 矢量加速。|
|**ExitBootServicesDelay**||||||在 `EXIT_BOOT_SERVICES` 事件后增加延迟（微秒级）。|
|**ForceOcWriteFlash**||||||启用对所有 OpenCore 系统变量的闪存写入。|
|**ForgeUefiSupport**||||||在 EFI 1.x 固件上部分实现 UEFI 2.x 支持。|
|**IgnoreInvalidFlexRatio**||||||修复 MSR_FLEX_RATIO（0x194）MSR 寄存器中的无效值。|
|**ReleaseUsbOwnership**||x|||x|尝试从固件驱动程序中解除 USB 控制器所有权。|
|**ReloadOptionRoms**||||||查询 PCI 设备并重新加载它们的 Option ROM（如果可用）。|
|**RequestBootVarRouting**|x|x||x|x|需要此项以使“启动磁盘”偏好设置窗格正常工作。|
|**ResizeGpuBars**||||||配置 GPU PCI BAR 大小。|
|**TscSyncTimeout**||||||用于调试 TSC 同步的实验性 Quirk。|
|**UnblockFsConnect**°|( )|( )||( )|( )|如果驱动器检测失败导致启动条目丢失时启用此选项。|

`°` `UnblockFsConnect`: 针对 HP 机器启用。
<details>
<summary><strong>第六代和第七代 Intel 小技巧</strong>（点击展开内容！）</summary>


## 6th and 7th Gen Intel CPUs (Desktop/Mobile)

### SMBIOS Requirements
- 7th Gen Desktop: 
	- [**iMac18,3**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac18,3) (for systems with discrete GPU, using iGPU for computing task only)
	- [**iMac18,1**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac18,1) (for systems with using iGPU for display output)
- 7th Gen Mobile/NUC: [**various**](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/kaby-lake.html#platforminfo)
- 6th Gen Desktop: [**iMac17,1**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac17,1) (macOS El Capitan)
- 6th Gen Mobile/NUC: [**various**](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/skylake.html#platforminfo)

### ACPI Quirks   
| CPU Family      | Kaby Lake    | 7th Gen       | Skylake    | 6th Gen |
|:----------------|:-----------:|:-------------:|:----------:|:-------:|
| **ACPI Quirks** | Desktop     | Mobile        | Desktop    | Mobile  |    
|                 |             | 	        |            |         |
|FadtEnableReset  |
|NormalizeHeaders |
|RebaseRegions    |
|ResetHwSig       | 
|ResetLogoStatus* |(x)|(x)|(x)|(x)|
|SyncTableIDs     |

`*`Default in `sample.plist`

### Booter Quirks
| CPU Family        | Kaby Lake    | 7th Gen      | Skylake    | 6th Gen |
|:------------------|:-----------:|:-------------:|:---------:|:-------:|
| **Booter Quirks** | Desktop     | Mobile        | Desktop   | Mobile  |
|	            |             |               |           |         |
|AllowRelocationBlock||||
|AvoidRuntimeDefrag|x|x|x|x|
|DevirtualiseMmio||||
|DisableSingleUser||||
|DisableVariableWrite||||
|DiscardHibernateMap||||
|EnableSafeModeSlide|x|x|x|x|
|EnableWriteUnprotector|x|x|x|x|
|ForceBooterSignature||||
|ForceExitBootServices||||
|ProtectMemoryRegions||||
|ProtectSecureBoot||||
|ProtectUefiServices||||
|ProvideCustomSlide|x|x|x|x|
|ProvideMaxSlide||||
|RebuildAppleMemoryMap||||
|ResizeAppleGpuBars||||
|SetupVirtualMap|x|x|x|x|
|SignalAppleOS||||
|SyncRuntimePermissions||||

### Kernel Quirks
| CPU Family        | Kaby Lake    | 7th Gen       | Skylake    | 6th Gen |
|:------------------|:-----------:|:-------------:|:----------:|:-------:|
| **Kernel Quirks** | Desktop     | Mobile        | Desktop    | Mobile  |
|                   |             |               |            |         |
|AppleCpuPmCfgLock||||
|AppleXcpmCfgLock|x|x|x|x
|AppleXcpmExtraMsrs||||
|AppleXcpmForceBoost||||
|CustomSMBIOSGuid*|( )|( )|( )|( )|
|DisableIoMapper|x|x|x|x
|DisableLinkeditJettison|x|x|x|x|
|DisableRtcChecksum||||
|ExtendBTFeatureFlags||||
|ExternalDiskIcons||||
|ForceSecureBootScheme||||
|IncreasePciBarSize||||
|LapicKernelPanic**|( )|( )|( )|( )|
|LegacyCommpage||||
|PanicNoKextDump|x|x|x|x|
|PowerTimeoutKernelPanic|x|x|x|x|
|ProvideCurrentCpuInfo||||
|SetApfsTrimTimeout|-1|-1|-1|-1|
|ThirdPartyDrives||||
|XhciPortLimit***|x|x|x|x

`*` `CustomSMBIOSGuid`: Enable for Dell or Sony VAIO Systems</br>
`**` `LapicKernelPanic`: Enable for HP Systems</br>
`***` `XhciPortLimit`: Disable for macOS 11.3 and newer – create a USB Port Map instead!

### UEFI Quirks
| CPU Family      | Kaby Lake    | 7th Gen       | Skylake   | 6th Gen |
|:----------------|:-----------:|:-------------:|:---------:|:-------:|
| **UEFI Quirks** | Desktop     | Mobile        | Desktop   | Mobile  |
|                 |             |               |           |         |
|ActivateHpetSupport||||
|DisableSecurityPolicy||||
|EnableVectorAcceleration||||
|ExitBootServicesDelay||||
|ForceOcWriteFlash||||
|ForgeUefiSupport||||
|IgnoreInvalidFlexRatio||||
|ReleaseUsbOwnership||||
|ReloadOptionRoms||||
|RequestBootVarRouting|x|x|x|x
|ResizeGpuBars||||
|TscSyncTimeout||||
|UnblockFsConnect*|( )|( )|( )|( )|

`*` `UnblockFsConnect`: Enable on HP Machines
</details>
<details>
<summary><strong>4th and 5th Gen Intel Quirks</strong> (Click to show content)</summary>

## 4th and 5th Gen Intel CPUs (Desktop/Mobile)

### SMBIOS Requirements
- 5th Gen Desktop: N/A
- 5th Gen Mobile/NUC: [**various**](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/broadwell.html#platforminfo)
- 4th Gen Desktop: [**iMac14,4**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac14,1) (Haswell with iGPU only) or [**iMac15,1**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac15,1) (Haswell with dGPU)
- 4th Gen Mobile/NUC: [**various**](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/haswell.html#platforminfo)

### ACPI Quirks   
| CPU Family      | Broadwell     | 5th Gen | Haswell | 4th Gen |
|:----------------|:-------------:|:-------:|:-------:|:-------:|
| **ACPI Quirks** | Desktop (N/A) | Mobile  | Desktop | Mobile  |    
|                 |               |         |         |         |
|FadtEnableReset|
|NormalizeHeaders|
|RebaseRegions|
|ResetHwSig| 
|ResetLogoStatus*||x|x|x|
|SyncTableIDs|

`*` Default in `sample.plist`

### Booter Quirks
| CPU Family        | Broadwell     | 5th Gen | Haswell | 4th Gen |
|:------------------|:-------------:|:-------:|:-------:|:-------:|
| **Booter Quirks** | Desktop (N/A) | Mobile  | Desktop | Mobile  |
|		    |		    |  	      |         |         |
|AllowRelocationBlock||||
|AvoidRuntimeDefrag||x|x|x|
|DevirtualiseMmio||||
|DisableSingleUser||||
|DisableVariableWrite||||
|DiscardHibernateMap||||
|EnableSafeModeSlide||x|x|x|
|EnableWriteUnprotector||x|x|x|
|ForceBooterSignature||||
|ForceExitBootServices||||
|ProtectMemoryRegions||||
|ProtectSecureBoot||||
|ProtectUefiServices||||
|ProvideCustomSlide*||x|x|x|
|ProvideMaxSlide||||
|RebuildAppleMemoryMap||||
|ResizeAppleGpuBars||||
|SetupVirtualMap||x|x|x|
|SignalAppleOS||||
|SyncRuntimePermissions||||

`*` `ProvideCustomSlide`: Used for Slide variable calculation. However, the necessity of this quirk is determined by "OCABC: Only N/256 slide values are usable!" message in the debug log. If the message "OCABC: All slides are usable! You can disable `ProvideCustomSlide`!" is present in your log, you can disable ProvideCustomSlide.

### Kernel Quirks
| CPU Family        | Broadwell | 5th Gen | Haswell | 4th Gen |
|:------------------|:---------:|:-------:|:-------:|:-------:|
| **Kernel Quirks** | Desktop (N/A) | Mobile  | Desktop | Mobile |
|                   |           |         |         |         |
|AppleCpuPmCfgLock||||
|AppleXcpmCfgLock°||x|x|x|
|AppleXcpmExtraMsrs||||
|AppleXcpmForceBoost||||
|CustomSMBIOSGuid*||( )|( )|( )
|DisableIoMapper||x|x|x|
|DisableLinkeditJettison||x|x|x|
|DisableRtcChecksum||||
|ExtendBTFeatureFlags||||
|ExternalDiskIcons||||
|ForceSecureBootScheme||||
|IncreasePciBarSize||||
|LapicKernelPanic**||( )|( )|( )
|LegacyCommpage||||
|PanicNoKextDump||x|x|x|
|PowerTimeoutKernelPanic||x|x|x|
|ProvideCurrentCpuInfo||||
|SetApfsTrimTimeout||-1|-1|-1|
|ThirdPartyDrives||||
|XhciPortLimit***||(x)|(x)|(X)|

`°` `AppleXcpmCfgLock`: Not needed if you can disable CFGLock in BIOS</br>
`*` `CustomSMBIOSGuid`: Enable for Dell or Sony VAIO</br>
`**` `LapicKernelPanic`: Enable for HP Systems</br>
`***` `XhciPortLimit`: Disable for macOS 11.3 and newer – create a USB Port Map instead!

### UEFI Quirks
| CPU Family      | Broadwell | 5th Gen | Haswell | 4th Gen |
|:----------------|:---------:|:-------:|:-------:|:-------:|
| **UEFI Quirks** | Desktop (N/A) | Mobile  | Desktop | Mobile|
|                 |           |         |         |         |
|ActivateHpetSupport||||
|DisableSecurityPolicy||||
|EnableVectorAcceleration||||
|ExitBootServicesDelay||||
|ForceOcWriteFlash||||
|ForgeUefiSupport||||
|IgnoreInvalidFlexRatio||x|x|x
|ReleaseUsbOwnership||x||x|
|ReloadOptionRoms||||
|RequestBootVarRouting||x||x
|ResizeGpuBars||-1|-1|-1
|TscSyncTimeout||||
|UnblockFsConnect*||( )|( )|( )

`*` `UnblockFsConnect`: Enable on HP Machines
</details>
<details>
<summary><strong>2nd and 3rd Gen Intel Quirks</strong> (Click to show content)</summary>

## 2nd and 3rd Gen Intel CPUs (Desktop/Mobile)

### SMBIOS Requirements
- 3rd Gen Desktop: 
	- [**iMac13,1**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac13,1).For system using the iGPU for driving a display.
	- [**iMac13,2**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac13,1). For systems using a dGPU for displaying and the iGPU for computing tasks only.
	- For Big Sur: **iMac14,4** (iGPU), **iMac15,1** (dGPU + iGPU)
- 3rd Gen Mobile/NUC: [**various**](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/ivy-bridge.html#platforminfo)
- 2nd Gen Desktop: [**iMac14,4**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac14,1) (Haswell with iGPU only) or [**iMac15,1**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac15,1) (Haswell with dGPU)
- 2nd Gen Mobile/NUC: [**various**](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/haswell.html#platforminfo)

### ACPI Quirks   
| CPU Family       | Ivy Bridge | 3rd Gen | Sandy Bridge | 2nd Gen |
|:----------------|:----------:|:--------:|:------------:|:-------:|
| **ACPI Quirks** | Desktop    | Mobile   | Desktop      | Mobile  |
|                 |            |          |              |         |
|FadtEnableReset|
|NormalizeHeaders|
|RebaseRegions|
|ResetHwSig| 
|ResetLogoStatus*|(x)|(x)|(x)|(x)|
|SyncTableIDs|

`*` Default in `sample.plist`

### Booter Quirks
| CPU Family        | Ivy Bridge | 3rd Gen | Sandy Bridge | 2nd Gen |
|:------------------|:----------:|:--------:|:-----------:|:-------:|
| **Booter Quirks** | Desktop    | Mobile   | Desktop     | Mobile  |
|                   |            |          |             |         |
|AllowRelocationBlock||||
|AvoidRuntimeDefrag|x|x|x|x|
|DevirtualiseMmio||||
|DisableSingleUser||||
|DisableVariableWrite||||
|DiscardHibernateMap||||
|EnableSafeModeSlide|x|x|x|x|
|EnableWriteUnprotector|x|x|x|x
|ForceBooterSignature||||
|ForceExitBootServices||||
|ProtectMemoryRegions||||
|ProtectSecureBoot||||
|ProtectUefiServices||||
|ProvideCustomSlide*|x|x|x|x
|ProvideMaxSlide||||
|RebuildAppleMemoryMap||||
|ResizeAppleGpuBars||||
|SetupVirtualMap|x|x|x|x|
|SignalAppleOS||||
|SyncRuntimePermissions||||

`*` `ProvideCustomSlide`: Used for Slide variable calculation. However, the necessity of this quirk is determined by "OCABC: Only N/256 slide values are usable!" message in the debug log. If the message "OCABC: All slides are usable! You can disable `ProvideCustomSlide`!" is present in your log, you can disable ProvideCustomSlide.

### Kernel Quirks
| CPU Family        | Ivy Bridge | 3rd Gen | Sandy Bridge | 2nd Gen |
|:------------------|:----------:|:-------:|:------------:|:-------:|
| **Kernel Quirks** | Desktop    | Mobile  | Desktop      | Mobile  |
|                   |            |         |              |         |
|AppleCpuPmCfgLock|x|x|x|x
|AppleXcpmCfgLock||||
|AppleXcpmExtraMsrs||||
|AppleXcpmForceBoost||||
|CustomSMBIOSGuid*||||
|DisableIoMapper|x|x|x|x
|DisableLinkeditJettison|x|x|x|x
|DisableRtcChecksum||||
|ExtendBTFeatureFlags||||
|ExternalDiskIcons||||
|ForceSecureBootScheme||||
|IncreasePciBarSize||||
|LapicKernelPanic**||||
|LegacyCommpage||||
|PanicNoKextDump|x|x|x|x
|PowerTimeoutKernelPanic|x|x|x|x
|ProvideCurrentCpuInfo||||
|SetApfsTrimTimeout||||
|ThirdPartyDrives||||
|XhciPortLimit***|x|x|x|x|

`*` `CustomSMBIOSGuid`: Enable for Dell or Sony VAIO Systems</br>
`**` `LapicKernelPanic`: Enable for HP Systems</br>
`***` `XhciPortLimit`: Disable if your system doesn't have USB 3.0 ports And if you are running macOS 11.3 and newer – create a USB Port Map instead!

### UEFI Quirks
| CPU Family      | Ivy Bridge | 3rd Gen | Sandy Bridge | 2nd Gen |
|:----------------|:----------:|:-------:|:------------:|:-------:|
| **UEFI Quirks** | Desktop    | Mobile  | Desktop      | Mobile  |
|                 |            |         |              |         |
|ActivateHpetSupport||||
|DisableSecurityPolicy||||
|EnableVectorAcceleration||||
|ExitBootServicesDelay||||
|ForceOcWriteFlash||||
|ForgeUefiSupport||||
|IgnoreInvalidFlexRatio|x|x|x|x
|ReleaseUsbOwnership||x||x
|ReloadOptionRoms||||
|RequestBootVarRouting||||
|ResizeGpuBars||||
|TscSyncTimeout||||
|UnblockFsConnect*|( )|( )|( )|( )|

`*` `UnblockFsConnect`: Enable on HP Machines
</details>
<details>
<summary><strong>AMD Quirks</strong> (Click to show content!)</summary>

## AMD Ryzen and Threadripper (17h and 19h)

### SMBIOS Requirements
- **Dektop**: [**various**](https://dortania.github.io/OpenCore-Install-Guide/AMD/zen.html#platforminfo)

### ACPI Quirks    
| CPU Family      | Ryzen and Threadripper |
|:----------------|:--------------------:|
| **ACPI Quirks** | Desktop              |    
|                 |                      |
|FadtEnableReset  |
|NormalizeHeaders |
|RebaseRegions    |
|ResetHwSig       | 
|ResetLogoStatus* |(x)|
|SyncTableIDs     |

`*`Default in `sample.plist`

### Boooter Quirks
| CPU Family         | Ryzen and Threadripper |
|:------------------ |:--------------------:|
| **Booter Quirks**  | Desktop              |    
|                    |                      |
|AllowRelocationBlock|
|AvoidRuntimeDefrag|x
|DevirtualiseMmio°|( )°|
|DisableSingleUser|
|DisableVariableWrite|
|DiscardHibernateMap|
|EnableSafeModeSlide|x
|EnableWriteUnprotector|
|ForceBooterSignature
|ForceExitBootServices
|ProtectMemoryRegions|
|ProtectSecureBoot
|ProtectUefiServices|
|ProvideCustomSlide|
|ProvideMaxSlide
|RebuildAppleMemoryMap|x|
|ResizeAppleGpuBars
|SetupVirtualMap|x|x
|SignalAppleOS
|SyncRuntimePermissions|x|

`°` `DevirtualiseMmio`: Enable for TRx 40</br>

### Kernel Quirks
- For AMD, enable `Kernel` > `Emulate`: `DummyPowerManagement`
- AMD also requires a lot of [**Kernel patches**](https://github.com/AMD-OSX/AMD_Vanilla/tree/master) to make macOS work.

| CPU Family         | Ryzen and Threadripper |
|:------------------ |:--------------------:|
| **kernel Quirks**  | Desktop              |    
|                    |                      |
|AppleCpuPmCfgLock||
|AppleXcpmCfgLock||
|AppleXcpmExtraMsrs||
|AppleXcpmForceBoost||
|CustomSMBIOSGuid||
|DisableIoMapper||
|DisableLinkeditJettison||
|DisableRtcChecksum||
|ExtendBTFeatureFlags||
|ExternalDiskIcons||
|ForceSecureBootScheme||
|IncreasePciBarSize||
|LapicKernelPanic||
|LegacyCommpage||
|PanicNoKextDump|x|
|PowerTimeoutKernelPanic|x|
|ProvideCurrentCpuInfo|x|
|SetApfsTrimTimeout||
|ThirdPartyDrives||
|XhciPortLimit°||

`°` `XhciPortLimit`: Not required on AMD systems, since these boards usually have 2 or more USB controllers with 10 Ports max.

### UEFI Quirks
| CPU Family       | Ryzen and Threadripper |
|:---------------- |:--------------------:|
| **UEFI Quirks**  | Desktop              |    
|                  |                      |
|ActivateHpetSupport||
|DisableSecurityPolicy||
|EnableVectorAcceleration|x|
|ExitBootServicesDelay||
|ForceOcWriteFlash||
|ForgeUefiSupport||
|IgnoreInvalidFlexRatio||
|ReleaseUsbOwnership||
|ReloadOptionRoms||
|RequestBootVarRouting|x|
|ResizeGpuBars||
|TscSyncTimeout||
|UnblockFsConnect°|( )|

`°` `UnblockFsConnect`: Enable on HP Machines

## AMD Bulldozer (15h) and Jaguar (16h)

### SMBIOS Requirements
- **Dektop**: [**various**](https://dortania.github.io/OpenCore-Install-Guide/AMD/fx.html#platforminfo)

### ACPI Quirks    
| CPU Family      | Bulldozer and Jaguar |
|:----------------|:------------------:|
| **ACPI Quirks** | Desktop            |    
|                 |                    |
|FadtEnableReset  |
|NormalizeHeaders |
|RebaseRegions    |
|ResetHwSig       | 
|ResetLogoStatus° |(x)|
|SyncTableIDs     |

`°`Default in `sample.plist`

### Boooter Quirks
| CPU Family         | Bulldozer and Jaguar |
|:------------------ |:------------------:|
| **Booter Quirks**  | Desktop            |    
|                    |                    |
|AllowRelocationBlock|
|AvoidRuntimeDefrag|x
|DevirtualiseMmio||
|DisableSingleUser|
|DisableVariableWrite|
|DiscardHibernateMap|
|EnableSafeModeSlide|x
|EnableWriteUnprotector|X
|ForceBooterSignature
|ForceExitBootServices
|ProtectMemoryRegions|
|ProtectSecureBoot
|ProtectUefiServices|
|ProvideCustomSlide°|X
|ProvideMaxSlide
|RebuildAppleMemoryMap|x|
|ResizeAppleGpuBars
|SetupVirtualMap|x|
|SignalAppleOS
|SyncRuntimePermissions||

`°` `ProvideCustomSlide`: If the message "OCABC: All slides are usable!" appears in the log, you can disable ProvideCustomSlide.

### Kernel Quirks
- For AMD, enable `Kernel` > `Emulate`: `DummyPowerManagement`
- AMD also requires a lot of [**Kernel patches**](https://github.com/AMD-OSX/AMD_Vanilla/tree/master) to make macOS work.

| CPU Family         | Bulldozer and Jaguar |
|:------------------ |:------------------:|
| **kernel Quirks**  | Desktop            |    
|                    |                    |
|AppleCpuPmCfgLock||
|AppleXcpmCfgLock||
|AppleXcpmExtraMsrs||
|AppleXcpmForceBoost||
|CustomSMBIOSGuid||
|DisableIoMapper||
|DisableLinkeditJettison||
|DisableRtcChecksum||
|ExtendBTFeatureFlags||
|ExternalDiskIcons||
|ForceSecureBootScheme||
|IncreasePciBarSize||
|LapicKernelPanic||
|LegacyCommpage||
|PanicNoKextDump|x|
|PowerTimeoutKernelPanic|x|
|ProvideCurrentCpuInfo|x|
|SetApfsTrimTimeout||
|ThirdPartyDrives||
|XhciPortLimit°||

`°` `XhciPortLimit`: Not required on AMD systems, since these boards usually have 2 or more USB controllers with 10 Ports max.

### UEFI Quirks
| CPU Family       | Bulldozer and Jaguar |
|:---------------- |:------------------:|
| **UEFI Quirks**  | Desktop            |    
|                  |                    |
|ActivateHpetSupport||
|DisableSecurityPolicy||
|EnableVectorAcceleration|x|
|ExitBootServicesDelay||
|ForceOcWriteFlash||
|ForgeUefiSupport||
|IgnoreInvalidFlexRatio||
|ReleaseUsbOwnership||
|ReloadOptionRoms||
|RequestBootVarRouting|x|
|ResizeGpuBars||
|TscSyncTimeout||
|UnblockFsConnect°|( )|

`°` `UnblockFsConnect`: Enable on HP Machines
</details>
