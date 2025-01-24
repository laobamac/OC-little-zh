# 禁用PCI串口适配器

在Hackintosh上，网络设置中可能会包含一个 `PCI串口适配器` 设备，这个设备对于网络连接并没有实际作用：

![PCI Serial](https://user-images.githubusercontent.com/76865553/179019541-b728d169-1675-4313-91a1-4288d6693ca1.png)

在IO注册表浏览器中搜索“serial”会发现它与 `Apple16X50.kext` 在 S/L/E 中相关：

![IOreg01](https://user-images.githubusercontent.com/76865553/178971557-01f0158d-7ab8-41e8-b3fe-5193e2058670.png)

从[源代码](https://github.com/apple-oss-distributions/Apple16X50Serial)来看，Apple16X50 似乎是一个(被)串(遗)口(忘)驱(的)动(史)，很可能是从PPC时代留(拉)下(出)来(来)的。它可能是为内部开发工具包中的串口或生产Mac中未安装的接口头保留下来的。看起来它只依赖于内核，似乎没有其他依赖关系。

## 操作步骤

遗憾的是，这个设备无法通过ACPI进行禁用，但你可以通过以下方法来阻止该kext的加载：

1. 从网络设置中删除PCI串口适配器条目。
2. 当系统问你是否要在下次连接时将设备添加回列表时，选择“是”(这样可以更方便地进行测试)。
3. 挂载你的EFI文件夹。
4. 打开你的 `config.plist` 文件。
5. 在 `Kernel/Block` 中添加以下规则：

	| 标识符 | 注释 | 启用 | 策略 | 架构 |
	|--------|------|:----:|:----:|:----:|
	| com.apple.driver.Apple16X50Serial | 阻止PCI串口适配器 | True | Disabled | Any |

6. 保存你的 `config.plist` 文件并重启。

## 测试

再次在IO注册表中搜索“serial”。`IOSerialBSDClient` 应该不见了：

![IOreg02](https://user-images.githubusercontent.com/76865553/178971604-4446dffe-27d4-4524-8734-0d1078f25d99.png)

同时，网络设置中的PCI串口适配器也会消失。

## 注意事项

- 我没有找到很多关于这个kext的信息，但 `IOSerialBSDClient` 似乎与[串口调制解调器检测](https://developer.apple.com/forums/thread/116061)有关。
- 如果禁用该kext后系统出现任何问题，可以重新启用它。到目前为止，我没有发现任何问题。
- 在Big Sur中，这个方法对不再有效，kext还是会被加载。但是在Monterey中又生效了，不排除是我的原因，难绷
