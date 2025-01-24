# 显卡

关于GPU和Intel iGPU的相关指南和修复，涵盖了OpenCore安装指南中没有涉及到的部分。

**目录**

- [**macOS中的GPU支持**](/11_Graphics/GPU/README.md)
	- [启用AMD (Big) Navi显卡](/11_Graphics/GPU/AMD_Navi/README.md)
	- [启用AMD Vega 56/64显卡](/11_Graphics/GPU/AMD_Vega/README.md)
	- [AMD Radeon调优](/11_Graphics/GPU/AMD_Radeon_Tweaks/README.md)
	- [老旧AMD显卡与macOS](https://web.archive.org/web/20170814210930/http://www.rampagedev.com/guides/graphic-cards-injection/) (ATI 4000到7000系列和AMD 200/300系列)
	- [禁用不支持的GPU](/11_Graphics/GPU/Disabling_unsupported_GPUs/README.md)
	- [禁用HDMI/DisplayPort的音频功能](/11_Graphics/GPU/Disabling_AppleGFXHDA/README.md)
	- [启用未检测到的AMD GPU](/11_Graphics/GPU/GPU_undetected/README.md)
	- [OpenCore中的GPU BAR大小设置](/11_Graphics/GPU/GPU-BAR_Size/README.md)

- [**AMD/Intel iGPU修复**](/11_Graphics/iGPU/README.md)
	- [启用AMD Vega iGPU](/11_Graphics/iGPU/AMD/README.md) 
	- [iGPU帧缓存列表](/11_Graphics/iGPU/iGPU_DeviceProperties.md)
	- [在macOS Ventura中启用Skylake显卡](/11_Graphics/iGPU/Skylake_Spoofing_macOS13/README.md)
	- [通过伪造ig-platform-id强制启用显卡](/11_Graphics/iGPU/Fake_ig-platform-id.md)
	- [在macOS Ventura中启用Metal 3和GPU选项卡](/11_Graphics/Metal_3/README.md)
	- [Comet Lake和Z500系列主板iGPU问题](/11_Graphics/iGPU/Cometlake_Z590/README.md)  
	- [注入EDID修复显示问题](/11_Graphics/Inject_EDID/README.md)
	- [禁用HDMI/DisplayPort的iGPU音频功能](/11_Graphics/iGPU/iGPU_disable_audio_over_HDMI-DP.md)
