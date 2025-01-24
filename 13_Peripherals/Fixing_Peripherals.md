# 修复外设问题

- **受影响的操作系统**：支持Apple Mobile File Integrity的macOS(Sierra及更新版本)。

## 受影响的设备和症状
- **网络摄像头**：
	- 网络摄像头在macOS中正常工作，可以在Facetime和/或Photobooth中使用，但在第三方应用程序中无法使用
	- 没有音频：无法从摄像头麦克风或耳机获取音频
	- 没有视频：在Zoom、Microsoft Teams、Skype等第三方视频会议应用中无法开启摄像头
- **无线鼠标**：
	- 通过Logitech Unifying Software配对Logitech无线鼠标时可能会遇到问题。如果鼠标也可以通过蓝牙配对，则没有问题。

## 原因
- 如果禁用了Apple Mobile File Integrity(AMFI)，则不会弹出授予第三方应用程序权限的提示。

## 解决方案
由于禁用AMFI首先需要禁用系统完整性保护(SIP)，因此重新启用SIP可能是解决方案。以下是四种解决此问题的不同方法…

### 解决方案1：添加`AMFIPass.kext`(最佳方案)
OpenCore Legacy Patcher 0.6.7的测试版引入了一个新的Kext，叫做`AMFIPass`，它允许在禁用SIP的情况下启动macOS并完全启用AMFI，即使已经应用了root补丁——否则这是不可能的。这样不仅增强了安全性，还解决了无法授予第三方应用程序摄像头和麦克风访问权限的问题。

#### 将AMFIPass添加到你的EFI和Config中：
- 从OpenCore Legacy Patcher仓库下载[**`AMFIPass.kext`**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/sonoma-development/payloads/Kexts/Acidanthera)并解压
- 挂载你的EFI
- 将kext添加到`EFI/OC/Kexts`
- 打开你的`config.plist`
- 将kext手动添加到Kernel/Add，或者在ProperTree中创建一个新的OC快照
- **可选**：调整`MinKernel`为需要禁用AMFI才能启动的内核版本。例如：`20.0.0`对应Big Sur，`21.0.0`对应Monterey，`22.0.0`对应Ventura等。
- 删除启动参数`amfi_get_out_of_my_way=0x1`或`AMFI=0x80`(如果存在)
- 保存你的config并重启

**结束**：现在，你可以启用AMFI并再次授予第三方应用程序访问麦克风和摄像头的权限！

> [!WARNING]
> 
> 在安装系统更新后，你可能仍然需要在重新应用root补丁之前添加`AMFI=0x80`。

### 解决方案2：重新启用SIP(并非总是可行)

- 将`csr-active-config`更改为`00000000`以重新启用SIP
- 保存你的config
- 重启

一旦重新启用SIP，启动要求访问摄像头/麦克风的应用时，系统会立即弹出授权提示。之后，这些应用将出现在隐私设置中的摄像头和麦克风列表中。

**问题**：如果你使用的是需要禁用SIP和/或AMFI才能启动的旧硬件，无法使用此方法。因此，你必须升级macOS，使用之前的安装版本，其中已经授予了这些权限，并将其携带到新系统中，或者使用方法3。

### 解决方案3：在安全模式下授予权限(如果禁用了SIP)

- 在`config.plist`中的Misc/Boot下启用`PollAppleHotkeys`
- 重启并进入安全模式(在启动选择器中按住Shift键并按回车)
- 运行需要访问麦克风/摄像头的应用
- 系统会请求授予访问麦克风/摄像头的权限
- 授权后，应用将出现在摄像头/麦克风的隐私设置中
- 正常重启
- 摄像头和麦克风将正常工作

**问题**： 

- 这种方法对Zoom有效，但Microsoft Teams在安全模式下无法运行。由于没有GUI，因此无法触发授予应用访问麦克风/摄像头的权限。
- 仅适用于可以在禁用SIP的情况下进入安全模式的情况。如果你使用OpenCore Legacy Patcher应用了任何Root补丁(例如重新安装iGPU/GPU驱动)，则在安全模式下启动时会卡住，因为图形驱动无法加载。在这种情况下，你也必须使用方法3。

### 解决方案4：手动添加权限(需要命令行技能)

如果无法启用SIP启动，你必须使用[tccplus](https://github.com/jslegendre/tccplus/releases)手动将权限添加到SQL3数据库，具体方法请参见：[无法授予应用程序特殊权限](https://dortania.github.io/OpenCore-Legacy-Patcher/ACCEL.html#unable-to-grant-special-permissions-to-apps-ie-camera-access-to-zoom)

> [!WARNING]
> 
> 本指南适用于*任何*需要访问麦克风和/或摄像头的第三方应用程序——包括数字音频工作站和视频编辑软件！

## 进一步的资源

- **更多关于TCC**：[TCC的作用与限制](https://eclecticlight.co/2023/02/10/privacy-what-tcc-does-and-doesnt)
- **更多关于AMFI**：[AMFI：检查你Mac上的文件完整性](https://eclecticlight.co/2018/12/29/amfi-checking-file-integrity-on-your-mac/)
- **macOS 13中的AMFI**：[Ventura如何检查应用程序的安全性](https://eclecticlight.co/2023/03/09/how-does-ventura-check-an-apps-security/)
