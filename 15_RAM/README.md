# 修复macOS中错误报告的RAM速度

- [修复macOS中错误报告的RAM速度](#修复macos中错误报告的ram速度)
	- [关于](#关于)
	- [修复方法](#修复方法)
	- [收集RAM数据](#收集ram数据)
		- [使用Windows](#使用windows)
		- [使用Linux（推荐）](#使用linux推荐)
	- [将内存数据添加到config.plist](#将内存数据添加到configplist)
	- [验证](#验证)
	- [进一步资源](#进一步资源)

---

## 关于
我最近购买了一台联想T490笔记本。检查安装的RAM时，我注意到在系统信息中报告的RAM速度为2400 MHz：

![2400](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/e068bb0e-d9e7-4e0f-a591-50a6ba992ac4)

但在Windows中，它正确地显示为2666 MHz：

![MemSpeed](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/41e21b50-d19c-4ac8-9c2e-5fbd615cfe01)

由于在Wintel系统中，RAM速度由BIOS/UEFI固件处理，我推测macOS中存在内存速度报告问题。

## 修复方法
为了修复macOS中错误报告的内存速度，你*可以*做以下操作。我说*可以*是因为这个修复纯粹是外观上的修复：

## 收集RAM数据

### 使用Windows
1. 运行Windows，打开命令提示符并输入：
	
	```bash
	wmic memorychip list full
	```
2. 输出应如下所示：
	```
	BankLabel=BANK 0
	Capacity=8589934592
	DataWidth=64
	Description=Physical Memory
	DeviceLocator=ChannelA-DIMM0
	FormFactor=12
	HotSwappable=
	InstallDate=
	InterleaveDataDepth=
	InterleavePosition=
	Manufacturer=Samsung
	MemoryType=0
	Model=
	Name=Physical Memory
	OtherIdentifyingInfo=
	PartNumber=M471A1K43BB1-CTD
	PositionInRow=
	PoweredOn=
	Removable=
	Replaceable=
	SerialNumber=00000000
	SKU=
	Speed=2667
	Status=
	Tag=Physical Memory 0
	TotalWidth=64
	TypeDetail=128
	Version=

	BankLabel=BANK 2
	Capacity=8589934592
	DataWidth=64
	Description=Physical Memory
	DeviceLocator=ChannelB-DIMM0
	FormFactor=12
	HotSwappable=
	InstallDate=
	InterleaveDataDepth=
	InterleavePosition=
	Manufacturer=Samsung
	MemoryType=0
	Model=
	Name=Physical Memory
	OtherIdentifyingInfo=
	PartNumber=M471A1K43CB1-CTD
	PositionInRow=
	PoweredOn=
	Removable=
	Replaceable=
	SerialNumber=397269CB
	SKU=
	Speed=2667
	Status=
	Tag=Physical Memory 1
	TotalWidth=64
	TypeDetail=128
	Version=
	```
3. 将数据保存为.txt文件，以便稍后在macOS中访问

### 使用Linux（推荐）
推荐使用Linux，因为Windows下的命令没有显示`AssetTag`。以下是直接从USB闪存驱动器运行Linux live版的说明，无需安装Linux。

1. 准备一个USB闪存驱动器，安装[**Ventoy**](https://github.com/ventoy/Ventoy)
2. 下载一个你选择的Linux发行版的.iso文件（例如Ubuntu或Zorin等）
3. 将.iso文件复制到Ventoy USB驱动器中
4. 从USB闪存驱动器启动
5. 在Ventoy菜单中，选择你的Linux发行版并点击“Normal Boot”
6. 选择“Try”选项而不是“Install”选项
7. 一旦进入桌面，打开终端并输入：
	```bash
	sudo dmidecode -t memory
	```
8. 输出应如下所示：
	```
	dmidecode 3.2
	Getting SMBIOS data from sysfs.
	SMBIOS 3.1.1 present.

	Handle 0x0002, DMI type 16, 23 bytes
	Physical Memory Array
		Location: System Board Or Motherboard
		Use: System Memory
		Error Correction Type: None
		Maximum Capacity: 16 GB
		Error Information Handle: Not Provided
		Number Of Devices: 2

	Handle 0x0003, DMI type 17, 40 bytes
	Memory Device
		Array Handle: 0x0002
		Error Information Handle: Not Provided
		Total Width: 64 bits
		Data Width: 64 bits
		Size: 8192 MB
		Form Factor: SODIMM
		Set: None
		Locator: ChannelA-DIMM0
		Bank Locator: BANK 0
		Type: DDR4
		Type Detail: Synchronous
		Speed: 2667 MT/s
		Manufacturer: Samsung
		Serial Number: 00000000
		Asset Tag: None
		Part Number: M471A1K43BB1-CTD    
		Rank: 1
		Configured Memory Speed: 2400 MT/s
		Minimum Voltage: Unknown
		Maximum Voltage: Unknown
		Configured Voltage: 1.2 V

	Handle 0x0004, DMI type 17, 40 bytes
	Memory Device
		Array Handle: 0x0002
		Error Information Handle: Not Provided
		Total Width: 64 bits
		Data Width: 64 bits
		Size: 8192 MB
		Form Factor: SODIMM	
		Set: None
		Locator: ChannelB-DIMM0
		Bank Locator: BANK 2
		Type: DDR4
		Type Detail: Synchronous
		Speed: 2667 MT/s
		Manufacturer: Samsung
		Serial Number: 397269CB
		Asset Tag: None
		Part Number: M471A1K43CB1-CTD    
		Rank: 1
		Configured Memory Speed: 2400 MT/s
		Minimum Voltage: Unknown
		Maximum Voltage: Unknown
	```
9. 将数据保存为.txt文件到USB闪存驱动器中，以便稍后在macOS中访问

> [!注意]
> 
> 看起来macOS显示的是“Configured Memory Speed”值，而不是“Speed”值。由于我的“Configured Memory Speed”是2400 MT/s，这就是为什么在macOS中报告的速度低于Windows的原因。但“Configured Memory Speed”实际上指的是*实际*运行的内存速度，而不是它能达到的最大速度。这可以在BIOS中更改，但笔记本BIOS通常不允许你配置内存速度。唉…

## 将内存数据添加到config.plist 
:warning: 在将任何数据输入`config.plist`之前，先阅读OpenCore的“Documentation.pdf”第10.4章：“内存属性”，以熟悉可用的参数！因为某些参数在OpenCore中使用不同的值，必须将其从十六进制转换为十进制。

1. 重启进入macOS
2. 运行OpenCore辅助工具，挂载EFI并打开`config.plist`
3. 转到`PlatformInfo/Memory`
4. 启用`CustomMemory`（禁用它将忽略整个`Memory`部分）
5. 输入你之前收集的相关数据，并确保它符合OpenCore的标准： ![CstmMem](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/b40dd4e2-aca6-454f-86bd-75ab8faf78c6)
6. 保存配置并重启

## 验证
- 运行系统信息
- 检查“内存”部分。现在应该报告正确的内存速度： <br>![2667](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/338f44f4-f7db-4bbf-91ca-53ec9afbf187)
- 完成

## 进一步资源
- [内存属性部分](https://www.insanelymac.com/forum/topic/345520-opencore-063-new-memory-properties-section/) 来自miliuco
