# OpenCore 配置小技巧
本节包含一些关于 OpenCore `config.plist` 的有用配置技巧。

<details>
<summary><b>目录</b>（点击展开）</summary><br>

- [OpenCore 配置小技巧](#opencore-配置小技巧)
	- [OpenCore 排错小贴士](#opencore-排错小贴士)
		- [排错流程](#排错流程)
		- [`MinDate`/`MinVersion` 的 APFS 驱动设置](#mindateminversion-的-apfs-驱动设置)
	- [I. 更新 config.plist 和修复错误](#i-更新-configplist-和修复错误)
		- [自动配置升级（推荐）](#自动配置升级推荐)
		- [手动升级和错误修复（旧方法）](#手动升级和错误修复旧方法)
		- [关于使用配置工具的个人建议](#关于使用配置工具的个人建议)
	- [II. 启动问题的快速修复](#ii-启动问题的快速修复)
	- [III. 安全设置](#iii-安全设置)
		- [如何禁用单用户模式](#如何禁用单用户模式)
		- [如何禁用系统完整性保护（SIP）](#如何禁用系统完整性保护sip)
	- [IV. 调整 Boot Picker 属性并启用鼠标支持](#iv-调整-boot-picker-属性并启用鼠标支持)
	- [V. 自定义启动选项](#v-自定义启动选项)
		- [在 BootPicker 中设置默认启动驱动器](#在-bootpicker-中设置默认启动驱动器)
		- [启用 Apple 热键功能](#启用-apple-热键功能)
		- [加速启动（结果可能因系统而异）](#加速启动结果可能因系统而异)
		- [启动方式选择](#启动方式选择)
			- [无 GUI 手动选择操作系统（默认）](#无-gui-手动选择操作系统默认)
			- [带 GUI 的手动选择操作系统（需要 OpenCanopy 和资源文件夹）](#带-gui-的手动选择操作系统需要-opencanopy-和资源文件夹)
			- [从“默认”卷自动启动操作系统（无 GUI）](#从默认卷自动启动操作系统无-gui)
			- [跳过 BootPicker 自动加载 macOS](#跳过-bootpicker-自动加载-macos)
	- [VI. 解决 NVRAM 问题](#vi-解决-nvram-问题)
		- [重置 NVRAM](#重置-nvram)
			- [OC ≤ 0.8.3](#oc--083)
			- [OC ≥ 0.8.4](#oc--084)
			- [重置 NVRAM 后保留启动条目](#重置-nvram-后保留启动条目)
		- [修复错误报告的 OpenCore 版本](#修复错误报告的-opencore-版本)
	- [VII. 禁止将 SMBIOS 注入到其他操作系统](#vii-禁止将-smbios-注入到其他操作系统)
	- [VIII. 在 OpenCore 和 Clover 之间交换 SMBIOS 数据](#viii-在-opencore-和-clover-之间交换-smbios-数据)
		- [手动方法](#手动方法)
			- [排错](#排错)
		- [使用 OCAT 导入/导出 SMBIOS 数据](#使用-ocat-导入导出-smbios-数据)
		- [Clover 用户的一键解决方案](#clover-用户的一键解决方案)
	- [更多资源](#更多资源)

</details>

## OpenCore 排错小贴士

### 排错流程

![OpenCore 排错流程](https://user-images.githubusercontent.com/76865553/135234918-2d0ce665-9037-4dd6-b0f4-e2b54c081160.png)

除了检查明显的设置（如 Booter、Kernel 和 UEFI Quirks），还需检查以下配置：

- `UEFI/ConnectDrivers` = true
- `Misc/Security/SecureBootModel` = Disabled
- `Misc/security/Vault` = Optional
- `UEFI/APFS/MinDate` = -1
- `UEFI/APFS/MinVersion` = -1
- 将 `UEFI/Drivers` 的结构与 OpenCore 包中的 `sample.plist` 比较（OC 0.7.3 格式有变）。

详细排错请参考 Dortania 的 [排错指南](https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/troubleshooting.html#table-of-contents)。

### `MinDate`/`MinVersion` 的 APFS 驱动设置
OpenCore 在 0.7.2 版本中引入了一个新的安全功能。如果 APFS 驱动程序的日期（`MinDate`）和版本（`MinVersion`）不符合特定标准，将会被禁止加载。

新默认值 `0` 和 `0` 支持 macOS Big Sur。如果运行旧版本 macOS，将看不到驱动器。禁用此功能，输入 `-1`，APFS 驱动将加载任何 macOS 版本。

**支持值列表：**

| `MinDate` | `MinVersion`     | 描述                                  |
|:---------:|:----------------:|:-------------------------------------|
| 0         | 0                | 默认，仅加载 Big Sur+ 的 APFS 驱动    |
| -1        | -1               | 禁用，允许加载任何 APFS 驱动         |
| 20210508  | 1677120009000000 | 需要 macOS ≥ Big Sur (11.4+)         |
| 20200306  | 1412101001000000 | 需要 macOS ≥ Catalina (10.15.4+)     |
| 20190820  | 9452750070000000 | 需要 macOS ≥ Mojave (10.14.6)        |
| 20180621  | 7480770080000000 | 需要 macOS ≥ High Sierra (10.13.6)   |

**来源**：[Acidanthera](https://github.com/acidanthera/OpenCorePkg/blob/master/Include/Acidanthera/Library/OcApfsLib.h)

> [!IMPORTANT]
>
> 出于安全考虑，您应该根据使用的 macOS 版本更改这些值。


## I. 更新 config.plist 和修复错误

### 自动配置升级（推荐）
自从发布 OpenCore Auxiliary Tools（[**OCAT**](https://github.com/ic005k/QtOpenCoreConfig)）以来，维护和更新 OpenCore 配置及文件的过程变得更加轻松。它可以自动更新/迁移任何过时的 `config.plist`，并提供最新的结构和功能集，还能更新 OpenCore、驱动程序和 Kext，并检查配置中的错误。更多细节请参阅我的 [OpenCore 更新指南](/D_Updating_OpenCore)。

### 手动升级和错误修复（旧方法）
在 OCAT 出现之前，我通常使用以下 4 个工具手动维护和更新配置，与最新的 `sample.plist` 进行比较并更新文件：

- **OCConfigCompare** – 比较 OpenCore 的 `sample.plist` 和你的 `config.plist` 之间的差异。
- **KextUpdater** – 更新 Kext、驱动程序等。
- **ProperTree** – 编辑配置并创建 OpenCore 快照。
- **OCValidate** – 检查配置是否存在格式错误。

手动升级配置和文件可能会非常耗时。既然 OCAT 现在可以通过几个简单的点击完成所有这些工作，我很高兴不再需要手动处理了，你也应该这样做。

### 关于使用配置工具的个人建议
“不使用配置工具”的建议已经逐渐演变为对所有配置工具的偏见，我认为这种看法是错误的。这一建议源于 config.plist 的结构经常发生变化的时期，那时配置工具开发者很难跟上新增功能的步伐，导致许多配置被破坏、系统无法启动。因此，大家建议不要使用它们。

随着时间的推移，这一建议变成了一个被机械重复的口号，而忽略了背后的原因。某些论坛、Reddit 和 Discord 中的一些意见领袖还会对使用这些工具的人进行排斥，这种现象并不健康。

然而，对于像 OpenCore Configurator 这样的应用来说，这一建议仍然成立。但 OCAT 在处理配置变化时更加灵活，它可以自动整合新增的配置键值，并与最新的 `sample.plist` 和 `OCValidate` 保持同步。此外，OCAT 还会自动备份你的配置，方便随时恢复。

因此，将 OCAT 与其他配置工具混为一谈是不公平的，那些所谓的意见领袖需要正视这一点。

## II. 启动问题的快速修复

如果系统在启动时没有进度条并停留在启动 Logo 上，即使引导和内核设置正确，也可以尝试更改以下设置：

- **Misc/Security/SecureBootModel** = `Disabled`。如果使用 `Default` 值无法启动，请暂时禁用 Secure Boot。一旦系统正常工作，建议检查 `SystemProductName` 的 Mac 模型是否支持 Apple 的 Secure Boot 功能。
- **Misc/Security/Vault** = `Optional`。禁用文件保险箱。如果设置为“Secure”但没有配置文件保险箱加密，可能会导致无法启动系统。

如果 BootPicker 中没有显示 macOS 分区（APFS 格式），请更改以下设置（适用于 OpenCore 0.7.2 及更新版本）：

- **UEFI/APFS**：将 `MinDate` 和 `MinVersion` 设置为 `-1`。这会禁用 APFS 驱动验证，无论 macOS 版本如何，驱动都会加载（从 macOS High Sierra 开始支持 APFS）。

> [!背景]
> 如果使用的操作系统版本早于 Big Sur，而 `MinDate` 和 `MinVersion` 的默认值为 `0`，APFS 驱动不会加载，导致无法显示 macOS 分区。这是一项安全功能，旨在确保 macOS 使用经过验证的 APFS 驱动。建议在安装过程中禁用以提高兼容性。

## III. 安全设置

### 如何禁用单用户模式
为了安全起见，建议禁用单用户模式，因为它可能被利用为绕过管理员账户密码保护的后门。启用以下选项即可：

**Misc/Security/DisableSingleUser** = `Yes`

### 如何禁用系统完整性保护（SIP）
1. 添加以下配置项来禁用 SIP：

   **NVRAM/Add/7C436110-AB2A-4BBB-A880-FE41995C9F82** &rarr; 将 `csr-active-config` 从 `00000000`（SIP 启用）更改为以下值：

   - `FF030000`（适用于 High Sierra）
   - `EF070000`（适用于 Mojave/Catalina）
   - `03080000`（适用于 Big Sur 及更新版本）
   - `EF0F0000`（适用于 Big Sur 及更新版本，禁用更多安全功能）

   **注意事项：**
   - 对于 Big Sur 和更新版本，建议使用 `67080000` 而不是 `FF0F0000`，以避免系统更新通知失效。
   - 如果系统分区的安全签名被破坏，`EF0F0000` 会通知系统更新，但会下载完整安装程序（大约 12 GB），而非增量更新。


2. 为避免每次更改 `csr-active-config` 后都需要重置 NVRAM，请在配置中添加以下参数：

   **NVRAM/Delete/7C436110-AB2A-4BBB-A880-FE41995C9F82** &rarr; `csr-active-config`

   这样在重启时，NVRAM 中现有的 `csr-active-config` 值会被删除，并替换为存储在 `NVRAM > Add` 下的新值。这种方法避免了每次都使用“Reset NVRAM”选项。

   要测试设置是否正确应用，请在重启后打开终端并输入 `csrutil status`。对于 `03080000` 值，输出结果应如下所示：

	```
	Configuration:
	Apple Internal: disbaled
	Kext Signing: disabled
	Filesystem Protections: disabled
	Debugging Restrictions: enabled
	DTrace Restrictions: enabled
	NVRAM Protections: enabled
	BaseSystem Verification: enabled
	```
	**[!NOTE]** 请参考 ["`csr-active-config` 标志解释"](/B_OC_Calculators/SIP_Flags_Explained.md) 理解该位掩码的工作方式。

## IV. 调整 Boot Picker 属性并启用鼠标支持

通过 **PickerAttributes**，可以自定义 BootPicker 的外观和操作方式。当前支持的参数如下：

- `1` = 启用启动项的自定义图标
- `2` = 启用启动项的自定义标题
- `4` = 为未配置自定义项的启动项预定义标签图像
- `8` = 更倾向于使用内置图标以匹配主题样式
- `16` = 启用鼠标光标
- `32` = 在内置选择器中启用额外的时间和调试信息（仅适用于 DEBUG 和 NOOPT 构建）
- `64` = 最小化 GUI（无关机和重启按钮）
- `128` = 启用“风味图标”，提供灵活的启动项内容描述（详细信息见 `Documentation.pdf`）

**示例：**

- `17` &rarr; 启用自定义图标和鼠标光标（从 OpenCore 0.6.7 起为默认设置）
- `19` &rarr; 启用自定义图标、自定义标题和鼠标光标
- `147` &rarr; 启用自定义图标、自定义标题、鼠标光标以及风味图标。推荐用于自定义主题。

## V. 自定义启动选项

### 在 BootPicker 中设置默认启动驱动器

要在 BootPicker 中设置默认启动驱动器，请启用以下配置选项：

- **ShowPicker** = `Yes`
- **AllowSetDefault** = `Yes`

在 **BootPicker** 中：

1. 选择驱动器/分区。
2. 按住 <kbd>Ctrl</kbd> 键，然后按 <kbd>Enter</kbd>。

之后，该分区将始终被预选为默认选项（除非重置 NVRAM）。

### 启用 Apple 热键功能

启用 **PollAppleHotKeys** = `Yes` 可以使用 Mac 上的键盘快捷键，在无需额外设置启动参数的情况下进入不同的启动模式（例如：Verbose、Safe 或单用户模式）。

**示例：**

- 开机时按住 <kbd>CMD</kbd><kbd>V</kbd> 将以 Verbose 模式启动 macOS，避免手动添加 `-v`。
- 按住 <kbd>⇧</kbd> 键（Shift）将以安全模式启动 macOS。

有关更多详细信息，请参考 OpenCore 包中的 `Configuration.pdf`。

### 加速启动（结果可能因系统而异）

将 **ConnectDrivers** 设置为 `No`。

如果开机后 BootPicker 出现较慢（例如超过 8 秒），可以尝试此选项缩短等待时间，特别是在笔记本电脑上。但是这样会禁用启动音，因为音频驱动 `AudioDxe.efi` 不会被加载。

> [!CAUTION]
>
> 在从 USB 闪存驱动器安装 macOS 前需要启用 `ConnectDrivers`，否则无法在 BootPicker 中看到驱动器。

### 启动方式选择

修改配置中的以下设置可以影响启动过程。以下是最常用的几种方式：

#### 无 GUI 手动选择操作系统（默认）
- **PickerMode** = `Builtin`
- **ShowPicker** = `Yes`

#### 带 GUI 的手动选择操作系统（需要 OpenCanopy 和资源文件夹）
- **PickerMode** = `External`
- **ShowPicker** = `Yes`

#### 从“默认”卷自动启动操作系统（无 GUI）
- **PickerMode** = `Default`
- **ShowPicker** = `No`

> [!CAUTION]
> 
> 如果系统安装了 Windows，启用此选项可能会有问题。因为如果重置 NVRAM，且未启用 `LauncherPath`，Windows 可能会占据引导菜单的首位，导致无法进入 OpenCore 的 BootPicker。

#### 跳过 BootPicker 自动加载 macOS
以下设置将从第一个 APFS 卷自动启动 macOS。与 `LauncherOption` 的 `Full` 或 `Short` 结合使用可以防止 Windows Boot Manager 占据引导菜单的首位。

**前提条件**：启用 `PollAppleHotkeys`

- **PickerMode** = `Builtin`
- **ShowPicker** = `Yes`

> [!NOTE]
> 
> 开机后按住 <kbd>X</kbd> 可以直接启动 macOS，即使屏幕显示黑屏而不是 BootPicker。

这是一种可靠的解决方法，适用于以下问题：

- BootPicker 不显示，而是一个黑屏（可能与 GOP 渲染有关）。
- 5 秒后（默认延迟时间），系统会启动到 Windows，因为 Windows 占据了 BootPicker 的首位。

💡 如果你希望避免将所有 SSDT 注入到 Windows，可以通过 BIOS 启动菜单引导 Windows，或者使用 [OpenCore_NO_ACPI](/O_OC_NO_ACPI)。与 Clover 不同，OpenCore 会将 ACPI 文件夹中所有启用的内容注入到任何操作系统中。

## VI. 解决 NVRAM 问题

### 重置 NVRAM

某些 BIOS 版本可能会受到 OpenCore 集成的 NVRAM 重置工具的影响。常见症状包括无法进入 BIOS 或某些 NVRAM 参数（如启动参数）未应用或无法删除等。老款 Lenovo 笔记本尤其容易受到影响。

因此，从 OpenCore 0.8.4 开始，OpenCore 包中新增了一个驱动程序 **ResetNvramEntry.efi**（此前为工具 `CleanNvram.efi`），与此类问题的 BIOS 更加兼容。如果你在重置 NVRAM 时遇到问题，可以尝试以下操作：

#### OC ≤ 0.8.3
1. 设置 **Misc/Security/AllowNvramReset** 为 `No`，以禁用 OpenCore 内置的 NVRAM 重置工具，避免在 BootPicker 中出现重复的 "CleanNVRAM" 条目。
2. 将 **CleanNvram.efi** 复制到 `EFI/OC/Tools`。
3. 将其添加到 `config.plist` 的 **Misc/Tools** 部分并启用。
4. 设置 **HideAuxiliary** = `Yes`（位于 **Misc/Boot** 下）。
5. 在 **Misc/Tools** 中找到 `CleanNvram`，并将 `Auxiliary` 设置为 `Yes`。
6. 保存配置并重启。
7. 在 BootPicker 中按下空格键以显示 "CleanNVRAM" 条目。
8. 选中图标并按回车键以重置 NVRAM。

> [!NOTE]
> 
> 自 OC 0.8.4 起，重置 NVRAM 的上述选项已被弃用。请删除：
> 
> - `AllowNvramReset` 键（从 `config.plist` 中删除）。
> - `CleanNvram.efi`（从 `config.plist` 和 `EFI/OC/Tools` 中删除）。

#### OC ≥ 0.8.4
在 OC 0.8.4 及更高版本中启用 NVRAM 重置的方法如下：

1. 将 **ResetNvramEntry.efi** 添加到 `EFI/OC/Drivers`。
2. 在 `config.plist` 中的 **UEFI/Drivers** 下添加 **ResetNvramEntry.efi** 并启用。
3. 保存配置并重启。
4. 在 BootPicker 中按下空格键以显示 "ResetNVRAM" 条目。
5. 选中图标并按回车键以重置 NVRAM。

#### 重置 NVRAM 后保留启动条目
通常情况下，重置 NVRAM 会重置 BIOS 引导菜单的顺序，导致 Windows Boot Manager 占据首位或引导分区丢失。为避免这种情况，请执行以下操作：

- 转到 **UEFI/Drivers/ResetNvramEntry.efi**。
- 在 **Arguments** 字段中添加 `--preserve-boot` 参数。

### 修复错误报告的 OpenCore 版本

有时候，NVRAM 中存储的 OpenCore 版本信息不会自动更新，因此在 Kext Updater 或 Hackintool 中显示的版本可能会不正确。从 OC 0.6.7 起，这一问题已经通过不再将版本信息写入 NVRAM 的方式得以修复，但错误的版本信息仍会驻留在 NVRAM 中，直到被删除。要修复此问题，可以按照以下步骤操作：

1. 在 **NVRAM/Delete/4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102** 下创建一个新子元素。
2. 将其命名为 `opencore-version`。
3. 保存配置并重启。
4. 重启后，正确的 OpenCore 版本信息应该会显示，此时你可以删除该条目。

或者，你也可以使用终端命令删除该键值，无需重启：


- 在 **NVRAM/Delete/4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102** 下创建一个子元素。
- 将其命名为 `opencore-version`。
- 保存配置并重启。
  重启后，正确的 OpenCore 版本应会显示，此时可以删除该条目。

或者，也可以使用终端命令删除键值（无需重启）：

```terminal
sudo nvram -d 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102:opencore-version
```

## VII. 禁止将 SMBIOS 注入到其他操作系统

为了避免 OpenCore 将 SMBIOS 信息注入到 Windows 或其他操作系统中，可能会引发注册问题，请更改以下设置：

- **Kernel/Quirks/CustomSMBIOSGuid**: `True`（默认值：`False`）
- **Platforminfo/UpdateSMBIOSMode**: `Custom`（默认值：`Create`）

**来源**：[避免将 SMBIOS 注入到其他操作系统](https://github.com/dortania/OpenCore-Install-Guide/tree/master/clover-conversion#optional-avoiding-smbios-injection-into-other-oses)

## VIII. 在 OpenCore 和 Clover 之间交换 SMBIOS 数据

### 手动方法
在 OpenCore 和 Clover 配置之间来回交换 SMBIOS 数据可能会有些混乱，因为两者对数据字段的命名和位置不同。

正确传输数据非常重要，否则每次切换 OpenCore 和 Clover 时，你都需要重新输入 Apple ID 和密码，这会导致设备在 Apple 账户中被识别为新的设备。为了避免这种情况，请按照以下步骤操作：

1. 将以下字段的数据从 OpenCore 的 `PlatformInfo/Generic` 复制到 Clover Configurator 的“SMBIOS”和“RtVariables”部分：

| OpenCore PlatformInfo/Generic | Clover SMBIOS/RtVariables    |
|------------------------------|------------------------------|
| SystemProductName            | ProductName                 |
| SystemUUID                   | SmUUID                      |
| ROM                          | ROM（在 `RtVariables` 下选择“from SMBIOS”，并粘贴 ROM 地址）|
| N/A in "Generic"             | Board-ID                    |
| SystemSerialNumber           | Serial Number               |
| MLB                          | 1. Board Serial Number（在 `SMBIOS` 下）</br>2. MLB（在 `RtVariables` 下）|

2. 勾选“Update Firmware Only”框。
3. 在下拉菜单中选择与你的 `ProductName` 一致的 Mac 型号，这会自动更新 BIOS 和 Firmware 等字段。
4. 保存配置并用 Clover 重启。

如果 SMBIOS 数据传输正确，你将不需要再次输入 Apple ID 和密码。

#### 排错
如果从 OpenCore 切换到 Clover（或相反）后需要重新输入 Apple ID 密码，说明使用的 SMBIOS 数据不一致，你需要找出数据不匹配的地方。可以通过 Hackintool 工具执行以下步骤来检查：

1. 挂载 EFI 分区。
2. 打开当前使用的引导管理器的配置文件。
3. 运行 Hackintool。在“系统”部分可以查看当前使用的 SMBIOS 数据：
   </br> ![SYSINFO](https://user-images.githubusercontent.com/76865553/166119425-8970d155-b546-4c91-8daf-ec308d16916f.png)
4. 检查配置中的参数是否与 Hackintool 显示的框选参数一致。
5. 如果参数不一致，修正并保存配置，确保与 Hackintool 的数据匹配。
6. 如果参数一致，打开另一个引导管理器的配置文件，再次与 Hackintool 显示的参数对比并调整。
7. 保存配置后重启。
8. 切换到另一个引导管理器并启动 macOS。
9. 如果数据正确，则不需要再次输入 Apple ID 密码（可使用 Hackintool 再次验证）。

### 使用 OCAT 导入/导出 SMBIOS 数据
除了手动复制 OpenCore 与 Clover 的 SMBIOS 数据，还可以使用 [**OpenCore Auxiliary Tools (OCAT)**](https://github.com/ic005k/OCAuxiliaryTools/releases)。OCAT 提供内置的导入/导出功能，可直接从 Clover 导入 SMBIOS 数据或将 SMBIOS 数据导出到 Clover 配置中：

![OCAT 导入导出功能](https://user-images.githubusercontent.com/76865553/162971063-cbab15fa-4c83-4013-a732-5486d4f00e31.png)

> [!IMPORTANT]
> 
> - 如果操作正确，切换引导管理器后无需再次输入 Apple ID 密码，macOS 会显示“此 Apple ID 现在用于此设备”。
> - 如果 macOS 提示输入 Apple ID 和邮件密码，说明配置有误。此时建议切换回 OpenCore 检查配置，否则设备可能被注册为新的 Mac。

### Clover 用户的一键解决方案
如果在生成 OpenCore 配置的 SMBIOS 数据时使用了以太网控制器的真实 MAC 地址（ROM），可以避免 SMBIOS 冲突。在 Clover 的“Rt Variables”部分，选择“from System”，即可解决问题。

## 更多资源
- 查看 Dortania 的优秀 OpenCore [安装后指南](https://github.com/dortania/OpenCore-Post-Install)，了解如何解决各种问题。
- OpenCore 包中包含的 **Documentation.pdf** 也提供了许多实用技巧。
