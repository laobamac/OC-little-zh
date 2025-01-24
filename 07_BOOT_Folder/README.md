# 添加和配置 `.contentFlavour` 和 `.contentVisibility`

- [添加和配置 `.contentFlavour` 和 `.contentVisibility`](#添加和配置-contentflavour-和-contentvisibility)
  - [关于](#关于)
  - [如何添加 `.contentFlavour` 和 `.contentVisibility`](#如何添加-contentflavour-和-contentvisibility)
  - [关于 `.contentVisibility`](#关于-contentvisibility)
    - [用法](#用法)
    - [InstanceIdentifier](#instanceidentifier)
  - [关于 `.contentFlavour`](#关于-contentflavour)
    - [配置](#配置)
  - [资源](#资源)

---

## 关于
OpenCore 0.8.8引入了两个新的隐藏文件：`.contentFlavour` 和 `.contentVisibility`，用于进一步控制BootPicker的行为。如果你在0.8.8版本之前使用过OpenCore，并且使用OCAT进行更新，这些文件可能缺失在你的`EFI/OC/BOOT`文件夹中。

## 如何添加 `.contentFlavour` 和 `.contentVisibility` 

- 挂载你的EFI
- 下载OpenCore包
- 解压它
- 进入 `Downloads/OpenCore-0.9.X-RELEASE/X64/EFI/BOOT`
- 按 <kbd>CMD</kbd>+<kbd>.</kbd> 显示隐藏文件
- 将 `.contentFlavour` 和 `.contentVisibility` 文件复制到 `EFI/OC/Boot`
- 再次按 <kbd>CMD</kbd>+<kbd>.</kbd> 隐藏文件夹和文件

## 关于 `.contentVisibility`
**`.contentVisibility`** 用于隐藏OpenCore的启动菜单中的启动项。与以前版本中必须计算 `ScanPolicy` 并在某些情况下与 `CustomEntries` 结合使用来使其工作的方法相比，这使得隐藏条目变得更加简单。

### 用法
只需将 `.contentVisibility` 文件放置在包含你想要从BootPicker中隐藏的操作系统引导程序的文件夹中。

`.contentVisibility` 文件可以使用TextEdit、Visual Studio Code和Xcode打开和编辑。它基本上包含一个单词(ASCII)：`Disabled`：

![visibility](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/101f23b6-06b2-4938-b741-468e27ffe6ac)

**选项** – 你可以通过使用以下词语来改变其行为：

- `Disabled` &rarr; 隐藏启动项。
- `Enabled` &rarr; 显示启动项。
- `Auxiliary` &rarr; 将启动项视为辅助项，只有按下空格键后才会显示。我将此选项用于我的USB闪存驱动器，它永久连接到我的系统，作为一个插入后即可使用的设备，包含我的EFI文件夹的工作备份。

**位置** – 你可以将文件放置在以下位置：

- `/System/Volumes/Preboot/{GUID}/.contentVisibility`
- `/System/Volumes/Preboot/.contentVisibility`
- `/Volumes/{ESP}/.contentVisibility`(不推荐)
- `/EFI/Boot` 文件夹(如果是在USB闪存驱动器上，设置为 `Auxiliary`，而不是 `Disabled`！)

**示例**：

- 要隐藏EFI文件夹，将其放在 `EFI/OC/BOOT` 中
- 要隐藏Windows，将其放在包含 `Microsoft/Boot` 文件夹的EFI中
- 对于其他操作系统，使用相同的原则。

### InstanceIdentifier
OpenCore 0.9.4添加了一个新功能，称为 `InstanceIdentifier`。这允许在配置中添加 `InstanceIdentifier`，描述使用的OpenCore实例。如果你有两个不同的OpenCore实例(使用不同的标识符)，你可以修改 `.contentVisibility` 文件来隐藏这些实例。

更多详情，请参见OpenCore文档第8.1.1章：启动算法。

## 关于 `.contentFlavour`

**`.contentFlavour`** 用于修改OpenCore的BootPicker中的启动项外观和感觉(包括音频辅助)。要启用它，必须将 `OC_ATTR_USE_FLAVOUR_ICON` 标志添加到 `PickerAtributes` 位掩码中。请查看[OpenCore计算器](/B_OC_Calculators)部分，了解如何生成该位掩码。

### 配置
要配置 `.contentFlavour` 文件，请遵循Acidanthera提供的详细[OpenCore内容风格](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Flavours.md)指南。

## 资源
OpenCorePkg [Pull Request #446](https://github.com/acidanthera/OpenCorePkg/pull/446): 添加可选的 `.contentVisibility` 
