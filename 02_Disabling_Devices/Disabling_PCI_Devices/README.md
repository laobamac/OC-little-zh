# 禁用PCI设备

## 关于
在某些情况下，你可能希望禁用 PCI 设备。比如，独显、声卡或通过 PCIe 连接的不兼容的 SD 读卡器。禁用 PCI 设备有两种方法：使用 SSDT 或阻止启用设备的 kext。

何时使用哪种方法取决于具体情况。使用 SSDT 是首选，但仅当相关设备是 PCI 总线上的子设备时才有效。

## 方法

- **方法 1**: [**通过SSDT**](/02_Disabling_Devices/Disabling_PCI_Devices/ACPI/README.md) (推荐)
- **方法 2**: [**通过Kexts**](/02_Disabling_Devices/Disabling_PCI_Devices/Block_Kexts/README.md)
