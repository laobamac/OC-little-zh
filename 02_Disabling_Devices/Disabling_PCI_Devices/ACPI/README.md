# 禁用PCI设备

- [禁用PCI设备](#禁用pci设备)
  - [介绍](#介绍)
  - [设备名](#设备名)
  - [SSDT禁用补丁示例](#ssdt禁用补丁示例)

## 介绍

在某些情况下，您可能希望禁用 PCI 设备。例如独显、声卡或通过 PCIe 连接的 SD 读卡器在 macOS 下无法正常工作。您可以使用自定义 SSDT 补丁来禁用 PCI 设备。

这些设备应具有以下条件：

- 是PCI总线的子设备(你总不能把PCI端口直接噶了吧
- **父设备** 定义一些变量，例如 `PCI_Config` 或 `SystemMemory`，其中偏移量 `0x55` 处数据的位 `D4` 是设备操作属性
- 它有一个 **子设备** 地址：`Name (_ADR， Zero)`  

## 设备名

- 新一点的机器上，**子设备**的名称通常是 **`PXSX`**，而**父设备**的名称是 **`RP01`**，**`RP02`**，**`RP03`** 等等。
- 早期的ThinkPad机器使用的**子设备**名称是 **`SLOT`** 或者 **没有** 名字，而**父设备**的名称是 **`EXP1`**，**`EXP2`**，**`EXP3`** 等等。
- 其他机器可能使用不同的名称。
- 笔记本电脑内建的无线网卡属于这样的设备。

## SSDT禁用补丁示例

- 戴尔Latitude 5480的SD卡属于PCI设备，设备路径：`_SB.PCI0.RP01.PXSX`
- 补丁文件：***SSDT-RP01.PXSX-disable***：
  ```asl
  External (_SB.PCI0.RP01, DeviceObj)
  Scope (_SB.PCI0.RP01)
      {
      OperationRegion (DE01, PCI_Config, 0x50, 0x01)
      Field (DE01, AnyAcc, NoLock, Preserve)
      {
              , 4,
          DDDD, 1
      }
  		//可能的开始
  		Method (_STA, 0, Serialized)
  		{
  				If (_OSI ("Darwin"))
  				{
  					Return (Zero)
  				}
  		}
  		//可能的结束
  }  
  Scope (\)
  {
      If (_OSI ("Darwin"))
      {
          \_SB.PCI0.RP01.DDDD = One
      }
  }


> [!CAUTION]
>
> - 如果一个 **父设备** 有多个 **子设备** ，请 **谨慎使用此方法**。
> - 使用 SSDT 时，将示例中的“RP01”替换为已禁用子设备所属的 **父设备**的名称，如示例中所示。
> - 如果禁用的设备已包含“_STA”方法，请忽略示例的 *可能开始* 到 *可能结束* 标记之间的内容。
> - 此方法不会从 PCI 总线中释放设备。
