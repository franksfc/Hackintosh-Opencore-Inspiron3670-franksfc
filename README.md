# # hackintosh-Inspiron3670-franksfc

# 我的opencore引导

我折腾的过程比较漫长，从下载黑果小兵的镜像用免驱独显，到拔掉独显定制opencore核显，最后不断完善具体的功能，直到黑苹果接近完美。给大家分享一下我的折腾成果，以及我的理由。

## 机型配置

| _Inspiron 3670_ |                                                 |
| :-------------- | ----------------------------------------------- |
| 主板            | B360                                            |
| cpu             | i5-9400                                         |
| 显卡            | UHD630(1.05GHZ)                                 |
| 网卡/蓝牙       | 奋威 T919（BCM94360CD）                         |
| 声卡            | Realtek ALC867 （layout id: 11)                 |
| 内存            | 8GB                                             |
| 硬盘            | KBG40ZNS256G NVMe KIOXIA 256GB+ ST1000DM010 1TB |
| 显示器          | DELL SE2417HG                                   |

## 已实现功能

- [x] 网卡蓝牙正常识别使用，支持airdrop 接力
- [x] VDA解码、核显满速
- [x] 声卡完美驱动
- [x] 修补HPET、RTC、TMR
- [x] ACPI注入缺失部件
- [x] 定制USB，快速充电
- [x] 定制核显接口，支持hdmi输出
- [x] 注入SBUS。摁下关机键跳出提示框
- [x] 加载原生电源管理，开启节能五项
- [x] 加载原生NVRAM
- [x] 完美hidpi
- [x] 可以解锁CFGlock
- [x] 目前测试支持休眠，完美睡眠

## 未实现功能

- [ ] [ ] VGA接口

## 不稳定功能

- [ ] [ ] 前置SD读卡器

## 1.0 Bios

### 1.1 Bios 设置

* 关闭Secure Boot

* 关闭Fast Boot

* SATAmode调整为AHCI

* 关闭PTT technology
* 关闭wake on USB（实际上不影响USB唤醒）
* 关闭wake on LAN
* 考虑关闭WIFI和Bluetooth，因为系统里不好驱动

### 1.2如何解锁Bios的CFGlock和DVMT

准备一个U盘，格式化为FAT32格式，并新建一个文件夹，命名为EFI。打开EFI文件夹，再新建一个名为 BOOT 的文件夹。
最后打开BOOT文件夹，把BOOTx64.efi文件放进去，这个同时也是解锁CFG的工具。打开UEFI Tool，然后打开你下载好的主板bios文件，不同的主板自行去官网下载。

* Ctrl+F搜索，选择GUID。
* 输入字符串:899407D799FE43D89A2179EC328CAC21。然后点击OK，下面会出来一栏，双击。接着会定位到Setup
* 右键Setup，选择第三个选项，输出FFS文件，记住路径。
* 保存后关闭软件，打开图中所指的那个软件
* 然后选择刚刚输出的FFS文件，然后点击Extract,输出一个txt文件，记住路径。
* 打开输出的txt文件，然后Ctrl+F搜索:CFG LOCK，然后定位到CFG LOCK，记住后面对应图片位置的代码。
* 搜索DVMT，记住后面最大值的代码
* 完成后重启电脑，进去主板快捷启动，选择U盘启动，然后进入了工具的主页面，
* 输入setup_var 05BE 0x0(关闭CFG lock）
* 输入setup_var 0x8DC 0x2（DVMT 64MB）
* 输入setup_var 0x8DD 0x3（DVMT Total MAX）
* 输入reboot重启。

这样你的bios的DVMT和CFG lock就弄好了！

### 1.3 DELL Bios 提取工具

https://github.com/vuquangtrong/Dell-PFS-BIOS-Assembler

然后将官网下载的BIOS程序拖到Dell_PFS_Extract上，就可以将里面的BIOS提取出来。

## 2.0 ACPI

ACPI是黑苹果最重要的一个课题，没有之一。ACPI是The Advanced Configuration and Power Interface，也就是“电源管理模式和配置管理的接口规范”，是bios中的一个高级功能模块。DSDT和SSDT是ACPI中的表格。DSDT的英文名称是The Differentiated System Description Table，它描述了你机器的设备，存在于你的bios中。MacOS对ACPI的支持不完整，只支持DSDT。黑苹果需要修改DSDT才能使硬件更好地驱动。有两种修改DSDT的方法。

第一种是重命名补丁。第二种是SSDT。SSDT就相当于DSDT的补丁，而重命名补丁直接在DSDT上修改。强烈推荐使用SSDT，因为SSDT中的代码可以实现操作系统的识别以及打断代码修正添加等功能，有更好的稳定性与兼容性。

由于ACPI也比较复杂。我就这台机器为例子，讲讲8、9代机器的ssdt配置。详细可以看Dortania和OC-little，做进一步研究。

### 2.1我需要哪些SSDT？

对于300系主板，主要需要：

* SSDT-PMCR或者通常成为SSDT-PMC

* SSDT-EC-USBX

* SSDT-PLUG

* SSDT-AWAC

* SSDR-PPMC

* SSDT-SBUS-MCHC

* 如果是DELL主板，可以再加一个SSDT-OCWork-dell

* 本机需要SSDT-IRQ

### 2.2 提取DSDT

尽管提取原始DSDT的方法非常多，我认为 `CLOVER` 的提取方法是最方便并且靠谱的。我们需要一个空的U盘或者空的ESP分区，我的教程是非常偏向小白的，所以这里提取我也会用到windows，以及Diskgenius这个软件，做最简单的示范。

* 进入Windows，插入U盘，打开[DiskGenius](https://www.diskgenius.cn/download.php)，选中我们的U盘，并选择顶部菜单栏的快速分区
  * 分区表类型：`GUID`
  * 不要创建新的 `ESP` 分区
  * 不要创建新的 `MSR` 分区
  * 分区格式为 `FAT32`
* 格式化完成后，放入我从黑果小兵镜像包提取出来的[EFI](https://blog.xjn819.com/tools/EFI.zip)放进去。这是一个 clover 引导，但并不能引导你的系统，只能提取 DSDT。
* 插上U盘，重启，通过U盘引导，看到Clover界面，我们按F4，这样原始的DSDT文件就收集好了。
* 重新通过OC引导进入系统，我们打开U盘，EFI_Clover_ACPI/Orgin下，有我们的原始ACPI内容，我们只需要DSDT.aml这个就行了，保存到安全的地方。

### 2.3 ACPI quirks设置

1. FadtEnableReset：False

2. NormalizeHeaders：False

3. RebaseRegions: False

4. ResetHwSig: False

5. ResetLogoStatus: False

6. SyncTableIds：False

## 3.0 Booter

### 3.1 quirks

1. **一般设置**

  打开debuglog，搜索： OCABC: MAT support is 1/0

  若MAT support is 1即MAT被支持，请打开**AvoidRuntimeDefrag、EnableSafeModeSlide、RebuildAppleMemoryMap、SyncRuntimePermissions**

  若MAT support is 0 即MAT不被支持，请打开**AvoidRuntimeDefrag、EnableSafeModeSlide、EnableWriteUnprotector**

2. **ProvideCustomSlide：NO**
   若debug log中出现 OCABC: Only N/256 slide values are usable!则需要打开

3. **ProtectUefiServices：NO**
   Z390主板请打开

4. **Slide的问题**

  故障提示如下：

  ```text
Error allocating 0x1197b pages at 0x0000000017a80000 alloc type 2
  ```

  先打开DevirtualiseMmio、ProvideCustomSlide，再做一次尝试。

  若仍然失败，请转到https://dortania.github.io/OpenCore-Install-Guide/extras/kaslr-fix.html#using-devirtualisemmio。

  Z390与HEDT主板可能需要，本主板不需要。

5. **ForceExitBootServices\SetupVirtualMap：NO**

  在启动失败时可尝试使用，一般不需要

6. **ProvideMaxSlide：0**

  官方解释：若在打开ProvideCustomSlide后出现了随机的启动失败，打开AppleDubug后debug log出现了AAPL: [EB|‘LD:LKC] } Err(0x9)的错误，则用slide=X来试最大的X的值，将它填入ProvideMaxSlide中。

  若没有这个故障，就填0

7. **AllowRelocationBlock:NO** 

  此功能用于10.7及更早的MacOS系统，这里不需要

8. **DisableVariableWrite：NO**

  非原生NVRAM的主板可能需要打开

9. **DisableSingleUser：NO**

  禁用单用户模式，不建议开启

9. **DiscardHibernateMap：YES**

   电脑从休眠(hibernation)中唤醒时,硬盘里的资料会恢复到内存中去，但这个时候 OC 的内核以及内核缓存等也会写入，这样可能导致冲突，这个选项是帮助我们解决这个问题的。

10. **ProtectMemoryRegions：NO**

    官方解释：The need for this quirk is determined by artifacts, sleep wake issues, and boot failures. This quirk is typically only required by very old firmware.

    如果对此不是很了解，不建议开启。

11. **ProtectSecureBoot：NO**
    请关闭安全启动，并关闭这个quirk

12. **ResizeAppleGpuBars：-1**

    - 此选项用于适配BIOS具备`Resizable BAR`功能的主板。BIOS支持此功能可以给Windows带来更好的显卡性能，但另一方面会导致MacOS不稳定。 本主板不需要此设置。
    - 填如下值对应相对功能：
      - 当值为`-1`时:关闭此功能 
      - 当值为`0`时: 预留1MB
      - 当值为`1`时: 预留2MB
      - 当值为`2`时: 预留4MB
      - 当值为`10`时: 预留1GB
      - 若要开启此功能建议填`0`

13. **ForceBooterSignature**

    休眠（hibernation）中唤醒使用OpenCore的SHA-1加密验证，一般不需要

## 4.0 DeviceProperties

### 4.1.1 核显

#### UHD 630（[Coffee Lake]((null)) 微架构，下文简称 CFL）

支持 macOS 10.14 或更新版本。

* 0x3EA50000（桌面版，缺省值）
* 0x3E9B0007（桌面版，推荐）
* 0x3EA50009（移动版，缺省值）

注意：使用第九代 Coffee Lake R 处理器时，需设定（仿冒）`IGPU` 的 `device-id` 为 `923E0000`。（如下所示）

_从 macOS Mojave 10.14.4 起，无需再设定此参数！_

#### 使用 WEG 自定义 FB 和 端口 补丁

一般来说WhateverGreen会自动完成大部分工作，不需要任何额外的Framebuffer补丁。

当出现以下情况可能需要使用额外的Framebuffer补丁：

* 在BIOS中无法设置超过32M的DVMT（framebuffer-stolenmem / framebuffer-fbmem）
* 为4K屏设定更大的VRAM（-unifiedmem）
* 禁用独显(disable-external-gpu)
* 启用支持4k的像素时钟补丁（enable-hdmi20）
* 禁用连接器以启用睡眠（framebuffer-pipecount / framebuffer-portcount / framebuffer-conX-type = -1）
* 更改连接器类型以匹配您的系统端口（framebuffer-conX-type）
* 等等

#### WEG 支持的自定义补丁列表

#### 语义补丁：

```
framebuffer-patch-enable (启用语义补丁的总开关)

framebuffer-framebufferid (要修改的 FB，一般保持默认即可)

framebuffer-mobile
framebuffer-pipecount
framebuffer-portcount
framebuffer-memorycount
framebuffer-stolenmem(给BIOS中DVMT增加内存大小）：
framebuffer-fbmem
framebuffer-unifiedmem (核显显存大小，调大一点可能能解决花屏）
framebuffer-cursormem (Haswell 专用补丁)
framebuffer-flags

framebuffer-camellia (集成显示控制器，仅与白苹果相关)

framebuffer-conX-enable (启用端口为 X 的修改)
framebuffer-conX-index
framebuffer-conX-busid
framebuffer-conX-pipe
framebuffer-conX-type
framebuffer-conX-flags
framebuffer-conX-alldata (完全替换端口信息)
framebuffer-conX-YYYYYYYY-alldata (在当前 FB 与 YYYYYY 匹配时完全替换端口信息)

X 是端口索引。
```

#### 二进制补丁：

```
framebuffer-patchN-enable (启用第 N 项补丁)
framebuffer-patchN-framebufferid (要修改的 FB，一般保持默认即可)
framebuffer-patchN-find
framebuffer-patchN-replace
framebuffer-patchN-count (要搜索的补丁号迭代数，默认为 1)
N 为补丁索引号: 0, 1, 2, ... 9
```

#### 部分补丁解释：

再次重申，所有DATA数据类型需要将数据两两一组倒过来填入，例如：`0x16260006`转换之后就是这样`06002616`填入，如下图：

当设置内存大小时，你可能想知道DATA是怎么计算出来的。用framebuffer-fbmem参数举例，当需要设置为48M之后它应填入的值是：`00000003`，这个也是转换后的值，所以原来的值应当是`03000000`，这是一个16进制的数字，转换成10进制是`50331648`。我们知道1M=1024KB，1KB = 1024B，所以，我们把转换成十进制之后的数字`50331648`除以1024然后再除以1024，得出的结果就是48了，所以这串数字代表的就是48M。

[点击这里前往进制转换网页](https://link.juejin.im/?target=https://tool.lu/hexconvert/)

1. `AAPL,ig-platform-id`（设备平台id，直接影响显卡是否能成功驱动）： 举例一些常用`笔记本`的核显id（均为DATA数据类型）

```
 - HD4600~HD5200：`0x0A260000`或`0x0A2E0008`
 - HD5300~HD6000：`0x16260006`
 - HD630：`0x3E9B0000`
```

2. `device-id`（设备id，可能是能让黑苹果正确显示设备信息，直接使用无需倒序）：

```
 - HD4600~HD5200：`12040000`
```

```
 - HD5300~HD6000：`16160000`
```

```
 - HD630：`3e9b0000`
```

```
   > 具体的`AAPL,ig-platform-id`和`device-id`的使用设置查看前文
```

3. `framebuffer-patch-enable`（是否启用framebuffer补丁，当然启用啊，不启用的话这篇文章还有什么用处）：

```
 - DATA数据：`01000000 `-> 1（启用） `00000000` -> 0（不启用）
 - NUMBER数据：`0`（不启用） `1`（启用）
```

4. `framebuffer-stolenmem`（给BIOS中DVMT添加一点内存大小，会影响高分屏，这个值必须大于32M，也不应该过高）：

```
 - 一般1080P屏幕的话，设置为48M就够用了：`00003001`
```

```
 - 当你的笔记本电脑屏幕是2k，你可以设置为64M：`00000004`
```

```
 - 4K屏的话，要设置为128M：`00000008`
```

```
   > 如果你的BIOS中可以设置DVMT的话并且你设置成为128M之后，可以不需要设置这个属性，或者这个属性设置小一点：`00003001` 保险起见，高分屏直接设置成128M比较稳，并且保证在BIOS能设置DVMT的情况下设置在64M或以下 （PS：这一部分可能有误，但是最后一句保险起见，高分屏直接设置成128M比较稳是试验过的）
```

5. `framebuffer-unifiedmem`（核显显存大小，调大一点可能能解决花屏）：

```
 - 2048M：`00000080`
 - 3072M：`000000C0`
```

6. framebuffer-cursormem（翻译成中文就是光标内存，会影响高分屏，比如高分屏花屏可能就是这个值不够大）：

```
 - 一般屏幕设置成9M大小就好：`00009000`
 - 高分屏的话最好直接设置成48M：`00000003`
```

7. `framebuffer-fbmem`（framebuffer内存大小，会影响高分屏）：

```
 - 一般屏幕设置成9M大小就好：00009000
 - 高分屏的话最好直接设置成48M：00000003
```

8. `framebuffer-conN-enable`（N为数字，显卡第N个输出接口是否启用，1为启用，0为不启用）：

```
 - DATA数据：`01000000` -> 1（启用） `00000000` -> 0（不启用）
 - NUMBER数据：`0`（不启用） `1`（启用）
```

9. `framebuffer-conN-type`（N为数字，显卡第N个输出接口的类型）：

```
 - HDMI输出：`00080000`
 - DP输出：`0004000`
```

10. `framebuffer-conN-index`（个人理解，显卡第N个输出接口的优先级，或者说是设置第N个输出口的位置）： 这个按个人需要设置，如果需要屏蔽这个输出口，可以设置成FFFFFFFF，也就是最大的数字，让它足够靠后，这样就达到了屏蔽效果！

#### HDMI 高分屏 60 fps 方案

为核显添加 `enable-hdmi20` 属性，或使用 `-cdfon` 启动参数代替，否则将会黑屏。

#### 笔者经验：

hidpi一直都是我头大的问题，一直有1600x900分辨率无法开启的问题。在反复探索后解决办法如下：
1，首先，开启隐藏的bios选项，开启DVMT64MB

2，删除framebuffer-stolenmem，这步十分关键，这个补丁会适得其反

3，framebuffer-unifiedmem 设为000000C0 即显存变为3072MB

4，用one-key-hidpi，开启hidpi



## 5.0 Kernel

### 5.1 我需要哪些kext？

**请密切关注kext的顺序！**

- [x] Lilu.kext
- [x] VirtualSMC.kext
- [x] WhateverGreen.kext
- [x] AppleALC.kext
- [x] NVMeFix.kext
- [x] RealtekRTL8111.kext
- [x] SMCDellSensors.kext
- [x] SMCProcessor.kext
- [x] XHCI-unsupported.kext
- [x] USBPorts.kext

注意：最后两个是USB定制的文件，后文会讲到



### 5.2 Quirks

1. **AppleCpuPmCfgLock:：NO**

   - 四代之前的 CPU，如果未解锁 CFG(即MSR0xE2) 请选择 `YES`。

2. **AppleXcpmCfgLock：NO** 

   - 四代之后的CPU若未解锁 CFG(即MSR0xE2) 请选择 `YES`。

3. **AppleXcpmExtraMsrs：NO** 

   - 主要在没有原生电源管理的CPU上启用，一般是 `Haswell-E`, `Broadwell-E`, `Skylake-X` 这三种 CPU 需要填写 `YES`。除此之外的 CPU 选择 `NO`。

4. **AppleXcpmForceBoost：NO**

   - 开启时将电脑的 cpu 频率锁定为最高频率。

5. **CustomPciSerialDevice：NO**

   串口的问题，没有设备不做讨论

6. **CustomSMBIOSGuid：NO** 

   - 戴尔笔记本专用项。

7. **DisableIoMapper：YES（NO）**

   - 禁用 vt-d。

8. **DisableLinkeditJettison：YES** 

   - 提升 `lilu` 等插件在 MACOS 11 系统的表现，用来替代 `keepsyms=1`。

9. **DisableRtcChecksum：NO**

   - 越过两条 rtc 检查 (0x58及0x59)。RTC 我们会更多地使用 `RTCMemoryFixup.kext` 来防止它。

10. **ExtendBTFeatureFlags：NO**

    - 代替 `BT4LEContinuityFixup.kext` 来实现 `continuity`。

11. **ExternalDiskIcons：NO**

    - 修复苹果系统把内部硬盘识别为外置硬盘时（黄色图标的硬盘）开启。

12. **ForceAquantiaEthernet：NO**

    Aquantia AQtion based 10GbE network cards相关，没有设备，不做讨论

13. **ForceSecureBootScheme：NO**

    - 只在MacOS安装在虚拟机并开启`SecurebootModel`时考虑开启。

14. **IncreasePciBarSize：NO** 

    - 解决卡 PCI configuration，一般卡 pci configuration 都是因为自己错误的设置和硬件问题。

15. **LapicKernelPanic：NO**

    - 适用于HP笔记本的内核奔溃选项。

16. **LegacyCommpage：NO** 

    - 老平台主板中使用 ssse3 需要开启来使用 macOS 10.4 - 10.6。

17. **PanicNoKextDump：NO**

    - 防止 `kext` 出错打报告而让我们看不到真正的 `panic` 原因，初始排错时最好打开。

18. **PowerTimeoutKernelPanic：YES** 

    - 一些设备自身的电源管理无法让系统进入睡眠而超时，导致内核奔溃，我有这个问题，故选择 `YES`。

19. **ProvideCurrentCpuInfo：NO** 

    - 给hyper-V虚拟化MacOS时提供cpu的信息，保证全核同步。

20. **SetApfsTrimTimeout：0（-1）** 

    * APFS文件系统根据空间是否`被使用`或者是`空的`来设计的，这种设计跟其他文件系统是不同的，一般被设计成`被使用`，`空的`或者`未映射`三种情况。这种不同可能带来在启动时系统需要更多的时间来做trim，甚至导致无法进入系统。这个选项是为了设计一个时间节点来终止trim进入系统。一般来说填入`-1`是不阻止这个trim过程，但如果你有这个问题，你可以设置一个时间，单位为微秒。

    * 此设置不影响系统内的Trims，只是在开机时跳过Trims检查
    * 在macOS 12.0及更高版本上，不再可能指定Trims超时。但是，可以通过设置0来禁用它。

21. **ThirdPartyDrives：NO**

- 开启 SATA 类 SSD 的 `trim` 功能，我没有 SATA 类的 ssd，我选择 `NO`。自行根据情况选择。

22. **XhciPortLimit：NO** 

- 解除 15 个 USB 端口限制。11.3系统不要开启，并自行定义usb端口。

## 6.0 Misc

![Misc](https://dortania.github.io/OpenCore-Install-Guide/assets/img/misc.39d503c8.png)

* 更详细的Debug设置请转到https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/debug.html

* 更详细的Security设置请转到https://dortania.github.io/OpenCore-Post-Install/universal/security.html

## 7.0 NVRAM

### 7.1 Config–NVRAM–Add 

```
4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14
    DefaultBackgroundColor      Data       <00000000> //默认开机背景色为黑色
 
7C436110-AB2A-4BBB-A880-FE41995C9F82
    boot-args                   String     Slide=1 darkwake=0 -v //slide=1 表示从第一组内存开始连续注入；darkwake=0 代表一键唤醒机器并偏好设置中节能选项的小憩功能。如果你要用小憩功能请填8； -v 是跑代码，在没装好稳定的黑果前我建议加上，方便定位错误，弄完后再删除 -v
    csr-active-config           Data       <e7030000> //关闭 SIP 保护
    nvda_drv                    Data       <31> //对 10.13 系统之前的N卡的相关设置，我们不做讨论。
    prev-lang:kbd               Data       <7a682d48616e733a323532> //语言设置相关，记得改成这个，这个是中文
    ForceDisplayRotationInEFI   Number     0   开机的UEFI界面是否需要调整，比如你是竖屏的，可以填0,90,180,270。请同时开启AppleEg2Info
    7C436110-AB2A-4BBB-A880-FE41995C9F82 //默认就行，如果需要使用 RTC 屏蔽选项，具体参考RTC综述
```

------

### 7.2 Config–NVRAM–Delete

NVRAM的数据不可被覆盖，必须先被删除再添加，我们这里按默认设置不必理会。

------

### 7.3 Config–NVRAM–LegacySchema

这里是模拟 `NVRAM` 的变量设置，大部分默认已经填好，我们只需添加两个变量即可。

打开 `7C436110-AB2A-4BBB-A880-FE41995C9F82` 这一栏，添加两个item如下：

```
item 11     String      efi-boot-device
item 12     String      efi-boot-device-data
```

------

### 7.4 Config–NVRAM–LegacyEnable

如果你的主板不支持原生 `NVRAM`，请一定要选择 `YES`，反之则选择 `NO`。

## 8.0 PlatformInfo

可以选择的机型有iMac与Macmini，因为iMac pro和Mac Pro没办法驱动核显。虽然iMac机型的性能释放更好，但是它对电能小憩以及休眠极其不友好，所以我换成了Mac mini的机型，只有这样才能完美地睡眠。

![PlatformInfo](https://dortania.github.io/OpenCore-Install-Guide/assets/img/smbios.46661610.png)

## 9.0 UEFI

没有什么可以讲的，直接上图

![UEFI](https://dortania.github.io/OpenCore-Install-Guide/assets/img/aptio-v-uefi.229e6a50.png)

## 10.0 如何定制USB？

macOS 10.14.1+ 的USB端口限制补丁已经失效了，因此无法一次配置所有端口。 [RehabMan](https://github.com/RehabMan)已更新 `USBInjectAll.kext` 并已包含用于排除端口组的引导标志。

1. 将 `USBInjectAll.kext` (用于端口发现) 放入 `EFI/CLOVER/kexts/Other`

2. 转到 工具栏 ▸ `USB` 来查看 USB 控制器列表。因为这里需要基于 USB控制器 您可能需要安装额外的 kexts:

* 8086:8CB1 和 macOS (10.11.1) ▸ 请使用 `XHCI-9-series.kext`
* 8086:8D31, 8086:A2AF, 8086:A36D, 8086:9DED ▸ `请使用 XHCI-unsupported.kext`
* 8086:1E31, 8086:8C31, 8086:8CB1, 8086:8D31, 8086:9C31, 8086:9CB1 ▸ `请使用 FakePCIID.kext + FakePCIID_XHCIMux.kext`

3. 如果您缺少了其中一个附加的 kexts，*请完成安装并立即重新启动*，然后再次运行 Hackintool

4. 转到 工具栏 ▸ `USB` 选项，使用下拉列表将每个端口设置为适当的接口类型

- [ ] [ ] 永久连接设备的USB端口（例如M.2蓝牙卡）应设置为 `Internal` (内建)
- [ ] [ ] 与 USB3 端口相连的 HSxx 端口 (USB2) 应设置为 `USB3`
- [ ] [ ] 内部集线器通常连接到端口PR11和PR21，因此应设置为 `Internal` (内建)
- [ ] [ ] USB Type-C 接口可以是9或10，这取决于硬件如何处理 USB Type-C 型设备/电缆的正反两种可能方向
- [ ] [ ] 如果 USB Type-C 在两个方向上使用相同的 SSxx，则它具有内建切换器，因此应设置为 `TypeC+Sw`
- [ ] [ ] 如果 USB Type-C 在两个方向使用不同的 SSxx，则它没有内建切换器，因此应设置为 `TypeC`

5. 使用 `导出` 按钮在桌面上生成 USB 修复文件，找到并添加 USBPorts.kext至Opencore

## 11.0 RTC综述

这篇文章试图教育用户解决RTC(CMOS)记忆体相关的错误导致无法开机，卡F1，无法关机，时钟停止，无法唤醒等问题。此文章为《使用OpenCore引导黑苹果》的补充内容，因原文中RTC相关内容较为散乱，而特意补充。

---

> 特别提醒：在使用以下方式修复rtc的过程之前，强烈建议先短接主板CMOS以及清除NVRAM后再进行操作！NVRAM和CMOS存储位置不同，请确定两件事都做了！！

### 11.1 RTC的问题表现

* 华硕等主板开机卡F1
* 开机卡在PCI Configuration代码处
* 睡眠后唤醒奔溃，得到的错误报告为：Sleep Wake Failure in EFI
* 无法关机
* 电脑时钟不走，或者睡眠唤醒后时钟不走

### 11.2 RTC如何工作

RTC为l2c下的从设备，rtc为系统提供时间，电源，硬件信息等指示数据，l2c主动联系从设备rtc获取信息，反馈给系统。这些信息都被注册在rtc的各个内存位置上，这些内存地址对应的数据可通过OC-liilte或者搜索自己主板型号的intel datasheet获得。

> 举例：
> 系统设定一个自动关机的时间，比如22:20分，此信息通过系统trigger到rtc，l2c扫到rtc信息，则自动关机。

### 11.3 苹果系统下RTC的问题如何产生？

问题主要是因为Apple。

* Apple公司使用自己定义的Apple RTC，而普通主板可能引用了最新的AWAC装置来代替传统RTC而导致无法开机。
* Apple公司使用自己定义的Apple RTC，Apple RTC记录的内存范围可能不被我们普通的主板所支持，超出了我们普通主板0x70-0x78范围而导致了错误。

### 11.4 如何解决上述问题

对于《苹果系统下RTC的问题如何产生？》，第一个问题是这样的：一些具有AWAC的设备我们可能需要禁用AWAC来启用Legacy RTC。这里可通过[OC-Little]((null))包中的禁用AWAC来实现。其中SSDT-AWAC.dsl更适用于主板DSDT代码特别多的主板，请根据情况二选一并转换成aml后使用。

---

对于第二个问题，它可能带来的影响包括时钟不走，关机不断电，开机卡f1，无法唤醒，唤醒崩溃等。这种问题就像我之前说的，是因为apple rtc写入到了普通主板rtc不支持的范围而导致的。

我之前在主文章有写到通过[二进制补丁](https://github.com/RehabMan/HP-ProBook-4x30s-DSDT-Patch/blob/master/config_parts/config_master.plist#L291L296)来解决开机卡F1，但这种方式其实是不妥当的，这是因为：

* 这种补丁会因为系统版本的更新而失效
* 并不能控制Efiboot来写入RTC
* 卡F1的因素过多，而二进制补丁不可能具有通用性

因此，OC团队提供了三种方式来更好的解决RTC引起的各种问题。

* 提供DisableRtcChecksum选项来忽略RTC地址中的0x58-0x59范围。但遗憾的是此quirk只适用于0x58-0x59地址损坏的rtc情况，并不具备通用性。
* ACDT团队提供了[RTCMemoryFixup.kext](https://github.com/acidanthera/RTCMemoryFixup/releases)让用户可以自主选择所有你想屏蔽的地址。此kext需要用二分法来使用。
* 将此kext放入OC/Kexts下面，并在config中加载它。
* 一般来说，我们CMOS的总内存池是从00-FF（这个是16进制，换算成10进制就是从0-255)，我们可以通过增加boot-args:rtcfx_exclude=00-FF来完全屏蔽cmos（当然这样写你完全失去了cmos记忆的功能了）
* 我们需要通过二分法来定位你出错的cmos位置。把00-FF分成两部分，也就是00-7F以及80-FF。我们分别填一次rtcfx_exclude=00-7F以及rtcfx_exclude=80-FF，试试看问题有没有解决。比如说我使用的rtcfx_exclude=80-FF是解决了，那我们继续对80-FF进行拆分为：0x80-0xBF 和 0xC0-0xFF。以此类推，直到你拆分到最后的那一段位置为止。
* ACDT团队提供了写入NVRAM的方式来屏蔽RTC地址。如图，RTC-blacklist中 8586878889 代表屏蔽85 86 87 89的RTC地址，如果屏蔽85以及86则填写8586。注16进制。
  [image:16C905AA-8D38-4B02-A9A3-687BE47C739A-57229-00004F488F926E16/4SjaDbtdNivyOVA.png]rtc-blacklist.png



## 12.0 非原生主板模拟NVRAM

对 OC 而言，`NVRAM` 是非常核心的一环，不管是原生还是模拟的。如果你是原生nvram的主板，请不必理会这章节。

So this section is for those who don't have native NVRAM, the most common hardware to have incompatible native NVRAM with macOS are X99 and some X299 series chipsets:

* X99
* X299

For B360, B365, H310, H370, Z390 users, make sure you have [SSDT-PMC (opens new window)](https://dortania.github.io/Getting-Started-With-ACPI/)both under EFI/OC/ACPI and config.plist -> ACPI -> Add. For more info on making and compiling SSDTs, please see ：

*Note*: 10th gen CPUs do not need this SSDT

这章节的主要内容为非原生 NVRAM 模拟生成 `nvram.plist`。

* 首先打开我们之前下载好的 OpenCore，进入目录下的 `Utilities/LogoutHook` 文件夹，你会看到 `LogoutHook.command` 文件。
* 打开 Terminal（终端）并输入 `cd ~` 返回到用户根目录，输入 `mv` 和一个空格，然后鼠标拖动 `LogoutHook.command` 文件到 Terminal（终端），再在终端输入 `~/.LogoutHook.command`

例如（不要直接复制这条命令行）
 mv _Users_xjn_Downloads_OpenCore-0.6.2-RELEASE_Utilities_LogoutHook_LogoutHook.command ~_.LogoutHook.command

* 最后在 Terminal（终端）中，输入以下命令按回车：

sudo defaults write com.apple.loginwindow LogoutHook _Users_$USER/.LogoutHook.command

 * 终端会提示要求你输入密码（密码打进去不会显示）。
* 重启，你会在 `ESP/EFI/` 下看到 `nvram.plist`，代表已经成功模拟了。

## 13.0 睡眠综述

### 13.1 强制睡眠

睡眠即醒很大程度上跟 USB 的定制相关，一般一个好的 USB 定制就能解决睡眠即醒的问题。当然还有很多无法解决的问题，比如蓝牙不能在 HUB 下进行内建，等等。甚至有些时候我们都不知道为什么黑果会睡不着，那有没有一个办法让黑果强制睡眠呢？答案是有的。经过我的摸索，有几种方法能达到强制睡眠的效果，只是方法不同而已，但主要围绕的还是 `0d/6d` 的数值来做一些工作。这些方法涉及了很多 OC 领域的一些小技巧，我也顺便展示给大家。

> `0d/6d` 补丁是阻止一些部件参与唤醒工作，这其中包括了xhc部件，意味着你无法使用鼠标键盘唤醒，只能用电源键唤醒。但若你有一组除了xhc之外的usb控制器，那把键盘鼠标插在那两个控制器上，可以在使用强制睡眠的情况下用键盘鼠标唤醒电脑。    

- [ ] - - -

#### 13.1.1 分辨0D/06

主板一般有5个部件是直接参与唤醒工作的，这五个部件分别是 XHC（USB控制器）、CNVW（CNVI网卡，如果你的主板自带或者预留了口的话）、GLAN（有线网卡）、XDCI（USB相关）、HDEF（音频）。旧的一些主板可能会有不同的命名，比如 XHC 有叫 EH01，HDEF 叫做 HDAS 等，这里不做讨论。而这些设备往往会直接影响睡眠，比如你输入：

```
log show --last 1d | grep "Wake reason"
```

我们会看到类似的输出结果

```
2020-10-31 03:35:45.196371+0800 0x74       Default     0x0                  0      0    kernel: (AppleACPIPlatform) AppleACPIPlatformPower Wake reason: XDCI CNVW
2020-10-31 03:35:45.196373+0800 0x74       Default     0x0                  0      0    kernel: (AppleACPIPlatform) AppleACPIPlatformPower Wake reason: XDCI CNVW
```

那么即 是`XDCI` `CNVW` 导致了睡眠出现了问题。于是，我们用几种方法去屏蔽或者说修改这些部件，来达到电脑正常睡眠的效果。

我们打开之前提取的SSDT，随便搜索五大部件中的一个，比如说 XDCI：
![截屏2019-11-07下午11.06.02.png](https://i.loli.net/2020/10/31/k5jDCx9c3WdJunh.png)

主要是看上图中 `XDCI` 下的 `_PRW` 属性值，可以直接看到 Return 的值为 `GPRW (0x6D, 0x04)`。其中 `6D` 这个数值看主板而定，有些主板叫做 `0D`，而后面 `04` 这个值的含义为 S4 级别的电源管理，即休眠甚至关机情况下的唤醒；有些后面的数值是 `03`，代表着 S3 级电源管理。这个我打一个大家比较熟悉的例子， `GLAN` 这个网卡部件的 `PRW` 值也是 `0x04`，为什么要是 `04` 呢？因为这样我们可以使用远程通过网络启动主机功能。

#### 13.1.2 方法一: OC 全局重命名强制睡眠

上一步中已经确认了你的主板是 `0D` 还是 `06`，打开 `OC little` 的 `06/0D` 补丁，选择合适自己主板的补丁集，比如我的是 `Name-6D更名.plist`。将补丁抄入自己的 `config.plist` 后重启生效。

> 全局重命名会导致其他系统无法通过 OC 引导开机，不建议使用。    

- [ ] - - -

#### 13.1.3 方法二: 沿用Clover版本的0D/06补丁&展示TgtBridge在OC下的用法

#### 3.7.1.2 方法二: 沿用Clover版本的0D/06补丁&展示TgtBridge在OC下的用法

宪武大大做的 clover 版本的 0d/6d 补丁，其实没啥必要讲，只是有留言问了 tgtbridge 在 oc 下怎么用，那我就展示一下吧。这个补丁原理是一样的，通过重命名的方式改 `_prw`。

直接下载宪武大大的[clover hotpatch](https://github.com/daliansky/P-little/tree/master/部件补丁包/11-1-睡了即醒(0D:6D)补丁)补丁包，打开plist文件。那我们拿出一组数据来讲解怎么把它翻译成oc版本：

```
Comment      String       XHC:_PRW to XPRW
Disabled     Boolean      True//此补丁并未生效，这里要改成false才会生效
Find         5F505257         //hex转text的含义即是：_PRW 
Replace      58505257         //hex转text的含义即是：XPRW 
TgtBridge    5848435F         //hex转text的含义即是：XHC_
```

这组改名是对 `XHC` 下的 PRW 改名为` xprw`，这样的话，之前 prw 下的 `(0x6D, 0x04)` 即不生效了。而指定 xhc 的方法即是使用了 tgtbridge，因为整张 dsdt 上有几十上百个 `_PRW`，你必须通过 tgtbridge 来指定到底是哪一个部件的 `_PRW`。

那么 OC 到底怎么使用 tgtbridge 来特定某一部件下的内容重命名呢？我们先把上面一段 clover 的补丁转换成 oc 的版本先吧：

```
Comment           String        XHC:_PRW to XPRW
Count             Number                      //需要重点解释
Enabled           Boolean       True          //表示应用此补丁，不应用选False
Find              Data          5F505257      //hex转text的含义即是：_PRW
Limit             Number        0             //这个按默认即可 不去管他
Mask              Data          <>            //这个按默认即可 不去管他
OemTableId        Data          <>            //这个按默认即可 不去管他
Replace           Data          58505257      //hex转text的含义即是：XPRW
ReplaceMask       Data          <>            //这个按默认即可 不去管他
Skip              Number                      //需要重点解释
TableLength       Number        0             //这个按默认即可 不去管他
TableSignature    Data          44534454      //hex转text的含义即是：DSDT，这里按默认即可，代表对dsdt进行修改
```

这里就是一个还没全部翻译好的 oc 版改名 xhc 的 prw。那么如何定位 xhc 下的 `_prw` 呢，主要是填写 `Count` 和 `Skip`。其实 oc 的 `tgtbridge` 是通过一个个数过去来定位具体哪一个位置的。比如xhc的prw是整张dsdt里面的第55个，那 `skip` 填 `54`，意味着跳过前 `54` 个，从第 `55` 个开始执行。那执行多少次呢？执行一次 `count` 就填 1；比如你要同时改第 `55` 个和 `56` 个，那 count 就填 `2`。说了这么多，我来实操一下吧：

打开dsdt，在左下角直接搜索_PRW，就能把整张表的_PRW筛选出来了：
![截屏2019-11-08上午2.57.43.png](https://i.loli.net/2020/10/31/EfxazsJNZLKmgv3.png)

我总共数了一下，一共有56个_PRW。我们再在主内容栏上按 ⌘ +F 搜索 xhc，直接找到 xhc 的 `_PRW`，刚好我们看到我的 xhc 实在整张表的倒数第 4 个，也就是正数第 53 个：

![截屏2019-11-08上午3.00.36.png](https://i.loli.net/2020/10/31/SzvArXcH4OimtR9.png)

那么我们就可以补充完整张表了：

```
Comment          String       XHC:_PRW to XPRW
Count            Number       1
Enabled          Boolean      True
Find             Data         5F505257
Limit            Number       0
Mask             Data         <>
OemTableId       Data         <>
Replace          Data         58505257
ReplaceMask      Data         <>
Skip             Number       52
TableLength      Number       0
TableSignature   Data         44534454
```

如果你想第 53、54、55 个都改掉，那 count 就写 3，意味着顺序执行3次。好了，就这样，有问题留言。

- [ ] - - -

#### 13.1.4 方法三：配合SSDT+重命名的强制睡眠补丁（推荐）

oc 不提倡用户直接全局重命名，如果真的要用重命名，也一定是搭配 ssdt 去做重命名，所以这个方法也是宪武大大和我最推荐的一种方法。

打开宪武大大的 OC-SSDT 包，找到 `0D/6D` 文件夹，打开 `SSDT-GPRW.dsl`。

```
// In config ACPI, GPRW to XPRW
// Find: 47505257 02
// Replace: 58505257 02 //这里提示你要应用这个补丁，你必须在config中的ACPI-PATCH里面加入如上重命名内容
//
DefinitionBlock ("", "SSDT", 2, "ACDT", "GPRW", 0)
{
  External(XPRW, MethodObj) //寻找dsdt表中叫做XPRW的内容，这是要你在config中先把gprw改名成xprw才会生效，这就是为什么这个补丁的重命名必须是这个ssdt和重命名一起用的原因，你第一个重命名不生效，这个ssdt也不会生效。
  Method (GPRW, 2, NotSerialized)
  {
    If (_OSI ("Darwin")) //为了不破坏dsdt完整性，这里做了系统判断，当你运行windows的时候，此ssdt不生效
     {
       If ((0x6D == Arg0)) //如果你的dsdt中是6D进行判断
       {
         Return (Package ()
         { 0x6D,
           Zero
          })
       }
 
       If ((0x0D == Arg0)) //如果你的dsdt中是0D进行判断
       {
         Return (Package ()
         { 0x0D,
           Zero
          })
        }
      }
  Return (XPRW (Arg0, Arg1)) //当运行mac系统时，如果你的dsdt中XPRW为6d，或者0d时返回为0，即屏蔽。
  }
}
```

这个 ssdt 不需要你改任何内容，打开后左上角另存为（save as), 其中文件格式(file format)必须选择 `ACPI Machine Language Binary`，文件名字就叫，记住后缀为 aml。记得将 `ssdt-gprw.aml` 放入 `EFI/OC/ACPI` 目录下，并在 `config.plist` 中加载此 aml 文件。

同时，我们需要在 `ACPI--Patch` 下增加一条全局重命名来配合此 SSDT。

```
Comment: GPRW to XPRW
Count:0
Enabled:YES
Find:<4750525702>
Limit:0
Mask:<>
OemTable:<>
Replace:<5850525702>
ReplaceMask:<>
Skip:0
TableLength:0
TableSignature:<>
```

### 13.2 Wake failure in EFI

这个问题非常复杂，但也经常出现。是属于疑难杂症的一类，它要求所有黑苹果问题全部解决

主要有这么几类：

* Opencore配置出现**错误**
* USB定制
* RTC的问题
* CPU不恰当的频率设置，如CPUFriend的定制
* 不恰当的PCIE设备的设置，如显卡
* Bios设置

这几项在前文已有涉及，不再赘述，如果仍然不能解决，建议继续折腾或者去论坛求助。

笔者的这个机器需要调整bios的设置，才能完美睡眠，否则就会睡死。具体的bios设置见上文。

## 14.0 CPUFriend

此章节对你的要求会相对高一点，并且请你具备如下条件：

* 有耐心
* CFG已经解锁（这也许不是必要条件）
* 已经打开原生电源管理。
* 此章节只适用于4代之后的CPU。
* 在调整CPU变频时，出现失误导致CPU温度过高，能有正确的处理能力保证CPU不烧毁，我对极端的后果不负责任。
* 此教程对无核显用户不友好，需要自己更多的领悟。
* 如果你不同意第五条，请不要看下去。

在Intel四代之后，苹果引入了新的内核级电源管理方式：`XCPM`（XNU CPU Power Management），这种新的管理方式可以高效地管理电源及变频。同时，苹果也推出了`HWP` (HardWare controlled Performance states)，这种技术可以快速根据特定程序的需求，作出变频转换。我们这个章节，本质上就是在加载`XCPM`的情况下，调整`HWP`来优化CPU的变频。

同时我要说的是，我在论坛上看到很多所谓的“变频”，有的甚至加载了50多个档位的变频，其实这种是完全没有意义的。我认为，变频是能在你需要的高频的时候快速进入高频状态，在关闭程序后又能很快回到低档位，换句话说，其实只需要三个档位就高了：睿频档，正常频率，以及低频档。

* 你需要搞清楚自己的cpu型号，并找到、换成与自己cpu型号最接近或者一样的白苹果型号，并且此型号必须带有`HWP`。一般新的笔记本机型都具有，台式机的话推荐iMac19.1以及Macmini8.1。可查询[此帖](https://blog.daliansky.net/A-command-to-teach-you-how-to-confirm-their-own-models-and-how-to-open-the-HWP.html)选取适合的机型。
* 执行如下命令：

cd ~/desktop
mkdir cpu
cd cpu
git clone https://github.com/corpnewt/CPUFriendFriend.git
git clone https://github.com/acidanthera/CPUFriend.git cp ~/desktop/cpu/CPUFriend/tools/ResourceConverter.sh ~/desktop/cpu/
CpuFriendFriend/CPUFriendFriend.command

* 你会看到如图的命令行，这里1 of 4代表第一段睿频的设置，以此类推，数值越大睿频越高，下面要求你填写的是最低的频率值，你想要低一点的800MHZ就填08，高一点的1300MHZ就填0D（注意大小写）
* * 填完前两段后，它会要求你填写EPP值，EPP值越低，性能表现越强。我们是填的前两段的低频率部分，我们可以选择节能型的，比如0x80，如果你想极致性能，可以填0x00。
* 直至你填完所有4段变频需求后，便会生成你的变频plist。我们执行以下命令:

cd ~/desktop
mkdir cpu
cd cpu
git clone https://github.com/corpnewt/CPUFriendFriend.git
git clone https://github.com/acidanthera/CPUFriend.git cp ~/desktop/cpu/CPUFriend/tools/ResourceConverter.sh ~/desktop/cpu/
CpuFriendFriend/CPUFriendFriend.command

* 我们会在桌面的CPU文件夹中找到你所需要的`ResourceConverter.sh`以及Mac-xxxxxxx.plist两个文件
* 执行以下命令生成你最终需要的`CPUFriendDataProvider.kext`,注意命令行中`Mac-AA95B1DDAB278B95.plist`，请替换成你自己的文件名，这样我们就可以在桌面的CPU文件夹下拿到`CPUFriendDataProvider.kext`。

cd ~/Desktop/cpu
./ResourceConverter.sh --kext ~/Desktop/cpu/Mac-AA95B1DDAB278B95.plist

* 我们再到CPUFriend的[release](https://github.com/acidanthera/CPUFriend/releases)页面下，下载最新的release版本，得到里面的CPUFriend.kext
* 将`CPUFriendDataProvider.kext`与`CPUFriend.kext`一起放到oc/kexts下,并在config中加载，注意：`CPUFriend.kext`应该放在`CPUFriendDataProvider.kext`的前面。
* 完成，自行测试。

## 15. 修补 DRM

请参考[WhateverGreen's DRM](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.Chart.md)



## Q&A：前置SD卡能不能驱动？

等了许久，前置SD卡终于是有驱动了：RealtekCardReader，但是驱动问题较大，每次睡眠就会意外弹出，并且容易系统崩溃，暂且不使用，观望中……

## *Credits* ：

[*https://blog.xjn819.com/post/opencore-guide.html*](https://blog.xjn819.com/post/opencore-guide.html)

[*https://blog.zuiyu1818.cn/posts/Hac_Intel_Graphics.html*](https://blog.zuiyu1818.cn/posts/Hac_Intel_Graphics.html)

[*https://blog.zuiyu1818.cn/posts/Hac_Intel_Graphics_simple.html*](https://blog.zuiyu1818.cn/posts/Hac_Intel_Graphics_simple.html)

https://www.insanelymac.com/forum/topic/342002-darkwake-on-macos-catalina-boot-args-darkwake8-darkwake10-are-obsolete/

https://www.insanelymac.com/forum/topic/299222-is-power-nap-possible-for-hackintosh/#comment-2037299

https://github.com/daliansky/OC-little

https://blog.daliansky.net/clover-user-manual.html

https://github.com/acidanthera/WhateverGreen/tree/master/Manual

https://blog.daliansky.net/OpenCore-BootLoader.html

https://blog.daliansky.net/Tutorial-Using-Hackintool-to-open-the-correct-pose-of-the-8th-generation-core-display-HDMI-or-DVI-output.html

https://blog.daliansky.net/Intel-FB-Patcher-tutorial-and-insertion-pose.html

https://blog.xjn819.com/post/rtc-issues-related-to-oc.html

https://blog.xjn819.com/post/kernel-panic-report-analysis.html

https://github.com/corpnewt/SSDTTime

https://blog.daliansky.net/From-Clover-To-OpenCore.html

https://oc.skk.moe

https://dortania.github.io/

https://macx.top/2768.html

https://www.longzc.cn/archives/330

https://www.longzc.cn/archives/308

https://win2mac.top/d07d3078.html

https://www.bilibili.com/read/cv6167464

https://apple.sqlsec.com

https://www.kancloud.cn/chandler/mac_os/482278#SSDTDSDT_9

https://www.bugprogrammer.me

https://blog.skk.moe/
