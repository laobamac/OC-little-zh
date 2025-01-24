# Kext加载顺序示例

**目录**

- [Kext加载顺序示例](#kext加载顺序示例)
	- [关于](#关于)
	- [Kext和内核补丁的处理顺序](#kext和内核补丁的处理顺序)
	- [Lilu和VirtualSMC优先](#lilu和virtualsmc优先)
	- [使用`MinKernel`和`MaxKernel`设置](#使用minkernel和maxkernel设置)
		- [内核支持表](#内核支持表)
	- [示例](#示例)
		- [示例1：强制性kext(最小要求)](#示例1强制性kext最小要求)
		- [示例2：ApplePS2SmartTouchPad + 插件(笔记本)](#示例2appleps2smarttouchpad--插件笔记本)
		- [示例3：VoodooPS2 + 触控板(笔记本)](#示例3voodoops2--触控板笔记本)
		- [示例4：VoodooPS2 + I2C(笔记本)](#示例4voodoops2--i2c笔记本)
		- [示例5：VoodooPS2 + VoodooRMI + VoodooSMBus(笔记本)](#示例5voodoops2--voodoormi--voodoosmbus笔记本)
		- [示例6：VoodooPS2 + VoodooRMI + VoodooI2C(笔记本)](#示例6voodoops2--voodoormi--voodooi2c笔记本)
		- [示例7：Broadcom WiFi和蓝牙](#示例7broadcom-wifi和蓝牙)
			- [修复AirportBrcmFixup生成大量崩溃报告的问题](#修复airportbrcmfixup生成大量崩溃报告的问题)
		- [示例8a：Intel WiFi(AirportItlwm)和蓝牙(IntelBluetoothFIrmware)](#示例8aintel-wifiairportitlwm和蓝牙intelbluetoothfirmware)
		- [示例8b：在多个版本的macOS中使用`AirportItlwm.kext`](#示例8b在多个版本的macos中使用airportitlwmkext)
			- [说明](#说明)
		- [示例9a：可能的桌面Kext顺序](#示例9a可能的桌面kext顺序)
		- [示例9b：可能的笔记本Kext顺序](#示例9b可能的笔记本kext顺序)
		- [示例10：在macOS 14+中启用Broadcom WiFi卡](#示例10在macos-14中启用broadcom-wifi卡)
			- [此部分暂时搁置，可以直接看我的视频进行学习！哔哩哔哩-在macOS Sequoia驱动博通网卡教程](#此部分暂时搁置可以直接看我的视频进行学习哔哩哔哩-在macos-sequoia驱动博通网卡教程)
		- [示例11：CPUFriend](#示例11cpufriend)
		- [示例12：在macOS Sequoia驱动Intel无线网卡](#示例12在macos-sequoia驱动intel无线网卡)
	- [注意事项和致谢](#注意事项和致谢)

---

## 关于
本章包含了一些配置示例，用于演示在OpenCore的`config.plist`中某些kext的加载顺序。与Clover不同，OpenCore按照`config.plist`中的`Kernel/Add`部分中列出的顺序加载kext。如果这个顺序不正确，系统可能无法启动、启动时崩溃，或者你添加的kext设备可能无法正常工作！因此，确保顺序正确至关重要。

一般来说，为其他kext提供附加功能的kext必须先加载。配置1包含了任何Hackintosh启动所需的最基本kext的加载顺序：

1. **Lilu.kext**
2. **VirtualSMC.kext** (+传感器插件) 或 **FakeSMC.kext** (+可选传感器插件)
3. **Whatevergreen**
4. **AppleALC**

以下配置示例显示了**WiFi**、**蓝牙**、**键盘**、**触控板**和其他必须按正确顺序加载的kext。没有正确顺序可能导致内核恐慌。`config.plist`中列出的kext如果没有实际存在于`EFI/OC/Kexts`文件夹中，也会导致问题。因此，确保kext按正确顺序加载并且`config.plist`中的内容与OC文件夹中的kext一一对应至关重要。

> [!WARNING]
>
> 要了解更多有关可用和受支持kext的信息，请阅读OpenCore GitHub上的[**Kext文档**](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Kexts.md)。

## Kext和内核补丁的处理顺序
OpenCore按以下顺序处理`config.plist`中的`Kernel`部分(从0.9.2版本开始，Commit 6a65dd1)：

- 处理`Block`
- 处理`Add`和`Force`
- 处理`Emulate`和`Quirks`
- 处理`Patch`

> [!WARNING]
> 
> 在OpenCore Commit 6a65dd1之前，`Add`和`Force`是最后处理的，这使得无法强制修补注入的kext。

## Lilu和VirtualSMC优先
虽然建议将**Lilu**和**VirtualSMC**首先加载以简化kext相关的故障排除，但***这并不是硬性要求***！**Lilu**和**VirtualSMC**只需要在依赖它们的kext之前加载即可。`ProperTree`通过交叉引用`CFBundleIdentifiers`与`OSBundleLibraries`来确保kext的正确加载顺序，这在创建配置快照时非常重要。

> [!提示]
> 
> 如果不确定，可以在ProperTree中创建一个新的快照(文件 &rarr; `OC Snapshot`)或者将**Lilu**和**VirtualSMC**放在配置文件顶部，以消除kext依赖问题！根据我的经验，首先加载Lilu和VirtualSMC也能减少启动时间。

## 使用`MinKernel`和`MaxKernel`设置

为kext应用`MinKernel`和`MaxKernel`设置非常有用，这样可以最大化你的`config.plist`与*不同*版本macOS的兼容性，而不必为不同的kext创建多个配置文件。

通过指定内核范围，你可以控制哪些kext在不同版本的macOS上加载，而不必手动启用/禁用kext。这样，你可以保留所有kext启用状态，只需在`MinKernel`和`MaxKernel`字段中输入相应的值即可控制加载哪些kext。

**`MinKernel`和`MaxKernel`设置尤其适用于**：

- **WiFi**和**蓝牙**kext，其中某些macOS版本需要不同的kext集合
- 仅在特定版本的macOS上需要的kext，如`CryptexFixup.kext`(仅限Ventura及更新版本)或`NoTouchID.kext`(仅限High Sierra到Mojave)

### 内核支持表

以下是macOS 10.4到macOS 15的内核版本范围表：

| OS X/macOS版本(名称) | MinKernel | MaxKernel | 架构 |
|-----------------------|:---------:|:---------:|:----:|
| macOS 15 (Sequoia)     | 24.0.0    | 24.99.99  | Intel (64位) / Apple Silicon (ARM) |
| macOS 14 (Sonoma)      | 23.0.0    | 23.99.99  | 同上 |
| macOS 13 (Ventura)     | 22.0.0    | 22.99.99  | 同上 |
| macOS 12 (Monterey)    | 21.0.0    | 21.99.99  | 同上 |
| macOS 11 (Big Sur)     | 20.0.0    | 20.99.99  | 同上 |

> [!提示]
>
> - 要检查macOS版本使用的内核，可以在终端中输入`uname -r`，或者在“系统信息”中查找“软件”部分。
> - 虽然`MaxKernel`可以达到`X.99.99`，但大多数情况下使用`X.9.9`就足够了。至今为止，没有任何版本的macOS使用超过`X.9.9`的内核。

## 示例
### 示例1：强制性kext(最小要求)
![config01](https://user-images.githubusercontent.com/76865553/140840864-e7596685-d2bf-426d-af92-4f23fa01f18a.png)</br>
任何额外的kext必须放在强制性kext之后。

### 示例2：ApplePS2SmartTouchPad + 插件(笔记本)
![Config2](https://user-images.githubusercontent.com/76865553/140813746-3d3ab6aa-949a-4b91-8c9b-c3dcd0fef77d.png)

### 示例3：VoodooPS2 + 触控板(笔记本)
![config3](https://user-images.githubusercontent.com/76865553/140813775-eb6ff60f-9ec3-4c9b-a768-f5e5a9e6868e.png)

### 示例4：VoodooPS2 + I2C(笔记本)
![config4](https://user-images.githubusercontent.com/76865553/140813798-a403f299-e85d-4fed-90f7-bea045384db5.png)

### 示例5：VoodooPS2 + VoodooRMI + VoodooSMBus(笔记本)
对于通过SMBus控制的Synaptics触控板，kext顺序如下：

![Voodoo_RMI_SMBUS](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/5bcb3370-2094-4c91-b813-c9eab5cd2901)

**来源**：[**VoodooSMBus**](https://github.com/VoodooSMBus/VoodooRMI#installation)

### 示例6：VoodooPS2 + VoodooRMI + VoodooI2C(笔记本)
对于通过I2C控制的Synaptics触控板，kext顺序如下：

![Voodoo_RMI_I2C](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/d7010ea1-89f3-4a10-b9ab-acb920bd180b)

**来源**：[**VoodooSMBus**](https://github.com/VoodooSMBus/VoodooRMI#installation)

### 示例7：Broadcom WiFi和蓝牙

![broadcomwifibt](https://github.com/user-attachments/assets/a5241530-af60-4a12-ac39-c430cdb129f7)

使用不被macOS原生支持的Broadcom WiFi/Bluetooth卡时，请注意以下几点：

1. kext必须按正确的顺序加载(否则启动崩溃)。如果不确定，可以在ProperTree中创建OC快照，它可以修正顺序。
2. 必须使用`MinKernel`和`MaxKernel`设置来控制不同版本的macOS加载哪些kext。
3. `AirportBrcomFixup`用于启用WiFi，包含两个附加kext作为插件(只应启用其中一个)：
	- `AirPortBrcmNIC_Injector.kext`(兼容macOS 10.13及更新版本)
	- `AirPortBrcm4360_Injector.kext`(兼容macOS 10.8到10.15)

4. 对于蓝牙，需要多个kext和组合：
	- `BlueToolFixup.kext`：适用于macOS 12及更新版本，包含固件数据(MinKernel 21.0)。
	- `BrcmFirmwareData.kext`：包含必要的固件，适用于macOS 10.8到11.x(MaxKernel 20.9.9)。
	- `BrcmPatchRAM.kext`：适用于10.10或更早版本。
	- `BrcmPatchRAM2.kext`：适用于macOS 10.11到10.14。
	- `BrcmPatchRAM3.kext`：适用于macOS 10.15到11.x。需要与`BrcmBluetoothInjector.kext`一起使用。
5. 随着macOS Sonoma(Darwin Kernel 23.0)的发布，Apple完全放弃了对Broadcom卡的支持！要重新启用Broadcom WiFi，必须：
	- [使用OpenCore Legacy Patcher应用根补丁](/14_OCLP_Wintel/Enable_Features/WiFi_Sonoma.md)
	- 添加额外的kext
	- 调整一些配置设置(见→[示例10](#example-10-enabling-legacy-broadcom-wifi-cards-in-macos-14))

> [!警告]
> 
> 不要将`BrcmFirmwareRepo.kext`添加到`EFI/OC/Kexts`！它不能通过启动管理器注入，必须安装到`/System/Library/Extensions`(macOS 10.11及更高版本为`/Library/Extensions`)。此时`BrcmFirmwareData.kext`不再需要。可以使用[**Kext-Droplet**](https://github.com/chris1111/Kext-Droplet-macOS)将kext直接安装到系统库中。

#### 修复AirportBrcmFixup生成大量崩溃报告的问题
我最近注意到，尽管我的WiFi卡在连接性和速度上表现良好，`com.apple.drive.Airport.Brcm4360.0`和`com.apple.iokit.IO80211Family`的崩溃报告仍然不断生成(位于/Library/Logs/CrashReporter/CoreCapture下)。

这个问题与智能连接(Smart Connect)有关，它是支持2.4 GHz和5 GHz基带的WiFi路由器的功能，可以使WiFi卡根据信号质量自动切换。关闭路由器中的智能连接功能即可解决此问题。

### 示例8a：Intel WiFi(AirportItlwm)和蓝牙(IntelBluetoothFIrmware)
![IntelBT](https://user-images.githubusercontent.com/76865553/196041542-9f6943dc-b500-408e-8d61-f15a6082d5f7.png)

> [!WARNING]
> 
> - 对于Intel WiFi，实际上有两个可以使用的kext：`Itlwm.kext`和`AirportItlwm.kext`。它们各有优缺点，因此使用哪一个取决于个人偏好([**了解更多**](https://openintelwireless.github.io/itlwm/FAQ.html))。
> - 对于在macOS Monterey及更新版本中使用Intel蓝牙，[**请阅读此内容**](https://openintelwireless.github.io/IntelBluetoothFirmware/FAQ.html#what-additional-steps-should-i-do-to-make-bluetooth-work-on-macos-monterey-and-newer)。

### 示例8b：在多个版本的macOS中使用`AirportItlwm.kext`

如你所知，有两个kext用于启用Intel卡的Wi-Fi支持：itlwm和AirportItlwm。与Itlwm不同，AirportItlwm要求为每个macOS版本提供不同的kext版本(目前支持macOS High Sierra到Sonoma)。

如果你安装了多个版本的macOS，并希望在所有版本中使用AirportItlwm，那么你需要在EFI文件夹中有不同版本的kext，以便Wi-Fi在所有macOS版本中正常工作。

#### 说明

1. 访问[https://github.com/OpenIntelWireless/itlwm/releases](https://github.com/OpenIntelWireless/itlwm/releases)
2. 点击“资产”下载你选择的AirportItlwm版本
3. 解压并重命名：通常我会加上下划线和操作系统名称，例如`AirportItlwm_Sonoma.kext`(不要添加空格！)
4. 将它们添加到`EFI/OC/Kexts`和`config.plist`
5. 禁用`itlwm.kext`(如果存在)
6. 然后，添加`MinKernel`和`MaxKernel`设置，以限制仅加载为该macOS版本设计的kext：<br>![Airportitlwm](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/f47386b8-34f9-42fc-bdb3-8f29b7b95777)
7. 保存配置

> [!重要]
> 
> - 添加正确的`MinKernel`和`MaxKernel`设置非常重要，否则Wi-Fi无法正常工作，且系统可能在多次注入kext时崩溃！
- 重命名kext时，你将无法通过像OCAT这样的工具自动获取更新。
- 在更新macOS Sonoma(14.3及更高版本)时，必须禁用`AirportItlwm.kext`，转而使用`itlwm.kext`并设置`SecureBootModel`为`Disabled`。更新前后，你可以恢复这些设置。

### 示例9a：可能的桌面Kext顺序
![config9](https://user-images.githubusercontent.com/76865553/140826181-073a2204-aacb-435e-970c-1823cd2786d1.png)

大多数Intel桌面配置至少会包含`Lilu`、`VirtualSMC`(插件可选)、`WhateverGreen`和`AppleALC`。这个示例不包括USB端口、以太网和Wi-Fi/BT的kext！

### 示例9b：可能的笔记本Kext顺序
![config9b](https://user-images.githubusercontent.com/76865553/140829571-525840b9-f7e5-4abb-8cd9-3aa0e31867a9.png)

这是一个可能的笔记本kext顺序示例。在这个示例中，触控板需要`VoodooPS2Controller`，Wi-Fi和蓝牙由Intel提供，以太网卡是Realtek的。根据笔记本的硬件，第10到第17个kext可能完全不同。

> [!WARNING]
> 
> - 戴尔用户可以添加`SMCDellSensors`用于温度监控和风扇控制。
- 如果笔记本配备了兼容的亮度传感器，可以添加`SMCLightSensor`。

### 示例10：在macOS 14+中启用Broadcom WiFi卡
#### 此部分暂时搁置，可以直接看我的视频进行学习！[哔哩哔哩-在macOS Sequoia驱动博通网卡教程](https://www.bilibili.com/video/BV1xe4oecEUk/?spm_id_from=333.1387.homepage.video_card.click)
1. 阻止**IOSkywalkFamily**： <br> ![Brcm_Sonoma1](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/54079541-ee2e-4848-bb80-9ba062363210)
2. 添加以下kext来自OCLP([**在此处找到**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/master/payloads/Kexts/Wifi)和[**在此处**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera))(根据需要调整`MinKernel`)： <br> ![Brcm_Sononma2](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/49c099aa-1f83-4112-a324-002e1ca2e6e7)
3. 保存并重启
4. 验证所有列出的kext是否已加载。输入`kextstat | grep -v com.apple`，检查列表。如果它们没有加载，将`-brcmfxbeta`启动参数添加到配置中。保存并重启，再次验证。
5. 使用OCLP 0.6.9或更高版本应用根补丁(可以在[此处找到开发版本](https://github.com/dortania/OpenCore-Legacy-Patcher/pull/1077#issuecomment-1646934494))
6. 如果“Networking: Modern Wireless”或“Networking Legacy Wireless”(根据你的卡片使用其中一个)未出现在可用补丁列表中，需要手动启用该选项并自行编译OpenCore Patcher。说明可以在[这里找到](https://github.com/5T33Z0/OC-Little-Translated/blob/main/14_OCLP_Wintel/WIiFi_Sonoma.md)
7. 重启。此后Wi-Fi应能正常工作(如果你的卡片支持)。

**兼容卡片**：目前仅支持少数WiFi卡。根据使用的卡片，你需要为Wi-Fi打补丁启用正确的选项：

- **现代**：
	- Broadcom BCM94350, BCM94360, BCM43602, BCM94331, BCM943224
- **遗留**：
	- Atheros芯片组
	- Broadcom BCM94322, BCM94328

### 示例11：CPUFriend
你可以使用**CPUFriend.kext**和一个数据注入器kext来修改macOS使用的CPU频率向量。

默认情况下，所选SMBIOS中的频率向量用于处理CPU电源管理。如果你的Hackintosh使用的CPU型号与所选SMBIOS对应的Mac型号中的CPU相同，则不需要使用这个kext。但如果你的系统使用的CPU与所选SMBIOS中的CPU型号不匹配(即你的CPU优于或劣于所选Mac中的CPU)，则应优化CPU电源管理，使CPU在macOS中正常工作。

**示例**：你使用的是[**iMac19,1**](https://everymac.com/ultimate-mac-lookup/?search_keywords=iMac19,1) SMBIOS，但你的CPU是[i7-9700](https://ark.intel.com/content/www/de/de/ark/products/191792/intel-core-i79700-processor-12m-cache-up-to-4-70-ghz.html)，这个CPU没有在该SMBIOS中定义。在这种情况下，你应使用CPUFriendFriend生成一个数据提供程序kext，为CPUFriend提供正确的CPU频率向量，从而注入到所选SMBIOS中。要使此功能正常工作，CPUFriend需要在CPUFriendDataprovider之前加载。

**步骤如下**：

1. 下载并运行[**CPUFriendFriend**](https://github.com/corpnewt/CPUFriendFriend)
2. 按照屏幕上的指示操作
3. 将[**CPUFriend.kext**](https://github.com/acidanthera/CPUFriend)和CPUFriendDataprovider kext添加到`EFI/OC/Kexts`和`config.plist`
4. kext加载顺序如下：<br> ![CPUFriend](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/2189b749-cf6e-4027-a36b-95a385e2c521)

> [!WARNING]
> 
> 有关CPU电源管理的更多信息，请参阅→[**启用CPU电源管理**](/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management)


### 示例12：在macOS Sequoia驱动Intel无线网卡
:warning: 当[zxystd](https://github.com/zxystd)发布Sequoia版本的Airportitlwm后，此方法不再有效！（z大yyds
请看VCR并搭配[OCLP-Mod](https://github.com/laobamac/OCLP-Mod)及其附属驱动食用。[哔哩哔哩-在macOS Sequoia驱动Intel无线网卡教程](https://www.bilibili.com/video/BV1r5WyeyE3k/?spm_id_from=333.337.search-card.all.click)

## 注意事项和致谢
- :warning: 本节中包含的plist仅为示例，**不能**用于任何系统。它们仅用于演示`Kernel/Add`部分中kext的加载顺序！
- 忽略截图中的红点。
- 示例2到6中列出的kext是用于PS2控制器(键盘、鼠标、触控板)。建议使用`config-2-PS2-Controller`plist作为起始点。
