# 笔记本 Broadwell

| 支持 | 版本 |
| :--- | :--- |
| 初始macOS支持 | OS X 10.10, Yosemite |
| 最后支持的操作系统 | macOS 12 Monterey |
| 注意 | 有关Ventura的信息，请参阅 [macOS 13 Ventura](../extras/ventura.md#dropped-cpu-support) |

## 起点

做一个config.Plist可能看起来很难，其实不然。这只是需要一些时间，本指南将告诉您如何配置所有内容，您不会被冷落。这也意味着如果你有问题，检查你的配置设置以确保它们是正确的。使用OpenCore需要注意的主要事项:

* **所有的属性必须定义**, 没有默认的OpenCore将出错，所以**不要删除任何节，除非明确地告诉你可以删除**。如果指南没有提到该选项，请将其保留为默认值。
* **Sample.plist不能按原样使用**, 你必须根据自己的系统进行配置
* **不要使用配置器**, 这些配置器很少遵守OpenCore的配置，甚至一些像Mackie的配置器还会添加Clover属性和破坏plists!

现在，我们来快速回顾一下我们需要的工具

* [ProperTree](https://github.com/corpnewt/ProperTree)
  * 通用plist编辑器
* [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)
  * 用于生成SMBIOS数据
* [Sample/config.plist](https://github.com/acidanthera/OpenCorePkg/releases)
  * 参见前一节获取方法: [config.plist 设置](../config.plist/README.md)

::: warning 注意

在设置OpenCore之前，请多次阅读本指南，并确保设置正确。请注意，图片并不总是最新的，所以请阅读下面的文字，如果没有提到，那么保持默认。

:::

## ACPI

![ACPI](../images/config/config-laptop.plist/broadwell/acpi.png)

### Add

::: tip 信息

这是你将为系统添加ssdt的地方，这些对于**引导macOS**非常重要，并且有许多用途，例如[USB 映射](https://sumingyd.github.io/OpenCore-Post-Install/usb/)，[禁用不支持的gpu](../extras/spoof.md)等。在我们的系统中，**甚至需要这些才可以启动**。制作指南:[**ACPI入门**](https://sumingyd.github.io/Getting-Started-With-ACPI/)

对于我们来说，我们需要一些ssdt来带回Clover提供的功能:

| 需要的SSDTs | 描述 |
| :--- | :--- |
| **[SSDT-PLUG](https://sumingyd.github.io/Getting-Started-With-ACPI/)** | 在Haswell和更新的版本上允许本地CPU电源管理，请参阅[ACPI入门指南](https://sumingyd.github.io/Getting-Started-With-ACPI/) 了解更多细节。 |
| **[SSDT-EC](https://sumingyd.github.io/Getting-Started-With-ACPI/)** | 修复嵌入式控制器，参见 [开始使用ACPI指南](https://sumingyd.github.io/Getting-Started-With-ACPI/) 了解更多详细信息。 |
| **[SSDT-GPIO](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/decompiled/SSDT-GPI0.dsl)** | 创建一个存根以便VoodooI2C可以连接，对于那些有问题的VoodooI2C工作可以尝试 [SSDT-XOSI](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-XOSI.aml) 代替。请注意，英特尔NUCs不需要这个 |
| **[SSDT-PNLF](https://sumingyd.github.io/Getting-Started-With-ACPI/)** | 修复亮度控制，请参阅[ACPI指南入门](https://sumingyd.github.io/Getting-Started-With-ACPI/) 了解更多详细信息。请注意，英特尔NUCs不需要这个 |

请注意，您**不应该**在这里添加您生成的`DSDT.aml`，它已经在您的固件中了。因此，如果存在，请删除`config plist`和EFI/OC/ACPI下的条目。

对于那些想要更深入地转储您的DSDT、如何制作这些ssdt以及编译它们的人，请参阅[**开始使用ACPI**](https://sumingyd.github.io/Getting-Started-With-ACPI/) **页面** 编译后的ssdt有一个**.aml**扩展名(已编译)，将被放入`EFI/OC/ACPI`文件夹，并且**必须**在你的配置文件`ACPI -> Add`下指定。

:::

### Delete

这将阻止某些ACPI表加载，对于我们来说，我们可以忽略它。

### Patch

::: tip 信息

本节允许我们通过OpenCore动态修改ACPI的部分内容(DSDT、SSDT等)。对我们来说，我们需要以下内容:

* OSI 重命名
  * 这在使用SSDT-xosi时是必需的，因为我们会将所有的OSI调用重定向到这个SSDT，**如果你使用SSDT-gpio则不需要**

| Comment | String | Change _OSI to XOSI |
| :--- | :--- | :--- |
| Enabled | Boolean | YES |
| Count | Number | 0 |
| Limit | Number | 0 |
| Find | Data | `5f4f5349` |
| Replace | Data | `584f5349` |

:::

### Quirks

与ACPI相关的设置，将所有内容保留为默认设置，因为我们不需要这些选项。

## Booter

![Booter](../images/config/config-universal/aptio-iv-booter.png)

本节专门讨论与使用OpenRuntime (AptioMemoryFix.efi的替代品)进行boot.efi补丁相关的问题

### MmioWhitelist

本节允许将通常被忽略的空格传递给macOS，当与`DevirtualiseMmio`配对时很有用。

### Quirks

::: tip 信息
与boot.efi补丁和固件修复相关的设置，我们将其保留为默认设置
:::
::: details 更深入的信息

* **AvoidRuntimeDefrag**: YES
  * 修复UEFI运行时服务，如日期，时间，NVRAM，电源控制等
* **EnableSafeModeSlide**: YES
  * 允许slide变量在安全模式下使用。
* **EnableWriteUnprotector**: YES
  * 需要从CR0寄存器移除写保护。
* **ProvideCustomSlide**: YES
  * 用于Slide变量计算。然而，这种怪异的必要性取决于 `OCABC: Only N/256 slide values are usable!` 调试日志中的消息。如果显示 `OCABC: All slides are usable! You can disable ProvideCustomSlide!` 在你的日志中，你可以禁用`ProvideCustomSlide`.
* **SetupVirtualMap**: YES
  * 修复了SetVirtualAddresses对虚拟地址的调用, 用于Gigabyte主板，以解决早期的内核崩溃
  
:::

## DeviceProperties

![DeviceProperties](../images/config/config-laptop.plist/broadwell/DeviceProperties.png)

### Add

从映射中设置设备属性。

::: tip PciRoot(0x0)/Pci(0x2,0x0)

本节是通过WhateverGreen的[Framebuffer补丁指南](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md) 建立的，用于设置重要的iGPU属性。

config.plist还没有这个部分，所以你必须手动创建它。

在设置iGPU时，下表应该有助于找到要设置的正确值。下面是一些值的解释:

* **AAPL,ig-platform-id**
  * 这是用于内部设置iGPU
* **Type**
  * 该条目是否建议用于笔记本电脑(即。内置显示器)或英特尔NUCs(即。独立盒子)

在设置iGPU属性时，通常遵循以下步骤。如果他们说的有任何不同，请遵循下表的配置说明:

1. 在最初设置config.plist时，只设置AAPL,ig-platform-id——这通常就足够了
2. 如果你启动后没有得到图形加速(7MB 显存和不透明的dock背景)，那么你可能需要尝试不同的`AAPL, g-platform-id`值，添加stolenmem补丁，甚至添加`device-id`属性。

| AAPL,ig-platform-id | Type | Comment |
| ------------------- | ---- | ------- |
| **`06002616`** | Laptop | Broadwell笔记本电脑推荐值 |
| **`02001616`** | NUC | Broadwell NUCs的推荐值 |

#### Configuration Notes

* 对于HD 5600，你需要将`device-id`伪造为`26160000`:

| Key | Type | Value |
| :--- | :--- | :--- |
| device-id | data | 26160000

* 在某些情况下，你不能在UEFI设置中将这些卡的DVMT-prealloc设置到96MB以上，你可能会遇到内核崩溃。通常它们被配置为32MB的DVMT-prealloc，在这种情况下，这些值被添加到iGPU属性中

| Key | Type | Value |
| :--- | :--- | :--- |
| framebuffer-patch-enable | Data | `01000000` |
| framebuffer-stolenmem | Data | `00003001` |
| framebuffer-fbmem | Data | `00009000` |

:::

::: tip PciRoot(0x0)/Pci(0x1b,0x0)

`layout-id`

* 应用AppleALC音频注入，你需要自己研究你的主板有哪个编解码器，并将其与AppleALC的布局匹配。[AppleALC支持编解码器](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs).
* 你可以直接删除这个属性，因为目前它还没有被我们使用

对于我们来说，我们将使用引导参数`alcid=xxx`来完成这一点。`alcid`将覆盖所有其他布局id。更多信息在[安装后页面](https://sumingyd.github.io/OpenCore-Post-Install/)

:::

### Delete

从映射中删除设备属性，我们可以忽略它

## Kernel

![Kernel](../images/config/config-universal/kernel-legacy-XCPM.png)

### Add

在这里，我们指定要加载哪些kext，以什么特定的顺序加载，以及每个kext适用于什么体系结构。默认情况下，我们建议保留ProperTree所做的操作，但对于32位cpu，请参见以下内容:

::: details 更深入的信息

你需要记住的主要事情是:

* 装载顺序
  * 记住，任何插件都应该在它的依赖**后面**加载
  * 这意味着像Lilu这样的kext **必须**出现在VirtualSMC、AppleALC、WhateverGreen等之前

提醒一下 [ProperTree](https://github.com/corpnewt/ProperTree) 用户可以运行 **Cmd/Ctrl + Shift + R** 以正确的顺序添加他们所有的kext，而无需手动输入每个kext。

* **Arch**
  * 这个kext支持的架构
  * 目前支持的值是 `Any`, `i386` (32位), 和`x86_64` (64位)
* **BundlePath**
  * kext的名称
  * 例如: `Lilu.kext`
* **Enabled**
  * 这里想必就不用多做解释了，启用或禁用kext
* **ExecutablePath**
  * 实际可执行文件的路径隐藏在kext中，您可以通过右键单击并选择`显示包内容`来查看kext的路径。一般来说，它们将是`Contents/MacOS/Kext`，但有些将Kext隐藏在`Plugin`文件夹下。请注意，kext中仅plist时不需要填充该属性。
  * 例如: `Contents/MacOS/Lilu`
* **MinKernel**
  * kext将被注入到的最低内核版本，有关可能的值，请参见下表
  * 例如. `12.00.00` 用于 OS X 10.8
* **MaxKernel**
  * kext将被注入到的最高内核版本，可能的值见下表
  * 例如. `11.99.99` 用于 OS X 10.7
* **PlistPath**
  * 隐藏在kext中的info.plist的路径
  * 例如: `Contents/Info.plist`
  
::: details 内核支持表

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
| 14 | 23.0.0 | 23.99.99 |
| 15 | 24.0.0 | 24.99.99 |

:::

### Emulate

用于欺骗不支持的cpu，如Pentiums和Celerons

* **Cpuid1Mask**: 不填写
* **Cpuid1Data**: 不填写

### Force

用于从系统卷中加载kext，仅适用于某些kext不存在于缓存中的旧操作系统(例如10.16中的IONetworkingFamily)。

对我们来说，我们可以忽略。

### Block

阻止某些kext的加载。与我们无关。

### Patch

为内核和kext打补丁。

### Quirks

::: tip 信息

与内核相关的设置，对我们来说，我们将更改以下内容：

| 选项 | 启用 | 说明 |
| :--- | :--- | :--- |
| AppleCpuPmCfgLock | NO | 如果运行10.10或更高版本，并且不能在BIOS中禁用`CFG-Lock`，则需要 |
| AppleXcpmCfgLock | YES | 如果在BIOS中禁用了`CFG-Lock`，是不需要的 |
| DisableIoMapper | YES | 如果在BIOS中禁用了`VT-D`，是不需要的 |
| LapicKernelPanic | NO | 惠普的机器需要这个选项 |
| PanicNoKextDump | YES | |
| PowerTimeoutKernelPanic | YES | |
| XhciPortLimit | YES | 如果运行macOS 11.3+，请禁用 |

:::

::: details 更深入的信息

* **AppleCpuPmCfgLock**: NO
  * 仅当BIOS中不能禁用CFG-Lock时需要
  * 仅适用于 Ivy Bridge 和更旧的
    * 注意:Broadwell及更老版本在运行10.10或更老版本时需要此功能
* **AppleXcpmCfgLock**: YES
  * 仅当BIOS中不能禁用CFG-Lock时需要
  * 仅适用于Haswell和更新版本
    * 注意:Ivy Bridge-E也包括在内，因为它支持XCPM
* **CustomSMBIOSGuid**: NO
  * 为UpdateSMBIOSMode设置为`Custom`时执行GUID补丁。通常与戴尔笔记本电脑有关
  * 通过UpdateSMBIOSMode自定义模式启用此选项也可以禁用SMBIOS注入到“非苹果”操作系统中，但我们不支持这种方法，因为它破坏了Bootcamp兼容性。使用风险自负
* **DisableIoMapper**: YES
  * 如果在BIOS中无法禁用或其他操作系统需要启用VT-D，则需要绕过VT-D，这是`dart=0`的更好替代方案，因为SIP可以在Catalina中继续运行
* **DisableIoMapperMapping**: NO
  * 如果您在使用Wi-Fi/以太网/Thunderbolt时遇到问题，启用了VT-D并且安装了超过16GB的内存，则需要使用。这个怪癖需要一个[打过补丁的DMAR表](https://dortania.github.io/Getting-Started-With-ACPI/Universal/dmar.html)，删除了保留内存区域
  * 我们建议禁用VT-D，所以我们不需要启用它
  * 13.2.1及以下版本不需要
* **DisableLinkeditJettison**: YES
  * 允许Lilu和其他kext在不需要`keepsyms=1`的情况下拥有更可靠的性能
* **DisableRtcChecksum**: NO
  * 防止AppleRTC写入主校验和(0x58-0x59)，这对于接收BIOS重置或在重启/关机后进入安全模式的用户是必需的
* **ExtendBTFeatureFlags** NO
  * 对于那些非apple /非fenvi卡有连续性问题的人很有帮助
* **LapicKernelPanic**: NO
  * 在AP核心lapic中断上禁用内核崩溃，一般HP系统需要。相当于Clover的`Kernel LAPIC`
* **LegacyCommpage**: NO
  * 解决了macOS中64位cpu的SSSE3要求，主要适用于64位Pentium 4 cpu (即Prescott)
* **PanicNoKextDump**: YES
  * 允许在发生内核严重故障时读取内核严重故障日志
* **PowerTimeoutKernelPanic**: YES
  * 帮助修复macOS Catalina中与苹果驱动程序权限变化相关的内核崩溃，尤其是数字音频。
* **SetApfsTrimTimeout**: `-1`
  * 为ssd上的APFS文件系统设置以微秒为单位的修剪超时时间，仅适用于macOS 10.14及更新版本的有问题的ssd。
* **XhciPortLimit**: YES
  * 这是15个端口限制补丁，不要依赖它，因为它不是一个保证修复USB接口的解决方案。如果可能，请创建一个[USB映射](https://sumingyd.github.io/OpenCore-Post-Install/usb/)。
  * 在macOS 11.3+中，[XhciPortLimit可能无法正常工作。](https://github.com/dortania/bugtracker/issues/162) 我们建议用户在升级前禁用此功能和 映射或 [从Windows映射](https://github.com/USBToolBox/tool). 你也可以安装macOS 11.2.3或更旧的版本。

原因是UsbInjectAll重新实现了内置的macOS功能，而没有适当的当前调优。只在一个plist-only kext中描述端口要简洁得多，这样就不会浪费运行时内存等

:::

### Scheme

与传统引导相关的设置(例如:10.4-10.6)，大多数情况下你可以跳过，但是对于那些计划启动旧操作系统的人，你可以查看以下内容:

::: details 更深入的信息

* **FuzzyMatch**: True
  * 用于忽略内核缓存的校验和，而不是选择可用的最新缓存。可以帮助提高10.6中许多机器的启动性能
* **KernelArch**: x86_64
  * 设置内核的arch类型，你可以在`Auto`、`i386`(32位)和`x86_64`(64位)之间选择。
  * 如果你正在启动需要32位内核的旧操作系统(例如:10.4和10.5)我们建议将其设置为`Auto`，并让macOS根据您的SMBIOS决定。支持的值见下表:
    * 10.4-10.5 — `x86_64`, `i386` 或 `i386-user32`
      * `i386-user32` 是32位的用户空间，所以32位的cpu必须使用这个(或cpu缺少SSSE3)
      * `x86_64` 仍然有一个32位的内核空间，但是在10.4/5中将确保64位的用户空间
    * 10.6 — `i386`, `i386-user32`, 或 `x86_64`
    * 10.7 — `i386` 或 `x86_64`
    * 10.8 或更新的 — `x86_64`

* **KernelCache**: Auto
  * 设置内核缓存类型，主要用于调试，因此我们建议使用`Auto`以获得最佳支持

:::

## Misc

![Misc](../images/config/config-universal/misc.png)

### Boot

::: tip 信息

| 选项 | 启用 | 说明 |
| :--- | :--- | :--- |
| HideAuxiliary | YES | 按空格键显示macOS recovery和其他辅助条目 |

:::

::: details 更深入的信息

* **HideAuxiliary**: YES
  * 此选项将在选择器中隐藏补充条目，例如macOS恢复和工具。隐藏辅助条目可以提高多磁盘系统的启动性能。您可以在选择器处按空格键来显示这些条目

:::

### Debug

::: tip 信息

有助于调试OpenCore引导问题(除了`DisplayDelay`，我们将更改所有内容):

| 选项 | 启用 |
| :--- | :--- |
| AppleDebug | YES |
| ApplePanic | YES |
| DisableWatchDog | YES |
| Target | 67 |

:::

::: details 更深入的信息

* **AppleDebug**: YES
  * 启用boot.efi的日志记录，用于调试。注意只有10.15.4及更高版本支持这个功能
* **ApplePanic**: YES
  * 尝试将内核崩溃记录到磁盘
* **DisableWatchDog**: YES
  * 禁用UEFI watchdog，可以帮助解决早期引导问题
* **DisplayLevel**: `2147483650`
  * 显示更多的调试信息，需要调试版本的OpenCore
* **SysReport**: NO
  * 有助于调试，例如转储ACPI表
  * 注意，这仅限于调试版本的OpenCore
* **Target**: `67`
  * 显示更多的调试信息，需要OpenCore的调试版本

这些值是基于 [OpenCore 调试](../troubleshooting/debug.md)中计算的值。

:::

### Security

::: tip 信息

安全性不用多说了吧，**不要跳过**。我们将修改以下内容:

| 选项 | 启用 | 说明 |
| :--- | :--- | :--- |
| AllowSetDefault | YES | |
| BlacklistAppleUpdate | YES | |
| ScanPolicy | 0 | |
| SecureBootModel | Default | 将此设置为`Default`，以便OpenCore自动设置与您的SMBIOS对应的正确值。下一页将详细介绍这个设置。 |
| Vault | Optional | 如果你不把它设置为Optional，你会后悔的，注意它是区分大小写的 |

:::

::: details 更深入的信息

* **AllowSetDefault**: YES
  * 允许按 `CTRL+Enter` 和`CTRL+Index` 在选择器中设置默认启动条目
* **ApECID**: 0
  * 用于网络个性化的安全启动标识符，由于macOS安装程序中的一个bug，目前这种方式是不可靠的，因此我们强烈建议您保留默认设置。
* **AuthRestart**: NO
  * 为FileVault 2启用身份认证重启，因此在重启时不需要密码。可以被认为是一个可选的安全风险
* **BlacklistAppleUpdate**: YES
  * 用于阻止固件更新，作为额外的保护级别，因为macOS Big Sur不再使用`run-efi-updater`变量

* **DmgLoading**: Signed
  * 确保只加载经过签名的dmg
* **ExposeSensitiveData**: `6`
  * 显示更多的调试信息，需要OpenCore的调试版本
* **Vault**: `Optional`
  * 我们不会处理这个，所以我们可以忽略，**你不会启动这个设置，因为安全**
  * 这是一个单词，省略此设置是不可以的。如果你不把它设置为`Optional`，你会后悔的，注意它是区分大小写的
* **ScanPolicy**: `0`
  * `0`允许您查看所有可用的驱动器，请参阅[安全](https://sumingyd.github.io/OpenCore-Post-Install/universal/security.html)部分了解更多详细信息。**如果设置为默认，将不会启动USB设备**
* **SecureBootModel**: Default
  * 在macOS中控制Apple的安全启动功能，请参阅[安全](https://sumingyd.github.io/OpenCore-Post-Install/universal/security.html) 部分了解更多细节。
  * 注意:用户可能会发现在已经安装的系统上升级OpenCore可能会导致早期引导失败。要解决这个问题，请参见这里:[卡在 OCB: LoadImage failed - Security Violation](/troubleshooting/extended/kernel-issues.md#stuck-on-ocb-loadimage-failed-security-violation)

:::

### Serial

用于串行调试(保持默认值)。

### Tools

用于运行OC调试工具，如shell, ProperTree的快照功能将为您添加这些。

### Entries

用于指定当OpenCore无法自然找到的不规则的引导路径。

这里不会介绍，更多信息请参见 8.6的[Configuration.pdf](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf)

## NVRAM

![NVRAM](../images/config/config-universal/nvram.png)

### Add

::: tip 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14

用于OpenCore的UI缩放，默认就可以很好的工作。有关更多信息，请参阅深入部分

:::

::: details 更深入的信息

引导路径，主要用于修改UI

* **DefaultBackgroundColor**: boot.efi使用的背景色
  * `00000000`: 西拉黑
  * `BFBFBF00`: 浅灰色

:::

::: tip 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102

OpenCore的NVRAM GUID，主要针对RTC内存修复用户

:::

::: details 更深入的信息

* **rtc-blacklist**: <>
  * 要与RTCMemoryFixup一起使用，请参阅这里以获取更多信息:[修复RTC写入问题](https://sumingyd.github.io/OpenCore-Post-Install/misc/rtc.html#finding-our-bad-rtc-region)
  * 大多数用户可以忽略这一节

:::

::: tip 7C436110-AB2A-4BBB-A880-FE41995C9F82

系统完整性保护位掩码

* **通用引导参数**:

| 引导参数 | 说明 |
| :--- | :--- |
| **-v** | 这将启用详细模式，在你启动时显示所有滚动的幕后文本，而不是苹果logo和进度条。它对任何黑苹果用户来说都是无价的，因为它可以让你深入了解引导过程，并可以帮助你识别问题、问题kext等。 |
| **debug=0x100** | 这禁用了macOS的watchdog，它有助于防止在内核出现严重错误时重新启动。这样你就**有希望**收集到一些有用的信息并按照提示来解决问题。 |
| **keepsyms=1** | 这是debug=0x100的配套设置，它告诉操作系统在内核出现故障时也打印这些符号。这可以提供一些更有用的见解，以了解造成崩溃本身的原因。 |
| **alcid=1** | 用于设置AppleALC的布局id，请参阅[支持的编解码器](https://github.com/acidanthera/applealc/wiki/supported-codecs) ，以确定针对特定系统使用哪种布局。更多信息在 [安装后页面](https://sumingyd.github.io/OpenCore-Post-Install/) |

* **特定于gpu的引导参数**:

| 引导参数 | 说明 |
| :--- | :--- |
| **-wegnoegpu** | 用于禁用除集成的Intel iGPU之外的所有其他gpu，对于那些想运行新版本的macOS，而他们的dGPU不支持的人很有用 |

* **csr-active-config**: `00000000`
  * '系统完整性保护' (SIP)的设置。通常建议通过恢复分区使用`csrutil`进行更改。
  * 默认情况下，csr-active-config设置为`00000000`，以启用系统完整性保护。您可以选择许多不同的值，但总的来说，为了最佳安全实践，我们建议启用此选项。更多信息可以在我们的故障排除页面中找到:[禁用SIP](../troubleshooting/extended/post-issues.md#disabling-sip)

* **run-efi-updater**: `No`
  * 这用于防止苹果的固件更新包安装的时候破坏引导顺序;这很重要，因为这些固件更新(意味着mac)将无法工作。

* **prev-lang:kbd**: <>
  * *需要`lang-COUNTRY:keyboard`格式的非拉丁键盘，建议保持空白，尽管你可以指定它(**示例配置中默认是俄语**):
  * 中文: `zh-Hans:252`(`7A682D48616E733A323532` 这是十六进制)
  * 完整列表可在这里找到[AppleKeyboardLayouts.txt](https://github.com/acidanthera/OpenCorePkg/blob/master/Utilities/AppleKeyboardLayouts/AppleKeyboardLayouts.txt)
  * 提示:`prevr-lang:kbd`可以被转换成String，所以你可以直接输入`zh-Hans:252`，而不是转换成十六进制
  * 提示2:`prevr-lang:kbd`可以设置为一个空白变量(例如:`<>`)，这将强制在首次启动时显示语言选择器。

| Key | Type | Value |
| :--- | :--- | :--- |
| prev-lang:kbd | String | zh-Hans:252 |

:::

### Delete

强制重写NVRAM变量，请注意，`Add` **不会覆盖** NVRAM中已经存在的值，所以像`boot-args`这样的值应该保持不变。

* **LegacySchema**
  * 用于赋值NVRAM变量，与`OpenVariableRuntimeDxe.efi`一起使用。仅适用于没有原生NVRAM的系统

* **WriteFlash**: YES
  * 允许所有添加的变量写入闪存。

## PlatformInfo

![PlatformInfo](../images/config/config-laptop.plist/broadwell/smbios.png)

::: tip 信息

为了设置SMBIOS信息，我们将使用CorpNewt的[GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)应用程序。

在Broadwell的例子中，我们选择了MacBookPro12,1 SMBIOS。典型的细分如下:

| SMBIOS | CPU 类型 | GPU 类型 | 显示尺寸 |
| :--- | :--- | :--- | :--- |
| MacBook8,1 | Dual Core 7W(Low End) | iGPU: HD 5300 | 12" |
| MacBookAir7,1 | Dual Core 15W | iGPU: HD 6000 | 11" |
| MacBookAir7,2 | Dual Core 15W | iGPU: HD 6000 | 13" |
| MacBookPro12,1 | Dual Core 28W(High End) | iGPU: Iris 6100 | 13" |
| MacBookPro11,2 | Quad Core 45W | iGPU: Iris Pro 5200 | 15" |
| MacBookPro11,3 | Quad Core 45W | iGPU: Iris Pro 5200 + dGPU: GT 750M | 15" |
| MacBookPro11,4 | Quad Core 45W | iGPU: Iris Pro 5200 | 15" |
| MacBookPro11,5 | Quad Core 45W | iGPU: Iris Pro 5200 + dGPU: R9 M370X | 15" |
| iMac16,1 | NUC Systems | HD 6000/Iris Pro 6200 |  N/A |

**注**:MacBook8,1在macOS Monterey中被删除。如果运行macOS Monterey，请选择其他型号。

运行GenSMBIOS，选择选项1下载MacSerial，选择选项3下载SMBIOS。这将给我们一个类似于下面的输出:

```sh
  #######################################################
 #               MacBookPro12,1 SMBIOS Info            #
#######################################################

Type:         MacBookPro12,1
Serial:       C02M9SYJFY10
Board Serial: C02408101J9G2Y7A8
SmUUID:       7B227BEC-660D-405F-8E60-411B3E4EF055
```

将 `Type` 部分复制到 Generic -> SystemProductName.

将 `Serial` 部分复制到 Generic -> SystemSerialNumber.

将 `Board Serial` 部分复制到 Generic -> MLB.

将 `SmUUID` 部分复制到 Generic -> SystemUUID.

我们将Generic -> ROM设置为苹果ROM(从真正的Mac中转储)，你的网卡Mac地址，或任何随机的Mac地址(可以是6个随机字节，在本指南中我们将使用`11223300 0000`。安装后，请跟随[修复iServices](https://sumingyd.github.io/OpenCore-Post-Install/universal/iservices.html)页面了解如何找到您的真实MAC地址)

**提醒你需要一个无效的序列号!当你在[苹果的检查覆盖页面](https://checkcoverage.apple.com)中输入你的序列号时，你会得到一条信息，比如“无法检查此序列号的覆盖范围”。**

**Automatic**: YES

* 基于Generic节而不是DataHub、NVRAM和SMBIOS节生成platformminfo

:::

### Generic

::: details 更深入的信息

* **AdviseFeatures**: NO
  * 用于当EFI分区不是Windows驱动器上的第一个分区

* **MaxBIOSVersion**: NO
  * 设置BIOS版本为Max，以避免在Big Sur+固件更新，主要适用于正版mac。

* **ProcessorType**: `0`
  * 设置为`0`用于自动类型检测，但是如果需要，这个值可以被覆盖。参见[AppleSmBios.h](https://github.com/acidanthera/OpenCorePkg/blob/master/Include/Apple/IndustryStandard/AppleSmBios.h) 获取可能的值

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

![UEFI](../images/config/config-universal/aptio-iv-uefi-laptop.png)

**ConnectDrivers**: YES

* 强制 .efi 驱动，更改为NO将自动连接添加的UEFI驱动。这可以使引导稍微快一点，但不是所有驱动程序都连接自己。例如某些文件系统驱动程序不能加载。

### Drivers

在这里添加你的.efi驱动程序。

下面列出的是必须在这里的

* HfsPlus.efi
* OpenRuntime.efi

::: details 更深入的信息

| Key | Type | 描述|
| :--- | :--- | :--- |
| Path | String | 文件在`OC/Drivers`目录中的路径 |
| LoadEarly | Boolean | 在安装NVRAM之前尽早加载驱动程序，如果使用旧的NVRAM，应该只启用`OpenRuntime.efi`和`OpenVariableRuntimeDxe.efi` |
| Arguments | String | 有些驱动程序接受这里指定的其他参数。 |

:::

### APFS

默认情况下，OpenCore只从macOS Big Sur及更新版本加载APFS驱动程序。如果引导macOS Catalina或更早版本，可能需要设置新的最低版本/日期。
不设置此选项会导致OpenCore找不到您的macOS分区!

macOS Sierra和更早的版本使用HFS代替APFS。如果引导旧版本的macOS，可以跳过本节。

::: tip APFS 版本

如果修改最小版本，需要同时设置MinVersion和MinDate。

| macOS 版本 | Min Version | Min Date |
| :------------ | :---------- | :------- |
| High Sierra (`10.13.6`) | `748077008000000` | `20180621` |
| Mojave (`10.14.6`) | `945275007000000` | `20190820` |
| Catalina (`10.15.4`) | `1412101001000000` | `20200306` |
| 没有限制 | `-1` | `-1` |

:::

### Audio

对于AudioDxe设置，我们将忽略(保留默认值)。这与mac系统中的音频支持无关。

* 有关AudioDxe和音频部分的进一步使用，请参见安装后页面:[添加GUI和启动铃声](https://sumingyd.github.io/OpenCore-Post-Install/)

### Input

与用于FileVault和热键支持的boot.efi键盘直通相关，将所有内容保留为默认值，因为我们不需要这些选项。更多详细信息:[安全和文件库](https://sumingyd.github.io/OpenCore-Post-Install/)

### Output

关于OpenCore的视觉输出，将所有内容保留为默认值，因为我们暂时不使用这些选项。

::: details 更深入的信息

| Output | Value | Comment |
| :--- | :--- | :--- |
| UIScale | `0` | `0`将根据分辨率自动设置<br/>`-1`将保持不变<br/>`1`为1x缩放，对于正常显示器<br/>`2`为2倍缩放，对于HiDPI显示器 |

:::

### ProtocolOverrides

主要适用于虚拟机，旧mac和FileVault用户。更多详细信息:[安全和文件库](https://sumingyd.github.io/OpenCore-Post-Install/)

### Quirks

::: tip 信息
关于UEFI环境的选项，我们将做以下更改:

| 选项 | 启用 | 说明 |
| :--- | :--- | :--- |
| IgnoreInvalidFlexRatio | YES | |
| ReleaseUsbOwnership | YES | |
| UnblockFsConnect | NO | 主要用于惠普主板 |

:::

::: details 更深入的信息

* **IgnoreInvalidFlexRatio**: YES
  * 修复了在BIOS中无法禁用MSR_FLEX_RATIO (0x194)的问题，所有基于skylake的系统都需要禁用
* **ReleaseUsbOwnership**: YES
  * 从固件驱动中释放USB控制器，当您的固件不支持EHCI/XHCI切换时需要。大多数笔记本电脑都有垃圾的固件，所以我们需要这个
* **DisableSecurityPolicy**: NO
  * 禁用固件中的平台安全策略，建议用于有bug的固件，其中禁用安全引导不允许加载第三方固件驱动程序。
  * 如果运行Microsoft Surface设备，建议启用此选项

* **RequestBootVarRouting**: YES
  * 将AptioMemoryFix从`EFI_GLOBAL_VARIABLE_GUID`重定向到`OC_VENDOR_VARIABLE_GUID`。当固件试图删除启动项时需要启用，建议在所有系统上启用，以确保正确的更新安装，启动磁盘控制面板的功能等。

* **UnblockFsConnect**: NO
  * 某些固件通过按驱动程序模式打开来阻塞分区句柄，这将导致无法安装文件系统协议。主要适用于没有列出驱动器的HP系统

:::

### ReservedMemory

用于将某些内存区域从操作系统中免除使用，主要与Sandy Bridge igpu或具有错误内存的系统相关。在本指南中没有涉及这个选项的用法

## 清理

现在你可以保存它，并将它放入EFI/OC下的EFI文件中。

对于那些有启动问题的人，请务必先阅读[故障排除部分](../troubleshooting/troubleshooting.md)，如果您的问题仍然没有得到解答，我们有大量的资源供您使用:

* [r/Hackintosh Subreddit](https://www.reddit.com/r/hackintosh/)
* [r/Hackintosh Discord](https://discord.gg/2QYd7ZT)

### Config reminders

**HP 用户**:

* Kernel -> Quirks -> LapicKernelPanic -> True
  * 否则，您将在LAPIC上出现内核崩溃
* UEFI -> Quirks -> UnblockFsConnect -> True

## Intel BIOS 设置

* 注意:大多数选项可能不会出现在你的固件中，我们建议尽可能匹配，但如果这些选项在你的BIOS中不可用，不要太担心

### 禁用

* 快速启动（Fast Boot）
* 安全引导（Secure Boot）
* 串口/COM端口（Serial/COM Port）
* 并口（Parallel Port）
* VT-d (如果将`DisableIoMapper`设置为YES，则可以启用)
* 兼容性支持模块(CSM)(**在大多数情况下必须关闭，当该选项启用时，像`gIO`这样的GPU错误/停顿很常见**)
* 雷电 Thunderbolt(用于初始安装，因为如果安装不正确，Thunderbolt可能会导致问题)
* Intel SGX
* Intel Platform Trust
* CFG Lock (MSR 0xE2写保护)(**必须关闭，如果你找不到选项，那么在Kernel -> Quirks下启用`AppleXcpmCfgLock`。你的hack将无法在启用CFG-Lock的情况下启动**)
  * 对于10.10和更老版本，你也需要启用AppleCpuPmCfgLock

### 启用

* VT-x
* 4G以上解码
* 超线程
* 执行禁止位
* EHCI/XHCI Hand-off
* 操作系统类型:Windows 8.1/10 UEFI模式(一些主板可能需要”其他操作系统”代替)
* DVMT预分配(iGPU内存):64MB或以上
* SATA 模式: AHCI

# 完成后，我们需要编辑一些额外的值。访问[苹果安全启动页面](../config.plist/security.md)
