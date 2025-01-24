# 如何在macOS Sonoma+中通过OpenCore Legacy Patcher重新启用以前支持的Wi-Fi/BT卡

> **适用版本**: OCLP 1.0.0+

**目录**

- [如何在macOS Sonoma+中通过OpenCore Legacy Patcher重新启用以前支持的Wi-Fi/BT卡](#如何在macos-sonoma中通过opencore-legacy-patcher重新启用以前支持的wi-fibt卡)
	- [技术背景](#技术背景)
		- [蓝牙如何处理？](#蓝牙如何处理)
		- [Intel如何处理？](#intel如何处理)
	- [操作步骤（Broadcom和Atheros）](#操作步骤broadcom和atheros)
		- [配置和EFI调整](#配置和efi调整)
		- [验证已添加的Kext](#验证已添加的kext)
		- [应用根补丁以重新启用Wi-Fi卡](#应用根补丁以重新启用wi-fi卡)
	- [操作步骤（Intel）](#操作步骤intel)
		- [Wi-Fi](#wi-fi)
		- [蓝牙](#蓝牙)
		- [Intel蓝牙故障排除](#intel蓝牙故障排除)
	- [致谢](#致谢)

---

## 技术背景
在macOS Sonoma开发初期，驱动旧版Wi-Fi/BT卡所需的kext和框架从系统中移除，导致常用的BT/Wi-Fi卡的Wi-Fi部分无法正常工作。

**这影响了以下Wi-Fi卡芯片组**：

- **"Modern"**卡：
    - **Broadcom**: `BCM94350`（包括`BCM94352`）、`BCM94360`、`BCM43602`、`BCM94331`、`BCM943224`
    - **所需Kexts**: IOSkywalkFamily, IO80211FamilyLegacy, AirPortBrcmNIC, AirportBrcmFixup, AirPortBrcmNIC_Injector.
- **"Legacy"**卡：
    - **Atheros**: `AR928X`、`AR93xx`、`AR242x`/`AR542x`、`AR5418`、`AR5416`（从未被Apple使用）
    - **Broadcom**: `BCM94322`、`BCM94328`
    - **所需Kexts**: corecaptureElCap, IO80211ElCap, AirPortAtheros40（仅针对Atheros）

借助Dortania的OpenCore Legacy Patcher（OCLP），通过修补一些系统文件（以及通过OpenCore注入额外的kexts），可以重新启用这些Wi-Fi卡。如果你想了解Wi-Fi修补如何工作，[请查看此帖](https://www.insanelymac.com/forum/topic/357087-macos-sonoma-wireless-issues-discussion/?do=findComment&comment=2809940)。

除了修补Apple产品中正式使用的Wi-Fi/BT卡外，OCLP还支持对Hackintosh中常见的第三方Broadcom芯片组进行Wi-Fi修补（通过`AirportBrcmFixup.kext`启用）：

- 设备ID `pci12e4,4357` → BCM43225
- 设备ID `pci12e4,43B1` → BCM4352
- 设备ID `pci12e4,43B2` → BCM4352（2.4 GHz）

### 蓝牙如何处理？

一般来说，如果a) 你的系统USB端口已正确映射，并且b) 所需的最新蓝牙kext已存在于EFI文件夹中，那么安装macOS 14后，蓝牙大概率能够正常工作。

macOS Sequoia需要在`config.plist`中添加额外的NVRAM条目，才能使蓝牙工作。

### Intel如何处理？

由于Apple从未在其Mac中使用Intel的Wi-Fi/BT卡，因此没有根补丁可用或需要。Wi-Fi和蓝牙由[**OpenIntelWireless**](https://openintelwireless.github.io/General/Installation.html)项目提供的kexts处理（除了`BluetoolFixup.kext`，该kext是[**BrcmPatchRAM**](https://github.com/acidanthera/BrcmPatchRAM)的一部分）。所需的kexts和配置设置也列在[**Kext加载顺序**](/10_Kexts_Loading_Sequence_Examples#example-8a-intel-wifi-airportitlwm-and-bluetooth-intelbluetoothfirmware)示例中。macOS Sequoia需要在`config.plist`中添加额外的NVRAM条目，才能使蓝牙工作。

## 操作步骤（Broadcom和Atheros）

### 配置和EFI调整
请将以下更改应用于你的config（或从plist示例中复制它们（→ [Legacy WiFi](/14_OCLP_Wintel/plist/Sonoma_WIFI_Legacy.plist) → [Modern WiFi](/14_OCLP_Wintel/plist/Sonoma_WIFI_Modern.plist)）并将列出的kexts添加到`EFI/OC/Kexts`文件夹：

配置部分 | 操作
:-------------:|-------
**Kernel/Add** | <ul><li> 对于**"Modern"**Wi-Fi卡，添加以下来自[OCLP Repo](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi)的**Kexts**： <ul><li> `IOSkywalkFamily.kext` <li> `IO80211FamilyLegacy.kext`（及其插件`AirPortBrcmNIC.kext`） <li> `AirportBrcmFixup`（及其插件`AirPortBrcmNIC_Injector.kext`），可从[这里](https://dortania.github.io/builds/?product=AirportBrcmFixup&viewall=true)获取 <li> [`AMFIPass.kext`](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera)（如果还没有） <li> 按照如下方式整理kexts，并添加`MinKernel`设置： <br> ![wifi_modern](https://github.com/laobamac/OC-little-zh/assets/76865553/ca04bf6f-71ee-47ed-b7b4-15359b88c17e)</ul> <li> 对于**"Legacy"**卡，添加以下kexts（可从[这里](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi)获取）：<ul> <li> `corecaptureElCap.kext` <li> `IO80211ElCap.kext`及其插件`AirPortAtheros40.kext`（仅对Atheros卡启用） <li> [`AMFIPass.kext`](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera)（如果还没有） <li> 按如下方式调整`MinKernel`设置： <br> ![wifi_legacy](https://github.com/laobamac/OC-little-zh/assets/76865553/ab112e90-a528-4354-a5a7-729900ebacf6)</ul><li> 对于**蓝牙**（macOS 12+）：<ul><li> 添加[`BlueToolFixup.kext`](https://github.com/acidanthera/BrcmPatchRAM) <li> 设置`MinKernel`为：`21.0.0`<li> 根据需要调整`MinKernel`和`MaxKernel`设置：<br>![brcmbeetee](https://github.com/user-attachments/assets/6f3a6349-ade7-41a3-b7b1-0370401b5279)
**Kernel/Block**| <ul> <li> 阻止 **IOSkywalkFamily**（仅适用于**Modern**Wi-Fi）： <br> ![Brcm_Sonoma1](https://github.com/laobamac/OC-little-zh/assets/76865553/54079541-ee2e-4848-bb80-9ba062363210)<li> **备注**：如果你使用的是USB以太网适配器并且使用ECM协议，还需添加[`ECM-Override.kext`](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Kexts/Misc/ECM-Override-v1.0.0.zip)以防止内核崩溃，因为Apple的DriverKit堆栈使用IOSkywalk处理ECM适配器。
**Misc/Security** | 将`SecureBootModel`更改为`Disabled`，以排除由安全冲突引起的早期内核崩溃。设置一切正常后，可以重新启用。
**NVRAM/Add/...-FE41995C9F82** |<ul><li> 更改`csr-active-config`为`03080000` <li> 可选：如果根补丁无法应用，则将`amfi=0x80`添加到`boot-args`（之后禁用） <li> 可选：如果应用根补丁后无法连接Wi-Fi，则将`-brcmfxbeta`添加到`boot-args`。<li> 可选：如果最新的Sonoma测试版未加载`AMFIPass.kext`，则将`-amfipassbeta`添加到`boot-args`。 <li> 在macOS Sequoia中，还需添加以下条目： <ul><li> `bluetoothInternalControllerInfo` <br>**类型**: `数据` <br>**值**: `00000000 00000000 00000000 0000`<li>`bluetoothExternalDongleFailed` <br> **类型**: `数据` <br>**值**: `00`<li>**截图**: ![](https://www.insanelymac.com/uploads/monthly_2024_12/Bildschirmfoto2024-12-15um14_18_08.png.d2057e458a2be3cefff9afe2ae40688f.png)</ul>
**NVRAM/Delete...-FE41995C9F82** | 添加以下参数： <ul> <li> `csr-active-config` <li> `boot-args` <li> `bluetoothInternalControllerInfo` <li> `bluetoothExternalDongleFailed`

> [!注意]
> 
> - 在应用这些设置后以及切换macOS版本时（如果安装了不同版本的macOS），请确保执行NVRAM重置
> - 我注意到，在应用这些更改后，蓝牙可能需要5到10秒才会可用。

### 验证已添加的Kext

在应用根补丁之前，最好先验证我们添加的kext是否已加载。

- 保存你的配置并重启
- 重置NVRAM
- 启动进入macOS Sonoma+
- 接下来，打开终端并输入`kextstat`
- 这将列出所有加载的kext。检查以下kext是否存在（使用<kbd>CMD</kbd>+<kbd>F</kbd>查找）：
	- **对于"Modern"Wi-Fi**： 
		- `IOSkywalkFamily`
		- `IO80211FamilyLegacy`
		- `AirportBrcmFixup`
		- `AirPort.BrcmNIC`
		- `Amfipass`
	- **对于"Legacy"Wi-Fi**：
		- `corecaptureElCap`
		- `IO80211ElCap`
		- `AirPortAtheros40`
		- `Amfipass`
- 如果这些kext没有加载，你需要检查你的配置和EFI/OC文件夹。
- 如果`Amfipass`没有加载，添加`-amfipassbeta`启动参数并重启
- 如果Wi-Fi可用，但无法连接任何接入点，添加`-brcmfxbeta`启动参数并重启。

一旦验证了所需的kext已加载，你可以继续使用OCLP应用根补丁。

### 应用根补丁以重新启用Wi-Fi卡

- 下载[**最新版本**](https://github.com/dortania/OpenCore-Legacy-Patcher/releases)的**OpenCore Legacy Patcher**并安装。
- 运行OCLP，点击“**Post Install Root Patch**”。
- 如果出现“Networking: Modern Wireless”或“Networking: Legacy Wireless”的补丁选项，点击“**开始根补丁**”（否则请查看→ 故障排除）
- 补丁完成后，重启
- 享受正常工作的Wi-Fi吧！


<details><summary><strong>Example: Modern Wi-Fi patching </strong> (Click to reveal!)</summary>

- Option "Networking Modern Wireless" is available: <br>![OCLP_Wifi](https://github.com/laobamac/OC-little-zh/assets/76865553/064d1ddb-fd91-4ddf-abb5-a8b00e37f3e2)
- Patching in Progress. In my case, it's for a BCM94352HMB (Modern): <br>![Progress](https://github.com/laobamac/OC-little-zh/assets/76865553/7f0ed6dc-67bf-4f89-99a1-ea72d518fbee)
- Wi-Fi is working again: <br>![access](https://github.com/laobamac/OC-little-zh/assets/76865553/11e57c0e-fd81-4aea-ae5b-475b7cf013b2)
</details>

---

## 操作步骤（Intel）

### Wi-Fi

对于启用Intel的Wi-Fi卡，实际上有两个可用的kext可以使用：`Itlwm.kext`和`AirportItlwm.kext`。虽然`AirportItlwm.kext`作为Wi-Fi驱动程序使用，但在使用`Itlwm.kext`时，卡将被识别为以太网控制器。这两个kext各有优缺点，使用哪个取决于个人偏好和使用的操作系统（[**了解更多**](https://openintelwireless.github.io/itlwm/FAQ.html)）。

与`Itlwm.kext`不同，`AirportItlwm.kext`需要根据每个macOS版本使用不同的kext变体（目前支持从macOS High Sierra到Sonoma）。在macOS Sequoia中，使用`itlwm.kext`是强制性的，才能使Wi-Fi工作。使用`Itlwm.kext`时，你还需要一个名为[**HeliPort**](https://github.com/OpenIntelWireless/HeliPort/releases)的附加应用程序来连接Wi-Fi热点。

如果你安装了多个版本的macOS，并希望在所有版本中使用`AirportItlwm.kext`，你需要：a) 重命名kext以便将它们都放入" kexts"文件夹中，b) 还需要调整它们的`MinKernel`和`MaxKernel`设置，这样只有正确的kext会被加载。

以下是如何组合`itlwm`和`AirportItlwm` kext的说明。在这种情况下，`itlwm`用于macOS Sequoia，而`AirportItlwm`用于macOS Big Sur到Sonoma。


1. 访问[**https://github.com/OpenIntelWireless/itlwm/releases**](https://github.com/OpenIntelWireless/itlwm/releases)
2. 点击“Assets”按钮
3. 下载你选择的`AirportItlwm`版本
4. 解压并重命名它们：我通常会在文件名后加上下划线和操作系统名称，例如`AirportItlwm_Sonoma.kext`（不要添加空格！）
5. 将它们添加到`EFI/OC/Kexts`文件夹和`config.plist`中
6. 禁用`itlwm.kext`（如果存在）
7. 接下来，添加`MinKernel`和`MaxKernel`设置，以限制kext仅加载为其设计的macOS版本：<br> ![intelwlan](https://github.com/user-attachments/assets/bf6ebd02-922b-42cf-9095-b103028ffa0d)
8. 保存配置并重启

> [!重要]
> 
> - 使用`AirportItlwm`或`Itlwm`之一——绝不要同时使用！
> - 使用`AirportItlwm`时，添加正确的`MinKernel`和`MaxKernel`设置是**强制的**。否则Wi-Fi将无法正常工作，并且在多次注入kext时系统可能崩溃！
> - 当重命名kext时，你将无法再通过像OCAT这样的工具自动获取kext更新。
> - 更新macOS Sonoma（14.3及更高版本）时，你**必须**禁用`AirportItlwm.kext`，改用`itlwm.kext`并将`SecureBootModel`设置为`Disabled`，然后再进行更新。否则，安装程序会崩溃（[更多信息](/W_Workarounds/macOS14.4.md)）。更新后，你可以重新启用`SecureBootModel`。


### 蓝牙

如果你的Intel Wi-Fi/BT卡已被[OpenIntelWireless项目支持](https://openintelwireless.github.io/itlwm/Compat.html)，则需要以下kext来启用macOS 14+中的蓝牙（→ [plist](/14_OCLP_Wintel/plist/Sonoma_WIFI_Intel.plist)）：

Kext | 备注 | `MinKernel` | `MaxKernel`
-----|---------|:-----------:|:-----------:
[**`BlueToolFixup.kext`**](https://github.com/acidanthera/BrcmPatchRAM) | macOS 12及更高版本的蓝牙启用器 | 21.0.0 | 
[**`IntelBluetoothFirmware.kext`**](https://github.com/OpenIntelWireless/IntelBluetoothFirmware) | Intel蓝牙固件 |  | 

**截图**：

![intelbt](https://github.com/user-attachments/assets/96062780-6a52-430c-a1ac-4b45515c81a6)

> [!注意]
> 
> - 在应用这些设置并切换macOS版本后（如果安装了多个版本的macOS），务必执行NVRAM重置。
> - 我注意到，在应用这些更改后，蓝牙可用可能需要10秒左右的时间。
> - 要在macOS Monterey及更高版本中使用Intel蓝牙，[**请阅读此文**](https://openintelwireless.github.io/IntelBluetoothFirmware/FAQ.html#what-additional-steps-should-i-do-to-make-bluetooth-work-on-macos-monterey-and-newer)。

### Intel蓝牙故障排除

如果在添加了这两个kext后蓝牙无法正常工作，请按以下步骤操作：

- 打开终端并输入 `NVRAM -p`
- 如果输出中包含 `bluetoothExternalDongleFailed=%01`，则需要将其更改为`%00`。
- 输入：`sudo NVRAM bluetoothExternalDongleFailed=%00`
- 然后输入 `sudo pkill bluetoothd` 来终止并重启蓝牙堆栈
 
## 致谢
- Acidanthera：为OpenCore和kexts提供支持
- Dortania：提供OpenCore Legacy Patcher
- Acquarius13：为OCLP源代码中需要编辑的内容提供了线索
- deeveedee：提醒我使用`brcmfxbeta`启动参数
- jrycm：提供关于如何让Intel卡在Sequoia 15.2上正常工作的Tips
