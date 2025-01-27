# 实用工具和资源
这是一个关于Hackintosh的有用免费软件、工具和资源的不完全列表。

## 引导管理器
- [**OpenCore Releases**](https://github.com/acidanthera/OpenCorePkg/releases) – OpenCore的官方RELEASE和DEBUG版本。
- [**OpenCore Nightly Builds**](https://dortania.github.io/builds/?product=OpenCorePkg&viewall=true) – 来自Dortania仓库的最新夜间版本。
- [**OpenCore_NO_ACPI_Build**](https://github.com/wjz304/OpenCore_NO_ACPI_Build/releases) – OpenCore的非官方分支，允许在其他操作系统中不注入ACPI表。
- [**Bootloader Chooser**](https://github.com/jief666/BootloaderChooser) – 一个预引导加载器，可以让你选择实际运行的引导管理器。这样，你可以轻松在Clover和OpenCore之间切换。
- [**Utilities**](/C_Utilities_and_Resources/OpenCore_Utilities.md) – 包含在OpenCore包中的工具。

## 应用和工具

### 获取macOS
用于直接从Apple服务器下载macOS并创建安装介质的工具。

- [**OpenCore Legacy Patcher**](https://github.com/dortania/OpenCore-Legacy-Patcher) – 用于修补macOS以在不受支持的系统上运行的应用程序。还可以下载macOS（Big Sur及更高版本）并制作USB安装盘。
- [**TINU**](https://github.com/ITzTravelInTime/TINU) – 一个开源的工具，用于创建可引导的macOS安装盘。
- [**MIST**](https://github.com/ninxsoft/Mist) (macOS Installer Super Tool) – 目前唯一可以下载ipsw固件（ARM）以及Intel系统macOS安装程序（适用于OSX 10.7.5及更高版本）的应用。也是具有最佳GUI和最多选项的工具。需要macOS Monterey或更高版本。
- [**Create macOS Install**](https://github.com/LAbyOne/Create-MacOS-Install) – 一个基于CLI的工具，用于下载并准备macOS（Snow Leopard或更高版本）、引导加载器和USB安装程序。
- [**gibMacOS**](https://github.com/corpnewt/gibMacOS) – 一个Python脚本，用于下载macOS（10.13至13）。唯一可以在Windows中创建互联网恢复USB安装程序的工具。
- [**StartOSInstall**](https://github.com/chris1111/Startosinstall-Ventura) – 脚本，用于将外部SSD转换为macOS安装程序（Big Sur及更高版本）。
- [**ISO Image Creator**](https://macmeup.com/create-iso-images/) – 将macOS安装程序转换为.iso镜像的应用程序。

### 故障排除/调试
- [**IORegistryExplorer**](https://github.com/utopia-team/IORegistryExplorer) – 用于从macOS的 [**I/OKit**](https://developer.apple.com/library/archive/documentation/DeviceDrivers/Conceptual/IOKitFundamentals/Introduction/Introduction.html#//apple_ref/doc/uid/TP0000011) 收集信息的应用程序。
- [**Hackintool**](https://github.com/headkaze/Hackintool) – 强大的后安装工具，所有Hackintosh用户都应该有。
- [**FindMask**](https://www.insanelymac.com/forum/topic/343572-for-hackers-an-utility-to-search-a-masked-string/?do=findComment&comment=2719931) – 用于在包含掩码代码的文件中搜索字节序列的CLI工具。

### 编辑plist、更新OpenCore和kext
用于保持OpenCore EFI文件夹清洁和更新的工具。

- [**OpenCore Auxiliary Tools (OCAT)**](https://github.com/ic005k/QtOpenCoreConfig) – 编辑OpenCore `config.plist` 的首选工具，支持挂载EFI、保存时迁移配置到最新版本，也可以更新OpenCore、驱动程序和kext到最新版本。强烈推荐。
- [**ProperTree**](https://github.com/corpnewt/ProperTree) – 基于Python的.plist编辑器，具有如快照生成和OpenCore配置错误检查等功能。
- [**PlistEDPlus**](https://github.com/ic005k/PlistEDPlus) – 免费的.plist编辑器。
- [**PlistOxide**](https://github.com/ChefKissInc/PlistOxide) – 用Rust编写的免费plist编辑器。
- [**BitmaskDecode**](https://github.com/corpnewt/BitmaskDecode) – 用于OpenCore的Python位掩码计算器。
- [**Meld for macOS**](https://yousseb.github.io/meld/) – 用于比较两个配置文件的应用程序。
- [**Kext Updater**](https://www.sl-soft.de/en/kext-updater/) – 一款出色的应用程序，用于检查和下载kext更新、引导管理器文件等。
- [**MountEFI**](https://github.com/corpnewt/MountEFI) – 挂载ESP分区的Python脚本。
- [**OCConfigCompare**](https://github.com/corpnewt/OCConfigCompare) – 比较你的配置与最新的sample.plist以迁移并验证它。适合喜欢手动更新配置的人。
- [**OC Anonymizer**](https://github.com/dreamwhite/OC-Anonymizer) – Python脚本，用于从config.plist中移除敏感数据并恢复一些默认设置。
- [**GenSMBIOS**](https://github.com/corpnewt/GenSMBIOS) – 用于生成SMBIOS数据的Python工具。
- [**OpCore Simplify**](https://github.com/lzhoang2801/OpCore-Simplify) – 第一个基于检测到的硬件自动生成OpenCore EFI的工具，包括配置设置、添加ACPI补丁、驱动程序和kext等。*我尚未在我的系统上测试，但看起来非常有前途。* **斜体部分是原作者的看法，我认为这类工具并没有实际意义，黑苹果是一项极其多变的技术，与其用这种工具还不如用OCAT预制的模板加以修改。**
### ACPI相关工具
用于获取、生成和编辑ACPI表的工具。

**编辑器（IDE）**
- [**MaciASL**](https://github.com/acidanthera/MaciASL) – 用于处理ACPI表的ASL/AML编辑器。
- [**Xiasl**](https://github.com/ic005k/Xiasl) – 跨平台ASL/AML编辑器，具有一些独特功能。

**工具**
- [**SSDTTime**](https://github.com/corpnewt/SSDTTime) – 用于转储DSDT和生成自动ACPI热补丁的Python脚本。
- [**MmioDevirt**](https://github.com/corpnewt/MmioDevirt) – 可分析OC引导日志并生成MMIO白名单条目的Python脚本。
- [**ssdtPRGen**](https://github.com/Piker-Alpha/ssdtPRGen.sh) – 用于生成旧版CPU电源管理SSDT的Python脚本。
- [**ACPI Rename**](https://github.com/corpnewt/ACPIRename) – 一个方便的Python脚本，用于分析DSDT。可以列出设备、方法和名称路径，并为二进制重命名生成HEX值。
- [**DarwinDumper**](https://bitbucket.org/blackosx/darwindumper/downloads/) – 用于转储ACPI表、I/O注册表等内容的应用程序。

### 音频
- [**PinConfigurator**](https://github.com/headkaze/PinConfigurator) – 用于导入和编辑AppleALC Layout-ID的PinConfig数据的应用程序。

### 笔记本电脑专用
- [**GenI2C_Refresh**](https://github.com/Baio1977/GenI2C) – 用于生成I2C控制触摸板的SSDT热补丁（与VoodooI2C.kext配合使用）的更新工具。

### macOS 应用程序（通用）
- [**About this Hack**](https://github.com/0xCUB3/About-This-Hack) – 一个简单的应用，可以在macOS中查看你的确切硬件信息。类似于macOS 13之前的“关于本机”窗口，但功能更多。
- [**DevCleaner for Xcode**](https://github.com/vashpan/xcode-dev-cleaner) – 用于清理各种Xcode缓存的应用，可以释放几十GB的存储空间。
- [**unpkg**](https://www.timdoug.com/unpkg/) – 一个实用工具，可以将.pkg文件解压到文件夹中，而不是直接安装到系统中。
- [**LuLu**](https://github.com/objective-see/LuLu) – 一个免费的开源macOS防火墙。
- [Find Any File](https://findanyfile.app/) – FAF可以找到Spotlight无法找到的文件，例如在网络（NAS）和其他外部存储卷上的文件，包和捆绑内隐藏的文件，以及通常被Spotlight排除搜索的文件夹（如System和Library文件夹中的内容）。

### macOS 虚拟化
- [**Create macOS VM Install**](https://github.com/rtrouton/create_macos_vm_install_dmg) – 用于为虚拟化软件（如VMware Fusion或Parallels Desktop）准备macOS安装磁盘映像的脚本。

### 驱动、硬件修补工具
用于在后安装中修补macOS，以便它可以在已终止支持的系统上运行。

- [**OpenCore Legacy Patcher**](https://github.com/dortania/OpenCore-Legacy-Patcher) – 可以安装在macOS 12及更高版本中被移除的NVIDIA和Intel显卡驱动程序、Apple过时硬件(比如T1)、网卡等。
- [**OCLP-Mod**](https://github.com/laobamac/OCLP-Mod) - 一个更加优秀的OCLP，针对国人优化了下载速度、进程逻辑并提供了镜像、KDK等下载服务。且增加了对AR92xx等高通网卡、所有itlwm支持的网卡补丁(截至20250126没发布Airportitlwm for Sequoia前需要补丁)
- [**GeForce Kepler Patcher**](https://github.com/chris1111/Geforce-Kepler-patcher) – 为macOS 12及更高版本中的Kepler显卡恢复NVIDIA驱动程序。
- [**Intel HD 4000 Patcher**](https://github.com/chris1111/Patch-HD4000-Monterey) – 用于安装macOS 12及更高版本中已移除的Intel HD 4000集成显卡驱动程序。

### 跨平台工具
- [**Python Installer**](https://www.python.org/downloads/) – 用于运行列表中Python工具的必要工具。
- [**USBToolbox**](https://github.com/USBToolBox/tool) – USB端口映射工具，使macOS 11.3及更高版本中的端口映射更简单。

### CPU相关工具
- [**freqVectorsEdit**](https://github.com/Piker-Alpha/freqVectorsEdit.sh) – 用于编辑X86PlatformPlugin.kext中Board-ID的plist频率向量的Python脚本。
- [**ssdtPRGen**](https://github.com/Piker-Alpha/ssdtPRGen.sh) – 用于生成旧版CPU电源管理SSDT的Python脚本。
- [**MacCPUID**](https://www.intel.com/content/www/us/en/download/674424/maccpuid.html) – 显示Intel CPU的详细信息（型号、功能、缓存等）。

### Windows 工具
- [**Windows Install**](https://sourceforge.net/projects/windows-install/) – 易于使用的工具，可以在macOS中从.iso文件安装Windows。
- [**BlueScreenView**](https://www.nirsoft.net/utils/blue_screen_view.html) – 显示有关蓝屏死机的信息，包括可能导致崩溃的驱动程序或模块的详细信息。
- [**USB Device Tree Viewer**](https://www.uwe-sieber.de/usbtreeview_e.html) – 一个有助于分析USB端口的工具。
- [**CrystalDiskInfo**](https://crystalmark.info/en/software/crystaldiskinfo/) – 检查硬盘、SSD和NVMe健康状况的好工具。
- [**GPU-Z**](https://www.techpowerup.com/gpuz/) – 提供有关GPU的有用信息。
- [**HWiNFO**](https://www.hwinfo.com/) – 获取有关系统和组件的深入信息的好工具。
- [**Intel Processor Identification Utility**](https://www.intel.com/content/www/us/en/download/12136/intel-processor-identification-utility-windows-version.html) – 顾名思义……

### 基于Web的应用程序
- [**Sanity Checker**](https://sanitychecker.ocutils.me/) – 检查OpenCore配置一致性的流行在线工具已回归！
- [**OpenCore ScanPolicy Generator**](https://oc-scanpolicy.vercel.app/)
- [**AnyPlist**](https://www.anyplist.com/#/) – 在线Plist编辑器。
- [**OpenCore ExposeSensitiveData Generator**](https://dreamwhite-oc-esd.vercel.app/)
- [**Clover Cloud Editor**](https://cloudclovereditor.altervista.org/cce/cce/index.php) – 尽管名字可能有误导，但这个工具是用于在线编辑OpenCore配置的。
- [**OpenCore Configurator Online**](https://galada.gitee.io/opencoreconfiguratoronline/) – 兼容OC ≤ 0.8.0版本。
- [**Big <> Little Endian Converter**](https://www.save-editor.com/tools/wse_hex.html)
- [**Base64 Decoder/Encoder**](https://www.base64decode.org/)

## 更多资源
- [**OpenCore 安装指南**](https://dortania.github.io/OpenCore-Install-Guide/)
- [**Acidanthera 文档**](https://github.com/acidanthera/bugtracker/blob/master/DOCUMENTS.md)
- [**I/O Kit 基础**](https://developer.apple.com/library/archive/documentation/DeviceDrivers/Conceptual/IOKitFundamentals/Introduction/Introduction.html#//apple_ref/doc/uid/TP0000011-CH204-TPXREF101) – 包括有关I/O注册表的解释。
