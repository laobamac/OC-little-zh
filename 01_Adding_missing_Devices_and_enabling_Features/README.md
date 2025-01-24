# 为macOS启用设备、功能

**目录**

- [为macOS启用设备、功能](#为macos启用设备功能)
  - [关于](#关于)
  - [虚拟设备属性](#虚拟设备属性)
  - [获取ACPI表](#获取acpi表)
    - [通过Clover保存ACPI (不再建议，没有必要了)](#通过clover保存acpi-不再建议没有必要了)
    - [使用 OpenCore 转储 ACPI 表(需要 Debug 版本)](#使用-opencore-转储-acpi-表需要-debug-版本)
    - [通过 `SSDTTime`转储ACPI和对应SSDT补丁 (最好的方法！)](#通过-ssdttime转储acpi和对应ssdt补丁-最好的方法)
  - [添加缺少的设备和功能](#添加缺少的设备和功能)
    - [功能性 SSDT](#功能性-ssdt)
    - [装饰性 SSDT(可选)](#装饰性-ssdt可选)
  - [将 '.dsl' 文件转换为 '.aml'](#将-dsl-文件转换为-aml)
  - [为不同版本的 macOS 应用不同的 ACPI 补丁](#为不同版本的-macos-应用不同的-acpi-补丁)
  - [SSDT 与 `config.plist`：了解属性注入优先级](#ssdt-与-configplist了解属性注入优先级)
  - [避免使用 Olarila/MalD0n 的补丁](#避免使用-olarilamald0n-的补丁)
  - [资源](#资源)

---

## 关于
在此存储库中提供的许多“SSDT”补丁中，有相当一部分用于在 macOS 中启用设备、服务或功能。它们可分为四大类：

- **虚拟设备**，例如假冒嵌入式控制器或环境光传感器等。这些只需要存在，macOS 就会正常启动，不需要额外操作。
- 存在于“DSDT”中但被供应商禁用的设备。这些设备通常在 Windows 下被视为“旧版/Legacy”设备，但 macOS 需要启动这些设备。它们仍然存在于系统的“DSDT”中并提供相同的功能，但被禁用以支持较新的设备。比如实时时钟(“RTC”)，它在现代**X86**机器(如 酷睿7th平台+ 和更新版本)上被禁用，取而代之的是“AWAC”时钟。此类的 SSDT 修补程序会禁用较新的设备，并通过反转其状态(“_STA”)来为 macOS 启用其“旧版/Legacy”设备。
- **ACPI 中不存在的设备，或者使用与 macOS 预期不同的名称才能正常工作**。SSDT 热补丁仅针对 macOS 重命名这些设备/方法，因此它们可以附加到 macOS 中的驱动和服务，但也可以在其他操作系统中按预期工作，例如：USB 和 CPU 电源管理、笔记本电脑显示器的背光控制等。
- 补丁组合，分阶段工作以重新定义设备或方法，使其适用于 macOS。首先，将原始设备/方法重命名，以便其 在macOS  中失效，比如，将 '_PRW' 重命名为 'XPRW'。然后注入新的 SSDT，该 SSDT 仅重新定义适用于 macOS 的设备或方法。然后，重新定义的设备/方法将注入回系统中，使其在 macOS 中按预期工作。比如：修复睡眠和唤醒问题或启用触控板。

:bulb: OpenCore 用户应避免使用二进制重命名来启用设备和方法，因为这些重命名将应用于整个系统，这可能会破坏其他操作系统。相反，应应用使用 “_OSI” 方法仅针对 macOS 重命名这些设备/方法的符合 ACPI 的 SSDT。除此之外i，你也可以使用OpenCore Mod来让ACPI修补只适用于Darwin，而在其他系统(比如Windows)不生效，仍采用主板内置的ACPI。

Clover 用户不必担心这一点，因为二进制重命名和 SSDT 热补丁不会注入到其他操作系统中(除非您告诉它这样做)。但是，如果您是切换到 OpenCore 的 Clover 用户，则必须调整您的 SSDT 或使用OC Mod，因为它们很可能不包含“_OSI”方法！

> [!警告] 
> 
> 不要将已知的设备注入 macOS！将相同的、已知的设备和属性注入到系统中，在大多数情况下，是完全不必要的 - 这样做没有任何好处，还会减慢启动过程。
>
> 这样做的唯一原因是在 **系统信息** 的 “PCI” 部分列出了已安装的 PCIe 设备。除此之外，所有检测到的设备将自动列在它们所属的相应类别中。所以真的没有必要这样做。
>
>:bulb: 你只需要注入 'DeviceProperties'，以防你需要修改设备、功能等的参数/属性。因此，不要将相同的、未修改的属性注入您最初获取它们的系统中！

## 虚拟设备属性
- **功能**:
  - 该设备已存在于 ACPI 系统中，相对较小且自包含在代码中。 
  - 原始设备具有规范的 **'_HID'** 或 **'_CID'** 参数。
  - 即使原始设备未被禁用，在 macOS 运行时添加虚拟设备也不会导致冲突。
- **需求**:
  - 假冒设备/虚拟设备的名称与ACPI表中使用的原始设备名称**不同**。
  - 补丁内容和原始设备主内容 **相同**。
  - 热补丁的 **'_STA'** 方法必须包含 [**'If (_OSI (“Darwin”))'**](https://uefi.org/specs/ACPI/6.4/05_ACPI_Software_Programming_Model/ACPI_Software_Programming_Model.html#osi-operating-system-interfaces) 方法或使用OpenCore Mod，以确保补丁仅适用于 macOS (Darwin Kernel)：
	```asl
	Method (_STA, 0, NotSerialized)
       {
            If (_OSI ("Darwin"))
            {
                Return (0x0F)
            }
            Else
            {
                Return (Zero)
            }
        }
	```
- **示例**: [**修复IRQ冲突**](/01_Adding_missing_Devices_and_enabling_Features/IRQ_and_Timer_Fix_(SSDT-HPET))

> [!重要]
>
> “SSDT”中使用的 [**Low Pin Count Bus**](https://www.intel.com/content/dam/www/program/design/us/en/documents/low-pin-count-interface-specification.pdf) 的名称和路径(通常为“LPC”或“LPCB”)必须与原始 ACPI 中使用的名称和路径匹配，补丁才能正常工作！

## 获取ACPI表
为了弄清楚你的系统需要哪些 SSDT，有必要研究您机器的 ACPI 表 - 更具体地说，系统的“DSDT”存储在主板的 BIOS/UEFI 中。下面列出了几个选项来获取它。

**需求**: FAT32 格式的 USB 驱动器(用于 Clover/OpenCore)和以下方法之一来转储系统的 ACPI 表。

### 通过Clover保存ACPI (不再建议，没有必要了)
 Clover可以在没有配置的情况下在几秒钟内转储ACPI，但需要你先创建一个Clover引导。并且提取出的文件大多无用，我更建议使用SSDTTime。


- 下载最新的 [**Release**](https://github.com/CloverHackyColor/CloverBootloader/releases) (CloverV2-xxxx.zip) 并解压
- 将“EFI”文件夹放在 USB 驱动器上。
- 从U按启动系统。
- 在引导菜单中按“F4”。屏幕应闪烁一次。
- 重新启动到您的操作系统
- 转储的 ACPI 表将存储在闪存驱动器上的“EFI\CLOVER\ACPI\origin”下。

### 使用 OpenCore 转储 ACPI 表(需要 Debug 版本)

- 下载 [**OC Debug EFI**](https://github.com/utopia-team/opencore-debug/releases) 并解压
- 将“EFI”文件夹放在 USB 驱动器上。
- 从闪存驱动器启动系统。
- 让代码跑完，直到到达基于文本的启动菜单。这大约需要一分钟
- 拔出 U 盘并重新启动到正常工作的操作系统。
- 将 USB 闪存驱动器放回原处。转储的 ACPI 表将位于 “SysReport” 中。

### 通过 `SSDTTime`转储ACPI和对应SSDT补丁 (最好的方法！) 

- 下载 **SSDTTime**
- 双击 'SSDTTime.bat'
- 您应该会看到一个包含很多选项的菜单。
- 按“p”转储“DSDT”
- 它将位于名为“Results”的子文件夹中。

> [!备注]
> 
> 如果您因为自动下载所需工具('iasl.exe' 和 'acpidump.exe')失败而收到错误消息，请手动下载它们 [这里](https://www.intel.com/content/www/us/en/download/774881/acpi-component-architecture-downloads-windows-binary-tools.html)，解压.zip，将两个可执行文件放在 “Scripts” 文件夹中，然后再次运行 'SSDTTime.bat' 文件。

## 添加缺少的设备和功能
下面列出了两类 ACPI 修补程序：基本(或功能)和非基本 SSDT。**功能** SSDT 是在 Wintel 系统上启动 macOS 的 **必需**。根据使用的 macOS 版本，可能需要一些 SSDT(例如“SSDT-ALS0”或“SSDT-PNLF”)，一些根据使用的 CPU 和/或芯片组(例如“SSDT-HPET”、“SSDT-AWAC”或“SSDT-PMCR”)需要。非必要(或 **装饰性**)SSDT 只能被视为一种改进。这些不是让您的黑苹果正常工作的必要条件。阅读上面内容并参考[Dortania的ACPI指南](https://dortania.github.io/Getting-Started-With-ACPI/)来选择。

修补程序必须放在 'EFI/OC/ACPI' 中，并添加到 'config.plist' 中(在 'ACPI/Add' 下)。OpenCore 接受扩展名为“.aml”和“.bin”的文件。

> [!NOTE]
> 
> 您可以使用 Python 脚本 [**SSDTTime**](/01_Adding_missing_Devices_and_enabling_Features/_SSDTTime) 自动生成大量相关的 SSDT 修补程序。

### 功能性 SSDT
下面列出了在 macOS 中添加或启用设备和功能的 SSDT。使用列出的搜索词在系统的“DSDT”中查找设备。如果未列出搜索词，则需要进一步分析“DSDT”才能应用热补丁。首先阅读热补丁的描述，了解您是否真的需要它以及如何正确应用它。

|SSDT|Description|Search term(s) in DSDT 
|:----:|-------------|:-------------------:|
[**SSDT-ALS0/ALSD**](/01_Adding_missing_Devices_and_enabling_Features/Ambient_Light_Sensor_(SSDT-ALS0)/README.md)|添加一个假的环境光传感器 (SSDT-ALS0) 或在 macOS 中启用现有的环境光传感器 (SSDT-ALSD)。也包含在 OpenCorePkg 中。|`ACPI0008`
[**SSDT-AWAC**](/01_Adding_missing_Devices_and_enabling_Features/System_Clock_(SSDT-AWAC)/README.md)|禁用 macOS 的 AWAC 系统时钟，并强制启用 RTC。适用于 300 系列和更新版本的芯片组。也包含在 OpenCorePkg 中。|`Device (AWAC)` or `ACPI000E`
[**SSDT-BRG0**](/11_Graphics/GPU/GPU_undetected/README.md)|用于启用位于中间 PCI 桥后面的未检测到的 AMD GPU，而无需为其分配 ACPI 设备名称。也包含在 OpenCorePkg 中。比如6900XTXH。| –
[**SSDT-Darwin**](/01_Adding_missing_Devices_and_enabling_Features/SSDT-Darwin/README.md)|增强了“_OSI”方法以允许 macOS 版本检测。允许为不同版本的 macOS 的设备注入不同的属性。
[**SSDT-DTGP**](/01_Adding_missing_Devices_and_enabling_Features/Method_DTGP/README.md)|添加 'DTPG' 方法。仅当方法已寻址但未包含在 SSDT 本身中时，才需要该方法。|–
[**SSDT-EC/-USBX**](/01_Adding_missing_Devices_and_enabling_Features/Embedded_Controller_(SSDT-EC)/README.md)|添加一个伪造的嵌入式控制器 (SSDT-EC) 并启用 USB 电源管理 (SSDT-EC-USBX)。也包含在 OpenCorePkg 中。|`PNP0C09`
[**SSDT-GPIO**](/01_Adding_missing_Devices_and_enabling_Features/OCI2C-GPIO_Patch/README.md)|启用 GPIO 设备.|–
[**SSDT-HPET**](/01_Adding_missing_Devices_and_enabling_Features/IRQ_and_Timer_Fix_(SSDT-HPET)/README.md)| 修复 IRQ 冲突。板载声音才能正常工作。|–
[**SSDT-I225V**](/01_Adding_missing_Devices_and_enabling_Features/Intel_I225-V_Fix_(SSDT-I225V))|修复了 Gigabyte 主板上的 Intel I225-V 以太网控制器。|–
[**SSDT-HV-…**](/01_Adding_missing_Devices_and_enabling_Features/Enabling_Hyper-V_(SSDT-HV-...)/README.md)|用于在 macOS 中启用 Hyper-V 的 SSDT 集。需要额外的 Kext 和二进制重命名。也包含在 OpenCorePkg 中。|–
[**SSDT-IMEI**](/01_Adding_missing_Devices_and_enabling_Features/Intel_MEI_(SSDT-IMEI)/README.md)|将 Intel 管理引擎接口添加到 ACPI。在较旧平台上进行 Intel iGPU 加速时需要。也包含在 OpenCorePkg 中。|`0x00160000`
[**SSDT-LAN**](/01_Adding_missing_Devices_and_enabling_Features/Fake_Ethernet_Controller_(LAN)/README.md)|如果包含的控制器本身不受支持，则添加一个假的以太网控制器。|–
[**SSDT-NAVI**](/11_Graphics/GPU/AMD_Navi/README.md)|在 macOS 中启用 AMD Navi GPU(这是什么奇怪的需求|–
[**SSDT-PLUG**](/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management/CPU_Power_Management_(SSDT-PLUG)/README.md)| 为 Intel CPU 启用 XNU CPU 电源管理 (XCPM)(仅 macOS 11 之前需要)。也包含在 OpenCorePkg 中。|–
[**SSDT-PM**](/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management/CPU_Power_Management_(Legacy)/README.md)|适用于 Intel CPU(第 1 代至第 3 代)的 CPU 电源管理。| –
[**SSDT-PMCR**](/01_Adding_missing_Devices_and_enabling_Features/PMCR_Support_(SSDT-PMCR)/README.md)|将 Apple 独占的“PCMR”设备添加到 ACPI(仅 300 系列需要)。也包含在 OpenCorePkg 中。|`PMCR` or</br> `APP9876`
[**SSDT-PNLF**](/01_Adding_missing_Devices_and_enabling_Features/Brightness_Controls_(SSDT-PNLF)/README.md)|为笔记本电脑屏幕添加背光控制。也包含在 OpenCorePkg 中。|–
[**SSDT-PWRB/SLPB**](/01_Adding_missing_Devices_and_enabling_Features/Power_and_Sleep_Button_(SSDT-PWRB_SSDT-SLPB)/README.md)|如果缺少，则添加电源和睡眠按钮设备(主要适用于笔记本电脑)。|`PNP0C0C`(Power), `PNP0C0E`(Sleep)
[**SSDT-RTC0**](/01_Adding_missing_Devices_and_enabling_Features/System_Clock_(SSDT-RTC0)/README.md) </br>[**SSDT-RTC0-RANGE**](https://dortania.github.io/Getting-Started-With-ACPI/Universal/awac-methods/manual-hedt.html#seeing-if-you-need-ssdt-rtc0-range)|添加一个假的 Real Time Clock。仅对(真正的)300 系列主板 (RTCO) 和 X299 (RTC0-Range) 是必需的！也包含在 OpenCorePkg 中。|`PNP0B00`
[**SSDT-SBUS-MCHC**](/01_Adding_missing_Devices_and_enabling_Features/System_Management_Bus_and_Memory_Controller_(SSDT-SBUS-MCHC)README.md)|修复了 macOS 中的系统管理总线和内存控制器。也包含在 OpenCorePkg 中。|`0x001F0003` or</br> `0x001F0004`
[**SSDT-UNC**](https://dortania.github.io/Getting-Started-With-ACPI/Universal/unc0.html) |禁用未使用的非核心桥，以防止 macOS 11+ 中出现内核崩溃。受影响的芯片组：X99、X79、C602、C612。也包含在 OpenCorePkg 中。|–
[**SSDT-XCPM**](/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management/Enabling_XCPM_on_Ivy_Bridge_CPUs/README.md)|SSDT 和内核补丁，并在 Ivy Bridge CPU 上强制启用 XCPM 电源管理。| –
[**SSDT-XOSI**](/01_Adding_missing_Devices_and_enabling_Features/OS_Compatibility_Patch_(XOSI)/README.md)|操作系统兼容性补丁。也包含在 OpenCorePkg 中。|–

### 装饰性 SSDT(可选)
下面列出的 SSDT 被视为非功能性 - 它们不是必需的。它们添加了真实 Mac 中存在的设备的虚拟版本。添加下面列出的任何表，除了模仿相应 Mac 型号(如 SMBIOS 中所定义)的 I/O 注册表的 ***外观*** 之外，***不会添加或启用任何功能。引用 OpenCore 的一位开发人员的话：

> 我们的机器需要这些设备是不合理的。它们存在于 Apple ACPI 中这一事实并不能使其成为我们 ACPI 的要求。
>
> – [**vit9696**](https://github.com/acidanthera/OpenCorePkg/pull/121#issuecomment-696825376)

|SSDT|Description|Search term(s) in DSDT
|:----:|-------------|:-------------------:|
[**SSDT-AC**](/01_Adding_missing_Devices_and_enabling_Features/AC_Adapter_(SSDT-AC)/README.md)|将 AC 适配器设备连接到 I/O 注册表中的“AppleACPIACAdapter”服务。现在不再需要，因为'VirtualSMC' 和 'SMCBatteryManager' 来处理这个问题。|`ACPI0003`
[**SSDT-ARTC**](/01_Adding_missing_Devices_and_enabling_Features/Fake_Apple_RTC_(SSDT-ARTC)/README.md)|将假的 'ARTC' 设备 (Apple Realtime Clock) 添加到 IOReg。适用于 Intel Core 9th Gen 及更新版本。使用与 'AWAC' 相同的 '_HID' 。| `ACPI000E` 
[**SSDT-DMAC**](/01_Adding_missing_Devices_and_enabling_Features/DMA_Controller_(SSDT-DMAC)/README.md)|将伪 DMA 控制器添加到设备树中。|`PNP0200` or</br> `DMAC`
[**SSDT-FWHD**](/01_Adding_missing_Devices_and_enabling_Features/Fake_Firmware_Hub_(SSDT-FWHD)/README.md)|将虚假固件集线器设备 ('FWHD') 添加到 IOReg。几乎所有基于 Intel 的 Mac 都在使用。|`INT0800`
[**SSDT-MEM2**](/01_Adding_missing_Devices_and_enabling_Features/SSDT-MEM2/README.md)|将“MEM2”设备添加到 iGPU(适用于第 4 代至第 7 代 Intel Core CPU)|`PNP0C01`
[**SSDT-PPMC**](/01_Adding_missing_Devices_and_enabling_Features/Platform_Power_Management_(SSDT-PPMC)/README.md)| 将伪造的平台电源管理控制器添加到 I/O 注册表中(仅限 100/200 系列芯片组)。|`0x001F0002` or</br> `Device (PPMC)`
[**SSDT-XSPI**](/01_Adding_missing_Devices_and_enabling_Features/Intel_PCH_SPI_Controller_(SSDT-XSPI)/README.md)|将伪造的 Intel PCH SPI 控制器添加到 IOReg。出现在第 10 代 Mac(和一些第 9 代移动 CPU)上。|`0x001F0005` 

---

## 将 '.dsl' 文件转换为 '.aml'
本节中的修补程序以反汇编的 ASL 文件 ('.dsl') 的形式提供，以便可以在 Web 浏览器、文本编辑器中查看它们。为了在 Bootloader 中使用它们，需要先将它们转换为 ASL 语言 ('.aml')。以下是如何执行此操作：

1. 点击您选择的 `.dsl` 文件的链接。这将显示文件中包含的代码。  
2. 下载该文件(右上角有一个下载按钮；“下载原始文件”)。  
3. 在 [**MaciASL**](https://github.com/acidanthera/MaciASL) 中打开它。  
4. 编辑文件(如有必要)。  
5. 点击“文件” > “另存为…”。  
6. 从“文件格式”下拉菜单中选择“ACPI 二进制”。  
7. 将其保存为“SSDT–…”(原始文件名)。  
8. 将 `.aml` 文件添加到 `EFI/OC/ACPI` 和您的 `config.plist`(在 `ACPI/Add` 下)。  
9. 保存并重启以进行测试。

## 为不同版本的 macOS 应用不同的 ACPI 补丁

ACPI 规范允许通过使用方法 ('_OSI') (Operating System Interface) 根据检测到的操作系统应用不同的设置。在 SSDT 中大量使用方法“If (_OSI (”Darwin“))”，以便仅在 macOS 运行时应用补丁。

如果您需要根据 macOS 版本为同一设备应用不同的补丁，例如，切换到不同的“AAPL，ig-platform-id”和帧缓冲补丁，以使不受支持的 iGPU 在较新的 macOS 版本中工作，该怎么办？默认情况下，这是不可能的，因为 ACPI 规范不支持此类功能。这本质上是一种全有或全无的情况：Darwin 内核要么正在运行，要么没有运行。

幸运的是，一个名为 [**OSIEnhancer**](https://github.com/b00t0x/OSIEnhancer) 较新的 kext 解决了这个问题。它支持对 OSI(“Darwin”)方法进行修改，从而更好地控制何时应用补丁。使用 OSIEnhancer，您可以指定 Darwin 内核和/或 macOS 版本，从而有效地引入类似于 kext 的“MinKernel”和“MaxKernel”设置的功能，适用于 ACPI 表。

由于 OSIEnhancer 是一个相对较新的 kext，因此可供参考的 SSDT 示例并不多 – 不过，可以在仓库上找到一些示例。它往往针对单个计算机定制，因为补丁通常取决于独特的硬件配置和有问题的 macOS 版本。这使得它更像是“每台计算机”的解决方案，要求用户创建自定义 SSDT 来满足其特定需求。因此，实现 OSIEnhancer 可能涉及一些试验和错误，以及对 ACPI 和系统特定要求的理解。

## SSDT 与 `config.plist`：了解属性注入优先级

在 OpenCore 中，通过 SSDT(辅助系统描述表)注入的属性通常优先于`config.plist`中定义的属性，前提是 SSDT 已正确实现和加载。此优先级是由于 macOS 在启动序列期间处理 ACPI 表和设备属性的方式的分层性质。

**理解层次结构**

1. **ACPI 等级 (SSDTs):**
   - **函数:** SSDT 用于定义或重写 ACPI 表，从而允许低级别硬件配置和属性注入。
   - **优先级:** 由于 SSDT 是在启动过程的早期加载的，因此它们可以在基本级别建立或修改设备属性。

2. **Bootloader 级别 (`config.plist`):**
   - **功能：** OpenCore 中的 `config.plist` 文件用于配置各种 bootloader 设置，包括设备属性注入。
   - 优先级：此处定义的属性在处理 ACPI 表后应用。如果 SSDT 已设置属性，则`config.plist`不能覆盖该属性。
  
**实际意义:**

- **USB 电源属性**：通过 SSDT 注入 USB 电源属性(例如，创建`USBX`设备)是确保 USB 功能正常的常见做法。这种方法通常比通过 `config.plist` 注入相同的属性更可取，因为它在较低级别集成了属性，从而产生更可靠的行为。

- **设备重命名：** 某些设备重命名任务(例如将`GFX0`更改为`IGPU`)可以在 SSDT 中更好地处理，或者使用 WhateverGreen 等 Kext 来处理，它们可以动态执行这些重命名。这种方法通常比通过 `config.plist` 尝试相同的重命名更安全、更有效。

**建议：**

- 将 SSDT 用于硬件级配置：对于需要低级硬件交互的任务，例如注入 USB 电源属性或重命名设备，建议通过 SSDT 实现这些更改。此方法可确保在引导过程的早期应用属性，从而实现更一致的行为。

- 使用 `config.plist` 进行 Bootloader 和高级设置：对于与 Bootloader 本身或高级系统设置相关的配置，`config.plist` 是合适的。这包括启动参数、启用或禁用 kext 等设置，以及其他特定于 bootloader 的选项。

**结论：**

虽然 SSDT 和`config.plist`都可用于在 OpenCore 中注入属性，但它们的应用程序的优先级和时间不同。SSDT 作为 ACPI 层的一部分，会提前处理，并且可以在稍后的启动过程中通过“config.plist”覆盖设置的属性。因此，对于关键硬件级属性注入，SSDT 通常是首选方法。

有关配置 OpenCore 和使用 SSDT 的更详细指南，您可以参考 [**Dortania的OpenCore配置文档**](https://dortania.github.io/docs/latest/Configuration.html)。 

## 避免使用 Olarila/MalD0n 的补丁

> [!CAUTION]
> 
> 避免使用来自 MalD0n/Olarila 的预制 OpenCore(和 Clover)EFI 文件夹，因为它们包含一个通用的`SSDT-OLARILA.aml`，它可以注入您的系统甚至可能不需要的各种设备。它还将“Olarila”品牌注入到“关于本机”部分。要删除它，请删除 'Device (_SB.PCI0 的 API 中。OLAR)' 和 `Device (_SB.PCI0 的 API 中。MALD)` 从这个 SSDT 中。或者更好的是：删除整个文件，并为系统实际需要的设备/功能添加单独的 SSDT。
## 资源
[**DarwinDumped**](https://github.com/khronokernel/DarwinDumped) – 由 khronokernel 提供的几乎所有 Mac 型号的 IORegistry 集合
