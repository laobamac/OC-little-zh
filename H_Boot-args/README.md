# Boot Arguments Explained
一个关于常用（及较少用）启动参数和设备属性的非完整列表，这些参数可以通过 OpenCore 和 Clover 等引导管理器注入。这里的信息不是简单地从它们的官方仓库复制粘贴的，而是尽量补充了更多背景信息。此外，为了更清晰地展示这些参数，我采用了表格而不是列表，便于复制条目以及快速识别与某些启动参数或属性相关的附加参数（例如开关和位掩码）。

<details>
<summary><b>目录</b>（点击展开）</summary>

- [Boot Arguments Explained](#boot-arguments-explained)
	- [调试](#调试)
	- [网络相关启动参数](#网络相关启动参数)
	- [其他有用的启动参数](#其他有用的启动参数)
	- [启动参数和 Kext 提供的设备属性](#启动参数和-kext-提供的设备属性)
		- [Lilu](#lilu)
		- [VirtualSMC](#virtualsmc)
		- [Whatevergreen](#whatevergreen)
			- [Global](#global)
			- [Board-id related](#board-id-related)
			- [Switching GPUs](#switching-gpus)
			- [Intel HD Graphics](#intel-hd-graphics)
			- [AMD Radeon](#amd-radeon)
				- [`unfairgva` Overrides](#unfairgva-overrides)
			- [NVIDIA](#nvidia)
			- [Backlight](#backlight)
			- [2nd Boot stage](#2nd-boot-stage)
		- [AirportBrcmFixup](#airportbrcmfixup)
		- [AppleALC](#applealc)
		- [BrcmPatchRAM](#brcmpatchram)
		- [BrightnessKeys](#brightnesskeys)
		- [CPUFriend](#cpufriend)
		- [CpuTscSync](#cputscsync)
		- [CryptexFixup](#cryptexfixup)
		- [DebugEnhancer](#debugenhancer)
		- [ECEnabler](#ecenabler)
		- [HibernationFixup](#hibernationfixup)
		- [NootedRed](#nootedred)
		- [NVMeFix](#nvmefix)
		- [RestrictEvents](#restrictevents)
		- [RTCMemoryFixup](#rtcmemoryfixup)
	- [Credits](#credits)

</details>

## 调试

|启动参数| 描述|
|:-------:|-----------|
|**`-v`**|详细模式（Verbose Mode）。用文本输出替换启动过程中的进度条，帮助识别问题。推荐与 `debug=0x100` 和 `keepsyms=1` 一起使用。
|**`-s`**|单用户模式（Single User Mode）。启动后进入终端模式，可用于修复系统问题。应通过 Quirk 禁用该模式，以防被用于绕过管理员账户密码。
|**`-x`**|安全模式（Safe Mode）。以最小系统扩展和功能启动 macOS。适用于 OpenCore Legacy Patcher 的 Root Patches 不再工作且无法启动系统的情况。
|**`-f`**|在 Clover 中启用无缓存启动（Cacheless Booting）。在 OpenCore 中需要更改 `Kernel/Scheme/KernelCache` 为 `Cacheless`。仅适用于 macOS 10.6 至 10.9。
|**`debug=0x100`**|禁用 WatchDog，防止内核恐慌时系统自动重启，以便查看错误信息（需启用详细模式）。
|**`keepsyms=1`**|配合 `debug=0x100` 使用，使操作系统在内核恐慌时也输出符号信息，便于排查问题。
|**`dart=0`**|禁用 VT-x/VT-d。现已通过 `DisableIOMapper` Quirk 替代。
|**`cpus=1`**|将 CPU 核心数限制为 1。适用于 macOS 无法启动或安装的情况。
|**`npci=0x2000`**/**`npci=0x3000`**|禁用与 `kIOPCIConfiguratorPFM64` 相关的 PCI 调试。可用于解决启动时卡在 **`PCI Start Configuration`** 的问题。**如果 BIOS 中支持 `Above4GDecoding`，则无需使用此参数。**
|**`-no_compat_check`**|禁用 macOS 兼容性检查，允许在不受支持的 SMBIOS/board-id 上启动 macOS。注意：<ul><li>安装 macOS 时仍需临时使用受支持的 SMBIOS。</li><li>如果此参数激活，则无法安装系统更新。添加 `RestrictEvents.kext` 和启动参数 `revpatch=sbvmm` 可解除此限制，但[**需 macOS 11.3 或更新版本**](/S_System_Updates)。</li></ul>
|**`-liludbgall`**|调试 Lilu kext 和插件（需要 Lilu 的 DEBUG 版本）。

> [!TIP]
>
> - 推荐用于首次安装 macOS 的启动参数：`-v`、`keepsyms=1` 和 `debug=0x100`
> - 更详细的调试指南请参考 [**此处**](https://caizhiyuan.gitee.io/opencore-install-guide/troubleshooting/kernel-debugging.html#efi-setup)

</details>

## 网络相关启动参数
| 启动参数 | 描述 |
|----------|------|
| **`dk.e1000=0`**（Big Sur）</br> **`e1000=0`**（Monterey） | 禁止 Intel I225-V 网卡使用 `com.apple.DriverKit-AppleEthernetE1000.dext`（Apple 的新驱动扩展），并改用 `AppleIntelI210Ethernet.kext`。这一设置是可选的，因为大多数搭载 I225-V 网卡的 400 系列及更新主板与 `.dext` 版本驱动兼容。然而，在某些仅支持 `.kext` 驱动的 Gigabyte 主板上，这是必须的。</br>:warning: 这些启动参数在 macOS Ventura 中已失效，因为 `.kext` 版本的驱动已从 `/S/L/E/` 下的 `IONetworkingFamily.kext` 中移除！ |

---

## 其他有用的启动参数
| 启动参数 | 描述 |
|----------|------|
| **`alcid=X`** | 为 AppleALC 选择一个 Layout ID，其中 `X` 需要是指定 Layout ID 的数值。请参考 [支持的编码器](https://github.com/acidanthera/applealc/wiki/supported-codecs) 以确定适合你系统音频 CODEC 的 Layout ID。 |
| **`amfi=0x80`** 或 **`amfi_get_out_of_my_way=1`** | 这两个参数功能相同，实际上是 [一个位掩码](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/data/amfi_data.py)。设置为 `0x80` 会禁用 Apple Mobile File Integrity (AMFI)。AMFI 是一个 macOS 内核模块，用于强制执行代码签名和库验证，以增强系统安全性。**注意：需要同时禁用 SIP！** |
| **`-force_uni_control`** | 强制启用 macOS Monterey 12.3+ 的 Universal Control 功能（通用控制）。 |

## 启动参数和 Kext 提供的设备属性

### Lilu
Lilu 的启动参数。请注意，Lilu 是一个补丁引擎，为 Hackintosh 生态系统中的许多其他 Kext 提供功能。如果你使用这些命令，特别是 `-liluoff`，请格外小心！

| 启动参数 | 描述 |
|----------|------|
| **`-liludbg`** | 启用调试打印（需要 Lilu 的 DEBUG 版本）。 |
| **`-liludbgall`** | 启用 Lilu 和所有加载插件的调试打印（仅适用于 DEBUG 二进制文件）。 |
| **`-liluoff`** | 禁用 Lilu（以及所有使用 Lilu 的插件 Kext）。 |
| **`-liluuseroff`** | 禁用 Lilu 的用户补丁功能（例如 `dyld_shared_cache` 的修改）。 |
| **`-liluslow`** | 启用传统用户补丁。 |
| **`-lilulowmem`** | 禁用内核解包（在恢复模式中禁用 Lilu）。 |
| **`-lilubeta`** | 在不受支持的 macOS 版本上启用 Lilu（默认支持 macOS 13 及更低版本）。 |
| **`-lilubetaall`** | 在不受支持的 macOS 上启用 Lilu 和 *所有* 已加载插件（需谨慎使用）。 |
| **`-liluforce`** | 无论模式、操作系统、安装程序或恢复模式如何，强制启用 Lilu。 |
| **`liludelay=1000`** | 添加 1 秒（1000 毫秒）的延迟以进行排错。 |
| **`lilucpu=N`** | 让 Lilu 和插件假设为第 N 个 CPUInfo::CpuGeneration。 |
| **`liludump=N`** | 让 Lilu 的 DEBUG 版本在 N 秒后将日志转储到 `/var/log/Lilu_VERSION_KERN_MAJOR.KERN_MINOR.txt`。 |

---

### VirtualSMC
一个高级的 Apple SMC 模拟器 Kext，需依赖 Lilu 才能完全发挥功能。此外包含附加的 [**传感器插件**](https://github.com/acidanthera/VirtualSMC/tree/master/Sensors)。

| 启动参数 | 描述 |
|----------|------|
| **`-vsmcdbg`** | 启用调试打印（需要 VirtualSMC 的 DEBUG 版本）。 |
| **`-vsmcbeta`** | 在不受支持的操作系统上启用 Lilu 增强功能（默认支持 macOS 13 及以下版本）。 |
| **`-vsmcoff`** | 禁用所有 Lilu 增强功能。 |
| **`-vsmcrpt`** | 将缺失的 SMC 键报告到系统日志中。 |
| **`-vsmccomp`** | 如果发现现有硬件 SMC 实现，则优先使用它。 |
| **`vsmcgen=X`** | 强制公开 X 代 SMC 设备（支持 1 和 2）。 |
| **`vsmchbkp=X`** | 设置 HBKP 转储模式（0 = 关闭，1 = 正常，2 = 无加密）。 |
| **`vsmcslvl=X`** | 设置值序列化级别（0 = 关闭，1 = 正常，2 = 包含敏感数据（默认））。 |
| **`smcdebug=0xff`** | 启用 AppleSMC 调试信息打印。 |
| **`watchdog=0`** | 禁用 WatchDog 计时器（当发生意外重启时使用）。 |


[**源码**](https://github.com/acidanthera/VirtualSMC)

### WhateverGreen
以下启动参数由 **WhateverGreen.kext** 提供并依赖该 kext 才能生效！请参考 [完整列表](https://github.com/acidanthera/WhateverGreen/blob/master/README.md#boot-arguments) 了解更多详细信息。

#### 全局

| 启动参数 | 属性 | 描述 |
|----------|:----:|------|
| **`-cdfon`** | `enable-hdmi20` | 启用 iGPU 和 dGPU 的 HDMI 2.0 补丁（macOS 11+ 不再支持）。 |
| **`-wegbeta`** | N/A | 在不支持的 macOS 版本上启用 WhateverGreen（默认支持 macOS 13 及更低版本）。 |
| **`-wegdbg`** | N/A | 启用调试打印（需要 DEBUG 版本的 WhateverGreen）。 |
| **`-wegoff`** | N/A | 禁用 WhateverGreen。 |

#### 与 Board-id 相关

| 启动参数 | 属性 | 描述 |
|----------|:----:|------|
| **`agdpmod=ignore`** | `agdpmod` | 禁用 AGDP 补丁（默认值为 `vit9696,pikera`，适用于外部 GPU）。 |
| **`agdpmod=pikera`** | `agdpmod` | 用 `board-ix` 替换 `board-id`，禁用 AMD Navi（RX 5000/6000 系列）的 Board-ID 检查，解决因 x6000 驱动程序框架差异引起的黑屏问题。虽然 Polaris 或 Vega 显卡不需要此补丁，但在多显示器设置中遇到黑屏问题时也可以使用。 |
| **`agdpmod=vit9696`** | `agdpmod` | 禁用 `board-id` 检查。如果启动 macOS 后屏幕变黑，该补丁可能有用。 |

#### 切换 GPU

| 启动参数 | 属性 | 描述 |
|----------|:----:|------|
| **`-wegnoegpu`** | `disable-gpu` | 禁用所有外部 GPU，仅启用 Intel CPU 的集成显卡。如果 GPU 与 macOS 不兼容，可以使用此选项（不一定总有效）。建议使用 SSDT 在 macOS 中禁用 GPU。 |
| **`-wegnoigpu`** | `disable-gpu` | 禁用集成显卡。 |
| **`-wegswitchgpu`** | `switch-to-external-gpu` | 检测到外部 GPU 时禁用集成显卡。 |

#### Intel HD Graphics

以下启动参数适用于 **WhateverGreen.kext**，并需该 kext 才能生效。更多详情请查阅 [官方手册](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md)。

| 启动参数 | 属性 | 描述 |
|----------|:----:|------|
| **`-igfxblr`** | `enable-backlight-registers-fix` | 修复 Kaby Lake、Coffee Lake 和 Ice Lake 平台上的背光寄存器问题。 |
| **`-igfxbls`** | `enable-backlight-smoother` | 平滑 Ivy Bridge 及更新平台的亮度过渡。详见 [手册](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#customize-the-behavior-of-the-backlight-smoother-to-improve-your-experience)。 |
| **`-igfxcdc`** | `enable-cdclk-frequency-fix` | 支持 Ice Lake CPU 家族的所有有效核心显示时钟（CDCLK）。详见 [手册](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#support-all-possible-core-display-clock-cdclk-frequencies-on-icl-platforms)。 |
| **`-igfxdbeo`** | `enable-dbuf-early-optimizer` | 修复 Ice Lake 平台的显示数据缓冲区（DBUF）问题。详见 [手册](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#fix-the-issue-that-the-builtin-display-remains-garbled-after-the-system-boots-on-icl-platforms)。 |
| **`-igfxdump`** | N/A | 将 IGPU 帧缓冲区 kext 转储到 `/var/log/AppleIntelFramebuffer_X_Y`（需要 DEBUG 版 WEG）。 |
| **`-igfxdvmt`** | `enable-dvmt-calc-fix` | 修复 Ice Lake 平台因 DVMT 预分配内存计算错误导致的内核恐慌。 |
| **`-igfxfbdump`** | N/A | 将原生和补丁后的帧缓冲表转储到 ioreg 的 `IOService:/IOResources/WhateverGreen`。 |
| **`-igfxhdmidivs`** | `enable-hdmi-dividers-fix` | 修复 Skylake、Kaby Lake 和 Coffee Lake 平台上高像素时钟率 HDMI 连接时的无限循环问题。 |
| **`-igfxi2cdbg`** | N/A | 启用 I2C-over-AUX 事务的详细输出（仅用于调试）。 |
| **`-igfxlspcon`** | `enable-lspcon-support` | 启用 LSPCON 芯片的驱动支持，用于将 DisplayPort 转换为 HDMI 2.0 输出。详见 [手册](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#lspcon-driver-support-to-enable-displayport-to-hdmi-20-output-on-igpu)。 |
| **`-igfxmlr`** | `enable-dpcd-max-link-rate-fix` | 应用最大链路速率修复补丁。 |
| **`-igfxmpc`** | `enable-max-pixel-clock-override` 和 `max-pixel-clock-frequency` | 增加最大像素时钟（替代 `CoreDisplay.framework` 的补丁）。 |
| **`-igfxnohdmi`** | `disable-hdmi-patches` | 禁用数字音频的 DP 转 HDMI 补丁。 |
| **`-igfxsklaskbl`** | N/A | 在 Skylake 平台上强制使用 Kaby Lake 图形 kext（需要 KBL `device-id` 和 `ig-platform-id`，macOS 13+ 不需要此补丁）。 |
| **`-igfxtypec`** | N/A | 强制启用 Type-C 平台的 DP 连接。 |
| **`-igfxvesa`** | N/A | 禁用图形加速并改为软件渲染。适用于 iGPU 和 dGPU 不兼容的情况，或更新 macOS 后使用过期 NVIDIA WebDrivers 导致显示在启动时无法点亮。 |
| **`igfxagdc=0`** | `disable-agdc` | 禁用 iGPU 的 AGDC（Apple 自动设备门控控制），解决 4K 显示器的唤醒问题。 |
| **`igfxfcms=1`** | `complete-modeset` | 在 Skylake 或 Apple 固件上强制完全模式设置。 |
| **`igfxfcmsfbs=`** | `complete-modeset-framebuffers` | 指定需要强制完全模式设置的连接器索引。格式为 64 位字节，例如，值 `0x010203` 指定连接器 1、2、3。传递 `-1` 禁用。 |
| **`igfxframe=frame`** | `AAPL,ig-platform-id`（Ivy Bridge 及更新）或 `AAPL,snb-platform-id`（仅 Sandy Bridge） | 将专用帧缓冲标识符注入 IGPU（仅用于测试）。 |
| **`igfxfw=2`** | `igfxfw` | 强制加载 Apple Graphics Unit Control (GUC) 固件，提升 Intel Quick Sync Video 性能。需要支持 Intel ME v12 或更新版本的芯片组（如 Z390、B360 等）。[存在问题](https://github.com/acidanthera/bugtracker/issues/800)，需逐个测试。 |
| **`igfxrpsc=1`** | `rps-control` | 启用 RPS 控制补丁，提升 Kaby Lake+ 芯片组的 iGPU 性能（适用于不支持 ME v12 的芯片组，如 Z370）。 |
| **`wegtree=1`** | `rebuild-device-tree` | 在加载 Apple GUC 固件后强制 WEG 重建 iGPU 的设备树。 |
| **`igfxgl=1`** | `disable-metal` | 禁用 Intel 的 Metal 支持。 |
| **`igfxmetal=1`** | `enable-metal` | 强制为 Intel 启用 Metal 支持，用于离线渲染。 |
| **`igfxonln=1`** | `force-online` | 强制所有显示器在线状态。解决 macOS 10.15.4+ 使用 Intel UHD 630 时的屏幕唤醒问题。 |
| **`igfxonlnfbs=MASK`** | `force-online-framebuffers` | 指定需要强制在线状态的连接器索引，格式类似于 `igfxfcmsfbs`。 |
| **`igfxpavp=1`** | `igfxpavp` | 强制启用 PAVP 输出。 |
| **`igfxsnb=0`** | N/A | 禁用 Sandy Bridge CPU 的 IntelAccelerator 名称修复。 |

#### AMD Radeon

以下启动参数适用于 **AMD Radeon GPUs**，由 **WhateverGreen.kext** 提供支持。

| 启动参数 | 属性 | 描述 |
|----------|:----:|------|
| **`-rad24`** | N/A | 强制显示为 24 位颜色模式。 |
| **`-radcodec`** | N/A | 强制在 AMDRadeonVADriver 中使用伪造的 PID。 |
| **`-raddvi`** | N/A | 启用 DVI 发射器修正（适用于 290X、370 等型号）。 |
| **`-radvesa`** | N/A | 完全禁用 ATI/AMD 视频加速，将 GPU 强制为 VESA 模式。适用于不受 macOS 支持的显卡或进行故障排除。Apple 内置的类似参数为 `-amd_no_dgpu_accel`。 |
| **`radpg=15`** | N/A | 禁用多个电源门控模式。适用于 Cape Verde GPU（如 Radeon HD 7730/7750/7770、R7 250/250X）。更多信息请参考 [Radeon FAQ](https://github.com/dreamwhite/WhateverGreen/blob/master/Manual/FAQ.Radeon.en.md)。 |
| **`shikigva=40`** + **`shiki-id=Mac-7BA5B2D9E42DDD94`** | N/A | 将 BoardID 替换为 `iMacPro1,1`，允许 Polaris、Vega 和 Navi GPU 处理所有类型的渲染任务。适用于期望 iGPU 的 SMBIOS，但在 macOS 11+ 中已废弃。详情请参考 [**Fixing DRM**](https://dortania.github.io/OpenCore-Post-Install/universal/drm.html#testing-hardware-acceleration-and-decoding)。 |
| **`CFG,CFG_USE_AGDC` (Type: Data; Value: 00)** | `disable-agdc` | 禁用 AMD GPU 的 AGDC（Apple 自动设备门控控制），解决使用 4K 显示器时的唤醒问题。 |
| **`unfairgva=x`** (x = 1 至 7) | N/A | 替代 macOS 11+ 中的 `shikigva`，解决 [**DRM**](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.Chart.md) 相关问题。`unfairgva` 是一个 3 位位掩码（1、2 和 4），可组合启用或选择不同功能。更多信息详见 [**这里**](https://www.insanelymac.com/forum/topic/351752-amd-gpu-unfairgva-drm-sidecar-featureunlock-and-gb5-compute-help/)。 |

##### `unfairgva` 重写选项

`unfairgva` 启动参数支持重写，用于配置 AMD GPU 的视频解码器偏好，以适配不同类型的 DRM 内容（如 Apple TV 和 iTunes 电影流媒体）。但这些偏好可能与操作系统的其他硬件媒体解码和编码方式不兼容，因此不推荐日常使用，仅在需要时启用。

- **强制 AMD DRM 解码器支持流媒体服务（如 Apple TV 和 iTunes）**：

	```shell
	defaults write com.apple.AppleGVA gvaForceAMDKE -boolean yes
	``` 
- **强制启用 AMD AVC 加速解码**：

	```shell
	defaults write com.apple.AppleGVA gvaForceAMDAVCDecode -boolean yes
	```
- **强制启用 AMD AVC 加速编码**：

	```shell
	defaults write com.apple.AppleGVA gvaForceAMDAVCEncode -boolean yes
	```
- **强制启用 AMD HEVC 加速解码**：

	```shell
	defaults write com.apple.AppleGVA gvaForceAMDHEVCDecode -boolean yes
	```
- **强制启用 AMD HEVC 加速编码**：

	```shell
	defaults write com.apple.AppleGVA disableGVAEncryption -string yes
	```
- **强制启用硬件加速视频解码（任意分辨率）**：

	```shell
	defaults write com.apple.coremedia hardwareVideoDecoder -string force
	``` 
- **禁用硬件加速视频解码（适用于 QuickTime 或 Apple TV）**：

	```shell 
	defaults write com.apple.coremedia hardwareVideoDecoder -string disable
	``` 

#### NVIDIA

| 启动参数 | 属性 | 描述 |
|----------|:----:|------|
| **~~`nvda_drv=1`~~** (≤ macOS 10.11) | N/A | 已废弃。从 macOS Sierra 开始，需要使用 NVRAM 键：<ul><li>**OpenCore**: `NVRAM/Add/7C436110-AB2A-4BBB-A880-FE41995C9F82/nvda_drv: 31` (**Type**: Data)</li><li>**Clover**: 在 `System Parameters` 下启用 `NvidiaWeb`</li></ul> |
| **`-ngfxdbg`** | N/A | 启用 NVIDIA 驱动错误日志记录。 |
| **`ngfxcompat=1`** | `force-compat` | 忽略 NVDAStartupWeb 的兼容性检查。 |
| **`ngfxgl=1`** | `disable-metal` | 禁用 NVIDIA 的 Metal 支持。 |
| **`ngfxsubmit=0`** | `disable-gfx-submit` | 禁用 macOS 10.13 的界面卡顿修复功能。 |
| **`nv_disable=1`** | N/A | 禁用硬件加速并改为 VESA 模式。适用于 macOS 升级期间，在驱动重新安装前确保显卡能够显示图像。**注意：不要与 `nvda_drv=1` 同时使用！** |
| **`agdpmod=pikera`** | `agdpmod` | 将 `board-id` 替换为 `board-ix`，禁用字符串比较，适用于非 iMac13,2/iMac14,2 的 SMBIOS。 |
| **`agdpmod=vit9696`** | `agdpmod` | 禁用 `board-id` 检查，解决启动 macOS 后屏幕变黑的问题。 |
| **`shikigva=1`** | N/A | 允许 iGPU 显示输出与 dGPU 同时使用，即使未使用无连接的帧缓冲器也可启用硬件解码。 |
| **`shikigva=4`** | N/A | 支持 Haswell 之后的硬件加速视频解码。可能需要与 `shikigva=12` 结合使用以修补必要的进程。 |
| **`shikigva=40`** | N/A | 将 BoardID 替换为 iMac14,2。适用于不期望使用 NVIDIA GPU 的 SMBIOS，但 WhateverGreen 通常会自动处理补丁。 |

---

#### 背光

| 启动参数 | 属性 | 描述 |
|----------|:----:|------|
| **`applbkl=3`** | `applbkl` | 启用 AMD Radeon RX 5000 系列显卡的 PWM 背光控制。详见 [此处](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.Radeon.en.md)。 |
| **`applbkl=0`** | `applbkl` | 禁用 AppleBacklight.kext 补丁（适用于 IGPU）。如需自定义 AppleBacklight 配置，请参考 [此处](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.OldPlugins.en.md)。 |

---

#### 第二阶段启动

| 启动参数 | 描述 |
|----------|------|
| **`gfxrst=1`** | 优先在第二阶段启动时绘制 Apple Logo，而不是复制帧缓冲区。 |
| **`gfxrst=4`** | 禁用第二阶段启动期间的帧缓冲区初始化交互。 |

---

**来源**：[**Dortania**](https://dortania.github.io/GPU-Buyers-Guide/misc/bootflag.html) 和 [**WhateverGreen**](https://github.com/acidanthera/WhateverGreen)

---

### AirportBrcmFixup

这是一个开源 Kext，用于为非原生的 Broadcom WiFi 网卡提供必要的补丁。

| 启动参数 | NVRAM 变量 | 描述 |
|----------|:---------:|------|
| **`-brcmfxdbg`** | N/A | 启用调试输出。 |
| **`-brcmfxbeta`** | N/A | 在不受支持的 macOS 上启用加载。 |
| **`-brcmfxoff`** | N/A | 禁用 Kext。 |
| **`-brcmfxwowl`** | N/A | 启用 WLAN 唤醒（默认禁用）。 |
| **`-brcmfx-alldrv`** | N/A | 允许为所有受支持驱动程序进行补丁，而不考虑当前系统版本。详见 [不同 macOS 版本中设备 ID 和 Kext 名称匹配](https://github.com/acidanthera/AirportBrcmFixup#matching-device-id-and-kext-name-in-different-macos-versions)。 |
| **`brcmfx-country=XX`** | `brcmfx-country=XX` | 将国家代码更改为 `XX`（可以是 US、CN、DE 等，或者使用 #a 表示通用）。 |
| **`brcmfx-aspm`** | N/A | 覆盖用于 `pci-aspm-default` 的值。 |
| **`brcmfx-wowl`** | N/A | 启用/禁用 WLAN 唤醒补丁。 |
| **`brcmfx-delay`** | N/A | 延迟加载 Broadcom 原生驱动程序的启动，单位为毫秒。适用于解决 Monterey 中的内核恐慌或 WiFi 设备丢失问题。建议从 15 秒延迟开始（`brcmfx-delay=15000`），逐步减少延迟直到发现启动不稳定为止。 |
| **`brcmfx-driver=0/1/2/3`** | 相同 | 仅启用特定的 Kext 加载：</br> `0` = AirPortBrcmNIC-MFG </br> `1` = AirPortBrcm4360 </br> `2` = AirPortBrcmNIC </br> `3` = AirPortBrcm4331 |

**来源**：[AirportBrcmFixup](https://github.com/acidanthera/AirportBrcmFixup)

---

### AppleALC

以下是适用于 **AppleALC** 的启动参数，这是一款常用的音频补丁 Kext。请注意，所有 Lilu 的启动参数同样适用于 AppleALC。

| 启动参数 | 属性 | 描述 |
|----------|:----:|------|
| **`alcid=X`** | `layout-id` | 选择音频 Layout ID，其中 `X` 是 Layout ID 的数值。例如，`alcid=1`。完整的 Layout ID 列表（按厂商和编解码器型号排序）请参考 [此处](https://github.com/dreamwhite/ChonkyAppleALC-Build)。 |
| **`-alcoff`** | N/A | 禁用 AppleALC（Bootmode 的 `-x` 和 `-s` 也会禁用 AppleALC）。 |
| **`-alcbeta`** | N/A | 在不受支持的系统（通常是未发布或旧系统）上启用 AppleALC。 |
| **`-alcdbg`** | N/A | 打印调试信息（需要 DEBUG 版本）。 |
| **`alcverbs=1`** | `alc-verbs` | 启用 alc-verb 支持。 |

**来源**：[AppleALC Wiki](https://github.com/acidanthera/AppleALC/wiki/Installation-and-usage)

---

### BrcmPatchRAM

BrcmPatchRAM 是一个 macOS 驱动程序，用于为基于 Broadcom RAMUSB 的设备应用 PatchRAM 更新。它会在每次启动或唤醒时为 Broadcom 蓝牙设备应用固件更新。由于该 Kext 包含多个需要根据硬件和 macOS 版本 [组合使用的 Kext](https://dortania.github.io/OpenCore-Post-Install/usb/#example-7-broadcom-wifi-and-bluetooth)，请按照官方仓库的说明进行正确配置。

| 启动参数 | 描述 |
|----------|------|
| **`-btlfxallowanyaddr`** | 在 macOS 12.4 及更高版本中禁用地址检查。Apple 在 `bluetoothd` 中引入了新地址检查，如果两个蓝牙设备地址相同，则会触发错误。 |
| **`bpr_initialdelay=X`** | 更改 `mInitialDelay`，其中 `X` 是与设备通信前的延迟时间（单位：毫秒）。默认值为 `100`。<br>**示例**：`bpr_initialdelay=300` |
| **`bpr_handshake=X`** | 覆盖 `mSupportsHandshake`，可选值：<br> `0` 表示在上传固件后等待 `bpr_preresetdelay` 毫秒，然后重置设备。<br> `1` 表示等待设备的特定响应，然后重置设备。默认值取决于设备标识符。 |
| **`bpr_preresetdelay=X`** | 更改 `mPreResetDelay`，其中 `X` 是设备接受固件所需的延迟时间（单位：毫秒）。如果 `bpr_handshake=1`，该值将被忽略。默认值为 `250`。<br>**示例**：`bpr_preresetdelay=500` |
| **`bpr_postresetdelay=X`** | 更改 `mPostResetDelay`，其中 `X` 是设备在固件上传后重置时初始化固件所需的延迟时间（单位：毫秒）。默认值为 `100`。<br>**示例**：`bpr_postresetdelay=400` |
| **`bpr_probedelay=X`** | 更改 `mProbeDelay`（已在 BrcmPatchRAM3 中移除），其中 `X` 是探测设备前的延迟时间（单位：毫秒）。默认值为 `0`。 |

> [!TIP] 对于部分用户的典型“从睡眠唤醒”问题，以下设置可能有效：
> 
> - `bpr_probedelay=100 bpr_initialdelay=300 bpr_postresetdelay=300`
> - `bpr_probedelay=200 bpr_initialdelay=400 bpr_postresetdelay=400`（稍长延迟）

**来源**：[BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM)

---

### BrightnessKeys

BrightnessKeys 根据 ACPI 规范（附录 B：视频扩展）自动处理亮度键快捷键。需要 Lilu 1.2.0 或更新版本。

| 启动参数 | 描述 |
|----------|------|
| **`-brkeysdbg`** | 启用调试打印（仅适用于 DEBUG 二进制文件）。 |

**来源**：[BrightnessKeys](https://github.com/acidanthera/BrightnessKeys)

---

### CPUFriend
用于优化支持 XCPM（Haswell 及更新）功能的 Intel CPU 的电源管理的 Kext。

| 启动参数 | 描述 |
|----------|------|
| **`-cpufdbg`** | 启用调试日志记录（仅适用于 DEBUG 版本）。 |
| **`-cpufoff`** | 完全禁用 CPUFriend。 |
| **`-cpufbeta`** | 在不受支持的 macOS 版本上启用 CPUFriend。 |

**来源**：[CPUFriend](https://github.com/acidanthera/CPUFriend/)

---

### CpuTscSync
Lilu 插件，结合了 VoodooTSCSync 的功能，同时在 TSC 不同步时禁用 `xcpm_urgency`。适用于解决从睡眠唤醒后的一些内核恐慌问题。

| 启动参数 | 描述 |
|----------|------|
| **`-cputsdbg`** | 启用调试输出。 |
| **`-cputsbeta`** | 在不受支持的 macOS 版本上启用 Kext 加载。 |
| **`-cputsoff`** | 禁用 Kext 加载。 |
| **`-cputsclock`** | 强制使用 `clock_get_calendar_microtime` 方法同步 TSC（与启动参数 `TSC_sync_margin` 使用相同方法）。 |

**来源**：[CpuTscSync](https://github.com/acidanthera/CpuTscSync)

---

### CryptexFixup
Lilu 插件，用于在 macOS Ventura 中安装 Rosetta Cryptex。这是为在旧版 Intel（Ivy Bridge 及更早）和 AMD（Bulldozer/Piledriver/Steamroller 及更早）CPU 上安装和引导 macOS 13 所必需的。更多详情请参考 [官方文档](https://dortania.github.io/OpenCore-Install-Guide/extras/ventura.html#dropped-cpu-support)。

| 启动参数 | 描述 |
|----------|------|
| **`-cryptoff`** | 禁用 Kext。 |
| **`-cryptdbg`** | 启用详细日志记录（仅适用于 DEBUG 版本）。 |
| **`-cryptbeta`** | 在 macOS 13 以上版本上启用。 |
| **`-crypt_allow_hash_validation`** | 禁用 `APFS.kext` 补丁。 |
| **`-crypt_force_avx`** | 强制在支持 AVX2.0 的系统上安装 Rosetta Cryptex。 |

**来源**：[CryptexFixup](https://github.com/acidanthera/CryptexFixup)

---

### DebugEnhancer
Lilu 插件，用于在 macOS 内核中启用调试输出。

| 启动参数 | 描述 |
|----------|------|
| **`-dbgenhdbg`** | 启用调试输出。 |
| **`-dbgenhbeta`** | 在不受支持的 macOS 版本上启用加载。 |
| **`-dbgenhoff`** | 禁用 Kext 加载。 |
| **`-dbgenhiolog`** | 将 IOLog 输出重定向到内核的 `vprintf`（与 `kdb_printf` 和 `kprintf` 类似）。 |

**来源**：[DebugEnhancer](https://github.com/acidanthera/DebugEnhancer)

---
### ECEnabler
Allows reading Embedded Controller fields over 1 byte long, vastly reducing the amount of ACPI modification needed (if any) for working battery status of Laptops. 

boot-arg | Description 
---------|------------
**`-eceoff`** | Disables ECEnabler  
**`-ecedbg`** | Enables debug logs  
**`-ecebeta`**| Removes upper macoOS limit (for macOS betas)  

[**Source**](https://github.com/1Revenger1/ECEnabler)

### HibernationFixup
Lilu plugin kext intended to fix hibernation compatibility issues. It's pretty complex in terms of configuration, so you're best to read the info provided in the repo!

boot-arg | Description 
---------|------------
**`-hbfx-dump-nvram`** | Saves NVRAM to a file nvram.plist before hibernation and after kernel panic (with panic info)
**`-hbfx-disable-patch-pci`**| Disables patching of IOPCIFamily (this patch helps to avoid hang & black screen after resume (restoreMachineState won't be called for all devices)
**`hbfx-patch-pci=XHC,IMEI,IGPU`** | Allows to specify explicit device list (and restoreMachineState won't  be called only for these devices). Also supports values `none`, `false`, `off`.
**`-hbfxdbg`**| Enables debugging output
**`-hbfxbeta`** | Enables loading on unsupported osx
**`-hbfxoff`** | Disables kext loading
**`hbfx-ahbm=abhm_value`** | Controls auto-hibernation feature, where "abhm_value" represenst an *arithmetic sum* (bitmask) of the following values: </br></br> `1` = `EnableAutoHibernation`: If this flag is set, system will hibernate instead of regular sleep (flags below can be used to limit this behavior)</br>  `2` = `WhenLidIsClosed` : Auto hibernation can happen when lid is closed (if bit is not set - no matter which status lid has)</br> `4` = `WhenExternalPowerIsDisconnected`: Auto hibernation can happen when external power is disconnected (if bit is not set - no matter whether it is connected) </br> `8` = `WhenBatteryIsNotCharging`: Auto hibernation can happen when battery is not charging (if bit is not set - no matter whether it is charging) </br> `16` = `WhenBatteryIsAtWarnLevel`: Auto hibernation can happen when battery is at warning level (macOS and battery kext are responsible for this level)</br> `32` = `WhenBatteryAtCriticalLevel`: Auto hibernation can happen when battery is at critical level (macOS and battery kext are responsible for this level) </br> `64` = `DoNotOverrideWakeUpTime`:	Do not alter next wake up time, macOS is fully responsible for sleep maintenance dark wakes </br> `128` = `DisableStimulusDarkWakeActivityTickle`: Disable power event kStimulusDarkWakeActivityTickle in kernel, so this event cannot trigger a switching from dark wake to full wake </br></br> **EXAMPLE**: `hbfx-ahbm=135`  would enable options associated with values `1`, `2`, `4` and `128`.

The following options can be stored in NVRAM and can be used instead of respective boot-args:

**GUID**: `NVRAM/Add/E09B9297-7928-4440-9AAB-D1F8536FBF0A`

NVRAM Key |Type  
----------|:---:
**`hbfx-dump-nvram`** | Boolean
**`hbfx-disable-patch-pci`** | Boolean
**`hbfx-patch-pci=XHC,IMEI,IGPU,none,false,off`**| String
**`hbfx-ahbm`** | Number

[**Source**](https://github.com/acidanthera/HibernationFixup)

### NootedRed
 Lilu plugin for AMD Vega iGPUs.

boot-arg | Description 
---------|------------
 `-CKFBOnly` | Enables "partial" acceleration 
 `-NRedDPDelay` | Adds delay to the boot sequence, useful if experiencing black screen on boot
 `-NRedDebug` | Enables detailed logging for `DEBUG` build

[**Source**](https://github.com/NootInc/NootedRed)

### NVMeFix
A set of patches for the Apple NVMe storage driver, IONVMeFamily. Its goal is to improve compatibility with non-Apple SSDs.

boot-arg | Description 
---------|------------
**`-nvmefdbg`** | Enables detailed logging for `DEBUG` build
**`-nvmefoff`** | Disables the kext
**`-nvmefaspm`** | forces ASPM L1 on all the devices. This argument is recommended exclusively for testing purposes!  For daily usage, inject `pci-aspm-default` device property with `<02 00 00 00>` value into SSD and bridge devices they are connected to instead. &rarr; &rarr; See [**Configuration**](https://github.com/acidanthera/NVMeFix/blob/master/README.md#configuration) for more details.

[**Source**](https://github.com/acidanthera/NVMeFix)

### RestrictEvents
boot-arg | NVRAM Key| Description 
---------|:--------:|------------
**`-revoff`**| –| Disables the kext
**`-revdbg`**| – | Enables verbose logging (in DEBUG builds)
**`-revbeta`**| – | Enables the kext on macOS < 10.8 or greater than 13
**`-revproc`**|–| Enables verbose process logging (in DEBUG builds)
**`revpatch=value`** |–| Enables patching as comma separated options. Default value is `auto`. See available values below.
**`revpatch=auto`** |YES| Enables `memtab,pci,cpuname`, without `memtab` and `pci` patches being applied on a real Mac.
**`revpatch=none`** |YES| Disables all patching 
**`revpatch=asset`** |YES| Allows Content Caching when `sysctl kern.hv_vmm_present` returns `1` on macOS 11.3 or newer
**`revpatch=cpuname`**|YES| Allows Custom CPU name in System Information
**`revpatch=diskread`** |YES| Disables "Uninitialized disk" warning in Finder
**`revpatch=f16c`** |YES| Resolves CoreGraphics crashing on Ivy Bridge CPUs by disabling f16c instruction set reporting in macOS 13.3 or newer
**`revpatch=memtab`** |YES|Enables memory tab in System Information on `MacBookAir` and `MacBookPro10,x`.
**`revpatch=pci`**|YES|Prevents PCI configuration warnings in System Settings on `MacPro7,1`
**`revpatch=sbvmm`** |YES| Forces VMM SB model, allowing OTA updates for unsupported models on macOS 11.3 or newer
**`revcpu=value`** |YES| Enables or disables CPU brand string patching. `1` = non-Intel default, `0` = Intel default
**`revcpuname=value`** |YES| Custom CPU brand string (max 48 characters, 20 or less recommended, taken from CPUID otherwise)
**`revblock=value`** |–| To block processes as comma separated options. Default value is `auto`.
**`revblock=auto`** |YES| Same as `pci`
**`revblock=none`**|YES| Disables all blocking
**`revblock=pci`**|YES| Prevents PCI and RAM configuration notifications on `MacPro7,1`
**`revblock=gmux`** |YES| Blocks `displaypolicyd` on Big Sur+ (for genuine MacBookPro9,1/10,1)
**`revblock=media`** |YES| Block `mediaanalysisd` on Ventura+ (for Metal 1 GPUs)

**NOTE**: NVRAM variables work the same way as the boot arguments, but have lower priority. They need to be added to the config under `NVRAM/Add/4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102`:

![revpatchrevblock](https://github.com/laobamac/OC-little-zh/assets/76865553/dcfad4c5-f646-4463-b6dc-2248d23c49cf)

[**Source**](https://github.com/acidanthera/RestrictEvents)

### RTCMemoryFixup
Kext providing a way to emulate some offsets in your CMOS (RTC) memory. 
More details about CMOS-related isses fan be found [here](/06_CMOS-related_Fixes).

boot-arg | Description 
:-------|------------
**`rtcfx_exclude=`** | The `=` is followed by values for offsets ranging from `00` to `FF` &rarr; Check repo description for details
**`-rtcfxdbg`** | Turns on debugging output

[**Source**](https://github.com/acidanthera/RTCMemoryFixup)

**To be continued…**

## Credits
- 1Revenger1 for ECEnabler
- Acidanthera for Lilu, VirtalSMC, Whatevergreen and others
- Miliuco and Andrey1970AppleLife for additional details about `igfxfw` and `rps-control` properties
