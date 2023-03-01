# 桌面 Coffee Lake

| 支持 | 版本 |
| :--- | :--- |
| 初始macOS支持 | macOS 10.13, High Sierra |

## 起点

制作一个config plist看起来很难，其实并不难。这只是需要一些时间，本指南将告诉您如何配置所有内容，您不会被冷落。这也意味着如果你有问题，检查你的配置设置以确保它们是正确的。OpenCore的主要注意事项:

* **所有属性必须定义**, 没有默认的OpenCore将出现错误，所以**不要删除节，除非明确告知**。如果指南没有提到该选项，请将其保留为默认值。
* **Sample.plist不能按原样使用**, 您必须配置您的系统
* **DO NOT USE CONFIGURATORS**, 这些配置器很少遵守OpenCore的规则，甚至一些像Mackie的配置器会添加Clover属性和破坏plists!

现在，我们来快速回顾一下我们需要的工具

* [ProperTree](https://github.com/corpnewt/ProperTree)
  * 通用plist编辑器
* [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)
  * 用于生成SMBIOS数据
* [Sample/config.plist](https://github.com/acidanthera/OpenCorePkg/releases)
  * 请参阅上一节有关如何获取: [config.plist 设置](../config.plist/README.md)

::: warning 警告

在设置OpenCore之前，请多次阅读本指南，并确保您已经正确设置了它。请注意，图像并不总是最新的，所以请阅读下面的文本，如果没有提到，则保留为默认值。

:::

## ACPI

![ACPI](../images/config/config.plist/coffeelake/acpi.png)

### Add

::: tip 信息

这是你将为系统添加ssdt的地方，这些对于**引导macOS**非常重要，并且有许多用途，例如 [USB 映射](https://sumingyd.github.io/OpenCore-Post-Install/usb/), [禁用不支持的gpu](../extras/spoof.md) 在我们的系统中，它甚至需要它才能引导**。制作指南可以在这里找到:[**开始使用ACPI**](https://sumingyd.github.io/Getting-Started-With-ACPI/)

我们需要几个ssdt来恢复Clover提供的功能:

| 需要SSDTs | 描述 |
| :--- | :--- |
| **[SSDT-PLUG](https://sumingyd.github.io/Getting-Started-With-ACPI/)** | 允许Haswell上的本地CPU电源管理，更新的版本，请参阅[ACPI入门指南](https://sumingyd.github.io/Getting-Started-With-ACPI/)了解更多详细信息。|
| **[SSDT-EC-USBX](https://sumingyd.github.io/Getting-Started-With-ACPI/)** | 修复嵌入式控制器和USB电源，参见[ACPI指南入门](https://sumingyd.github.io/Getting-Started-With-ACPI/) 了解更多详细信息。 |
| **[SSDT-AWAC](https://sumingyd.github.io/Getting-Started-With-ACPI/)** | 这是[300系列RTC补丁](https://www.hackintosh-forum.de/forum/thread/39846-asrock-z390-taichi-ultimate/?pageNo=2), 大多数B360, B365, H310, H370, Z390和一些Z370板需要，防止系统启动macOS。替代方案是 [SSDT-RTC0](https://sumingyd.github.io/Getting-Started-With-ACPI/) 当AWAC SSDT由于缺少遗留RTC时钟而不兼容时，要检查您是否需要它以及使用哪个，请参阅[ACPI入门](https://sumingyd.github.io/Getting-Started-With-ACPI/) 页面. |
| **[SSDT-PMC](https://sumingyd.github.io/Getting-Started-With-ACPI/)** | 所以300系列主板(非z370)不会在ACPI中声明FW芯片为MMIO，所以XNU忽略由UEFI内存映射声明的MMIO区域。这个SSDT恢复了对NVRAM的支持。详见 [ACPI入门指南](https://sumingyd.github.io/Getting-Started-With-ACPI/)|

请注意，您**不应该**在这里添加您生成的`DSDT.aml`，它已经在您的固件中了。因此，如果存在，请删除`config.plist`和EFI/OC/ACPI下的条目。

对于那些想要更深入地了解转储DSDT、如何制作这些ssdt并编译它们的人，请参阅 [**ACPI入门**](https://sumingyd.github.io/Getting-Started-With-ACPI/) **页面** 编译后的ssdt具有 **.aml** 扩展名(已编译)，将被保存在`EFI/OC/ACPI`文件夹中，并且**必须**在你的配置文件`ACPI -> Add`下指定。

:::

### Delete

这将阻止某些ACPI表加载，对于我们来说，我们可以忽略它。

### Patch

本节允许我们通过OpenCore动态修改ACPI的部分内容(DSDT、SSDT等)。对我们来说，我们的补丁由我们的ssdt处理。这是一个更简洁的解决方案，因为这将允许我们使用OpenCore引导Windows和其他操作系统

### Quirks

与ACPI相关的设置，将所有内容保留为默认设置，因为我们不需要这些选项。

## Booter

![Booter](../images/config/config-universal/hedt-booter.png)

本节专门讨论与使用OpenRuntime (AptioMemoryFix.efi的替代品)进行boot.efi补丁相关的问题

### MmioWhitelist

This section is allowing devices to be passthrough to macOS that are generally ignored, for us we can ignore this section.

### Quirks

::: tip 信息
与boot.efi补丁和固件修复相关的设置，对我们来说，我们需要更改以下内容:

| 选项 | 启用 | 注释 |
| :--- | :--- | :--- |
| DevirtualiseMmio | YES | |
| EnableWriteUnprotector | NO | |
| ProtectUefiServices | YES | Z390系统需要 |
| RebuildAppleMemoryMap | YES | |
| ResizeAppleGpuBars | -1 | 如果您的固件支持增加GPU条大小(即可缩放条支持)，请将其设置为 `0` |
| SyncRuntimePermissions | YES | |
:::

::: details 更深入信息

* **AvoidRuntimeDefrag**: YES
  * 修复了UEFI运行时服务，如日期，时间，NVRAM，电源控制等
* **DevirtualiseMmio**: YES
  * 减少被盗内存占用，扩展`slide=N`值的选项，并对修复Z390上的内存分配问题非常有帮助。需要 `ProtectUefiServices` 在 IceLake 和 Z390 Coffee Lake上.
* **EnableSafeModeSlide**: YES
  * 允许slide变量在安全模式下使用。
* **EnableWriteUnprotector**: NO
  * 这一选项和RebuildAppleMemoryMap通常会发生冲突，建议在较新的平台上启用后者并禁用此条目。
  * 然而，由于原始设备制造商没有使用最新的EDKII版本，您可能会发现上述组合将导致早期启动失败。这是由于缺少`MEMORY_ATTRIBUTE_TABLE`，因此我们建议禁用RebuildAppleMemoryMap并启用EnableWriteUnprotector。有关此问题的更多信息见[故障排除部分](/troubleshooting/extended/kernel-issues.md#stuck-on-eb-log-exitbs-start).
* **ProtectUefiServices**: NO
  * 保护UEFI服务不被固件覆盖，主要与vm、Icelake和Z390系统有关。
  * 如果在Z390上，**启用此选项**。
* **ProvideCustomSlide**: YES
  * 用于Slide变量计算。然而，这种选项的必要性取决于`OCABC: Only N/256 slide values are usable!` 调试日志中的消息。如果显示 `OCABC: All slides are usable! You can disable ProvideCustomSlide!` 您可以禁用`ProvideCustomSlide`.
* **RebuildAppleMemoryMap**: YES
  * 生成与macOS兼容的内存映射，可能会在一些笔记本电脑OEM固件上崩溃，所以如果您收到早期引导失败，禁用此功能。
* **ResizeAppleGpuBars**: -1
  * 启动macOS时，如果设置为`0`，将减少GPU PCI条的大小，设置为`-1`则禁用
  * 使用此选项可以设置其他PCI Bar值，但可能导致不稳定
  * 只有在固件中启用了对Resizable BAR的支持时，才需要将此属性设置为0。
* **SetupVirtualMap**: YES
  * 修复了SetVirtualAddresses对虚拟地址的调用，在Skylake和更新的版本上不需要。一些固件如Gigabyte可能仍然需要它，如果没有它，内核将会崩溃。
* **SyncRuntimePermissions**: YES
  * 修正了与MAT表的对齐，并要求使用MAT表启动Windows和Linux，也推荐用于macOS。主要适用于重建苹果内存地图用户。

:::

## DeviceProperties

![DeviceProperties](../images/config/config.plist/coffeelake/DeviceProperties.png)

### Add

从映射设置设备属性。

::: tip PciRoot(0x0)/Pci(0x2,0x0)

本节通过WhateverGreen的[Framebuffer补丁指南](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md) 设置，用于设置重要的iGPU属性。 **如果你有`-F`系列CPU，你可以忽略这一部分，因为你没有iGPU**

config.plist还没有这个部分，所以你必须手动创建它。

`AAPL,ig-platform-id` 是macOS用来确定iGPU驱动程序如何与我们的系统交互的，可以选择的两个值如下:

| AAPL,ig-platform-id | 注释 |
| :--- | :--- |
| **`07009B3E`** | 使用桌面iGPU驱动显示器时使用 |
| **`00009B3E`** | 如果不起作用，可以替换`07009B3E` |
| **`0300913E`** | 桌面iGPU仅用于计算任务而不驱动显示器时使用的 |

* **注意**:在macOS 10.15.5及更新版本中，使用`07009B3E`时黑屏似乎有很多问题，如果你遇到类似的问题，请尝试切换到`00009B3E`

我们还添加了另外两个属性，`framebuffer-patch-enable`和`framebuffer-stolenmem`。第一个选项允许通过WhateverGreen.kext打补丁，第二个选项将最小被盗内存设置为19MB。这通常是不必要的，因为可以在BIOS中配置(建议64MB)，但在没有配置时必须配置。

* **注意**:无头帧缓冲区(其中dGPU是显示出来的)不需要`framebuffer-patch-enable`和`framebuffer-stolenmem`

对于在B360、B365、H310、H370、Z390上啰嗦模式后出现黑屏问题的用户，请参见 [BusID iGPU 补丁](https://sumingyd.github.io/OpenCore-Post-Install/gpu-patching/intel-patching/busid.html) 页面

| Key | Type | Value |
| :--- | :--- | :--- |
| AAPL,ig-platform-id | Data | `07009B3E` |
| framebuffer-patch-enable | Data | `01000000` |
| framebuffer-stolenmem | Data | `00003001` |

(这是一个没有dGPU和BIOS选项没有iGPU内存的桌面UHD 630的例子)

:::

::: tip PciRoot(0x0)/Pci(0x1b,0x0)

`layout-id`

* 应用AppleALC音频注入，你需要自己研究你的主板有哪个编解码器，并将其与AppleALC的布局匹配。[AppleALC支持编解码器](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs).
* 你可以直接删除这个属性，因为目前它还没有被使用

对于我们来说，我们将使用引导参数`alcid=xxx`来完成此操作。`alcid`将覆盖所有其他布局id。更多信息请参见[安装后页面](https://sumingyd.github.io/OpenCore-Post-Install/)

:::

### Delete

从映射中删除设备属性，我们可以忽略它

## Kernel

![Kernel](../images/config/config-universal/kernel-modern-XCPM.png)

### Add

在这里，我们指定要加载哪些kext，以什么特定的顺序加载，以及每个kext适用于什么体系结构。默认情况下，我们建议保留ProperTree所做的操作，但对于32位cpu，请参见以下内容:

::: details 更深入信息

最主要的你需要记住的是:

* 装载顺序
  * 记住，任何插件都应该在它的依赖项之后加载
  * 这意味着像Lilu这样的kext **必须**出现在VirtualSMC、AppleALC、WhateverGreen等之前

提醒一下[ProperTree](https://github.com/corpnewt/ProperTree)用户可以运行**Cmd/Ctrl + Shift + R**以正确的顺序添加他们所有的kext，而无需手动输入每个kext。

* **Arch**
  * 该kext支持的体系结构
  * 目前支持的值是 `Any`, `i386` (32位), 和 `x86_64` (64位)
* **BundlePath**
  * kext的名称
  * 示例: `Lilu.kext`
* **Enabled**
  * 不言自明，启用或禁用kext
* **ExecutablePath**
  * 实际可执行文件的路径隐藏在kext中，你可以通过右键单击并选择`显示包内容`来查看你的kext的路径。一般来说，它们将是`Contents/MacOS/Kext`，但有些将Kext隐藏在`Plugin`文件夹下。请注意，plist中仅kext不需要填充该属性。
  * 示例: `Contents/MacOS/Lilu`
* **MinKernel**
  * kext将被注入到的最低内核版本，有关可能的值，请参见下表
  * 示例. `12.00.00` 用于 OS X 10.8
* **MaxKernel**
  * kext将被注入到的最高内核版本，可能的值见下表
  * 示例. `11.99.99` 用于 OS X 10.7
* **PlistPath**
  * 隐藏在kext中的`info.plist`的路径
  * 示例: `Contents/Info.plist`

::: details Kernel Support Table

| OS X 版本 | MinKernel | MaxKernel |
| :--- | :--- | :--- |
| 10.4 | 8.0.0 | 8.99.99 |
| 10.5 | 9.0.0 | 9.99.99 |
| 10.6 | 10.0.0 | 10.99.99 |
| 10.7 | 11.0.0 | 11.99.99 |
| 10.8 | 12.0.0 | 12.99.99 |
| 10.9 | 13.0.0 | 13.99.99 |
| 10.10 | 14.0.0 | 14.99.99 |
| 10.11 | 15.0.0 | 15.99.99 |
| 10.12 | 16.0.0 | 16.99.99 |
| 10.13 | 17.0.0 | 17.99.99 |
| 10.14 | 18.0.0 | 18.99.99 |
| 10.15 | 19.0.0 | 19.99.99 |
| 11 | 20.0.0 | 20.99.99 |
| 12 | 21.0.0 | 21.99.99 |
| 13 | 22.0.0 | 22.99.99 |

:::

### Emulate

用于欺骗不支持的cpu，如Pentiums和Celerons

* **Cpuid1Mask**: 不填写
* **Cpuid1Data**: 不填写

### Force

用于从系统卷中加载kext，只适用于某些特定的kext不在缓存中的旧操作系统。(例如 IONetworkingFamily 在 10.6).

对我们来说,我们可以忽略。

### Block

阻止某些kext加载。与我们无关。

### Patch

对内核和kext进行补丁。

### Quirks

::: tip 信息

与内核相关的设置，我们将启用以下功能:

| 选项 | 是否启用 | 注释 |
| :--- | :--- | :--- |
| AppleXcpmCfgLock | YES | 如果在BIOS中禁用了`CFG-Lock`，则不需要 |
| DisableIoMapper | YES | 如果在BIOS中禁用了`VT-D`，则不需要 |
| LapicKernelPanic | NO | 惠普的机器需要这个选项 |
| PanicNoKextDump | YES | |
| PowerTimeoutKernelPanic | YES | |
| XhciPortLimit | YES | 如果运行macOS 11.3+则禁用 |

:::

::: details 更深入信息

* **AppleCpuPmCfgLock**: NO
  * 仅当BIOS中不能禁用CFG-Lock时需要
  * 只适用于Ivy Bridge 及以上
    * 注意:Broadwell及更老版本在运行10.10或更老版本时需要此功能
* **AppleXcpmCfgLock**: YES
  * 仅当BIOS中不能禁用CFG-Lock时需要
  * 仅适用于Haswell和更新版本
    * 注意:Ivy Bridge-E也包括在内，因为它支持XCPM
* **CustomSMBIOSGuid**: NO
  * 为UpdateSMBIOSMode设置为`Custom`时执行GUID补丁。通常与戴尔笔记本电脑有关
  * 通过UpdateSMBIOSMode自定义模式启用此选项也可以禁用SMBIOS注入到“非苹果”操作系统中，但我们不支持这种方法，因为它破坏了Bootcamp兼容性。使用风险自负
* **DisableIoMapper**: YES
  * 如果在BIOS中无法禁用或其他操作系统需要禁用VT-D，则需要绕过VT-D，这是`dart=0`的更好替代方案，因为SIP可以在Catalina中继续运行
* **DisableLinkeditJettison**: YES
  * 允许Lilu和其他人在不需要`keepsyms=1`的情况下拥有更可靠的性能
* **DisableRtcChecksum**: NO
  * 防止AppleRTC写入主校验和(0x58-0x59)，这对于接收BIOS重置或在重启/关机后进入安全模式的用户是必需的
* **ExtendBTFeatureFlags** NO
  * 对于那些非apple /非fenvi卡有连续性问题的人很有帮助
* **LapicKernelPanic**: NO
  * 在AP core lapic中断上禁用内核恐慌，这通常是HP系统需要的。相当于Clover的 `Kernel LAPIC`
* **LegacyCommpage**: NO
  * 解决macOS中64位cpu的SSSE3需求，主要与64位Pentium 4 cpu(即Prescott)相关
* **PanicNoKextDump**: YES
  * 允许在发生内核严重故障时读取内核严重故障日志
* **PowerTimeoutKernelPanic**: YES
  * 帮助修复macOS Catalina中与Apple驱动程序有关的电源变化的内核恐慌，最显著的是数字音频。
* **SetApfsTrimTimeout**: `-1`
  * 为ssd上的APFS文件系统设置微调超时，以微秒为单位，仅适用于macOS 10.14和更新版本有问题的ssd。
* **XhciPortLimit**: YES
  * 这实际上是15端口限制补丁，不要依赖它，因为它不是修复USB的保证解决方案。如果可能，请创建一个[USB映射](https://sumingyd.github.io/OpenCore-Post-Install/usb/)。
  * macOS 11.3+， [XhciPortLimit可能无法按预期功能。](https://github.com/dortania/bugtracker/issues/162) 我们建议用户在升级前禁用此选项和映射或[从Windows映射](https://github.com/USBToolBox/tool). 您也可以安装macOS 11.2.3或更老版本。

:::

### Scheme

与传统引导(即 10.4-10.6)相关的设置，大多数情况下你可以跳过，但是对于那些计划引导传统操作系统的人，你可以看到以下内容:

::: details 更深入信息

* **FuzzyMatch**: True
  * 用于忽略kernelcache的校验和，而不是选择可用的最新缓存。可以帮助提高10.6中许多机器的引导性能
* **KernelArch**: x86_64
  * 设置内核的arch类型，你可以在`Auto`、`i386`(32位)和`x86_64`(64位)之间选择。
  * 如果你正在启动需要32位内核的旧操作系统(即ie 10.4和10.5)，我们建议将其设置为`Auto`，并让macOS根据你的SMBIOS来决定。支持的值见下表:
    * 10.4-10.5 — `x86_64`, `i386` 或 `i386-user32`
      * `i386-user32` 指向32位用户空间，因此32位cpu必须使用它(或缺少SSSE3的cpu)
      * `x86_64` 仍然使用32位内核空间，但是可以确保10.4/5的64位用户空间
    * 10.6 — `i386`, `i386-user32`, 或 `x86_64`
    * 10.7 — `i386` 或r `x86_64`
    * 10.8 或更新r — `x86_64`

* **KernelCache**: Auto
  * 设置内核缓存类型，主要用于调试，因此我们建议使用`Auto`以获得最佳支持

:::

## Misc

![Misc](../images/config/config-universal/misc.png)

### Boot

::: tip 信息

| 选项 | 是否启用 | 注释 |
| :--- | :--- | :--- |
| HideAuxiliary | YES | 按空格键显示macOS恢复等辅助项 |

:::

::: details 更深入信息

* **HideAuxiliary**: YES
  * 此选项将在选择器中隐藏补充条目，例如macOS recovery和tools。隐藏辅助条目可以提高多磁盘系统的启动性能。您可以在选择器处按空格键来显示这些条目

:::

### Debug

::: tip 信息

有助于调试OpenCore引导问题(除了`DisplayDelay`，我们将更改所有内容):

| 选项 | 是否启用 |
| :--- | :--- |
| AppleDebug | YES |
| ApplePanic | YES |
| DisableWatchDog | YES |
| Target | 67 |

:::

::: details 更多深入信息

* **AppleDebug**: YES
  * 启用boot.efi日志记录，用于调试。注意只有10.15.4及更高版本支持这个功能
* **ApplePanic**: YES
  * 试图将内核异常记录到磁盘
* **DisableWatchDog**: YES
  * 禁用UEFI watchdog，可以帮助解决早期引导问题
* **DisplayLevel**: `2147483650`
  * 显示更多的调试信息，需要调试版本的OpenCore
* **SysReport**: NO
  * 有助于调试，例如转储ACPI表
  * 注意，这仅限于调试版本的OpenCore
* **Target**: `67`
  * 显示更多调试信息，需要OpenCore的调试版本

这些值是基于[OpenCore调试](../troubleshooting/debug.md)中计算的值。

:::

### Security

::: tip 信息

安全性是非常不言自明的，**不要跳过**。我们将修改以下内容:

| 选项 | 是否启用 | 注释 |
| :--- | :--- | :--- |
| AllowSetDefault | YES | |
| BlacklistAppleUpdate | YES | |
| ScanPolicy | 0 | |
| SecureBootModel | Default | 将此设置为 `Default` ，以便OpenCore自动设置与您的SMBIOS对应的正确值。下一页将详细介绍这个设置。 |
| Vault | Optional | 这是一个单词，省略此设置是不行的。如果您没有将其设置为Optional，您将会后悔，注意它是区分大小写的 |

:::

::: details 更深入信息

* **AllowSetDefault**: YES
  * 允许`CTRL+Enter`和`CTRL+Index`在选择器中设置默认启动设备
* **ApECID**: 0
  * 用于网络个性化的安全启动标识符，由于macOS安装程序中的一个bug，目前这种方式是不可靠的，因此我们强烈建议您保留默认设置。
* **AuthRestart**: NO
  * 启用FileVault 2的认证重启，重启时不需要密码。可以被认为是可选的安全风险吗
* **BlacklistAppleUpdate**: YES
  * 用于阻止固件更新，作为额外的保护级别，因为macOS Big Sur不再使用`run-efi-updater`变量

* **DmgLoading**: Signed
  * 确保只加载经过签名的dmg
* **ExposeSensitiveData**: `6`
  * 显示更多调试信息，需要OpenCore的调试版本
* **Vault**: `Optional`
  * 我们不会处理vault，所以我们可以忽略它，**您将无法使用这个set来引导。**
  *这是一个单词，省略此设置是不可选的。如果你不把它设置为`Optional`，你会后悔的，注意它是区分大小写的
* **ScanPolicy**: `0`
  * `0`允许您查看所有可用的驱动器，请参阅[安全](https://sumingyd.github.io/OpenCore-Post-Install/universal/security.html) 部分了解更多详细信息。**如果设置为默认，将不会启动USB设备**
* **SecureBootModel**: Default
  * 在macOS中控制Apple的安全启动功能，请参阅[Security](https://sumingyd.github.io/OpenCore-Post-Install/universal/security.html) 部分了解更多细节。
  * 注意:用户可能会发现在已经安装的系统上升级OpenCore可能会导致早期引导失败。要解决这个问题，请参见这里: [卡在 OCB: LoadImage failed - Security Violation](/troubleshooting/extended/kernel-issues.md#stuck-on-ocb-loadimage-failed-security-violation)

:::

### Serial

用于串行调试(一切都保留默认值)。

### Tools

用于运行OC调试工具，如shell, ProperTree的snapshot函数将为您添加这些。

### Entries

用于指定无法通过OpenCore自然找到的不规则引导路径。

这里不会介绍更多信息，请参阅 [Configuration.pdf](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf) 的8.6

## NVRAM

![NVRAM](../images/config/config-universal/nvram.png)

### Add

::: tip 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14

用于OpenCore的UI缩放，default对我们有用。更多信息请参见深入部分

:::

::: details 更深入信息

启动器路径，主要用于UI修改

* **DefaultBackgroundColor**: boot.efi使用的背景色
  * `00000000`: 西拉黑
  * `BFBFBF00`: 浅灰色

:::

::: tip 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102

OpenCore的NVRAM GUID，主要与RTCMemoryFixup用户相关

:::

::: details 更深入信息

* **rtc-blacklist**: <>
  * 与RTCMemoryFixup一起使用，参见这里了解更多信息:[修复RTC写入问题](https://sumingyd.github.io/OpenCore-Post-Install/misc/rtc.html#finding-our-bad-rtc-region)
  * 大多数用户可以忽略此部分

:::

::: tip 7C436110-AB2A-4BBB-A880-FE41995C9F82

系统完整性保护位掩码

* **通用引导参数**:

| boot-args | 描述 |
| :--- | :--- |
| **-v** | 启用详细模式，在你启动时显示所有滚动的幕后文本，而不是苹果logo和进度条。它对任何使用黑苹果的人都是无价的，因为它让你了解引导过程的内部情况，并可以帮助你识别问题、问题kext等 |
| **debug=0x100** | 该命令禁用macOS的watchdog，该watchdog有助于防止在内核出现严重错误时重新启动。这样你就**有希望**收集到一些有用的信息并解决问题。 |
| **keepsyms=1** | 这是debug=0x100的伴随设置，它告诉操作系统在内核出现紧急情况时也打印符号。这可以提供一些更有用的见解，以了解造成崩溃本身的原因。 |
| **alcid=1** | 用于为AppleALC设置layout-id，请参阅[支持的编解码器](https://github.com/acidanthera/applealc/wiki/supported-codecs) 以确定适用于特定系统的布局。关于这方面的更多信息，请参阅[安装后页面](https://sumingyd.github.io/OpenCore-Post-Install/) |

* **特定于gpu的引导参数**:

| boot-args | 描述 |
| :--- | :--- |
| **agdpmod=pikera** | 用于在一些Navi gpu (RX 5000 & 6000系列)上禁用板ID检查，没有这个你会得到一个黑屏。**如果你没有Navi请不要使用**(即Polaris和Vega卡不应该使用这个) |
| **-radcodec** | 用于允许官方不支持的AMD gpu(欺骗)使用硬件视频编码器 |
| **radpg=15** | 用于禁用一些电源门控模式，有助于正确初始化基于AMD Cape Verde的gpu |
| **unfairgva=1** | 用于在支持的AMD gpu上修复硬件DRM支持 |
| **nvda_drv_vrl=1** | 用于在macOS Sierra和High Sierra的Maxwell和Pascal卡上启用NVIDIA的Web驱动程序 |
| **-wegnoegpu** | 用于禁用除集成的Intel iGPU之外的所有其他gpu，对于那些想运行新版本的macOS，而他们的dGPU不支持的人很有用 |

* **csr-active-config**: `00000000`
  * 设置“系统完整性保护”(SIP)。通常建议通过恢复分区使用`csrutil`进行更改。
  * 默认情况下，csr-active-config设置为`00000000`，以启用系统完整性保护。您可以选择许多不同的值，但总的来说，为了最佳安全实践，我们建议启用此选项。更多信息可以在我们的故障排除页面中找到:[禁用SIP](../troubleshooting/extended/post-issues.md#disabling-sip)

* **run-efi-updater**: `No`
  * 这用于防止苹果的固件更新包安装和破坏引导顺序;这很重要，因为这些固件更新(意味着mac)将无法工作。

* **prev-lang:kbd**: <>
  * 需要`lang-COUNTRY:keyboard`格式的非拉丁键盘，建议保持空白，尽管你可以指定它(**示例配置中默认是俄语**):
  * 中文: `zh-Hans:252`(十六进制为`7A682D48 616E733A 323532`)
  * 完整列表可在 [AppleKeyboardLayouts.txt](https://github.com/acidanthera/OpenCorePkg/blob/master/Utilities/AppleKeyboardLayouts/AppleKeyboardLayouts.txt)中找到
  * 提示:`prevr-lang:kbd`可以被转换成字符串，所以你可以直接输入`zh-Hans:252`，而不是转换成十六进制
  * 提示2:`prevr-lang:kbd`可以设置为一个空白变量(例如`<>`)，这将强制在首次启动时显示语言选择器。

| Key | Type | Value |
| :--- | :--- | :--- |
| prev-lang:kbd | String | zh-Hans:252 |

:::

### Delete

::: tip 信息

强制重写NVRAM变量，请注意，`Add` **不会覆盖** NVRAM中已经存在的值，所以像`boot-args`这样的值应该保持不变。对我们来说，我们将更改以下内容:

| 选项 | 启用 |
| :--- | :--- |
| WriteFlash | YES |

:::

::: details 更深入信息

* **LegacySchema**
  * 用于赋值NVRAM变量，与`OpenVariableRuntimeDxe.efi`一起使用。仅适用于没有原生NVRAM的系统

* **WriteFlash**: YES
  * 允许所有添加的变量写入闪存。

:::

## PlatformInfo

![PlatformInfo](../images/config/config.plist/coffeelake/smbios.png)

::: tip 信息

为了设置SMBIOS信息，我们将使用CorpNewt的[GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 应用程序。

在这个Coffee Lake的例子中，我们将选择iMac19,1 SMBIOS——这是为了兼容性而故意这么做的。Coffee Lake主要使用两种SMBIOS:

| SMBIOS | 硬件 |
| :--- | :--- |
| iMac19,1 | Mojave和更新的 |
| iMac18,3 | 用于High Sierra和更老的 |

* 当你有Pascal或Maxwell dGPU并且仅限于带有Web驱动的macOS版本时，你将使用18,3

运行GenSMBIOS，选择选项1下载MacSerial，选择选项3选择SMBIOS。这会给我们一个类似下面的输出:

```sh
  #######################################################
 #               iMac19,1 SMBIOS Info                  #
#######################################################

Type:         iMac19,1
Serial:       C02XG0FDH7JY
Board Serial: C02839303QXH69FJA
SmUUID:       DBB364D6-44B2-4A02-B922-AB4396F16DA8
```

`Type`部分复制到Generic -> SystemProductName。

`Serial`部分复制到Generic -> SystemSerialNumber。

`Board Serial` 部分复制到 Generic -> MLB.

`SmUUID` 部分复制到 Generic -> SystemUUID.

我们将Generic -> ROM设置为苹果ROM(从真正的Mac中转储)，你的网卡Mac地址，或任何随机的Mac地址(可以是6个随机字节，在本指南中我们将使用`11223300 0000`。安装后，请跟随[修复isservices](https://sumingyd.github.io/OpenCore-Post-Install/universal/iservices.html) 页面了解如何找到您的真实MAC地址)

**提醒你需要一个无效的串行!当你在[苹果的检查覆盖页面](https://checkcoverage.apple.com)中输入你的序列号时，你会得到一条信息，比如“无法检查此序列号的覆盖范围”。"**

**Automatic**: YES

* 基于Generic节而不是DataHub、NVRAM和SMBIOS节生成platformminfo

:::

### Generic

::: details 更多深入信息

* **AdviseFeatures**: NO
  * 当EFI分区不是Windows驱动器上的第一个分区时使用

* **MaxBIOSVersion**: NO
  * 将BIOS版本设置为Max，以避免Big Sur+的固件更新，主要适用于正版mac。

* **ProcessorType**: `0`
  * 设置为`0`用于自动类型检测，但是如果需要，这个值可以被覆盖。参见 [AppleSmBios.h](https://github.com/acidanthera/OpenCorePkg/blob/master/Include/Apple/IndustryStandard/AppleSmBios.h) 获取可能的值

* **SpoofVendor**: YES
  * 将供应商字段替换为Acidanthera，在大多数情况下使用苹果作为供应商通常不安全

* **SystemMemoryStatus**: Auto
  * 在SMBIOS信息中设置内存是否焊接，纯粹用于修饰，因此我们建议使用`Auto`

* **UpdateDataHub**: YES
  * 更新数据中心字段

* **UpdateNVRAM**: YES
  * 更新NVRAM字段

* **UpdateSMBIOS**: YES
  * 更新SMBIOS字段

* **UpdateSMBIOSMode**: Create
  * 用新分配的EfiReservedMemoryType替换表，在需要`CustomSMBIOSGuid`的Dell笔记本电脑上使用`Custom`
  * 设置为`Custom`并启用`CustomSMBIOSGuid`也可以禁用SMBIOS注入到“非apple”操作系统中，但是我们不支持这种方法，因为它破坏了Bootcamp的兼容性。使用风险自负

:::

## UEFI

![UEFI](../images/config/config-universal/aptio-v-uefi.png)

**ConnectDrivers**: YES

* 强制 .efi 驱动程序，更改为NO将自动连接添加的UEFI驱动程序。这可以使启动稍微快一些，但不是所有的驱动程序都自己连接。例如某些文件系统驱动程序不能加载。

### Drivers

在这里添加你的.efi驱动程序。

在这里的驱动必须有:

* HfsPlus.efi
* OpenRuntime.efi

::: details 更深入信息

| Key | Type | 描述 |
| :--- | :--- | :--- |
| Path | String | `OC/Drivers` 目录下文件的路径 |
| LoadEarly | Boolean | 在NVRAM设置之前加载驱动程序，如果使用传统NVRAM，应该只启用`OpenRuntime.efi`和`OpenVariableRuntimeDxe.efi` |
| Arguments | String | 有些驱动程序接受这里指定的其他参数。 |

:::

### APFS

默认情况下，OpenCore只从macOS Big Sur及更新版本加载APFS驱动程序。如果你启动的是macOS Catalina或更早的版本，你可能需要设置一个新的最低版本/日期。
不设置这个会导致OpenCore找不到你的macOS分区!

macOS Sierra和更早的版本使用HFS而不是APFS。如果要启动旧版本的macOS，可以跳过本节。

::: tip APFS 版本

如果改变了最小版本，则需要设置MinVersion和MinDate。

| macOS 版本 | Min Version | Min Date |
| :------------ | :---------- | :------- |
| High Sierra (`10.13.6`) | `748077008000000` | `20180621` |
| Mojave (`10.14.6`) | `945275007000000` | `20190820` |
| Catalina (`10.15.4`) | `1412101001000000` | `20200306` |
| No restriction | `-1` | `-1` |

:::

### Audio

与AudioDxe设置相关，我们将忽略它(保持默认设置)。这与macOS中的音频支持无关。

* 进一步使用AudioDxe和音频部分，请参见安装后页面: [添加GUI和启动铃声](https://sumingyd.github.io/OpenCore-Post-Install/)

### Input

与用于FileVault和热键支持的boot.efi键盘传递相关，将所有内容保留为默认值，因为我们不需要这些选项。更多详细信息:[Security and FileVault](https://sumingyd.github.io/OpenCore-Post-Install/)

### Output

关于OpenCore的视觉输出，将所有内容保留为默认值，因为我们不需要这些选项。

::: details 更深入信息

| Output | Value | 注释 |
| :--- | :--- | :--- |
| UIScale | `0` |`0`将根据分辨率自动设置<br/>`-1`将保持它不变<br/>`1`表示1倍缩放，用于正常显示<br/>`2`表示2倍缩放，对于HiDPI显示 |

:::

### ProtocolOverrides

主要针对虚拟机、传统mac和FileVault用户。更多详细信息:[Security and FileVault](https://sumingyd.github.io/OpenCore-Post-Install/)

### Quirks

::: tip 信息
关于UEFI环境的选项，对于我们来说，我们将改变以下内容:

| Quirk | Enabled | 注释 |
| :--- | :--- | :--- |
| UnblockFsConnect | NO | 主要用于惠普主板 |

:::

::: details 更深入信息

* **DisableSecurityPolicy**: NO
  * 禁用固件中的平台安全策略，建议用于有bug的固件，其中禁用安全引导不允许第三方固件驱动加载。
  * 如果运行微软Surface设备，建议启用此选项

* **RequestBootVarRouting**: YES
  * 将AptioMemoryFix从`EFI_GLOBAL_VARIABLE_GUID`重定向到`OC_VENDOR_VARIABLE_GUID`。当固件试图删除启动项时需要启用，建议在所有系统上启用，以确保正确的更新安装，启动磁盘控制面板的功能等。

* **UnblockFsConnect**: NO
  * 一些固件块分区处理通过驱动模式打开它们，这导致文件系统协议无法安装。主要适用于没有列出驱动器的HP系统

:::

### ReservedMemory

用于将某些内存区域从操作系统中免除使用，主要与Sandy Bridge igpu或具有错误内存的系统相关。在本指南中没有涉及这种选项的使用

## 清理

现在你可以保存它，并将其放入EFI/OC下的EFI中。

对于那些有启动问题的人，请确保首先阅读[故障诊断部分](../troubleshooting/troubleshooting.md) 如果您的问题仍然没有答案，我们有大量的资源供您使用:

* [r/Hackintosh Subreddit](https://www.reddit.com/r/hackintosh/)
* [r/Hackintosh Discord](https://discord.gg/2QYd7ZT)

## Intel BIOS 设置

* 注意:大多数选项可能不会出现在您的固件中，我们建议尽可能匹配，但如果这些选项在您的BIOS中不可用，也不要太担心

### 禁用

* 快速启动（Fast Boot）
* 安全引导（Secure Boot）
* 串口/COM端口（Serial/COM Port）
* 并口（Parallel Port）
* VT-d (如果将`DisableIoMapper`设置为YES，则可以启用)
* 兼容性支持模块 (CSM) (**在大多数情况下必须关闭，当该选项启用时，像`gIO`这样的GPU错误/停顿很常见**)
* Thunderbolt (用于初始安装，因为如果安装不正确，Thunderbolt可能会导致问题)
* Intel SGX
* Intel Platform Trust
* CFG Lock (MSR 0xE2写保护)(**此选项必须关闭，如果你找不到该选项，则在Kernel -> Quirks下启用`AppleXcpmCfgLock`。你的黑苹果将无法在启用CFG-Lock的情况下启动**)

### 启用

* VT-x
* 4G以上解码（Above 4G Decoding）
* 超线程（Hyper-Threading）
* 执行禁用位（Execute Disable Bit）
* EHCI/XHCI切换（EHCI/XHCI Hand-off）
* 操作系统类型:Windows 8.1/10 UEFI模式(一些主板可能需要”其他操作系统”代替)
* DVMT预分配（DVMT Pre-Allocated）(iGPU内存): 64MB或更高
* SATA模式:AHCI

# 完成后，我们需要编辑一些额外的值。转到 [苹果安全引导页](../config.plist/security.md)
