# Haswell 笔记本

| 支持 | 版本 |
| :--- | :--- |
| 最低版本 | OS X 10.8, Mountain Lion |
| 最高版本 | macOS 12 Monterey |
| 注释 | 有关Ventura的信息, 请查看 [macOS 13 Ventura](../extras/ventura.md#dropped-cpu-support) |

## 起点

配置config.plist可能看起来很难，但事实并非如此。只是得花一点时间，但是你需要的一切文档中都有。这也意味着，如果你遇到问题请重新检查你的配置设置是否准确无误。一些提示:

* **所有的属性必须定义**，如果你在编辑plist，请确保config中没有落下任何一项(键和值)，所有需要的属性(键)都得有，否则会导致OpenCore崩溃。另外，**除非已经明确写明，否则不要删除节点**！如果指南中没有对某个值的明确说明，如果指南没有提到这个选项，那就保持默认值。
* **Sample.plist不能直接怼上去**，必须根据你的硬件配置一步一步的修改好！
* **不要使用配置器**, 这些工具很少会按照OpenCore的配置规则来，甚至连Mackie这样的工具都会因为添加Clover的属性导致OC的配置损坏！

知道了这些以后，快速过一遍要用的工具：

* [ProperTree](https://github.com/corpnewt/ProperTree)
  * plist通用编辑器
* [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)
  * 用来生成SMBIOS数据
* [Sample/config.plist](https://github.com/acidanthera/OpenCorePkg/releases)
  * 从这里获取: [config.plist Setup](../config.plist/README.md)

::: warningtip 

在配置 OpenCore 之前，请多读几次本文档，并确保配置的准确无误。请注意，截图并不一定是最新的，因此请看它们下面的文字，如果文字没说，则保留为默认值。

:::

## ACPI

![ACPI](../images/config/config-laptop.plist/haswell/acpi.png)

### 添加(Add)

::: tip 提示

这是给你系统放SSDT的地方，这些配置对 **启动macOS** 来说非常重要。SSDT可以实现许多功能： [定制USB](https://dortania.github.io/OpenCore-Post-Install/usb/), [禁用不支持的显卡(GPU)](../extras/spoof.md) 等等. **我们的系统没了它甚至启动不了**. 关于怎么制作SSDT，请阅读: [**ACPI 入门指南**](https://dortania.github.io/Getting-Started-With-ACPI/)

对我们这平台来说，我们需要几个SSDT来恢复以前Clover有的功能：

| 需要的 SSDT | 描述 |
| :--- | :--- |
| **[SSDT-PLUG](https://dortania.github.io/Getting-Started-With-ACPI/)** | 在Haswell以及更高的版本中启用原生CPU电源管理，更多请参考 [ACPI 入门指南](https://dortania.github.io/Getting-Started-With-ACPI/) |
| **[SSDT-EC](https://dortania.github.io/Getting-Started-With-ACPI/)** | 修复EC控制器(embedded controller)，更多请参考 [ACPI 入门指南](https://dortania.github.io/Getting-Started-With-ACPI/) |
| **[SSDT-GPIO](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/decompiled/SSDT-GPI0.dsl.zip)** | 创建一个接口让Voodool2C可以连接，如果你在使用Voodool2C的过程中遇到了问题，请尝试用 [SSDT-XOSI](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-XOSI.aml) 代替 注意：Intel NUC 不需要这个 |
| **[SSDT-PNLF](https://dortania.github.io/Getting-Started-With-ACPI/)** | 屏幕亮度调节补丁，更多请参考 [ACPI 入门指南](https://dortania.github.io/Getting-Started-With-ACPI/) 注意：Intel NUC 不需要这个 |

注意，你 **不应该** 在这里添加 `DSDT.aml`，因为你的固件中已经有了。所以，如果原来有的话请删掉它。

如果你想要深入了解 提取DSDT、制作并编译这些SSDT, 请参考 [**ACPI 入门指南**](https://dortania.github.io/Getting-Started-With-ACPI/)。编译好的SSDT有 **.aml** 后缀(汇编语言) 将会放在 `EFI/OC/ACPI` 中，**并且** 在 `ACPI -> Add` 中正确设置。

:::

### 删除(Delete)

这的选项可以阻止某些ACPI表加载，但是我们并不需要它。

### 补丁(Patch)

::: tip 提示

我们可以在这里通过OpenCore动态修改ACPI中的各个部分（DSDT，SSDT等）对于本平台，我们需要以下内容：
* 重命名OSI
  * 使用SSDT-XOSI时必须添加，因为我们通过它将所有 OSI 调用重定向到此 SSDT, **如果您使用的是 SSDT-GPIO，则不要添加**

| Comment | String | Change _OSI to XOSI |
| :--- | :--- | :--- |
| Enabled | Boolean | YES |
| Count | Number | 0 |
| Limit | Number | 0 |
| Find | Data | `5f4f5349` |
| Replace | Data | `584f5349` |

英文翻译可能不容易理解，有关此表格的具体信息，请参阅[表单信息]()

:::

### 选项(Quirks)

这些是与 ACPI 相关的设置，请将此处的所有内容保留为默认值，因为我们没有用到。

## 引导(Booter)

![Booter](../images/config/config-universal/aptio-iv-booter.png)

本节专门介绍与 AptioMemoryFix.efi的替代品OpenRuntime 对boot.efi进行修补时用到选项。

### Mmio白名单(MmioWhitelist)

这里可以允许将通常被忽略的空间传递到 macOS，在与 `DevirtualiseMmio` 搭配时很有用。

### 选项(Quirks)

::: tip 提示
这些是与boot.efi修补和固件修复相关的设置，请将此处的所有内容保留为默认值，因为我们没有用到。
:::
::: details 详细信息

* **AvoidRuntimeDefrag**: YES
  * 修复UEFI运行时服务，如日期，时间，NVRAM，电源管理等
* **EnableSafeModeSlide**: YES
  * 允许在安全模式下使用slide变量。
* **EnableWriteUnprotector**: YES
  * 从 CR0 寄存器中删除写保护。
* **ProvideCustomSlide**: YES
  * 用于计算slide变量。但是，是否需要此选项由debug日志中的`OCABC: Only N/256 slide values are usable!`确定。如果日志中有`OCABC: All slides are usable! You can disable ProvideCustomSlide!`，则可以禁用ProvideCustomSlide。
* **SetupVirtualMap**: YES
  * 修复了对虚拟地址的 SetVirtualAddresses 调用，技嘉主板需要它来解决一开始的内核崩溃。
  
:::

## 设备属性(DeviceProperties)

![DeviceProperties](../images/config/config-laptop.plist/haswell/DeviceProperties.png)

### 添加(Add)

Sets device properties from a map.

::: tip PciRoot(0x0)/Pci(0x2,0x0)

这里根据WhateverGreen的 [帧缓冲(Framebuffer)修补指南](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md) 设置重要的iGPU属性。

默认的config.plist没有自带这部分，因此您必须手动添加它。

下表应该有助于找到要设置的正确值和一些有关这些设置说明：

* **AAPL,ig-platform-id**
  * This is used internally for setting up the iGPU
* **Type**
  * Whether the entry is recommended for laptops(ie. with built-in displays) or for Intel NUCs(ie. stand alone boxes)

Generally follow these steps when setting up your iGPU properties. Follow the configuration notes below the table if they say anything different:

1. 第一次配置时只需要设置 AAPL,ig-platform-id 就够了
2. 如果你能正常启动且没有图形加速 (7MB 显存且Dock栏没有毛玻璃), 这代表着你可能需要尝试不同的 `AAPL,ig-platform-id` 值, 添加stolenmem补丁, 甚至是添加 `device-id` 属性.

| AAPL,ig-platform-id | Type | Comment |
| ------------------- | ---- | ------- |
| **`0500260A`** | 笔记本 | 一般 HD 5000、HD 5100、HD 5200 用这个 |
| **`0600260A`** | 笔记本 | 一般 HD 4200、HD 4400、HD 4600，**另外** 还得加一个`device-id`(在前面) |
| **`0300220D`** | NUC | 一般所有的Haswell平台的NUC都用这个，**另外** HD 4200/4400/4600 还得加一个 `device-id` (在前面) |

#### 配置说明

除了 AAPL,ig-platform-id 之外，由于一些问题，您还需要添加从 6MB (00006000) 到 9MB 的游标字节大小补丁:

| Key | Type | Value |
| :--- | :--- | :--- |
| framebuffer-patch-enable | Data | `01000000` |
| framebuffer-cursormem | Data | `00009000` |

**对于 HD 4200、HD 4400、HD 4600 的特殊说明**:

还需要 device-id 来冒仿支持的显卡:

| Key | Type | Value |
| :--- | :--- | :--- |
| device-id | Data | `12040000` |

:::

::: tip PciRoot(0x0)/Pci(0x1b,0x0)

`layout-id`

* AppleALC音频注入，你需要自己找出适合你主板的codec，并且设置codec对应的layout [AppleALC支持的Codec](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs).
* 请删除这个属性，现在他对我们没用

对我们来说，应该添加引导参数(boot-args) `alcid=xxx` 来实现这个。`alcid` 将会覆盖所有存在的layout-id。有关这部分的详细信息，请参见 [Post-Install Page](https://dortania.github.io/OpenCore-Post-Install/)

:::

### 删除(Delete)

从map中删除设备属性，我们并不需要它。

## 内核(Kernel)

![Kernel](../images/config/config-universal/kernel-legacy-XCPM.png)

### 添加(Add)

在这里指定要加载的 kext、加载顺序以及标明每个 kext 的用途。默认情况下，我们建议保留ProperTree的操作，但是对于32位CPU，请参见以下内容：

::: details 详细信息

您主要需要注意的事情是:

* 加载顺序
  * 记住，任何插件都应该在其依赖项之后加载
  * 这就意味着像 Lilu 这样的 kext **必须** 在 VirtualSMC、AppleALC、WhateverGreen 等之前加载。

提醒，如果你在用 [ProperTree](https://github.com/corpnewt/ProperTree) 可以用 **Cmd/Ctrl + Shift + R** 来以以正确的顺序添加所有 kext。

* **Arch**
  * 此 kext 支持的架构
  * 目前支持的值为 `Any` (通用)、`i386` (32 位) 和 `x86_64` (64 位)
* **BundlePath**
  * kext名称
  * 例如: `Lilu.kext`
* **Enabled**
  * 很明显，启用或禁用 kext
* **ExecutablePath**
  * 实际可执行文件的路径藏在 kext 中，您可以通过右键并选择 `显示包内容` 来查看 kext 的路径。一般情况下，它们是 `Contents/MacOS/Kext`，但有些在 `Plugin` 文件夹下藏了更多kext。请注意，只有plist的kext 不需要填写此内容。
  * 比如: `Contents/MacOS/Lilu`
* **MinKernel**
  * 加载您的kext的最低内核版本，请参阅下表以获取可能的值
  * 比如: OS X 10.8 是 `12.00.00`
* **MaxKernel**
  * 加载您的kext的最高内核版本，请参阅下表以获取可能的值
  * 比如: OS X 10.7 是 `11.99.99`
* **PlistPath**
  * 藏在kext中的 `info.plist` 的路径
  * 比如: `Contents/Info.plist`
  
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

:::

### 冒仿(Emulate)

不受支持的 CPU（如奔腾和赛扬）需要冒仿

* **Cpuid1Mask**: 留空
* **Cpuid1Data**: 留空

### 强制(Force)

用于从系统分区中加载 kext，仅与缓存中不存在某些 kext 的旧操作系统相关（如10.6中的IONetworkingFamily）

对于我们，跳过即可。

### 阻止(Block)

阻止某些kext加载，我们不需要。

### 修补(Patch)

修补内核和 kext。

### 选项(Quirks)

::: tip 提示

与内核相关的设置，我们将启用以下内容：

| Quirk | Enabled | Comment |
| :--- | :--- | :--- |
| AppleCpuPmCfgLock | NO | 如果运行 10.10 或更低版本并且无法在 BIOS 中禁用 `CFG-Lock` ，则需要 |
| AppleXcpmCfgLock | YES | 如果在BIOS中禁用了 `CFG-Lock` 则不需要 |
| DisableIoMapper | YES | 如果在BIOS中禁用了 `VT-D` 则不需要 |
| LapicKernelPanic | NO | 惠普(HP)准系统需要这个选项 |
| PanicNoKextDump | YES | |
| PowerTimeoutKernelPanic | YES | |
| XhciPortLimit | YES | 如果运行macOS 11.3+请禁用 |

:::

::: details 详细信息

* **AppleCpuPmCfgLock**: NO
  * 只在BIOS中不能禁用 CFG-Lock 时启用
  * 只适用于 Ivy Bridge 及以下
    * 注意：Broadwell 及更低在运行 10.10 或更低版本时需要这样做
* **AppleXcpmCfgLock**: YES
  * 只在BIOS中不能禁用 CFG-Lock 时启用
  * 只适用于 Haswell 及以上
    * 注意：Ivy Bridge-E 也包括在内，因为它具有XCPM功能
* **CustomSMBIOSGuid**: NO
  * UpdateSMBIOSMode设置为`Custom`时执行 GUID 修补。通常与戴尔笔记本电脑有关
  * 使用UpdateSMBIOSMode的`Custom`模式时启用此选项也可以禁用SMBIOS注入"非Apple"操作系统(禁止Windows识别为MacBook)，但我们不推荐方法，因为它破坏了BootCamp的兼容性。使用风险自负
* **DisableIoMapper**: YES
  * 如果无法在 BIOS 中禁用或其他操作系统需要 VT-D，则需要绕过 VT-D，这是`dart=0`的更好替代方案，因为 SIP 可以在Catalina保持打开状态
* **DisableLinkeditJettison**: YES
  * 允许 Lilu 和其他kext在没有`keepsyms=1`的情况下以更高的性能运行
* **DisableRtcChecksum**: NO
  * 防止 AppleRTC 最初写入校验(0x58-0x59)，这是遇到 BIOS 重置或在重新启动/关机后进入安全模式的人所必需的
* **ExtendBTFeatureFlags** NO
  * 对那些在使用非苹果/非Fenvi卡时遇到许多问题的人很有帮助
* **LapicKernelPanic**: NO
  * 禁用 AP 核心中断造成的内核崩溃，惠普(HP)准系统通常需要此选项。相当于Clover的`Kernel LAPIC`
* **LegacyCommpage**: NO
  * 解决了 macOS 中 64 位 CPU 的 SSSE3 要求，主要与 64 位奔腾 4 CPU 相关（即Prescott）
* **PanicNoKextDump**: YES
  * 允许在发生内核崩溃时读取内核崩溃日志
* **PowerTimeoutKernelPanic**: YES
  * 帮助修复与 macOS Catalina 中 Apple 电源驱动程序的更改有关的内核崩溃，尤其是数字音频(digital audio)。
* **SetApfsTrimTimeout**: `-1`
  * 为 SSD 上的 APFS 文件系统设置trim超时（单位：微秒），仅适用于在 macOS 10.14 及更高版本发生问题的SSD用户。
* **XhciPortLimit**: YES
  * 这是 15 端口限制的补丁，不要依赖它，因为它不是修复 USB 的稳定解决方案。如果可能，请创建一个 [USB map](https://dortania.github.io/OpenCore-安装后/usb/)
  * 在 macOS 11.3+, [XhciPortLimit 可能无法正常运行](https://github.com/dortania/bugtracker/issues/162) 我们建议禁用这个选项并在升级macOS之前 [在Windows下定制USB](https://github.com/USBToolBox/tool). 当然，也可以选择安装macOS 11.2.3或者更低的版本。

UsbInjectAll在没有适当的电流调整的情况下重新实现了内置的macOS功能。仅在单 plist 的 kext 中描述您的端口要比UsbInjectAll整洁得多，也不会浪费运行时内存等

:::

### 方案(Scheme)

与传统(legacy)引导（OS X 10.4-10.6）相关的设置，对于大多数人可以跳过，但如果你想引导旧系统，设置可以在下面找到：


::: details 详细信息

* **FuzzyMatch**: True
  * 忽略内核缓存(kernelcache)的校验和(checksums)，选择最新的可用缓存。可以帮助提高 10.6 中许多电脑上的启动速度
* **KernelArch**: x86_64
  * 设置内核的架构类型, 你可以选择 `Auto`, `i386` (32位), `x86_64` (64位).
  * 如果您要启动需要 32 位内核（10.4和10.5）的旧系统，我们建议您将其设置为`Auto`，并让 macOS 根据您的 SMBIOS 做出决定。有关支持的值，请参阅下表
    * 10.4-10.5 — `x86_64`, `i386` or `i386-user32`
      * `i386-user32` 指 32 位用户空间(userspace)，32 位 CPU 必须使用此空间 (或缺少 SSSE3 的 CPU)
      * `x86_64`仍有 32 位内核空间(kernelspace)，但将确保 64 位用户空间在 10.4/5 中
      
    * 10.6 — `i386`, `i386-user32`, or `x86_64`
    * 10.7 — `i386` or `x86_64`
    * 10.8 及更新版本 — `x86_64`

* **KernelCache**: Auto
  * 设置内核缓存类型，主要用于调试，因此我们建议使用`Auto`以获得最佳支持


:::

## Misc

![Misc](../images/config/config-universal/misc.png)

### Boot

::: tip 提示

| Quirk | Enabled | Comment |
| :--- | :--- | :--- |
| HideAuxiliary | YES | 隐藏杂项，按空格键显示macOS恢复和其他辅助条目 |

:::

::: details 详细信息

* **HideAuxiliary**: YES
  * 此选项将在选取器中隐藏多余条目，例如 macOS Recovery和Tools(如Reset NVRAM)。隐藏辅助项可能会提高多磁盘系统上的引导性能。您可以在选取器上按空格键以显示这些条目

:::

### Debug

::: tip 提示

对于调试OpenCore启动问题很有帮助(我们将更改所有内容*除了* `DisplayDelay`):

| Quirk | Enabled |
| :--- | :--- |
| AppleDebug | YES |
| ApplePanic | YES |
| DisableWatchDog | YES |
| Target | 67 |

:::

::: details 详细信息

* **AppleDebug**: YES
  * 启用 boot.efi 日志记录，这对调试很有用。这仅在 10.15.4 及更高版本上支持
* **ApplePanic**: YES
  * 尝试将内核崩溃log保存到磁盘
* **DisableWatchDog**: YES
  * 禁用 UEFI WatchDog，可以帮助解决一开始的启动问题
* **DisplayLevel**: `2147483650`
  * 显示更多调试信息，需要 OpenCore 的debug版
* **SysReport**: NO
  * 有助于调试，例如提取 ACPI 表
  * 仅限 OpenCore 的debug版
* **Target**: `67`
  * 显示更多调试信息，需要 OpenCore 的debug版

这些值根据[OpenCore 调试](../troubleshooting/debug.md)计算

:::

### 安全(Security)

::: tip 提示

安全性也很重要，**不要跳过**。我们将更改以下内容:

| Quirk | Enabled | Comment |
| :--- | :--- | :--- |
| AllowSetDefault | YES | |
| BlacklistAppleUpdate | YES | |
| ScanPolicy | 0 | |
| SecureBootModel | Default | 保留为 `Default`，OpenCore会自动设置与您的SMBIOS相对应的正确值，下面会详细介绍。 |
| Vault | Optional | 不能省略此设置。如果不将其设置为Optional，你会后悔的，注意，它区分大小写 |

:::

::: details 详细信息

* **AllowSetDefault**: YES
  * 允许使用 `CTRL+Enter` 或 `CTRL+Index` 设置默认启动项
* **ApECID**: 0
  * 用于锁定专有安全启动标识符，由于 macOS 安装程序中的错误，目前此选项不可靠，因此我们强烈建议您将其保留为默认值。
* **AuthRestart**: NO
  * 为FileVault 2启用重启验证，重新启动时不需要密码。可以被视为安全风险，因此可选
* **BlacklistAppleUpdate**: YES
  * 用于阻止固件更新的额外保护，因为 macOS Big Sur 不再使用`run-efi-updater`变量


* **DmgLoading**: Signed
  * 仅加载已签名的dmg
* **ExposeSensitiveData**: `6`
  * 显示更多调试信息，需要 OpenCore 的debug版
* **Vault**: `Optional`
  * 我们不需要验证，所以跳过，**将这个选项设置为Secure你将不能启动**
  * 不能省略此设置。如果不将其设置为Optional，你会后悔的，注意，它区分大小写
* **ScanPolicy**: `0`
  * `0` 允许您查看所有可用的驱动器，有关详细信息，请参考 [安全选项](https://dortania.github.io/OpenCore-Post-Install/universal/security.html) **保留默认值将不会从USB设备启动**
* **SecureBootModel**: Default
  * 控制 macOS 中苹果的安全启动功能，有关详细信息，请参考 [安全选项](https://dortania.github.io/OpenCore-Post-Install/universal/security.html) 
  * 注意：用户可能会发现在已安装的系统上升级OpenCore可能会导致一开始的启动失败。要解决此问题，请参阅此处： [卡在 OCB: LoadImage failed - 安全违规](/troubleshooting/extended/kernel-issues.md#stuck-on-ocb-loadimage-failed-security-violation)

:::

### 串行(Serial)

串口调试选项 (保留默认值)

### 工具(Tools)

用于运行 OC 调试工具（如 shell），ProperTree 的snapshot功能将为您添加这些工具

### Entries

用于指定 OpenCore 找不到的不规则引导路径。
此处不涉及，有关详细信息，请参阅 [Configuration.pdf](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf) 8.6

## NVRAM

![NVRAM](../images/config/config-universal/nvram.png)

### 添加(Add)

::: tip 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14

用于OpenCore的UI样式设置，默认值很合适。有关详细信息，请参阅深入部分

:::

::: details 详细信息

引导路径，主要用于 UI 修改

* **DefaultBackgroundColor**: boot.efi背景颜色
  * `00000000`: Syrah Black(黑色)
  * `BFBFBF00`: Light Gray(亮灰色)

:::

::: tip 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102

OpenCore的NVRAM GUID，主要与RTCMemoryFixup有关。

:::

::: details 详细信息

* **rtc-blacklist**: <>
  * 与 RTCMemoryFixup 一起使用，更多请参考: [修复 RTC 写入问题](https://dortania.github.io/OpenCore-Post-Install/misc/rtc.html#finding-our-bad-rtc-region)
  * 大多数情况下跳过即可

:::

::: tip 7C436110-AB2A-4BBB-A880-FE41995C9F82

系统完整性保护代码(bitmask)

* **General Purpose boot-args**:

| boot-args | 描述 |
| :--- | :--- |
| **-v** | 这将启用啰嗦(verbose)模式，该模式显示启动时滚动的所有幕后文本，而不是 Apple Logo 和进度条。它对任何 Hackintosher(搞黑苹果的人) 来说都是无价的，因为它可以让您深入了解启动过程，并可以帮助您识别问题、问题 kext 等。|
| **debug=0x100** | 这将禁用macOS的watchdog，这有助于阻止在内核崩溃时自动重启。这样*更利于*收集一些有用的信息并解决问题。|
| **keepsyms=1** | 这是 debug=0x100 的配套设置，它告诉操作系统在内核崩溃时也打印符号。这可以提供有关导致崩溃本身的原因和更有用的信息。 |
| **alcid=1** | 用于设置 AppleALC 的layout-id，参考：[支持的 codec](https://github.com/acidanthera/applealc/wiki/supported-codecs) 找出你的声卡codec，更详细的信息请参考 [Post-Install Page](https://dortania.github.io/OpenCore-Post-Install/) |

* **GPU-Specific boot-args**:

| boot-args | 描述 |
| :--- | :--- |
| **-wegnoegpu** | 禁用iGPU以外的GPU，对于独显不支持的人很有用 |

* **csr-active-config**: `00000000`
  * 系统完整性保护(SIP)的设置。一般建议通过恢复分区使用`csrutil`更改此设置。
  * 默认的情况下，csr-active-config 设置为 '00000000'，这将启用系统完整性保护。您可以选择许多不同的值，但总的来说，我们建议保持启用此功能来保证安全。更多信息可以在我们的故障排除页面中找到：[禁用 SIP](../troubleshooting/extended/post-issues.md#disabling-sip)

* **run-efi-updater**: `No`
  * 这用于防止 Apple 的固件更新包安装和破坏启动顺序，这很重要，因为这些适用于 Mac的固件更新将不起作用

* **prev-lang:kbd**: <>
  * 非拉丁(non-latin)键盘需要以 `lang-COUNTRY:keyboard`格式填写，建议留空尽管您可以指定它（**示例配置(sample.config)中的默认值为俄语**）:
  * 美国: `en-US:0`(十六进制`656e2d55533a30`)
  * 中国: `zh-Hans:0`(十六进制`7A682D48616E733A30`)
  * 完整列表可以在这里找到 [AppleKeyboardLayouts.txt](https://github.com/acidanthera/OpenCorePkg/blob/master/Utilities/AppleKeyboardLayouts/AppleKeyboardLayouts.txt)
  * 提示: `prev-lang:kbd` 可以是String，当他是String时直接填入 `zh-Hans:0` 即可，不需要转换HEX
  * 提示2: `prev-lang:kbd` 可以留空(如 `<>`)，此时macOS将会在首次安装时要求你选择语言

| Key | Type | Value |
| :--- | :--- | :--- |
| prev-lang:kbd | String | zh-Hans:0 |

:::

### 删除(Delete)

强制重新写入NVRAM的值，注意，`Add` **不会覆盖** 已存在的值，所以像 `boot-args` 这样的值应单独保留

* **LegacySchema**
  * 用于分配NVRAM变量，与`OpenVariableRuntimeDxe.efi`搭配。仅本机不自带 NVRAM 的系统才需要

* **WriteFlash**: YES
  * 将所有添加的变量写入闪存

## PlatformInfo

![PlatformInfo](../images/config/config-laptop.plist/haswell/smbios.png)

::: tip 提示

我们使用CorpNewt的 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 来生成SMBIOS信息。

对于Haswell的这个例子，我们选择了MacBookPro11,1 SMBIOS。通常细分如下：

| SMBIOS | CPU 类型 | GPU 型号 | 显示屏尺寸 |
| :--- | :--- | :--- | :--- |
| MacBookAir6,1 | Dual Core 15W | iGPU: HD 5000 | 11" |
| MacBookAir6,2 | Dual Core 15W | iGPU: HD 5000 | 13" |
| MacBookPro11,1 | Dual Core 28W | iGPU: Iris 5100 | 13" |
| MacBookPro11,2 | Quad Core 45W | iGPU: Iris Pro 5200 | 15" |
| MacBookPro11,3 | Quad Core 45W | iGPU: Iris Pro 5200 + dGPU: GT 750M | 15" |
| MacBookPro11,4 | Quad Core 45W | iGPU: Iris Pro 5200 | 15" |
| MacBookPro11,5 | Quad Core 45W | iGPU: Iris Pro 5200 + dGPU: R9 M370X | 15" |
| Macmini7,1 | NUC Systems | HD 5000/Iris 5100 | N/A |

**注释**: macOS Monterey只支持以下SMBIOS

::: details Monterey支持的SMBIOS表

| SMBIOS | CPU 类型 | GPU 型号 | 显示屏尺寸 |
| :--- | :--- | :--- | :--- |
| MacBookPro11,4 | Quad Core 45W | iGPU: Iris Pro 5200 | 15" |
| MacBookPro11,5 | Quad Core 45W | iGPU: Iris Pro 5200 + dGPU: R9 M370X | 15" |
| Macmini7,1 | NUC Systems | HD 5000/Iris 5100 | N/A |

:::

运行 GenSMBIOS, 选择选项1用于下载MacSerial，选择选项3用于选择SMBIOS。 你将会看到类似以下输出：

```sh
  #######################################################
 #               MacBookPro11,1 SMBIOS Info            #
#######################################################

Type:         MacBookPro11,1
Serial:       C02M9SYJFY10
Board Serial: C02408101J9G2Y7A8
SmUUID:       7B227BEC-660D-405F-8E60-411B3E4EF055
```

把 `Type` 复制到 Generic -> SystemProductName.

把 `Serial` 复制到 Generic -> SystemSerialNumber.

把 `Board Serial` 复制到 Generic -> MLB.

把 `SmUUID` 复制到 Generic -> SystemUUID.

把 Generic -> ROM 设置为 Apple的ROM (在白苹果中提取的)或你的 NIC MAC地址或随机MAC地址 (6字节随机数(12位)，这里使用`11223300 0000`. 装完macOS之后按照 [修复 iServices](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html) 来找出你的真实MAC地址)

**提醒, 需要一个无效的序列号！在 [Apple 的序列号检查](https://checkcoverage.apple.com)中输入序列号时，您应该会收到一条消息，例如"无法找到相关信息"。**

**Automatic**: YES
* 基于Generic部分而不是DataHub生成NVRAM和SMBIOS部分平台信息

:::

### 通用(Generic)

::: details 详细信息

* **AdviseFeatures**: NO
  * EFI分区不在第一个的Windows设备需要

* **MaxBIOSVersion**: NO
  * 将 BIOS 版本设置为 Max 以避免 Big Sur+ 中的固件更新，主要适用于白苹果。

* **ProcessorType**: `0`
  * 设置为`0`表示自动检测类型，但如果需要，可以覆盖此值，参考 [AppleSmBios.h](https://github.com/acidanthera/OpenCorePkg/blob/master/Include/Apple/IndustryStandard/AppleSmBios.h) 中的值

* **SpoofVendor**: YES
  * 将供应商字段替换为Acidanthera，在大多数情况下使用Apple作为供应商通常不安全

* **SystemMemoryStatus**: Auto
  * 在SMBIOS信息中设置内存是否焊接(插槽是否插入)，纯属装饰，所以我们设置`Auto`

* **UpdateDataHub**: YES
  * 更新Data Hub字段

* **UpdateNVRAM**: YES
  * 更新NVRAM字段

* **UpdateSMBIOS**: YES
  * 更新SMBIOS字段

* **UpdateSMBIOSMode**: Create
  * 将表替换为新分配的EfiReservedMemoryType，在需要`CustomSMBIOSGuid`选项的戴尔(Dell)准系统上选择`Custom`
  * 使用UpdateSMBIOSMode的`Custom`模式时启用此选项也可以禁用SMBIOS注入"非Apple"操作系统(禁止Windows识别为MacBook)，但我们不推荐方法，因为它破坏了BootCamp的兼容性。使用风险自负

:::

## UEFI

![UEFI](../images/config/config-universal/aptio-iv-uefi-laptop.png)

**ConnectDrivers**: YES

* 强制 .efi 驱动连接, 设置为 NO 将自动连接添加的 UEFI 驱动程序。这可以使启动速度稍快，但并非所有驱动都会自动连接。例如某些文件系统驱动可能无法加载。

### Drivers

在这里添加你的 .efi 驱动.

一定得有的驱动只有:

* HfsPlus.efi
* OpenRuntime.efi

::: details 详细信息

| Key | Type | 描述 |
| :--- | :--- | :--- |
| Path | String | 在 `OC/Drivers` 文件夹中的Path |
| LoadEarly | Boolean | 在设置 NVRAM 之前加载, 只应该在传统NVRAM的 `OpenRuntime.efi`、 `OpenVariableRuntimeDxe.efi` 上启用 |
| Arguments | String | 某些驱动程序接受此处指定的额外参数 |

:::

### APFS

默认情况下，OpenCore 只为 macOS Big Sur 及更高版本的APFS分区加载驱动。如果您正在启动 macOS Catalina 或更旧的版本，您可能需要设置其他的最低版本/日期。
不设置此选项可能会导致OpenCore找不到您的APFS macOS分区！
macOS Sierra 和更旧版本使用 HFS 而非 APFS。如果引导旧版本的 macOS，则可以跳过此部分。


::: tip APFS 日期(版本)

更改最低版本需要同时设置最小版本和最小日期

| macOS Version | Min Version | Min Date |
| :------------ | :---------- | :------- |
| High Sierra (`10.13.6`) | `748077008000000` | `20180621` |
| Mojave (`10.14.6`) | `945275007000000` | `20190820` |
| Catalina (`10.15.4`) | `1412101001000000` | `20200306` |
| 不检测(允许加载任何版本) | `-1` | `-1` |

:::

### 声音(Audio)

与 AudioDxe 设置相关，忽略即可（保留为默认值）。这与 macOS 中的音频(声卡)支持无关。

* 有关 AudioDxe音频 的使用，请参阅安装后页面: [添加GUI和启动音效](https://dortania.github.io/OpenCore-Post-Install/)

### 输入(Input)

与用于FileVault和Hotkey支持的boot.efi键盘快捷键相关，将此处的所有内容保留为默认值，我们用不到这些选项。有关更多详细信息，请参阅此处: [安全性和FileVault](https://dortania.github.io/OpenCore-Post-Install/)


### 输出(Output)

关于OpenCore的显示输出，请将所有内容保留为默认值，我们用不到这些选项。


::: details 详细信息

| Output | Value | Comment |
| :--- | :--- | :--- |
| UIScale | `0` | `0`将根据分辨率自动设置<br/>`-1`将保持不变<br/>`1` 1x缩放(普通显示器)<br/>`2` 2x缩放(HiDPI) |

:::

### ProtocolOverrides

主要与虚拟机、老旧 Mac 和FileVault用户相关，更多请参考: [安全性和FileVault](https://dortania.github.io/OpenCore-Post-Install/)

### Quirks

::: tip 提示
Relating to quirks with the UEFI environment, for us we'll be changing the following:

| Quirk | Enabled | Comment |
| :--- | :--- | :--- |
| IgnoreInvalidFlexRatio | YES | |
| ReleaseUsbOwnership | YES | |
| UnblockFsConnect | NO | 惠普(HP)用户一般需要 |

:::

::: details 详细信息

* **IgnoreInvalidFlexRatio**: YES
  * 修复了无法在 BIOS 中禁用 MSR_FLEX_RATIO(0x194) 的造成的问题，这是所有基于 Skylake 平台之前的平台所必需的
* **ReleaseUsbOwnership**: YES
  * 从固件驱动程序释放 USB 控制器，当您的固件不支持 EHCI/XHCI 切换时需要。大多数笔记本电脑都有垃圾的固件，所以我们也需要这个
* **DisableSecurityPolicy**: NO
  * 禁用固件中的平台安全策略，建议用于禁用安全启动不允许加载第三方固件驱动程序的错误固件
  * 如果你用的是Microsoft Surface，建议启用

* **RequestBootVarRouting**: YES
  * 将 AptioMemoryFix 从`EFI_GLOBAL_VARIABLE_GUID`重定向到`OC_VENDOR_VARIABLE_GUID`。当固件尝试删除启动项时需要，建议在所有系统上启用以 正确安装更新、让启动磁盘设置生效 等。

* **UnblockFsConnect**: NO
  * By Driver模式下某些固件会因为打开分区句柄而破坏了它们，导致文件系统协议装载失败，主要适用于没有列出驱动器(硬盘)的惠普(HP)准系统

:::

### ReservedMemory

用于从操作系统中去除某些内存区域，主要与 Sandy Bridge 下的 iGPU 或有内存故障的系统相关。本文档未涵盖此选项的使用。

## 整理归位

现在，准备好config将它保存并放入 EFI(分区) 下的 EFI/OC 中。

如果你的引导不能正常启动，请务必先看 [故障排除](../troubleshooting/troubleshooting.md) ，如果你的问题还没解决，请向我们求助:

* [r/Hackintosh Subreddit](https://www.reddit.com/r/hackintosh/)
* [r/Hackintosh Discord](https://discord.gg/2QYd7ZT)

对于中国的用户，请去：
* [r/我们的QQ群]()
* [r/PCBeta远景论坛]()
* [r/远景论坛的QQ群]()

### 配置提示

**惠普(HP) 用户**:

* Kernel -> Quirks -> LapicKernelPanic -> True
  * 否则会在 LAPIC 上出现内核崩溃
* UEFI -> Quirks -> UnblockFsConnect -> True

## Intel BIOS 设置

* 注意：这些选项中的大多数可能不存在于您的固件中，我们建议尽可能匹配，但如果其中许多选项在您的 BIOS 中不可用，请不要担心

### 禁用(Disable)

* Fast Boot
* Secure Boot
* Serial/COM Port
* Parallel Port
* VT-d (如果设置 `DisableIoMapper` 为 YES 则可以启用)
* Compatibility Support Module (CSM) (**普遍需要关闭，GPU 卡住/错误像 `gIO` 这样的一般是因为此选项是启用的**)
* Thunderbolt(对于第一次安装，因为如果设置不正确，Thunderbolt 可能会导致问题)
* Intel SGX
* Intel Platform Trust
* CFG Lock (MSR 0xE2 写保护)(**此设置必须禁用，如果你找不到这个选项请打开 Kernel -> Quirks 中的 `AppleXcpmCfgLock`，你的黑苹果就可以在CFG-Lock没关的情况下启动**)
  * 如果你是10.10或更低的版本，可能还需要打开 AppleCpuPmCfgLock

### 启用(Enable)

* VT-x
* Above 4G Decoding
* Hyper-Threading
* Execute Disable Bit
* EHCI/XHCI Hand-off
* OS type: Windows 8.1/10 UEFI Mode (一些主板需要设置为"Other OS")
* DVMT Pre-Allocated(iGPU 显存): 64MB or higher
* SATA Mode: AHCI

# 以上全部完成后，我们需要编辑几个额外的值。前往 [Apple Secure Boot 页面](../config.plist/security.md)了解更多
