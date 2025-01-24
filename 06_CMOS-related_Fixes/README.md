# CMOS相关问题修复
**CMOS**内存保存重要数据，如日期、时间、硬件配置信息、辅助设置信息、启动设置、休眠信息等。

人话就是：主板上存储BIOS类数据的，你拔电池作用就是清CMOS

在实际 Mac 上，非易失性 RAM (NVRAM) 用于此目的。就像 CMOS 一样，NVRAM 也由小电池供电，这使得它可以在系统电源关闭时保留数据。

在 Wintel 系统上运行 macOS 时，有时会在关闭或重新启动系统时触发某些 BIOS/固件上的 CMOS 重置。

您可以使用以下修复程序之一来解决此问题：

- [**CMOS 重置修复**](/06_CMOS-related_Fixes/CMOS_Reset_Fix/README.md) – 用于修复 macOS 在关机或重启后触发的 CMOS 重置。
- [**模拟 CMOS 内存**](/06_CMOS-related_Fixes/Emulating_CMOS/README.md) – 用于修复“AppleRTC”与 BIOS/UEFI 固件之间的冲突。在尝试修复休眠时可能会有所帮助。