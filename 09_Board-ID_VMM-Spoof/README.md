# 在macOS 11.3及更新版本中使用不受支持的Board-ID

**目录**

- [在macOS 11.3及更新版本中使用不受支持的Board-ID](#在macos-113及更新版本中使用不受支持的board-id)
	- [解释](#解释)
	- [系统要求](#系统要求)
	- [使用场景](#使用场景)
	- [操作步骤](#操作步骤)
	- [验证](#验证)
	- [备注](#备注)
	- [鸣谢](#鸣谢)

---

## 解释
**OpenCore Legacy Patcher** (OCLP) 0.3.2 引入了一组Booter和Kernel补丁，这些补丁利用了Big Sur引入的macOS虚拟化功能 (VMM)，让macOS“以为”它运行在虚拟机中：

> Parrotgeek1的VMM补丁集会强制 `kern.hv_vmm_present` 始终返回 `True`。当 `hv_vmm_present` 返回 `True` 时，**`OSInstallerSetupInternal`** 和 **`SoftwareUpdateCore`** 将设置 **`VMM-x86_64`** Board-ID，而操作系统的其余部分会继续使用原始ID。
>
> - 修补 `kern.hv_vmm_present` 比手动设置VMM CPUID更好，因为它支持原生的CPU和GPU电源管理功能。

**来源**：[**OCLP issue 543**](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/543)

这种方法可以在macOS 11.3及更新版本中为不支持的Board-ID/CPU安装和接收OTA系统更新。虽然OCLP的主要目标是为旧Mac安装OpenCore和macOS，但这些补丁也可以在Hackintosh中使用，同时使用指定的SMBIOS提升CPU和GPU的电源管理，尤其是在笔记本电脑上。我成功在我的 [联想T530 ThinkPad](https://github.com/5T33Z0/Lenovo-T530-Hackinosh-OpenCore) 上运行macOS Sonoma。

尽管早在这些补丁出现之前，通过 `-no_compat_check` 引导参数就可以在不支持的SMBIOS上安装macOS，但如果启用了该引导参数，则无法接收OTA系统更新。

> [!警告]
>
> - 自 `RestrictEvents.kext` v1.1.3 版本起，这些Kernel补丁已集成到kext中，因此无需单独添加。如果你的 `config.plist` 中仍然包含[这些Kernel补丁](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist#L2163-L2282)，请禁用或删除它们！
> - 在 `RestrictEvents.kext` 发布之前，这些Kernel补丁对蓝牙功能有负面影响，因为启用VMM Board-ID会跳过加载蓝牙设备固件。这个问题现在已解决（→ [更多详情](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1076)）。

## 系统要求
**最低macOS版本**：Big Sur 11.3或更新版本（Darwin Kernel 20.4+）是***必须的***！</br>

**Intel CPU系列**：

- 第一代Intel Core CPU（需 [SurPlus Kernel Patches](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist#L2103-L2162)）
- Sandy Bridge（需 [SurPlus Kernel Patches](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist#L2103-L2162)）
- Ivy Bridge
- Haswell/Broadwell
- Skylake（在macOS 13+中继续使用SMBIOS `iMac17,1` 需要额外的[iGPU伪装](/11_Graphics/iGPU/Skylake_Spoofing_macOS13)，以支持Intel HD 530。）

> [!注意]
>
> 7到10代Intel Core CPU目前不需要这种伪装，因为macOS仍支持它们。

## 使用场景
1. **在不支持的CPU和SMBIOS/Board-ID上安装macOS 11.3+**。
2. **启用系统更新**。同时，这些补丁可以绕过以下情况下的系统更新问题：
   - 使用的SMBIOS属于带T1/T2安全芯片的Mac机型，例如：
     - MacBookPro15,1 (`J680`), 15,2 (`J132`), 15,3 (`J780`), 15,4 (`J213`)
     - MacBookPro16,1 (`J152F`), 16,2 (`J214K`), 16,3 (`J223`), 16,4 (`J215`)
     - MacBookAir8,1 (`J140K`), 8,2 (`J140A`)
     - 等等。

详细解释见原文。

## 操作步骤
- 挂载你的EFI分区。
- 使用ProperTree打开 `config.plist`。
- 将OCLP [**`Booter/Patch`**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist#L220-L243) 中的“跳过Board ID检查”补丁复制到你的 `config.plist`，并启用。
- 将 [**`RestrictEvent.kext`**](https://github.com/acidanthera/RestrictEvents/releases) 1.1.3或更新版本添加到 `EFI/OC/Kext` 文件夹，并在 `config.plist` 中配置。
- 删除 `-no_compat_check` 引导参数（如果存在）。
- 添加 `revpatch=sbvmm` 到引导参数或作为NVRAM变量。
- 可选但推荐：在 `PlatformInfo/Generic` 中选择正确的SMBIOS并生成新序列号（例如使用OCAT或GenSMBIOS）。
- 保存配置并重启。
- 安装macOS 12或更新版本。

## 验证
详细的测试截图和过程可以展开[这里](#proof)查看。

## 备注
- 升级到macOS 12+后，需要重新安装旧版显卡驱动，例如：Intel HD Graphics、NVIDIA Kepler等。
- 有关在不支持的平台上运行macOS Ventura及更新版本的详细配置，请参考 [**OCLP Wintel**](/14_OCLP_Wintel/README.md)。

## 鸣谢
- [**VMM使用说明**](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/543#issuecomment-953441283)
- Dortania团队开发的 [**OpenCore Legacy Patcher**](https://github.com/dortania/OpenCore-Legacy-Patcher)
- parrotgeek1提供的 [**VMM补丁**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/4a8f61a01da72b38a4b2250386cc4b497a31a839/payloads/Config/config.plist#L1222-L1281)
- reenigneorcim开发的 [**SurPlus**](https://github.com/reenigneorcim/SurPlus)
