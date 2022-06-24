# # hackintosh-Inspiron3670-franksfc
image

# 我的opencore引导
我折腾的过程比较漫长，从下载黑果小兵的镜像用免驱独显，到拔掉独显定制opencore核显，最后不断完善具体的功能，直到黑苹果接近完美。给大家分享一下我的折腾成果，以及我的理由。

## 机型配置：
| _Inspiron 3670_ |                                                 |
| :------------ | ----------------------------------------------- |
| 主板          | B360                                            |
| cpu           | i5-9400                                         |
| 显卡          | UHD630(1.05GHZ)                                  |
| 网卡/蓝牙     | 奋威 T919（BCM94360CD）                |
| 声卡          | Realtek ALC867 （layout id: 11)                 |
| 内存          | 8GB                                             |
| 硬盘          | KBG40ZNS256G NVMe KIOXIA 256GB+ ST1000DM010 1TB |
| 显示器        | DELL SE2417HG                                   |

## 已实现功能
- [x]  网卡蓝牙正常识别使用，支持airdrop 接力
- [x]  VDA解码、核显满速
- [x]  声卡完美驱动
- [x]  修补HPET、RTC、TMR
- [x]  ACPI注入缺失部件
- [x]  定制USB，快速充电
- [x]  定制核显接口，支持hdmi输出
- [x]  注入SBUS。摁下关机键跳出提示框
- [x]  加载原生电源管理，开启节能五项
- [x]  加载原生NVRAM
- [x] 完美hidpi
- [x] 可以解锁CFGlock
- [x] 目前测试支持休眠，完美睡眠

## 未实现功能：
- [ ] [ ] VGA接口

## 不稳定功能：
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

### 2.3 提取DSDT
尽管提取原始DSDT的方法非常多，我认为 `CLOVER` 的提取方法是最方便并且靠谱的。我们需要一个空的U盘或者空的ESP分区，我的教程是非常偏向小白的，所以这里提取我也会用到windows，以及Diskgenius这个软件，做最简单的示范。

* 进入Windows，插入U盘，打开[DiskGenius](https://www.diskgenius.cn/download.php)，选中我们的U盘，并选择顶部菜单栏的快速分区
	* 分区表类型：`GUID`
	* 不要创建新的 `ESP` 分区
	* 不要创建新的 `MSR` 分区
	* 分区格式为 `FAT32`
* 格式化完成后，放入我从黑果小兵镜像包提取出来的[EFI](https://blog.xjn819.com/tools/EFI.zip)放进去。这是一个 clover 引导，但并不能引导你的系统，只能提取 DSDT。
* 插上U盘，重启，通过U盘引导，看到Clover界面，我们按F4，这样原始的DSDT文件就收集好了。
* 重新通过OC引导进入系统，我们打开U盘，EFI_Clover_ACPI/Orgin下，有我们的原始ACPI内容，我们只需要DSDT.aml这个就行了，保存到安全的地方。

### 2.4 原生电源管理
DSDT中搜索 `Processor`，如：

```
    Scope (_SB)
    {
        Processor (PR00, 0x01, 0x00001810, 0x06){}
        Processor (PR01, 0x02, 0x00001810, 0x06){}
        Processor (PR02, 0x03, 0x00001810, 0x06){}
        Processor (PR03, 0x04, 0x00001810, 0x06){}
        Processor (PR04, 0x05, 0x00001810, 0x06){}
        Processor (PR05, 0x06, 0x00001810, 0x06){}
        Processor (PR06, 0x07, 0x00001810, 0x06){}
        Processor (PR07, 0x08, 0x00001810, 0x06){}
        Processor (PR08, 0x09, 0x00001810, 0x06){}
        Processor (PR09, 0x0A, 0x00001810, 0x06){}
        Processor (PR10, 0x0B, 0x00001810, 0x06){}
        Processor (PR11, 0x0C, 0x00001810, 0x06){}
        Processor (PR12, 0x0D, 0x00001810, 0x06){}
        Processor (PR13, 0x0E, 0x00001810, 0x06){}
        Processor (PR14, 0x0F, 0x00001810, 0x06){}
        Processor (PR15, 0x10, 0x00001810, 0x06){}
    }

```

根据查询结果，选择注入文件 SSDT-PLUG-_SB.PR00

### 2.5 300系主板开启原生NVRAM
打开你的DSDT，搜索 `001F0000`，确定自己的

* lpc部件名字，如图示，我的lpc部件名叫`LPCB`，请根据实际情况记录
* lpc的路径，如图左下角红线提示，我的LPC路径在`_SB_.PCI0.LPCB`

* 下载SSDT-PMC.dsl根据自己的dsdt将第二行External和Scope中的设备路径改为你的。本机型不需要修改。注意Scope中_SB后没有“下划线”为`_SB.PCI0.LPCB`

### 2.6 EC控制器

#### Finding the ACPI path
search for `PNP0C09`. You should get something similar:

From the above example we see 2 main things:

* Name of our embedded controller
	* In this case being `EC0`
* Pathing of our embedded controller
	* `PC00.LPC0`

But now we get into edge case territory, what fun!

The main ones to check for are:

* Multiple PNP0C09's show up
* No PNP0C09 show up
* PNP0C09 already named `EC`
* PNP0C09 already has an `_STA` method

#### Edits to the sample SSDT
Now that we have our ACPI path, lets grab our SSDT and get to work:

* SSDT-EC-USBX(opens new window)
	* For Skylake and newer and all AMD systems
* SSDT-EC(opens new window)
	* For Broadwell and older

Now when opening this SSDT, you'll notice a few things. Mainly:

* Some code is commented out

* This is code for disabling our EC
* Laptops users SHOULD NOT uncomment this

* There's a new EC called

```
Device (EC)
```

* DO NOT RENAME THIS, this will be the EC we give to macOS

Before:

```text
External (_SB_.PCI0.LPCB, DeviceObj) <- Rename this

Scope (_SB.PCI0.LPCB) <- Rename this
```

Following the example pathing we found, the SSDT should look something like this:

After:

```text
External (_SB_.PC00.LPC0, DeviceObj) <- Renamed

Scope (_SB.PC00.LPC0) <- Renamed
```

#### Multiple PNP0C09's show up
When multiple PNP0C09 show up, we need to next check for the following properties:

* `_HID` (Hardware ID)
* `_CRS` (Current Resource Settings)
* `_GPE` (General Purpose Events)

What these signify is whether this PNP0C09 device is real or not, as per the ACPI spec . So one's matching the above criteria are the one's we want to disable.

* Note: If _STA shows up as well, you'll need to go here: [PNP0C09 already has an `_STA` method](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-methods/manual.html#pnp0c09-already-has-an-sta-method)

#### No PNP0C09 show up
When this happens, you'll only need to create a "dummy" EC for macOS.

Try searching for any devices named: "LPCB", "LPC0", "LPC", "SBRG", "PX40". If you have any of these, try using the LPC pathing of each of those device in place of the Embedded Controller's pathing.

Note that DO NOT uncomment the EC disabling code as there are no devices that are considered "EC" in your machine.

#### PNP0C09 already named `EC`
Congrats! No need to create an SSDT-EC! However you will still want USBX if you're Skylake or newer.

Prebuilt can be grabbed here: SSDT-USBX.aml

#### PNP0C09 already has an `_STA` method
This is the equivalent of not having an EC as we can't control it with our SSDT-EC, instead we'll need to create a "dummy" EC for macOS. You'll still want to find the PCI and LPC pathing for this device. So follow the guide as if you were creating a laptop SSDT-EC/USBX.

Example of an EC with STA already:

### 2.7 IRQ补丁

### 2.8 SBUS-MCHC 补丁

### 2.9 开启节能5项

### 2.10 AWAC补丁

### 2.11 品牌机（DELL等）特殊补丁

### 2.11 屏蔽不需要的设备

### 2.12 如何加载SSDT？
左上角另存为（save as), 其中文件格式(file format)必须选择 `ACPI Machine Language Binary`，文件名字随便写吧，我就叫 `ssdt-pmc.aml`，记住后缀为aml。记得将 `ssdt-pmc.aml` 放入 `EFI/OC/ACPI` 目录下，并在 `config.plist` 中添加加载此aml文件。

### 2.13 ACPI quirks设置

1. FadtEnableReset
Type: plist boolean
Failsafe: false
Description: Provide reset register and flag in FADT table to enable reboot and shutdown.

NOTE: Mainly required on legacy hardware and a few newer laptops. Can also fix power-button shortcuts. Not recommended unless required.

2. NormalizeHeaders
Type: plist boolean
Failsafe: false
Description: Cleanup ACPI header fields to workaround macOS ACPI implementation flaws that result in boot crashes. Reference: Debugging AppleACPIPlatform on 10.13 by Alex James (also known as theracermaster). The issue was fixed in macOS Mojave (10.14).

3. RebaseRegions
Type: plist boolean
Failsafe: false
Description: Attempt to heuristically relocate ACPI memory regions. Not recommended.

ACPI tables are often generated dynamically by the underlying firmware implementation. Among the position- independent code, ACPI tables may contain the physical addresses of MMIO areas used for device configuration, typically grouped by region (e.g. OperationRegion). Changing firmware settings or hardware configuration, upgrading or patching the firmware inevitably leads to changes in dynamically generated ACPI code, which sometimes results in the shift of the addresses in the aforementioned OperationRegion constructions.

For this reason, the application of modifications to ACPI tables is extremely risky. The best approach is to make as few changes as possible to ACPI tables and to avoid replacing any tables, particularly DSDT tables. When this cannot be avoided, ensure that any custom DSDT tables are based on the most recent DSDT tables or attempt to remove reads and writes for the affected areas.

When nothing else helps, this option could be tried to avoid stalls at PCI Configuration Begin phase of macOS booting by attempting to fix the ACPI addresses. It is not a magic bullet however, and only works with the most typical cases. Do not use unless absolutely required as it can have the opposite effect on certain platforms and result in boot failures.

4. ResetHwSig
Type: plist boolean
Failsafe: false
Description: Reset FACS table HardwareSignature value to 0.

This works around firmware that fail to maintain hardware signature across the reboots and cause issues with waking from hibernation.

5. ResetLogoStatus
Type: plist boolean
Failsafe: false
Description: Reset BGRT table Displayed status field to false.

This works around firmware that provide a BGRT table but fail to handle screen updates afterwards.

6. SyncTableIds
Type: plist boolean
Failsafe: false
Description: Sync table identifiers with the SLIC table.

This works around patched tables becoming incompatible with the SLIC table causing licensing issues in older Windows operating systems.

## 3.0 Booter

### 3.1 quirks

1. AllowRelocationBlock
Type: plist boolean
Failsafe: false
Description: Allows booting macOS through a relocation block.

The relocation block is a scratch buffer allocated in the lower 4 GB used for loading the kernel and related structures by EfiBoot on firmware where the lower memory region is otherwise occupied by (assumed) non-runtime data. Right before kernel startup, the relocation block is copied back to lower addresses. Similarly, all the other addresses pointing to the relocation block are also carefully adjusted. The relocation block can be used when:

• No better slide exists (all the memory is used)
• slide=0 is forced (by an argument or safe mode)
• KASLR (slide) is unsupported (this is macOS 10.7 or older)

This quirk requires ProvideCustomSlide to be enabled and typically also requires enabling AvoidRuntimeDefrag to function correctly. Hibernation is not supported when booting with a relocation block, which will only be used if required when the quirk is enabled.

_Note_: While this quirk is required to run older macOS versions on platforms with used lower memory, it is not compatible with some hardware and macOS 11. In such cases, consider using EnableSafeModeSlide instead.

2. AvoidRuntimeDefrag
Type: plist boolean
Failsafe: false
Description: Protect from boot.efi runtime memory defragmentation.

This option fixes UEFI runtime services (date, time, NVRAM, power control, etc.) support on firmware that uses SMM backing for certain services such as variable storage. SMM may try to access memory by physical addresses in non-SMM areas but this may sometimes have been moved by boot.efi. This option prevents boot.efi from moving such data.

_Note_: Most types of firmware, apart from Apple and VMware, need this quirk.

3. DevirtualiseMmio
Type: plist boolean
Failsafe: false
Description: Remove runtime attribute from certain MMIO regions.

This quirk reduces the stolen memory footprint in the memory map by removing the runtime bit for known memory regions. This quirk may result in an increase of KASLR slides available but without additional measures,it is not necessarily compatible with the target board. This quirk typically frees between 64 and 256 megabytes of memory, present in the debug log, and on some platforms, is the only way to boot macOS, which otherwise fails with allocation errors at the bootloader stage.

This option is useful on all types of firmware, except for some very old ones such as Sandy Bridge. On certain firmware, a list of addresses that need virtual addresses for proper NVRAM and hibernation functionality may be required. Use the MmioWhitelist section for this.

4. DisableSingleUser
Type: plist boolean
Failsafe: false
Description: Disable single user mode.

This is a security option that restricts the activation of single user mode by ignoring the CMD+S hotkey and the -s boot argument. The behaviour with this quirk enabled is supposed to match T2-based model behaviour. Refer to this archived article to understand how to use single user mode with this quirk enabled.

5. DisableVariableWrite
Type: plist boolean
Failsafe: false
Description: Protect from macOS NVRAM write access.

This is a security option that restricts NVRAM access in macOS. This quirk requires OC_FIRMWARE_RUNTIME protocol implemented in OpenRuntime.efi.

_Note_: This quirk can also be used as an ad hoc workaround for defective UEFI runtime services implementations that are unable to write variables to NVRAM and results in operating system failures.

6. DiscardHibernateMap
Type: plist boolean
Failsafe: false
Description: Reuse original hibernate memory map.

This option forces the XNU kernel to ignore a newly supplied memory map and assume that it did not change after waking from hibernation. This behaviour is required by Windows to work. Windows mandates preserving runtime memory size and location after S4 wake.

_Note_: This may be used to workaround defective memory map implementations on older, rare legacy hardware. Examples of such hardware are Ivy Bridge laptops with Insyde firmware such as the Acer V3-571G. Do not use this option without a full understanding of the implications.

7. EnableSafeModeSlide
Type: plist boolean
Failsafe: false
Description: Patch bootloader to have KASLR enabled in safe mode.

This option is relevant to users with issues booting to safe mode (e.g. by holding shift or with using the -x boot argument). By default, safe mode forces 0 slide as if the system was launched with the slide=0 boot argument.

• This quirk attempts to patch the boot.efi file to remove this limitation and to allow using other values (from 1 to 255 inclusive).

• This quirk requires enabling ProvideCustomSlide.
_Note_: The need for this option is dependent on the availability of safe mode. It can be enabled when booting to safe mode fails.

8. EnableWriteUnprotector
Type: plist boolean
Failsafe: false
Description: Permit write access to UEFI runtime services code.

This option bypasses WˆX permissions in code pages of UEFI runtime services by removing write protection (WP) bit from CR0 register during their execution. This quirk requires OC_FIRMWARE_RUNTIME protocol implemented in OpenRuntime.efi.

_Note_: This quirk may potentially weaken firmware security. Please use RebuildAppleMemoryMap if the firmware supports memory attributes table (MAT). Refer to the OCABC: MAT support is 1/0 log entry to determine whether MAT is supported.

9. ForceBooterSignature
Type: plist boolean
Failsafe: false
Description: Set macOS boot-signature to OpenCore launcher.

Booter signature, essentially a SHA-1 hash of the loaded image, is used by Mac EFI to verify the authenticity of the bootloader when waking from hibernation. This option forces macOS to use OpenCore launcher SHA-1 hash as a booter signature to let OpenCore shim hibernation wake on Mac EFI firmware.

_Note_: OpenCore launcher path is determined from LauncherPath property.

10. ForceExitBootServices
Type: plist boolean
Failsafe: false
Description: Retry ExitBootServices with new memory map on failure.

Try to ensure that the ExitBootServices call succeeds. If required, an outdated MemoryMap key argument can be used by obtaining the current memory map and retrying the ExitBootServices call.

_Note_: The need for this quirk is determined by early boot crashes of the firmware. Do not use this option without a full understanding of the implications.

11. ProtectMemoryRegions
Type: plist boolean
Failsafe: false
Description: Protect memory regions from incorrect access.

Some types of firmware incorrectly map certain memory regions:

• The CSM region can be marked as boot services code, or data, which leaves it as free memory for the XNU kernel.

• MMIO regions can be marked as reserved memory and stay unmapped. They may however be required to be accessible at runtime for NVRAM support.

This quirk attempts to fix the types of these regions, e.g. ACPI NVS for CSM or MMIO for MMIO.

_Note_: The need for this quirk is determined by artifacts, sleep wake issues, and boot failures. This quirk is typically only required by very old firmware.

12. ProtectSecureBoot
Type: plist boolean
Failsafe: false
Description: Protect UEFI Secure Boot variables from being written.

Reports security violation during attempts to write to db, dbx, PK, and KEK variables from the operating system.

_Note_: This quirk attempts to avoid issues with NVRAM implementations with fragmentation issues, such as on the MacPro5,1 as well as on certain Insyde firmware without garbage collection or with defective garbage collection.

13. ProtectUefiServices
Type: plist boolean
Failsafe: false
Description: Protect UEFI services from being overridden by the firmware.

Some modern firmware, including on virtual machines such as VMware, may update pointers to UEFI services during driver loading and related actions. Consequently, this directly obstructs other quirks that affect memory management, such as DevirtualiseMmio, ProtectMemoryRegions, or RebuildAppleMemoryMap, and may also obstruct other quirks depending on the scope of such.

GRUB shim makes similar on-the-fly changes to various UEFI image services, which are also protected against by this quirk.

_Note 1_: On VMware, the need for this quirk may be determined by the appearance of the “Your Mac OS guest might run unreliably with more than one virtual core.” message.

_Note 2_: This quirk is needed for correct operation if OpenCore is chainloaded from GRUB with BIOS Secure Boot enabled.

14. ProvideCustomSlide
Type: plist boolean
Failsafe: false
Description: Provide custom KASLR slide on low memory.

This option performs memory map analysis of the firmware and checks whether all slides (from 1 to 255) can be used. As boot.efi generates this value randomly with rdrand or pseudo randomly rdtsc, there is a chance of boot failure when it chooses a conflicting slide. In cases where potential conflicts exist, this option forces macOS to select a pseudo random value from the available values. This also ensures that the slide= argument is never passed to the operating system (for security reasons).

_Note_: The need for this quirk is determined by the OCABC: Only N/256 slide values are usable! message in the debug log.

15. ProvideMaxSlide
Type: plist integer
Failsafe: 0
Description: Provide maximum KASLR slide when higher ones are unavailable.

This option overrides the maximum slide of 255 by a user specified value between 1 and 254 (inclusive) when ProvideCustomSlide is enabled. It is assumed that modern firmware allocates pool memory from top to bottom, effectively resulting in free memory when slide scanning is used later as temporary memory during kernel loading. When such memory is not available, this option stops the evaluation of higher slides.

_Note_: The need for this quirk is determined by random boot failures when ProvideCustomSlide is enabled and the randomized slide falls into the unavailable range. When AppleDebug is enabled, the debug log typically contains messages such as AAPL: [EB|‘LD:LKC] } Err(0x9). To find the optimal value, append slide=X, where X is the slide value, to the boot-args and select the largest one that does not result in boot failures.

16. RebuildAppleMemoryMap
Type: plist boolean
Failsafe: false
Description: Generate macOS compatible Memory Map.

The Apple kernel has several limitations on parsing the UEFI memory map:

• The Memory map size must not exceed 4096 bytes as the Apple kernel maps it as a single 4K page. As some types of firmware can have very large memory maps, potentially over 100 entries, the Apple kernel will crash on boot.

• The Memory attributes table is ignored. EfiRuntimeServicesCode memory statically gets RX permissions while all other memory types get RW permissions. As some firmware drivers may write to global variables at runtime, the Apple kernel will crash at calling UEFI runtime services unless the driver .data section has a EfiRuntimeServicesData type.

To workaround these limitations, this quirk applies memory attribute table permissions to the memory map passed to the Apple kernel and optionally attempts to unify contiguous slots of similar types if the resulting memory map exceeds 4 KB.

_Note 1_: Since several types of firmware come with incorrect memory protection tables, this quirk often comes paired with SyncRuntimePermissions.

_Note 2_ : The need for this quirk is determined by early boot failures. This quirk replaces EnableWriteUnprotector on firmware supporting Memory Attribute Tables (MAT). This quirk is typically unnecessary when using OpenDuetPkg but may be required to boot macOS 10.6, and earlier, for reasons that are as yet unclear.

17. ResizeAppleGpuBars Type: plist integer

```
**Failsafe**: -1
 **Description**: Reduce GPU PCI BAR sizes for compatibility with macOS.
```

This quirk reduces GPU PCI BAR sizes for Apple macOS up to the specified value or lower if it is unsupported. The specified value follows PCI Resizable BAR spec. While Apple macOS supports a theoretical 1 GB maximum, in practice all non-default values may not work correctly. For this reason the only supported value for this quirk is the minimal supported BAR size, i.e. 0. Use -1 to disable this quirk.

For development purposes one may take risks and try other values. Consider a GPU with 2 BARs: • BAR0 supports sizes from 256 MB to 8 GB. Its value is 4 GB.

• BAR1 supports sizes from 2 MB to 256 MB. Its value is 256 MB.

_Example 1_: Setting ResizeAppleGpuBars to 1 GB will change BAR0 to 1 GB and leave BAR1 unchanged. _Example 2_: Setting ResizeAppleGpuBars to 1 MB will change BAR0 to 256 MB and BAR0 to 2 MB. _Example 3_: Setting ResizeAppleGpuBars to 16 GB will make no changes.

_Note_: See ResizeGpuBars quirk for general GPU PCI BAR size configuration and more details about the technology.

18. SetupVirtualMap
Type: plist boolean
Failsafe: false
Description: Setup virtual memory at SetVirtualAddresses.

Some types of firmware access memory by virtual addresses after a SetVirtualAddresses call, resulting in early boot crashes. This quirk workarounds the problem by performing early boot identity mapping of assigned virtual addresses to physical memory.

_Note_: The need for this quirk is determined by early boot failures.

19. SignalAppleOS
Type: plist boolean
Failsafe: false
Description: Report macOS being loaded through OS Info for any OS.

This quirk is useful on Mac firmware, which loads different operating systems with different hardware configurations. For example, it is supposed to enable Intel GPU in Windows and Linux in some dual-GPU MacBook models.

20. SyncRuntimePermissions
Type: plist boolean
Failsafe: false
Description: Update memory permissions for the runtime environment.

Some types of firmware fail to properly handle runtime permissions:

• They incorrectly mark OpenRuntime as not executable in the memory map.
• They incorrectly mark OpenRuntime as not executable in the memory attributes table. • They lose entries from the memory attributes table after OpenRuntime is loaded.
• They mark items in the memory attributes table as read-write-execute.

This quirk attempts to update the memory map and memory attributes table to correct this.

_Note_: The need for this quirk is indicated by early boot failures (note: includes halt at black screen as well as more obvious crash). Particularly likely to affect early boot of Windows or Linux (but not always both) on affected systems. Only firmware released after 2017 is typically affected.

## 4.0 DeviceProperties
这个位置是关于设备的属性，比如

### 4.1利用hackintool驱动核显
随着`macOS Mojave`的发行，之前的通过`Clover` ▸ `KextsToPatch` 通过修补帧缓冲的方法已经失效了，尤其是 `SkyLake` 及以后架构。您现在必须使用 `Lilu` + `WhateverGreen`+`FB Patcher` 的方式来驱动您的显卡。

#### 4.1.1初步动作

* 删除 `FakePCIID`，`IntelGraphicsFixup`，`NvidiaGraphicsFixup`,`Shiki` 和 `CoreDisplayFixup`
* 关闭 `Clover` 里面关于 `Graphics` 注入的参数，这些参数包括：
	* config.plist ▸ Graphics ▸ 🔲 Inject ATI
	* config.plist ▸ Graphics ▸ 🔲 Inject Intel
	* config.plist ▸ Graphics ▸ 🔲 Inject NVidia
	* config.plist ▸ Graphics ▸ ig-platform-id= `清空`
	* config.plist ▸ Devices ▸ FakeID ▸ IntelGFX= `清空`
* 关闭 `Clover` 里面关于`Acpi` ▸ `DSDT` 的修复：
	* 🔲 `AddHDMI`
	* 🔲 `FixDisplay`
	* 🔲 `FixIntelGfx`
	* 🔲 `AddIMEI`
* `Devices` 禁用 `UseIntelHDMI`
* 移除 `boot argument` 参数：`-disablegfxfirmware`
* 移除 `IGPU` 和 `HDMI` 部分的全部内容，包括：
	* config.plist ▸ Devices ▸ Arbitrary
	* config.plist ▸ Devices ▸ Properties
	* config.plist ▸ Devices ▸ AddProperties
* 从以下位置删除任何与 `IGPU` 和 `HDMI` 相关的 `SSDT` 和 `DSDT` ：
	* Clover ▸ ACPI ▸ patched

#### 4.1.2使用方法

1. 打开应用：`Hackintool.app`

2. 通过菜单项：`缓冲帧`选择 `macOS 10.13.6` / `macOS 10.14`
3. 选择显卡对应的处理器架构，比如

* `Intel UHD Graphics 630`就需要选择`Coffee Lake`
* `Intel HD Graphics 620`就需要选择`Kaby Lake`
* `Intel HD Graphics 520`就需要选择`Skylake`，等等，

之后选择`平台 ID`，这个就是能正确驱动你的显卡的 ID，至于这个 ID 如何确定，请参考[黑苹果必备：Intel核显platform ID整理及smbios速查表](https://blog.daliansky.net/Intel-core-display-platformID-finishing.html) 并针对白苹果所使用的 ID 选取适合您的 `平台 ID`。

如果您想了解更多信息，请阅读[针对 Whatevergreen 的缓冲帧修补教程（英文）](https://www.tonymacx86.com/threads/guide-intel-framebuffer-patching-using-whatevergreen.256490/)。中文版：[Coffee Lake帧缓冲区补丁及UHD630 Coffee Lake ig-platform-id数据整理](https://blog.daliansky.net/Coffee-Lake-frame-buffer-patch-and-UHD630-Coffee-Lake-ig-platform-id-data-finishing.html) 和 [教程：利用Hackintool打开第8代核显HDMI输出的正确姿势](https://blog.daliansky.net/Tutorial-Using-Hackintool-to-open-the-correct-pose-of-the-8th-generation-core-display-HDMI-or-DVI-output.html)

4. 点击`应用补丁`按钮，在`通用`选项中勾选`设备/属性`，`自动侦测变化`，`全部`，`接口`，`显存`这几个选项；

5. 在`高级`选项中勾选`DVMT pre-alloc 32 MB`，`显存 2048MB`，`禁用 eGPU`，`启用 HDMI20(4K)`，`将 DP 映射到 HDMI`，`FB 端口数限制`

6. 勾选`设备`，选择`平台 ID` 相对应的 `设备 ID`，这通常跟你的显卡名称相吻合

7. 点击`生成补丁`生成显卡驱动补丁

8. 通过菜单项：`文件` ▸ `导出` ▸ `Config.plist`，将该补丁无损注入到 Clover 的配置文件 Config.plist 中

#### 几个例子：

* Coffee Lake（八代）平台：Intel UHD Graphics 630 (移动端)

* Kabe Lake（七代）平台：Intel HD Graphics 620 / Intel UHD Graphics 620 (移动端)

* CPU 架构：Kaby Lake
* 平台 ID：`0x59160000`
* 通用和高级界面同上勾选
* 仿冒图形卡 ID 选择：`0x5916: Intel Graphics 620`

* Sky Lake（六代）平台：Intel HD Graphics 530 (移动端)

* CPU 架构：Skylake
* 平台 ID：`0x191B0000`
* 通用和高级界面同上勾选
* 仿冒图形卡 ID 选择：`0x191B: Intel Graphics 530`

* Haswell（四代）平台：Intel HD Graphics 4600 (移动端)

* CPU 架构：Haswell
* 平台 ID：`0x0A260006`
* 通用和高级界面同上勾选
* 仿冒图形卡 ID 选择：`0x0A26: Intel Graphics 4600`

* 注意⚠️：以上设置仅为较为通用的设置，对于某些设备，可能存在疏漏或者冗余

* [查看此处]((null))来利用 Hackintool 深度定制 Whatevergreen 补丁

*最后通过菜单项：`文件` ▸ `导出` ▸ `Config.plist`，将该补丁导出到配置文件*

### 4.1.3利用Hackintool打开第8代核显HDMI/DVI输出的正确姿势

> 本篇文章，适用于使用了较新主板并且搭配英特尔核显的用户。如果你的板载HDMI无法正常工作，那么可以参考本教程的方案来对你的HDMI进行缓冲帧修复从而修复HDMI输出 。    

#### 工作原理
不同主板上的板载视频接口有许多不同的组合，有些是1个HDMI+2个DP，有些则是2个HDMI，还有可能是DVI和VGA。（当然，VGA在MacOS里是不能使用的，请记住这点）。

当MacOS初始化IGPU驱动（称为`AppleIntelFramebuffer`）时，它并不知道主板上的接口是什么类型的。但是，它会根据你选择的 平台 ID，作出默认假设。例如，对于 平台 ID `0x3E9B0007`，默认情况下它将所有接口视为DP接口，如果将DP显示器连接上，它就会立即工作。但是，如果将HDMI或DVI显示器连接上 ，就没有图像显示。 这是因为此时MacOS认为这些显示器是DP接口的。

主板上每个视频接口都与其对应的接口号相关，彼此不同。而在MacOS中，最多允许核显连接3个外部显示器，接口号为`5`,`6`和`7`。我们所要做的，正是确定主板上什么接口连接到HDMI，什么接口连接到DVI，什么接口连接到DP，搞清接口号与物理接口的映射关系，然后填写缓冲帧表向MacOS提供映射信息。这种映射的接口也叫cons，任何软件接口都可以映射到这些接口。

* 3个 连接接口(connectors) 名称分别为：
	* `con0`
	* `con1`
	* `con2`
* 接口号5,6和7可以作为软件的索引(Index)，索引号分别为1,2和3。它们有如下对应关系：
	* 索引号1(Index 1)始终指向物理接口5
	* 索引号2(Index 2)始终指向物理接口6
	* 索引号3(Index 3)始终指向物理接口7

例如：

* 如果我们想告诉MacOS物理接口6是HDMI类型，我们就标记此接口的索引号为2。
* 如果我们想告诉MacOS物理接口5是DVI类型，我们就标记标记此接口的索引号为1（注意：DVI和HDMI在MacOS中等效）。
* 如果我们想告诉MacOS物理接口7是DP类型，我们就标记此接口的索引号为3。

除了标记索引之外，我们还需要为每个索引指定一个`总线ID`。每种类型接口的`总线ID`值是有适用范围的，它们的可用范围见下表：

附表：接口类型和总线ID的对应表

|  DP  |   HDMI   | DVI  |
| :--: | :------: | :--: |
| 0x02 | 0x01 | 0x01 |
| 0x04 |   0x02   | 0x02 |
| 0x05 |   0x04   | 0x04 |
| 0x06 |   0x06   | 0x06 |

在这里面

* DP灵活多变，允许使用`总线ID` 有0x02,0x04,0x05,0x06，每个值理论上适用于任何主板。
* HDMI非常严格，只允许使用以下`总线ID`: 0x01,0x02,0x04,0x06，而且部分主板只接受这些值中的一种或两种。例如，技嘉 Z390只接受0x04。
* DVI与HDMI相同，使用相同的`总线ID`，甚至使用相同的`类型`。

#### 准备开始
接下来，我们首先要确定每个物理接口的类型。完成这个以后，其余部分的工作量会相对小些。我们要明确的有三个东西：

* 接口5（索引1）的类型
* 接口6（索引2）的类型
* 接口7（索引3）的类型

如图所示

然后整理出一个像这样的表格：

|      | Ports接口           | Indexs索引  | Types类型   | 总线ID           |
| :--- | :------------------ | :---------- | :---------- | :--------------- |
|      | 0x05                | 1           |             |                  |
|      | 0x06                | 2           |             |                  |
|      | 0x07                | 3           |             |                  |
| 备注 | Port 0x05,0x06,0x07 | Index 1,2,3 | HDMI_DP_DVI | 0x01,02,04,05,06 |

备注：

* 步骤1：确定物理接口类型
* 步骤2：为每个索引分配总线ID和类型，数值请参照上面整理出的`接口类型和总线ID的对应表`

在开始之前要做的事情

1. 点击[这里](https://www.tonymacx86.com/threads/release-hackintool-v1-7-7.254559/)下载Hackintool。

2. 安装`Lilu`和`WhateverGreen`

3. 使用正确的图形设备ID和值启动计算机，这些都可以在四叶草中轻松完成。

以下方法均可参考使用：

* 方法1：通过`Clover Configurator`直接配置（推荐）

```
 - `Devices` ▸ `Fake ID` ▸ `IntelGFX` ▸ 输入适当的设备ID（例如`0x3E9B8086`）
```


```
 - `Graphics` ▸ 勾选 `Inject Intel`
```

```
 - `Graphics` ▸ `ig-platform-id` ▸ 单击下拉菜单并选择适当的ID（例如`0x3E9B0007`）
```


* 方法2：在`Clover Configurator`的`Devices`页面中添加自定义属性`Properties`

下面的`PciRoot(0x0)/Pci(0x2,0x0)`,`AAPL,ig-platform-id`, 和`device-id`值必须要替换为适合你的

```
<key>Properties</key>
<dict>
    <key>PciRoot(0x0)/Pci(0x2,0x0)</key>
    <dict>
        <key>AAPL,ig-platform-id</key>
        <data>BwCbPg==</data>
        <key>device-id</key>
        <data>mz4AAA==</data>
        <key>framebuffer-patch-enable</key>
        <data>AQAAAA==</data>
    </dict>
</dict>
```

4. 对于大多数Coffee Lake桌面处理器，可注入设备ID 0x3E9B以及 平台ID 0x3E9B0007。当然，也可以参考此[framebuffer修补指南](https://www.tonymacx86.com/threads/guide-intel-framebuffer-patching-using-whatevergreen.256490/)来确定适合你的值，它将使你接口的驱动程序正常加载。如果显卡驱动的加速不能正常加载，这篇文章的内容将毫无意义；而那些将所有接口的索引号设置为 -1的任何 平台ID 叫做_无接口_Platform ID，这样的ID因为会把所有输出接口屏蔽，因此必须避免使用。例如Platform ID 0x3E920003就是这样的，如下所示：

5. 显卡驱动的加速正常工作时，主板的HDMI和DVI接口不能工作。此时你必须将显示器连接到主板上工作正常的视频接口（比如DP，一般是笔记本）或者驱动独立显卡并将显示器连接到独立显卡上（台式机，能屏蔽核显的笔记本）。

6. 运行Hackintool确定显卡的工作状况。如果GPU信息正确显示如下图所示，那么您就可以继续了。如果你看到GPU：??? ，那么你就需要重新开始或者求助其他人。

7. 列出主板上的视频接口（如HDMI，DP，DVI），不包括VGA。

8. 为测试每个视频接口，你需要为每种类型的视频接口准备显示器与连接线。当然没有的话也可以继续，但这会增加一些不确定因素。

9. 确保显卡加速驱动正确加载后，从config.plist中清除以下设置（使用Clover Configurator来完成）但是不要重新启动：

* Device ▸ Fake ID ▸ IntelGFX ▸ 清空该条目。
* Graphics ▸ 🔲 Inject Intel ▸ 取消选中该复选框。
* Graphics ▸ ig-platform-id ▸ 清空该条目。

10. 保存config.plist并退出Clover Configurator。

#### 准备工作

1. 运行Hackintool。从顶部菜单栏中选择缓冲帧并选择macOS 10.14。

1. 从应用补丁菜单中，选择应用当前补丁，可看到其前面打钩（这个选项用于显示改动后的设置，不勾选在Hackintool中将不能看到更改）。那是因为您未标记 /`应用当前补丁`/。

注意：如果要查看以前应用的设置，只需再次选择 /`应用当前补丁`/。

1. 选择合适的平台ID（不包括无意义平台ID）。

2. 这里我们以平台ID 0x3E9B0007为例。单击Connectors选项卡，程序列出了接口映射表 。我们可以在其中分配`Index`（索引号），`总线ID`，`通道`，`类型`和`标识符`。我们现在看到的就是`con0`，`con1`和`con2`的映射表，但此时它们的值没有意义。

3. 如果此时有显示器连接，其中一栏将红色突出显示。这里我使用的只有DP输出显示器，所以只会突出它，它显示红色这栏属于DP连接。接下来我需要确定它的接口号，只需要单击红色这栏并从窗口右下角读取接口号就行了。一个确定下来，就去确定第二个。我们现在知道了`接口5` - `索引1` 是DP输出，所以结果如下：

#### 开始操作

1. 始终让主显示器保持连接状态。

2. 将另一个接口与该类型的显示器连接。有可能所有的DP连接都会亮起，但DVI和HDMI可能会亮，也可能不会亮。

3. Hackintool中的一栏将亮为红色。因为我的主板有两个DP接口，所以我现在从第一个接口上拔下电缆并将其插入第二个接口，此时另一栏就会亮为红色。接着，我们需要再次通过单击红色栏来确定接口号。我们可以看到，第二个DP接口的接口号为6。现在我们的修改如图所示：


4. 由于我的主板只有3个视频接口，而且我知道我的HDMI输出无法正常工作，因此无需尝试连接HDMI电缆就可以得知信息。当然你也可以接上一个来验证你所期待的黑屏。通过这个排除过程，我们得出结论：

* 接口5（索引1）是DP输出
* 接口6（索引2）是DP输出
* 接口7（索引3）_必然是_ HDMI

5. 现在看下类型和索引这两列。我们看到所有三栏都被错误地设置为了 DP，索引号分别为1,2和3。前两个似乎没问题，但我们刚刚确定的索引3必然是HDMI，因此这个输出类型（Type）存在问题。

6. 此时，我们从下拉菜单中将索引3的类型 更正为HDMI并将其总线ID设置为0x04。为什么我们选择了0x04呢？确实，总线ID还有其它可能的值，但我们需要从某处开始并一次测试一种可能性。由于总线ID 0x04当前已分配给索引2，因此我们在它们之间交换一下值。让索引2使用总线ID 0x06（这是DP的有效总线ID之一），索引3使用0x04。此时结果如下：

7. 然后我们测试下变化。单击应用补丁选项卡，并在显示的选择通用和高级子选项卡：（我这里用的Coffee Lake处理器，所以_设备ID_在_高级_子选项卡应设置为_0x3E9B：Intel UHD Graphics 630_。你得根据自己的CPU 选择最合适的_设备ID_。）

8. 有时Hackintool会因为某些复选框的变化而重置接口设置的界面。因此，请在此时返回 接口 页面，再次检查您的设置是否完全正确并对错误项作出修正。然后返回 应用补丁页面，单击 生成补丁。

9. 接着，我们挂载四叶草的EFI分区（用Clover configurator或是其他方法），然后从Hackintool菜单栏中选择文件 ▸ 导出 ▸ Clover config.plist，如下所示：

10. 从出现的文件浏览器中，定位到四叶草的config.plist。Hackintool将自动备份现有文件，并以毫无破坏性的方式将补丁直接注入config.plist。

11. 接下来重启系统。当Mojave启动时，登录进入系统并将主板的HDMI接口连接到显示器上的HDMI输入，测试其输出是否正常。此时它可能不会工作，但如果工作，我们就完成了修改。

12. 如果HDMI（或DVI）无法正常工作，那么继续尝试使用其它允许的总线ID。再次运行Hackintool，选择缓冲帧 ▸ macOS 10.14并验证应用补丁 ▸ 应用当前补丁被正确勾选。然后重复步骤5到13，但要使用以下列出的不同总线ID：

13. 如果您的视频接口少于3个，有两个方案可用于禁用未使用的索引。方案1是将索引号设置为-1。方案2是保持索引值不变，但将总线ID设置为0x00。根据一些用户反馈，方案2可能是值得最先尝试的。例如，如果您在接口0x07（索引3）处有一个HDMI，您可以尝试下面的任意配置。

```
附表：接口0x07的单个HDMI的可能配置
组合1:
Index 3: 总线ID 0x04, 类型 HDMI
Index 1: 总线ID 0x00, 类型 DUMMY
Index 2: 总线ID 0x00, 类型 DUMMY
Index -1: 总线ID 0x00, 类型 DUMMY
组合2:
Index 1: 总线ID 0x00, 类型 DUMMY
Index 3: 总线ID 0x04, 类型 HDMI
Index 2: 总线ID 0x00, 类型 DUMMY
Index -1: 总线ID 0x00, 类型 DUMMY
组合3:
Index 1: 总线ID 0x00, 类型 DUMMY
Index 2: 总线ID 0x00, 类型 DUMMY
Index 3: 总线ID 0x04, 类型 HDMI
Index -1: 总线ID 0x00, 类型 DUMMY
```

1. 为了减少显卡故障并防止一些（罕见的）引导故障，建议启用`disablegfxfirmware`复选框，如图所示。

#### 收尾工作
这个工作相对简单，因为默认情况下三个接口中的两个已经配置正确，而且几乎没有什么变化情况。但是，如果连接另一个视频接口时没有亮起怎么办？在这种情况下，你就必须不断尝试各种组合。如果其中一个接口是默认已知的，那么你就只需要再知道一个或两个接口的值。

比如，你已经知道接口5（索引1）是DP，而这是你能获得的全部信息。那么，如果下一个要配置的接口是HDMI，就可以尝试将HDMI分配给接口6（索引2）并适当设置其接口类型和总线ID并重新启动。如果HDMI仍未正常工作，则可以将其分配给接口7（索引3）并适当设置其类型和总线ID。如果仍然失败，就需要在接口6上尝试不同的总线ID，然后再在接口7上尝试，直到找到正确组合。

成功启用HDMI（或DVI）后，我们可以再次运行Hackintool来检查接口号。首先，我们看到标记为HDMI的栏现在亮为红色（第一栏是我们的DP连接），并且如果我们点击HDMI栏本身，我们可以确认接口号确实是0x07。 这样，我们的 工作就结束了。


### 4.1.3 直接在DeviceProperties中注入属性
很多小伙伴问我核显怎么驱动，为什么我驱动了HD4600却只有7m缓存，我开机花屏难受等等。。。

说实话，我很幸运用到的EFI直接就驱动了HD4600，所以一直对核显驱动就没怎么上心，直到某天受到群友蓝幽鞭策，才开始查阅相关资料，终于弄懂了前因后果。

文章虽然看起来长，步骤思路很简单：

##### 清理以前的驱动→获取 IGPU 的设备路径→填入ig-platform-id→填入device-id
不要被本文的长度吓到了，举个例子：驱动完HD4600也就这点东西：

或者可以查看[精简版本](https://blog.zuiyu1818.cn/posts/Hac_Intel_Graphics_simple.html)

#### 缩写解释
| 缩写 | 解释                          |
| ---- | ----------------------------- |
| FB   | Framebuffer（缓冲帧）         |
| WEG  | Lilu.kext和WhateverGreen.kext |

#### 软件界面
出现这种界面，则是使用Xcode打开的Plist文件，若你觉得Xcode太大不想安装，也可以选择[PlistEdit Pro](https://www.fatcatsoftware.com/plisteditpro/)


[Xcode](https://files.zuiyu1818.cn/Mac/Xcode_Graphics_disable.png)

#### 下载相关
| 名称                | 地址                                                         |
| ------------------- | ------------------------------------------------------------ |
| Clover Configurator | [点击下载](https://mackie100projects.altervista.org/download-clover-configurator/) |
| PlistEdit Pro       | [点击下载](https://www.fatcatsoftware.com/plisteditpro/)     |
| WhateverGreen       | [点击下载](https://github.com/acidanthera/WhateverGreen)     |
| Lilu                | [点击下载](https://github.com/acidanthera/Lilu)              |
| gfxutil             | [点击下载](https://github.com/acidanthera/gfxutil/releases)  |

#### Clover目录
所有的clover目录都是指`/EFI/EFI/CLOVER`

#### DATA数据填入
由于Clover的特性，所有的DATA类型数据都必须两两一组倒序填入，例如：`0x0A160000`转换之后就是`0000160A`，如下图：

#### 为什么要使用Lilu + WhateverGreen
随着`macOS Mojave`的发行，之前的通过`Clover` ▸ `KextsToPatch` 通过修补帧缓冲的方法已经失效了，尤其是 `SkyLake` 及以后架构。您现在必须使用 `Lilu` + `WhateverGreen`+`FB Patcher` 的方式来驱动您的显卡。

WhateverGreen将取代Lilu的所有其他视频补丁插件，它目前已经合并了WhateverGreen，IntelGraphicsFixup，NvidiaGraphicsFixup，Shiki和CoreDisplayFixup

#### 启用核显的通常步骤

1. 修正有关设备的 ACPI 名称（核显自身名为 IGPU，英特尔® 管理引擎（英文缩写: IMEI）名为 IMEI）。
2. 如若必要，将 核显 / IMEI 的 设备 ID 仿冒为合适的型号。
3. 指定正确的缓冲帧。（英文: Framebuffer, 下文简称 缓冲帧 为 FB）（即 AAPL,ig-platform-id（适用于 Ivy Bridge 或更新微架构）或 AAPL,snb-platform-id（仅适用于 Sandy Bridge 微架构）) 一组正确的 FB 应当正确地包含了可用的输出端口以及该核显的其他属性。
4. 某些与核显相关的其他设备中已包含相关属性。

其中，第 1 步和第 4 步由 WhateverGreen 自动完成。其可运行在 macOS 10.8 及更高版本，这大大简化了正确启用核显的步骤。

#### 准备工作

1. 在 BIOS 中设置核显所需的内存量（即 预分配 DVMT，英文: DVMT Pre-Allocated）为 32 MB, 64 MB, 96 MB 等，与使用的 FB 值相关。如要使用最大值（英文: DVMT Total），请设为 MAX。

[BIOS设置](https://files.zuiyu1818.cn/Mac/bios.png)

2. 移除以下驱动（如果曾经使用）

* AzulPatcher4600
* AppleBacklightFixup
* CoreDisplayFixup
* FakePCIID_Intel_HD_Graphics
* FakePCIID_Intel_HDMI_Audio
* FakePCIID.kext（不使用其他基于 FakePCIID 的插件时）
* IntelGraphicsFixup
* IntelGraphicsDVMTFixup
* NvidiaGraphicsFixup
* Shiki

> 这些驱动文件通常位于Clover的`kexts/Other`文件夹中    

3. 关闭所有Clover中的显卡注入

* config.plist ▸ Graphics ▸ Inject ▸ ATI = NO
* config.plist ▸ Graphics ▸ Inject ▸ Intel = NO
* config.plist ▸ Graphics ▸ Inject ▸ NVidia = NO
* config.plist ▸ Graphics ▸ ig-platform-id = `清空`
* config.plist ▸ Devices ▸ FakeID ▸ IntelGFX = `清空`


[关闭Graphics显卡注入](https://files.zuiyu1818.cn/Mac/Clover_Graphics_disable.png)


[关闭Device中的显卡注入](https://files.zuiyu1818.cn/Mac/Clover_Graphics_disable.png)

如果你还不确定还可以打开plist文件查看是否显示为NO
[Xcode](https://files.zuiyu1818.cn/Mac/Xcode_Graphics_disable.png)

4. 禁用Clover中Apci的以下DSDT补丁

* AddHDMI
* FixDisplay
* FixIntelGfx
* AddIMEI
* FixHDA

5. 关闭Clover中Devices的`UseIntelHDMI`

6. 删除引导参数：`-disablegfxfirmware`


[删除引导参数](https://files.zuiyu1818.cn/Mac/Clover_boot_HD.png)

7. 删除以下位置所有的和IGPU、HDMI相关条目（一般来说清空就行了）：

* config.plist ▸ Devices ▸ Arbitrary
* config.plist ▸ Devices ▸ Properties
* config.plist ▸ Devices ▸ AddProperties


[删除IGPU](https://files.zuiyu1818.cn/Mac/Clover_Device_IGPU.png)

8. 删除或禁用以下 ACPI 重命名补丁: `GFX0 to IGPU`, `PEGP to GFX0`, `HECI to IMEI`, `MEI to IMEI`, `HDAS to HDEF`, `B0D3 to HDAU`

[禁用Acpi补丁](https://files.zuiyu1818.cn/Mac/Clover_Acpi_Patch.png)

若以上都做完了，恭喜你完成了清理工作。

#### 添加Lilu + WhateverGreen驱动
下载[Lilu](https://github.com/acidanthera/Lilu) 和 [WhateverGreen](https://github.com/acidanthera/WhateverGreen)驱动，选择release版本，解压并将.kext文件置于Clover的`kexts/Other`文件夹中

> 若你想要查看调试输出信息，请选择两者的的debug版本    

#### 获取IGPU的设备路径
下载并使用[gfxutil](https://github.com/acidanthera/gfxutil/releases)工具，将gfxutil文件解压至桌面，打开终端输入如下代码：

```
$ cd Desktop
$ ./gfxutil -f IGPU
DevicePath = PciRoot(0x0)/Pci(0x2,0x0)
```

其中DevicePath后面显示的`PciRoot(0x0)/Pci(0x2,0x0)`就是IGPU的设备路径

#### ig-platform-id（核心步骤）
我们需要制定正确的Framebuffer，一组正确的正确的 FB 应当正确地包含了可用的输出端口以及该核显的其他属性，所以我们需要注入属性。

打开config.plist，并在Device中的`Properties` 添加以下内容：

* `AAPL,ig-platform-id` 或 `AAPL,snb-platform-id（仅适用于 Sandy Bridge 微架构）`
* 设备 `IGPU` 的 `device-id`（需要仿冒时）
* 设备 `IMEI` 的 `device-id`（需要仿冒时）
* 部分补丁设定（必要时）

> 注意逗号区分中英文！！！    
上述属性应使用十六进制代码表示，并且需要 两两一组 倒序 输入。如 `0x0A260006` 应该用 `0600260A` 表示

##### 很多人问倒序怎么倒的，这里画个图解释一下

```
0x`指16进制，在这咱可以不管，提取出后面的`0A260006`，两两一组`0A 26 00 06`过程如下图所示，最终得到`0600260A
```

下面分别提供了适用于不同微架构的常用 `IGPU` 和 `IMEI` 属性模版。

[Xcode](https://files.zuiyu1818.cn/Mac/FB_template_xcode.png)

[clover中的设置](https://files.zuiyu1818.cn/Mac/FB_template_Clover.png)

> 如果某个属性不是必需的，请完全删除掉；如果某个 `PciRoot` 位置不存在，也请彻底删除！    
选择一个适合的 FB。 首先试试推荐值，如果失败，则逐个尝试其他值.在寻找合适的 FB 时，可以临时通过启动参数设置，此时 `Properties` 部分中的 FB 设置将被忽略。如: `igfxframe=0x0166000B`

[启动参数设置igfxframe](https://files.zuiyu1818.cn/Mac/Clover_Boot_Graphics.png)

注意！ 此处格式与 `Properties` 部分的格式不同，这里应正序 输入并保留 `0x` 前缀，如上例所示。

* 如未指定 FB，将会使用默认值；

* 如未设定 FB 并且存在独立显卡，将使用一组空 FB。

#### HD 2000/3000（[Sandy Bridge]((null)) 微架构，下文简称 SNB）
支持 macOS 10.7 至 10.13.6，本文适用于 10.8 到 10.13.6。在旧版本系统上请使用传统驱动方式。从 macOS 10.14 起，HD 2000/3000 已经不再支持。

此方法无法开启 Metal。

_推荐的 FB 配置_：

* 0x00030010（桌面版，缺省值）
* 0x00010000（移动版，缺省值）

通常 SNB 平台无需指定 FB，与 `board-id` 相对应的一组 FB 将会被自动使用。不过，在使用不基于 SNB 平台的 SMBios 时，则需指定 FB。（如使用 `HD 3000` + 基于 `Ivy Bridge` 平台的 `MacBookPro9,1` 时，则需指定 FB）

> 注意！为 SNB 平台指定 FB 时，属性名应为 `AAPL,snb-platform-id`，这与其他平台不同。    
对于桌面版，需设定（仿冒）`device-id` 为 `26010000`。

在基于 [7 系列芯片组](https://ark.intel.com/content/www/cn/zh/ark/products/series/98460/intel-7-series-chipsets.html?_ga=2.100876037.569501178.1553421075-527540512.1553334841) 的主板上使用基于 `SNB` 微架构的处理器时（如在 `Z77` 芯片组上使用基于 `SNB` 微架构的 `i7-2600` 时），需设定（仿冒）`IMEI` 的 `device-ID` 为 `3A1C0000`。

#### HD 2500/4000（[Ivy Bridge]((null)) 微架构，下文简称 Ivy）
支持 macOS 10.8 或更新版本。

_推荐的 FB 设置_：

* 0x0166000A（桌面版，缺省值）
* 0x01620005（桌面版）
* 0x01660003（移动版，缺省值）
* 0x01660009（移动版）
* 0x01660004（移动版）

在基于 [6 系列芯片组](https://ark.intel.com/content/www/cn/zh/ark/products/series/98461/intel-6-series-chipsets.html?_ga=2.2193906.333725926.1553422863-527540512.1553334841) 的主板上使用基于 `Ivy` 微架构的处理器时（如在 `Z68` 芯片组上使用基于 `Ivy` 微架构的 `i7-3770` 时），需设定（仿冒）`IMEI` 的 `device-ID` 为 `3A1E0000`。（如下所示）

#### Intel HD Graphics 4200-5200（[Haswell]((null)) 微架构）
支持 macOS 10.9 或更新版本。

_推荐的 FB 设置_：

* 0x0D220003（桌面版，缺省值）
* 0x0A160000（移动版，缺省值）
* 0x0A260005（移动版，推荐）
* 0x0A260006（移动版，推荐）

对于 桌面版 HD 4400 以及_所有_移动版核显，需设定（仿冒）`IGPU` 的 `device-id` 为 `12040000`。

#### HD 5300-6300（[Broadwell]((null)) 微架构，下文简称 BDW）
支持 macOS 10.10.2 或更新版本。

_推荐的 FB 设置_：

* 0x16220007（桌面版，缺省值）
* 0x16260006（移动版，缺省值）。

#### HD 510-580（[Skylake]((null)) 微架构，下文简称 SKL）
支持 macOS 10.11.4 或更新版本。

_推荐的 FB 设置_：

* 0x19120000（桌面版，缺省值）
* 0x19160000（移动版，缺省值）

#### HD 610-650（[Kaby Lake]((null)) 微架构，下文简称 KBL）
支持 macOS 10.12.6 或更新版本。

_推荐的 FB 设置_：

* 0x59160000（桌面版，缺省值）
* 0x59120000（桌面版，推荐）
* 0x591B0000（移动版，缺省值）

对于 UHD 620 ([Kaby Lake Refresh](https://en.wikipedia.org/wiki/Kaby_Lake#List_of_8th_generation_Kaby_Lake_R_processors)需设定（仿冒）`IGPU` 的 `device-id` 为 `16590000`

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

### 4.1.4 1080p显示器过于模糊，如何开启hidpi
#### 笔者经验：
hidpi一直都是我头大的问题，一直有1600x900分辨率无法开启的问题。在反复探索后解决办法如下：
1，首先，开启隐藏的bios选项，开启DVMT64MB

2，删除framebuffer-stolenmem，这步十分关键，这个补丁会适得其反

3，framebuffer-unifiedmem 设为000000C0 即显存变为3072MB

4，用one-key-hidpi，开启hidpi

关于 HiDPi 几年前之前写过一篇，感兴趣的网友可以去看看：[HiDPI 是什么？以及黑苹果如何开HiDPI](https://www.sqlsec.com/2018/09/hidpi.html)

这个这个教程里面就只写关键的操作了，不再啰嗦了。

#### HiDPi 是啥
简单来说是苹果一直使用的显示技术，通过多个像素点合成一个像素点来提高清晰度，当显示器达到苹果定义的视网膜标准时，会自动开启 HiDPi，MBP 全系列都开启了 HiDPi，如果你自己购买 4k 显示器的话，默认也是开启 HiDPi 的，或者使用 4k 笔记本黑苹果的时候（XPS 系列），也是自动开启 HiDPi 的。

HiDPi 因为牺牲像素点的原因，虽然看上去会比原来清晰，但是实际看上去的分辨率会低，相当于是牺牲分辨率换清晰度，所以 1080P 显示器开启 HiDPi 的话，最终显示的效果接近于 720P，这么看的话，2k 分辨率的设备更适合 HiDPi（高不成 低不就）。

#### 开启 HiDPi 效果前
ZenBoook 13 默认情况下的效果：

可以看到图片的字体还是比较模糊的，看着不是很舒服。

#### 开启 HiDPi 效果后
Zenbook 13 开启 HiDPi 后：

可以看到明显清晰了很多，看上去很清爽，虽然牺牲了一点分辨率，但是值！

#### 如何开启 HiDPi
实际上开启 HiDPi 并不复杂，有成熟的轮子工具可以使用了：脚本的 Github 项目地址: [GitHub - xzhih/one-key-hidpi: Enable macOS HiDPI](https://github.com/xzhih/one-key-hidpi)

只需要一条命令即可开启 HiDPi：

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/xzhih/one-key-hidpi/master/hidpi.sh)"
Copy
```

不过官方项目在 Github 很容易被墙，如果访问 Github 出现网络超时的情况，可以使用下面我放在国内的脚本命令：

```bash
sh -c "$(curl -fsSL https://html.sqlsec.com/hidpi.sh)"
Copy
```

国内的脚本是定期同步 Github 官方脚本的，主要代码转移到了国内的码云，基本上国内是可以很顺畅的使用的。

#### 脚本使用细节一
上面是国光常用的选项，因为这是一个双屏笔记本，所以列出了 2 个显示器，如果你们外接显示器的话，这里也会显示 2 个显示器。另外选择显示器 ICON 图标这里大家也可以根据自己的喜好来。

#### 脚本使用细节二

一般情况下 1080P 的显示器，分辨率配置这里选择 1 和 2 都可以，具体大家自己尝试看看。

不过有些笔记本的分辨率比较小众，比如之前的 驰为 Corebook X 14，它的分辨率为：2160x1440

这个时候就得自己拿计算器算好分辨率了，最终驰为这个 笔记本的 HiDPi 分辨率为：

1920x1280 1620x1080 1380x920 1080x720

其中在 1620x1080 面积会比较大，但是清晰度不及 1380x920，居然网友们可以根据自己的喜好来。

#### 开启 HiDPi 的设置
设置 - 显示器 里面选择缩放，可以看到开启 HiDPi 的显示器的效果，有小窗口的图标：

不过有些显示器是没有这个小窗口的，不过这种情况下也会有 HiDPi 的文字提醒：


#### 没开 HiDPi 的设置
没有开 HiDPi 的显示器选择缩放的话，只会通过文字形式（无 HiDPi 标识）列出来各个分辨率：

#### 开启 HiDPi 的缺点
凡是必有利弊，除了开启 HiDPi 会牺牲分辨率（显示面积）以外，还会加重显卡负担，如果你的核显或者 CPU 本身性能就薄弱的话，就会造成明显的卡顿现象。

说到这里有朋友肯定会问了？为啥会加重显卡负担呢？看下面这种开启 HiDPi 的图你就明白了：

因为 HiDPi 是牺牲分辨率换清晰度的，当你设置的 UI 分辨率要高一点的时候，必然需要更大的分辨率，原来的物理分辨率并不能满足，此时显卡会模拟输出更高的分辨率，这也就是 HiDPi 可能会造成系统卡顿的原因。（常见于核显，独显的话，无脑开启不会错）

## 5.0 Kernel

### 5.1 我需要哪些kext？

### 5.2 Quirks

1. AppleCpuPmCfgLock
Type: plist boolean
Failsafe: false
Requirement: 10.4
Description: Disables PKG_CST_CONFIG_CONTROL (0xE2) MSR modification in AppleIntelCPUPowerManage- ment.kext, commonly causing early kernel panic, when it is locked from writing.

Some types of firmware lock the PKG_CST_CONFIG_CONTROL MSR register and the bundled ControlMsrE2 tool can be used to check its state. Note that some types of firmware only have this register locked on some cores. As modern firmware provide a CFG Lock setting that allows configuring the PKG_CST_CONFIG_CONTROL MSR register lock, this option should be avoided whenever possible.

On APTIO firmware that do not provide a CFG Lock setting in the GUI, it is possible to access the option directly:

1. (a)  Download UEFITool and IFR-Extractor.

2. (b)  Open the firmware image in UEFITool and find CFG Lock unicode string. If it is not present, the firmware

may not have this option and the process should therefore be discontinued.

3. (c)  Extract the Setup.bin PE32 Image Section (the UEFITool found) through the Extract Body menu option.

4. (d)  Run IFR-Extractor on the extracted file (e.g. ./ifrextract Setup.bin Setup.txt).

5. (e)  Find CFG Lock, VarStoreInfo (VarOffset/VarName): in Setup.txt and remember the offset right after

it (e.g. 0x123).

6. (f)  Download and run Modified GRUB Shell compiled by brainsucker or use a newer version by datasone.

7. (g)  Enter setup_var 0x123 0x00 command, where 0x123 should be replaced by the actual offset, and reboot.

Warning: Variable offsets are unique not only to each motherboard but even to its firmware version. Never ever

try to use an offset without checking.
On selected platforms, the ControlMsrE2 tool can also change such hidden options. Pass desired argument: lock,

unlock for CFG Lock. Or pass interactive to find and modify other hidden options. As a last resort, consider patching the BIOS (for advanced users only).

2. AppleXcpmCfgLock
Type: plist boolean
Failsafe: false
Requirement: 10.8 (not required for older)
Description: Disables PKG_CST_CONFIG_CONTROL (0xE2) MSR modification in XNU kernel, commonly causing early kernel panic, when it is locked from writing (XCPM power management).

_Note_: This option should be avoided whenever possible. Refer to the AppleCpuPmCfgLock description for details.

3. AppleXcpmExtraMsrs
Type: plist boolean
Failsafe: false
Requirement: 10.8 (not required for older)
Description: Disables multiple MSR access critical for certain CPUs, which have no native XCPM support.

This is typically used in conjunction with the Emulate section on Haswell-E, Broadwell-E, Skylake-SP, and similar CPUs. More details on the XCPM patches are outlined in acidanthera/bugtracker#365.

_Note_: Additional not provided patches will be required for Ivy Bridge or Pentium CPUs. It is recommended to use AppleIntelCpuPowerManagement.kext for the former.

4. AppleXcpmForceBoost
Type: plist boolean
Failsafe: false
Requirement: 10.8 (not required for older)
Description: Forces maximum performance in XCPM mode.

This patch writes 0xFF00 to MSR_IA32_PERF_CONTROL (0x199), effectively setting maximum multiplier for all the time.

_Note_: While this may increase the performance, this patch is strongly discouraged on all systems but those explicitly dedicated to scientific or media calculations. Only certain Xeon models typically benefit from the patch.

5. CustomPciSerialDevice
Type: plist boolean
Failsafe: false
Requirement: 10.7
Description: Performs change of PMIO register base address on a customised PCI serial device.

The patch changes the PMIO register base address that the XNU kernel uses for serial input and output, from that of the default built-in COM1 serial port 0x3F8, to the base address stored in the first IO BAR of a specified PCI device or to a specific base address (e.g. 0x2F8 for COM2).

_Note_: By default, serial logging is disabled. serial=3 boot argument, which enables serial input and output, should be used for XNU to print logs to the serial port.

_Note 2_: In addition to this patch, kext Apple16X50PCI0 should be prevented from attaching to have kprintf method working properly. This can be achieved by setting (i.e. Delete, then Add) the class-code property of the PCI serial port device to FFFFFFFF in DeviceProperties section. As an alternative solution, a code- less kext PCIeSerialDisable.kext shown in the spoiler PCIeSerialDisable.kext_Contents_Info.plist at acidanthera/bugtracker#1954, may also be used.

_Note 3_: For this patch to be correctly applied, Override must be enabled with all keys properly set in Custom, under section Misc->Serial.

_Note 4_ : This patch is for PMIO support and is therefore not applied if UseMmio under section Misc->Serial->Custom is false. For MMIO, there are boot arguments pcie_mmio_uart=ADDRESS and mmio_uart=ADDRESS that allow the kernel to use MMIO for serial port access.

_Note 5_: The serial baud rate must be correctly set in both BaudRate under section Misc->Serial->Custom and via serialbaud=VALUE boot argument, both of which should match against each other. The default baud rate is 115200.

6. CustomSMBIOSGuid

Type: plist boolean

Failsafe: false
Requirement: 10.4
Description: Performs GUID patching for UpdateSMBIOSMode Custom mode. Usually relevant for Dell laptops.

7. DisableIoMapper
Type: plist boolean
Failsafe: false
Requirement: 10.8 (not required for older)
Description: Disables IOMapper support in XNU (VT-d), which may conflict with the firmware implementation.

_Note 1_: This option is a preferred alternative to deleting DMAR ACPI table and disabling VT-d in firmware preferences, which does not obstruct VT-d support in other systems in case they need this.

_Note 2_: Misconfigured IOMMU in the firmware may result in broken devices such as ethernet or Wi-Fi adapters. For instance, an ethernet adapter may cycle in link-up link-down state infinitely and a Wi-Fi adapter may fail to discover networks. Gigabyte is one of the most common OEMs with these issues.

8. DisableLinkeditJettison
Type: plist boolean
Failsafe: false
Requirement: 11
Description: Disables __LINKEDIT jettison code.

This option lets Lilu.kext, and possibly other kexts, function in macOS Big Sur at their best performance levels without requiring the keepsyms=1 boot argument.

9. DisableRtcChecksum
Type: plist boolean
Failsafe: false
Requirement: 10.4
Description: Disables primary checksum (0x58-0x59) writing in AppleRTC.

_Note 1_: This option will not protect other areas from being overwritten, see RTCMemoryFixup kernel extension if this is desired.

_Note 2_: This option will not protect areas from being overwritten at firmware stage (e.g. macOS bootloader), see AppleRtcRam protocol description if this is desired.

10. ExtendBTFeatureFlags
Type: plist boolean
Failsafe: false
Requirement: 10.8-11
Description: Set FeatureFlags to 0x0F for full functionality of Bluetooth, including Continuity.

_Note_: This option is a substitution for BT4LEContinuityFixup.kext, which does not function properly due to late patching progress.

11. ExternalDiskIcons
Type: plist boolean
Failsafe: false
Requirement: 10.4
Description: Apply icon type patches to AppleAHCIPort.kext to force internal disk icons for all AHCI disks.

_Note_: This option should be avoided whenever possible. Modern firmware typically have compatible AHCI controllers.

12. ForceAquantiaEthernet
Type: plist boolean
Failsafe: false
Requirement: 10.15.4
Description: Enable Aquantia AQtion based 10GbE network cards support.

This option enables Aquantia AQtion based 10GbE network cards support, which used to work natively before macOS 10.15.4.

_Note_: In order for Aquantia cards to properly function, DisableIoMapper must be disabled, DMAR ACPI table must not be dropped, and VT-d must be enabled in BIOS.

_Note 2_: While this patch should enable ethernet support for all Aquantia AQtion series, it has only been tested on AQC-107s based 10GbE network cards.

13. ForceSecureBootScheme
Type: plist boolean
Failsafe: false
Requirement: 11
Description: Force x86 scheme for IMG4 verification.

_Note_: This option is required on virtual machines when using SecureBootModel different from x86legacy.

14. IncreasePciBarSize
Type: plist boolean
Failsafe: false
Requirement: 10.10
Description: Allows IOPCIFamily to boot with 2 GB PCI BARs.

Normally macOS restricts PCI BARs to 1 GB. Enabling this option (still) does not let macOS actually use PCI devices with larger BARs.

_Note_: This option should be avoided whenever possible. A need for this option indicates misconfigured or defective firmware.

15. LapicKernelPanic
Type: plist boolean
Failsafe: false
Requirement: 10.6 (64-bit)
Description: Disables kernel panic on LAPIC interrupts.

16. LegacyCommpage
Type: plist boolean
Failsafe: false
Requirement: 10.4 - 10.6
Description: Replaces the default 64-bit commpage bcopy implementation with one that does not require SSSE3, useful for legacy platforms. This prevents a commpage no match for last panic due to no available 64-bit bcopy functions that do not require SSSE3.

17. PanicNoKextDump
Type: plist boolean
Failsafe: false
Requirement: 10.13 (not required for older)
Description: Prevent kernel from printing kext dump in the panic log preventing from observing panic details. Affects 10.13 and above.

18. PowerTimeoutKernelPanic
Type: plist boolean
Failsafe: false
Requirement: 10.15 (not required for older)
Description: Disables kernel panic on setPowerState timeout.

An additional security measure was added to macOS Catalina (10.15) causing kernel panic on power change timeout for Apple drivers. Sometimes it may cause issues on misconfigured hardware, notably digital audio, which sometimes fails to wake up. For debug kernels setpowerstate_panic=0 boot argument should be used, which is otherwise equivalent to this quirk.

19. ProvideCurrentCpuInfo
Type: plist boolean
Failsafe: false
Requirement: 10.8 (10.14)
Description: Provides current CPU info to the kernel.

This quirk works differently depending on the CPU:

• For Microsoft Hyper-V it provides the correct TSC and FSB values to the kernel, as well as disables CPU topology validation (10.8+).

• For KVM and other hypervisors it provides precomputed MSR 35h values solving kernel panic with -cpu host.

• For Intel CPUs it adds support for asymmetrical SMP systems (e.g. Intel Alder Lake) by patching core count to thread count along with the supplemental required changes (10.14+).

20. SetApfsTrimTimeout
Type: plist integer
Failsafe: -1
Requirement: 10.14 (not required for older)
Description: Set trim timeout in microseconds for APFS filesystems on SSDs.

The APFS filesystem is designed in a way that the space controlled via the spaceman structure is either used or free. This may be different in other filesystems where the areas can be marked as used, free, and _unmapped_. All free space is trimmed (unmapped/deallocated) at macOS startup. The trimming procedure for NVMe drives happens in LBA ranges due to the nature of the DSM command with up to 256 ranges per command. The more fragmented the memory on the drive is, the more commands are necessary to trim all the free space.

Depending on the SSD controller and the level of drive fragmenation, the trim procedure may take a considerable amount of time, causing noticeable boot slowdown. The APFS driver explicitly ignores previously unmapped areas and repeatedly trims them on boot. To mitigate against such boot slowdowns, the macOS driver introduced a timeout (9.999999 seconds) that stops the trim operation when not finished in time.

On several controllers, such as Samsung, where the deallocation process is relatively slow, this timeout can be reached very quickly. Essentially, it means that the level of fragmentation is high, thus macOS will attempt to trim the same lower blocks that have previously been deallocated, but never have enough time to deallocate higher blocks. The outcome is that trimming on such SSDs will be non-functional soon after installation, resulting in additional wear on the flash.

One way to workaround the problem is to increase the timeout to an extremely high value, which at the cost of slow boot times (extra minutes) will ensure that all the blocks are trimmed. Setting this option to a high value, such as 4294967295 ensures that all blocks are trimmed. Alternatively, use over-provisioning, if supported, or create a dedicated unmapped partition where the reserve blocks can be found by the controller. Conversely, the trim operation can be mostly disabled by setting a very low timeout value, while 0 entirely disables it. Refer to this article for details.

_Note_: The failsafe value -1 indicates that this patch will not be applied, such that apfs.kext will remain untouched.

_Note 2_: On macOS 12.0 and above, it is no longer possible to specify trim timeout. However, it can be disabled by setting 0.

_Note 3_ : Trim operations are _only_ affected at booting phase when the startup volume is mounted. Either specifying timeout, or completely disabling trim with 0, will not affect normal macOS running.

笔者经验：

这个BG4的固态硬盘在macOS的支持并不好，开机时间十分漫长，需要将SetApfsTrimTimeout设置为0，这时候macOS就会禁止TRIM，即使系统信息中TRIM support依然是支持。
反转了，在关掉Trim之后，会出现”宗卷哈希值不匹配“，甚至出现冻屏、卡死的现象，再三衡量一下，还是吧Trim重新开启，SetApfsTrimTimeout设置为-1

21. ThirdPartyDrives
Type: plist boolean
Failsafe: false
Requirement: 10.6 (not required for older)
Description: Apply vendor patches to IOAHCIBlockStorage.kext to enable native features for third-party drives, such as TRIM on SSDs or hibernation support on 10.15 and newer.

_Note_: This option may be avoided on user preference. NVMe SSDs are compatible without the change. For AHCI SSDs on modern macOS version there is a dedicated built-in utility called trimforce. Starting from 10.15 this utility creates EnableTRIM variable in APPLE_BOOT_VARIABLE_GUID namespace with 01 00 00 00 value.

22. XhciPortLimit
Type: plist boolean
Failsafe: false
Requirement: 10.11 (not required for older)

```
**Description**: Patch various kexts (AppleUSBXHCI.kext, AppleUSBXHCIPCI.kext, IOUSBHostFamily.kext) to remove USB port count limit of 15 ports.
```

_Note_: This option should be avoided whenever possible. USB port limit is imposed by the amount of used bits in locationID format and there is no possible way to workaround this without heavy OS modification. The only valid solution is to limit the amount of used ports to 15 (discarding some). More details can be found on AppleLife.ru.

## 6.0 Misc

### 6.1 Boot

### 6.2 Debug
Debug

此博客有很多疏漏，你可能会碰到很多奇怪的问题而无法解决。一般来说，一张开机-v的截图只能解决一些很简单的问题，一些帮助者可能通过经验判断你的错误所在，但这样的图无法定位错误，而Acidanthera提供了非常非常强大的排错功能，我们可以利用它收集错误报告，在网络上获得帮助。

如果我们需要debug报告，我们需要将所有的Acidanthera的kext以及OC bootloader替换成debug版本，所有的debug版本都会在github中提供。我们下载debug版本的Opencore，替换：

* EFI_BOOT_
	* BOOTx64.efi
* EFI_OC_Bootstrap/
	* Bootstrap.efi
* EFI_OC_Drivers/
	* OpenRuntime.efi
* EFI_OC_
	* OpenCore.efi
* 下载所有Acidanthera的debug版本kexts，并替换到EFI_OC_kexts

修改config.plist中的如下内容：

* AppleDebug `YES`
* ApplePanic `YES`
* DisableWatchdog `YES`
* Target `67`
* DisplayLevel `2147483714`
* PickerMode `Builtin`

在Config.plist_NVRAM/7C436110-AB2A-4BBB-A880-FE41995C9F82_boot-args栏目中增加

* `-liludbgall liludump=30`

保存后重启，你会得到：

* _EFI_下找到开机启动日志 (e.g. opencore-2020-11-16-083514.txt)
* _var_log下得到所有Acidanthera的kext的日志(e.g.Lilu_1.5.0_20.1.txt)

如果无法进入系统，则只需要第一份日志。

你可以通过这两份日志快速定位错误，或者在网路上寻求帮助。

### 6.3 Security
So something that makes OpenCore truly special is how it's been built with security in mind which is quite rare especially in the Hackintosh community. Well here we'll be going through and setting up some of OpenCore's great Security features:

* FileVault
	* Apple's built-in drive encryption
* Vault
	* OpenCore's semi-secure boot, used for snapshotting OpenCore so no unwanted changes happen
* ScanPolicy
	* OpenCore's drive policy, determines what types of disks show up in OpenCore's boot menu
* OpenCore Password Setup
	* Enable password in OpenCore boot menu
* Apple Secure Boot
	* Apple's variant of secure boot in the macOS kernel

#### 6.3.1 FileVault
FileVault is macOS's builtin drive encryption, and with OpenCore support for it has been drastically improved compared to the legacy Clover drivers.

To start, you'll need the following .efi drivers:

* OpenRuntime.efi
	* [OpenUsbKbDxe.efi (opens new window)](https://github.com/acidanthera/OpenCorePkg/releases)for DuetPkg users(systems without UEFI support)

Do not use VirtualSMC.efi with OpenCore, its already baked inside. You do however require VirtualSMC.kext still

Setting in your config.plist:

* Misc -> Boot
	* `PollAppleHotKeys` set to YES(While not needed can be helpful)
* Misc -> Security
	* `AuthRestart` set to YES(Enables Authenticated restart for FileVault 2 so password is not required on reboot. Can be considered a security risk so optional)
* NVRAM -> Add -> 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14
	* `UIScale` set to `02` for high resolution small displays
* UEFI -> Input
	* `KeySupport` set to YES(Only when using OpenCore's builtin input, users of OpenUsbKbDxe should avoid)
* UEFI -> Output
	* `ProvideConsoleGop` to YES
* UEFI -> ProtocolOverrides
	* `FirmwareVolume` set to YES
	* `HashServices` set to YES for Broadwell and older(this includes X99), this is needed for systems with broken SHA-1 hashing
* UEFI -> Quirks
	* `RequestBootVarRouting` set to YES
	* `ExitBootServicesDelay` set to `3000`-`5000` if you receive `Still waiting for root device` on Aptio IV firmwares(Broadwell and older)

With all this, you can proceed to enable FileVault like on a normal mac under `System Preferences -> Security & Privacy -> FileVault`

For UI issues, see [Fixing Resolution and Verbose](https://dortania.github.io/OpenCore-Post-Install/cosmetic/verbose.html)

#### 6.3.2Vault
What is vaulting?

Well vaulting is based around 2 things, vault.plist and vault.sig:

* vault.plist: a "snapshot" of your EFI
* vault.sig: validation of vault.plist

This can be seen as secure boot for OpenCore, so no one can modify it and get in without your permission.

The specifics of vaulting is that a 256 byte RSA-2048 signature of vault.plist will be shoved into our OpenCore.efi. This key can either be shoved into [OpenCoreVault.c (opens new window)](https://github.com/acidanthera/OpenCorePkg/blob/master/Platform/OpenCore/OpenCoreVault.c)before compiling or with `sign.command` if you already have OpenCore.efi compiled.

Do note that nvram.plist won't be vaulted so users with emulated NVRAM still have risk of someone adding/removing certain NVRAM variables

Settings in your config.plist:

* ```
Misc -> Security -> Vault
```

:

* `Basic`: Requires just vault.plist to be present, mainly used for filesystem integrity verification
* `Secure`: Requires both vault.plist and vault.sig, used for best security as vault.plist changes require a new signature

* ```
Booter -> ProtectSecureBoot:
```

```
YES
```

* Needed with Insyde firmwares for fixing secure boot keys and reporting violations

Setting up vault:

Grab OpenCorePkg and open the `CreateVault` folder, inside we'll find the following:

* `create_vault.sh`
* `RsaTool`
* `sign.command`

The last one is what we care about: `sign.command`

So when we run this command, it'll look for the EFI folder located beside our Utilities folder, so we want to bring either our personal EFI into the OpenCorePkg folder or bring Utilities into our EFI folder:

Now we're ready to run `sign.command`:

Disabling Vault after setup:

If you're doing heavy troubleshooting or have the need to disable Vault, the main things to change:

* Grab a new copy of OpenCore.efi
* `Misc -> Security -> Vault` set to Optional
* Remove `vault.plist` and `vault.sig`

#### 6.3.3 ScanPolicy
What this quirk allows to prevent scanning and booting from untrusted sources. Setting to `0` will allow all sources present to be bootable but calculating a specific ScanPolicy value will allow you a greater range of flexibility and security.

To calculate the ScanPolicy value, you simply add up all the hexadecimal values(with a hexadecimal calculator, you can access this from the built-in macOS calculator app with `⌘+3`). Once it's all added up, you would add this hexadecimal value to ScanPolicy(you will need to convert it to a decimal value first, Xcode will automatically convert it when you paste it)

`0x00000001 (bit 0)` — OC_SCAN_FILE_SYSTEM_LOCK

* restricts scanning to only known file systems defined as a part of this policy. File system drivers may not be aware of this policy, and to avoid mounting of undesired file systems it is best not to load its driver. This bit does not affect dmg mounting, which may have any file system. Known file systems are prefixed with OC_SCAN_ALLOW_FS_.

`0x00000002 (bit 1)` — OC_SCAN_DEVICE_LOCK

* restricts scanning to only known device types defined as a part of this policy. This is not always possible to detect protocol tunneling, so be aware that on some systems it may be possible for e.g. USB HDDs to be recognized as SATA. Cases like this must be reported. Known device types are prefixed with OC_SCAN_ALLOW_DEVICE_.

`0x00000100 (bit 8)` — OC_SCAN_ALLOW_FS_APFS

* allows scanning of APFS file system.

`0x00000200 (bit 9)` — OC_SCAN_ALLOW_FS_HFS

* allows scanning of HFS file system.

`0x00000400 (bit 10)` — OC_SCAN_ALLOW_FS_ESP

* allows scanning of EFI System Partition file system.

`0x00010000 (bit 16)` — OC_SCAN_ALLOW_DEVICE_SATA

* allow scanning SATA devices.

`0x00020000 (bit 17)` — OC_SCAN_ALLOW_DEVICE_SASEX

* allow scanning SAS and Mac NVMe devices.

`0x00040000 (bit 18)` — OC_SCAN_ALLOW_DEVICE_SCSI

* allow scanning SCSI devices.

`0x00080000 (bit 19)` — OC_SCAN_ALLOW_DEVICE_NVME

* allow scanning NVMe devices.

`0x00100000 (bit 20)` — OC_SCAN_ALLOW_DEVICE_ATAPI

* allow scanning CD/DVD devices.

`0x00200000 (bit 21)` — OC_SCAN_ALLOW_DEVICE_USB

* allow scanning USB devices.

`0x00400000 (bit 22)` - OC_SCAN_ALLOW_DEVICE_FIREWIRE

* allow scanning FireWire devices.

`0x00800000 (bit 23)` — OC_SCAN_ALLOW_DEVICE_SDCARD

* allow scanning card reader devices.

`0x01000000 (bit 24)` — OC_SCAN_ALLOW_DEVICE_PCI

* allow scanning devices directly connected to PCI bus (e.g. VIRTIO).

By default, ScanPolicy is given a value of `0x10F0103`(17,760,515) which is the combination of the following:

* OC_SCAN_FILE_SYSTEM_LOCK
* OC_SCAN_DEVICE_LOCK
* OC_SCAN_ALLOW_FS_APFS
* OC_SCAN_ALLOW_DEVICE_SATA
* OC_SCAN_ALLOW_DEVICE_SASEX
* OC_SCAN_ALLOW_DEVICE_SCSI
* OC_SCAN_ALLOW_DEVICE_NVME
* OC_SCAN_ALLOW_DEVICE_PCI

And lets just say for this example that you want to add OC_SCAN_ALLOW_DEVICE_USB:

```
0x00200000` + `0x10F0103` = `0x12F0103
```

And converting this to decimal gives us `19,857,667`

#### 6.3.4 OpenCore Menu Password
With OpenCore 0.6.1 and newer, users are able to set a SHA-512 password to ensure best security with their setups. This will enable a password prompt whenever elevated tasks are required. This includes:

* Showing boot menu
* Booting non-default OSes and tools(ie. not blessed by Startup Disk or Bootcamp Utility)
* Resetting NVRAM
* Booting non-default modes(ie. Verbose or Safe Mode via hotkeys)

With OpenCore 0.6.7, a new tool called `ocpasswordgen` was added to aid users in generating passwords.

To start, lets grab OpenCore 0.6.7 or newer and run the `ocpasswordgen` binary under `Utilities/ocpasswordgen/`. It'll prompt you to create a password:

![](hackintosh-Inspiron3670-franksfc/ocpasswordgen.eb011d95.png)img

For this example, we chose `Dortania` as the password. `ocpasswordgen` then popped out 2 important values we need for our config.plist:

* PasswordHash: Hash of the password
* PasswordSalt: Ensures 2 users with the exact same password do not do not have the same hash

Next let's open our config.plist and add these values to Misc -> Security:

* Note: Don't forget to also enable `EnablePassword`

Once these changes have been made, you can save and reboot the machine. Now when you enter OpenCore's menu, you should receive a prompt:

Enter your password and you should get your regular boot options:

* Note: Between typing the password and entering the menu, some older machines and VMs can take 30 seconds+ to finish verification. Please be patient

#### 6.3.5 *Apple Secure Boot*

* Note: DmgLoading, SecureBootModel and ApECID require [OpenCore 0.6.1 (opens new window)](https://github.com/acidanthera/OpenCorePkg/releases)or newer
* Note 2: macOS Big Sur requires OpenCore 0.6.3+ for proper Apple Secure Boot support

#### What is Apple Secure Boot

* Information based off of [vit9696's thread (opens new window)](https://applelife.ru/posts/905541), [Apple's T2 docs (opens new window)](https://www.apple.com/euro/macbook-pro-13/docs/a/Apple_T2_Security_Chip_Overview.pdf)and [Osy's Secure Boot page(opens new window)](https://osy.gitbook.io/hac-mini-guide/details/secure-boot)

To best understand Apple Secure Boot, lets take a look at how the boot process works in Macs vs OpenCore in regards to security:

As we can see, there's several layers of trust incorporated into Apple Secure Boot:

* OpenCore will verify the boot.efi manifest (e.g. boot.efi.j137ap.im4m) to ensure that boot.efi was signed by Apple and can be used by this Secure Boot model.
	* For non-zero ApECID, OpenCore will additionally verify the ECID value, written in the boot.efi manifest (e.g. boot.efi.j137ap.XXXXXXXX.im4m), to ensure that a compromised hard drive from a different machine with the same Secure Boot model cannot be used in your computer.
* boot.efi will verify the kernelcache to ensure it has not been tampered with
* apfs.kext and AppleImage4 ensure your System Volume's snapshot has not been tampered with(Only applicable with Big Sur+)

Not all of these verifications are required to boot, but they're all possible for those who want maximum security. Currently information regarding firmware-based Secure Boot is not covered however all Apple Secure Boot options are detailed below.

#### DmgLoading
Quite a simple setting however important in regards to Apple Secure Boot. This setting allows you to set load policy with DMGs in OpenCore. By default we recommend using `Signed` however for best security `Disabled` may be preferred.

Possible options for `Misc -> Security -> DmgLoading`:

| Value    | Comment                                                      |
| :------- | :----------------------------------------------------------- |
| Any      | Allows all DMGs to load in OpenCore, however this option will cause a boot failure if Apple Secure Boot is enabled |
| Signed   | Allows only Apple-signed DMGs like macOS installers to load  |
| Disabled | Disables all external DMG loading, however internal recovery is still allowed with this option |

#### SecureBootModel
SecureBootModel is used set the Apple Secure Boot hardware model and policy, allowing us to enable Apple's Secure Boot with any SMBIOS even if the original SMBIOS did not support it(ie. no T2 present on pre-2017 SMBIOS). Enabling SecureBootModel is the equivalent of ["Medium Security" (opens new window)](https://support.apple.com/HT208330), for Full Security please see [ApECID](https://dortania.github.io/OpenCore-Post-Install/universal/security/applesecureboot.html#apecid)

Currently the following options for `Misc -> Security -> SecureBootModel` are supported:

| Value     | SMBIOS                                   | Minimum macOS Version |
| :-------- | :--------------------------------------- | :-------------------- |
| Disabled  | No model, Secure Boot will be disabled.  | N/A                   |
| Default   | Currently set to x86legacy               | 11.0.1 (20B29)        |
| j137      | iMacPro1,1 (December 2017)               | 10.13.2 (17C2111)     |
| j680      | MacBookPro15,1 (July 2018)               | 10.13.6 (17G2112)     |
| j132      | MacBookPro15,2 (July 2018)               | 10.13.6 (17G2112)     |
| j174      | Macmini8,1 (October 2018)                | 10.14 (18A2063)       |
| j140k     | MacBookAir8,1 (October 2018)             | 10.14.1 (18B2084)     |
| j780      | MacBookPro15,3 (May 2019)                | 10.14.5 (18F132)      |
| j213      | MacBookPro15,4 (July 2019)               | 10.14.5 (18F2058)     |
| j140a     | MacBookAir8,2 (July 2019)                | 10.14.5 (18F2058)     |
| j152f     | MacBookPro16,1 (November 2019)           | 10.15.1 (19B2093)     |
| j160      | MacPro7,1 (December 2019)                | 10.15.1 (19B88)       |
| j230k     | MacBookAir9,1 (March 2020)               | 10.15.3 (19D2064)     |
| j214k     | MacBookPro16,2 (May 2020)                | 10.15.4 (19E2269)     |
| j223      | MacBookPro16,3 (May 2020)                | 10.15.4 (19E2265)     |
| j215      | MacBookPro16,4 (June 2020)               | 10.15.5 (19F96)       |
| j185      | iMac20,1 (August 2020)                   | 10.15.6 (19G2005)     |
| j185f     | iMac20,2 (August 2020)                   | 10.15.6 (19G2005)     |
| x86legacy | Non-T2 Macs in 11.0(Recommended for VMs) | 11.0.1 (20B29)        |

#### Special Notes with SecureBootModel

* The

```
Default
```

value is not recommended as if you plan to use this with ApECID for full security, we recommend setting a proper value (i.e. closest to your SMBIOS or versions of macOS you plan to boot) since the

```
Default
```

value is likely to be updated in the future.

* In addition, `Default` is set to `x86legacy` which will breaking booting High Sierra through Catalina.
* `x86legacy` is not required for normal Mac models without T2's, any of the above values are supported.

* The list of cached drivers may be different, resulting in the need to change the list of Added or Forced kernel drivers.

* ie. IO80211Family cannot be injected in this case, as it is already present in the kernelcache

* Unsigned and several signed kernel drivers cannot be used

* This includes Nvidia's Web Drivers in 10.13

* System volume alterations on operating systems with sealing, like macOS 11, may result in the operating system being unbootable.

* If you plan to disable macOS's APFS snapshots, please remember to disable SecureBootModel as well

* Certain boot errors are more likely to be triggered with Secure Boot enabled that were previously not required

* Commonly seen with certain APTIO IV systems where they may not require IgnoreInvalidFlexRatio and HashServices initially however Secure Boot does.

* On older CPUs (ie. before Sandy Bridge) enabling Apple Secure Boot might cause slightly slower loading by up to 1 second

* Operating systems released before Apple Secure Boot landed (ie. macOS 10.12 or earlier) will still boot until UEFI Secure Boot is enabled. This is so,

* This is due to Apple Secure Boot assuming they are incompatible and will be handled by the firmware just like Microsoft Windows is

* Virtual Machines will want to use

```
x86legacy
```

for Secure Boot support

* Note using any other model will require `ForceSecureBootScheme` enabled

 Due to an annoying bug on Apple's end, certain systems may be missing the secure boot files themselves on the drive. Because of this, you may get issues such as:

```text
OCB: LoadImage failed - Security Violation
```

To resolve, run the following in macOS:

```bash
# First, find your Preboot volume
diskutil list

# From the below list, we can see our Preboot volume is disk5s2
/dev/disk5 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +255.7 GB   disk5
                                 Physical Store disk4s2
   1:                APFS Volume ⁨Big Sur HD - Data⁩       122.5 GB   disk5s1
   2:                APFS Volume ⁨Preboot⁩                 309.4 MB   disk5s2
   3:                APFS Volume ⁨Recovery⁩                887.8 MB   disk5s3
   4:                APFS Volume ⁨VM⁩                      1.1 MB     disk5s4
   5:                APFS Volume ⁨Big Sur HD⁩              16.2 GB    disk5s5
   6:              APFS Snapshot ⁨com.apple.os.update-...⁩ 16.2 GB    disk5s5s
# Now mount the Preboot volume
diskutil mount disk5s2

# CD into your Preboot volume
# Note the actual volume is under /System/Volumes/Preboot in macOS
# however in Recovery it's simply under /Volumes/Preboot
cd /System/Volumes/Preboot

# Grab your UUID
ls
 46923F6E-968E-46E9-AC6D-9E6141DF52FD
 CD844C38-1A25-48D5-9388-5D62AA46CFB8

# If multiple show up(ie. you dual boot multiple versions of macOS), you will
# need to determine which UUID is correct.
# Easiest way to determine is printing the value of .disk_label.contentDetails
# of each volume.
cat ./46923F6E-968E-46E9-AC6D-9E6141DF52FD/System/Library/CoreServices/.disk_label.contentDetails
 Big Sur HD%

cat ./CD844C38-1A25-48D5-9388-5D62AA46CFB8/System/Library/CoreServices/.disk_label.contentDetails
 Catalina HD%

# Next lets copy over the secure boot files, recovery will need different commands

# Example commands for inside macOS
# Replace CD844C38-1A25-48D5-9388-5D62AA46CFB8 with your UUID value
cd ~
sudo cp -a /usr/standalone/i386/. /System/Volumes/Preboot/CD844C38-1A25-48D5-9388-5D62AA46CFB8/System/Library/CoreServices

# Example commands for Recovery
# Replace Macintosh\ HD and CD844C38-1A25-48D5-9388-5D62AA46CFB8 with
# your System Volume's name and Preboot's UUID
cp -a /Volumes/Macintosh\ HD/usr/standalone/i386/. /Volumes/Preboot/CD844C38-1A25-48D5-9388-5D62AA46CFB8/System/Library/CoreServices
```

Now you can enable SecureBootModel and reboot without issue! And since we're not editing the system volume itself we don't need to worry about disabling SIP or breaking macOS snapshots.

#### ApECID
ApECID is used as an Apple Enclave Identifier, what this means is it allows us to use personalized Apple Secure Boot identifiers and achieve ["Full Security" (opens new window)](https://support.apple.com/HT208330)as per Apple's secure boot page(when paired with SecureBootModel).

To generate your own ApECID value, you'll want some form of cryptographically secure random number generator that will output a 64-bit integer. Below we provide an example that can be run if [Python 3 (opens new window)](https://www.python.org/downloads/)is installed on your machine:

```py
python3 -c 'import secrets; print(secrets.randbits(64))'
```

With this unique 64-bit int, you can now enter it under Misc -> ApECID in your config.plist

However before setting ApECID, there's a few things we need to note:

* Fresh installs with ApECID set to a non-zero value will require a network connection at install time for verification
* SecureBootModel should have a defined value instead of `Default` to avoid issues if the value were to change in later OpenCore versions.
* Pre-existing installs will need to personalize the volume, for this you'll need to first reboot into recovery and run the following command(Replace `Macintosh HD` with your system's volume name):

```sh
# Run this command after setting your ApECID value
# You'll also need an active network connection in recovery to run this command
bless --folder "/Volumes/Macintosh HD/System/Library/CoreServices" --bootefi --personalize
```

And something to note when reinstalling macOS 10.15 or older is that you may receive "Unable to verify macOS" error message. To work around this issue, you'll want to allocate a dedicated RAM disk of 2 MBs for macOS personalization by entering the following commands in the macOS recovery terminal before starting the installation:

```sh
disk=$(hdiutil attach -nomount ram://4096)
diskutil erasevolume HFS+ SecureBoot $disk
diskutil unmount $disk
mkdir /var/tmp/OSPersonalizationTemp
diskutil mount -mountpoint /var/tmp/OSPersonalizationTemp $disk
```

## 7.0NVRAM

### 7.1 系统Boot-args
众所周知，在 Mac 设备启动时通过 Command + V 或者 Command +S 这类快捷键改变启动行为。但是除此以外，通过 NVRAM 或第三方引导程序（如 Clover）中也可以设置启动参数。macOS 内置了许多启动参数，可以用于专业用户调试或排除故障。

在 macOS 中你可以通过 `nvram` 工具设置启动参数：

```bash
sudo nvram boot-args="-v"
```

当然，对于使用第三方引导程序（如 Clover 等）的用户来说，可以在 `config.plist` 文件、或者在启动菜单中设置引导参数。

- [ ] - - -
* `-v`：以「啰嗦模式」启动，等价于 Mac 设备上的快捷键 Command + V。

* `-x`：以「安全模式」启动，等价于 Mac 设备上的长按 Shift 键。在这一模式下 macOS 会加载尽可能少的内核扩展（kext）文件。

* `-s`：以「单用户模式」启动，等价于 Mac 设备上的快捷键 Command + S。这一模式将会启动终端模式，你可以用这种方式修复你的系统。

* `f`：旧版的「安全模式」。

* `-f`：启动时强制重建内核扩展（kext）缓存。

* `config`：指定用于代替 `com.apple.Boot.plist` 的配置文件。`config=sukka`将会加载 `/Library/Preferences/SystemConfiguration/sukka.plist`

* `arch=x86_64`：在 OS X Snow Leopard 中，即使已经内置了 64 位内核，系统也依然默认启动 32 位内核。这个启动参数可以使系统强制使用 64 位内核。如果要使系统总是以 32 位内核启动，请将 `x86_64` 部分替换为 `i386`。在某些情况下，第三方 kext 可能只有 32 位或 64 位，需要启动到相应的内核类型才能加载。

* `-legacy`：启动到 32 位内核。

* `maxmem=32`：将可寻址内存限制为指定的容量，本例中为 32 GB。如果没有这一启动参数，macOS 会将内存限制设置为硬件可以寻址的最大容量、亦或是安装的内存容量。

* `cpus=1`：限制系统中活动 CPU 的数量。苹果的开发者工具有一个选项用于启用或禁用系统中的一些 CPU，但你也可以通过这个参数指定要使用的 CPU 数量。在某些情况下，这也许有助于省电、或者你正在调试 X86 电源驱动，否则这一选项真的没什么别的作用。

* `rd=disk0s1`：强制指定启动分区。

* `rp`：根目录位置。

* `trace`：Kernel Trace 缓冲区大小。

* `iog=0x0`：这一参数强制 macOS 在笔记本上 不使用 Clamshell 模式。此时当你外接了显示器和键盘，合盖后笔记本不会睡眠、但内置显示器将会关闭。

* `serverperfmode`：为 macOS Server 开启 [性能模式](https://support.apple.com/en-gb/HT202528)

* `_panicd_ip=11.4.5.14`：设置一个 Kernel Panic 收集服务器的 IP 地址，日志将会通过 UDP 协议发送给这个 IP 的 1069 端口。

* `panicd_port`：修改日志发送端口（默认为 1069）。

* ```
debug=0x144
```

：这一参数结合了内核调试功能来显示内核进程的额外信息，这在 Kernel Panic 时很有用。可用的参数包括以下这些：

* 0x1：DB_HALT，在引导时暂停，直到外部调试串口已经连接并被识别
* 0x2：DB_PRT，将内核的 `printf()` 函数输出的信息打印到 `Console.app`
* 0x4：DB_NMI，启用内核调试功能，包括生成非屏蔽中断（NMI）。在 Power Mac 上，只需简单地按下电源键就能产生 NMI。在笔记本电脑上，在按下电源键时必须按住命令键。如果按住电源键超过五秒钟，系统将关闭电源。在「系统偏好设置」中更改启动盘时 DB_NMI 位将被清除。
* 0x8：DB_KPRT，将 `kprintf()` 产生的内核调试输出发送到远程输出设备，通常是一个调试串口（如果有的话）。注意， `kprintf()` 的输出是同步的。
* 0x10：DB_KDB，使用 KDB 代替 GDB 作为默认的内核调试器。与 GDB 不同，KDB 必须被显式编译到内核中。此外，基于 KDB 的调试需要原生的串口硬件（而不是基于 USB 的串口适配器）。
* 0x20：SB_SLOG，启用将杂项诊断记录到系统日志中。设置了这个位后，`load_shared_file()` 内核函数会记录额外的信息。
* 0x40：DB_ARP，允许跨局域网调试内核。
* 0x80：DB_KDP_BP_DIS，已经被弃用，用于支持旧版的 GDB。
* 0x100：DB_LOG_PI_SCRN，禁用五国、而把 Kernel Panic 的相关数据直接打印在屏幕上。这一参数还可以用于 Core Dump。
* 0x200：DB_KDP_GETC_ENA，在 Kernel Panic 后启用快捷键（c 继续，r 重启，k 进入 KDB）
* 0x400：DB_KERN_DUMP_ON_PANIC，当 Kernel Panic 时触发一次 Core Dump。
* 0x800：DB_KERN_DUMP_ON_NMI，当产生 NMI 时触发一次 Core Dump。
* 0x1000：DB_DBG_POST_CORE，等待调试器连接（如果使用 GDB）或在 NMI 触发的内核转储后等待调试器（如果使用 KDB）。如果没有设置 DB_DBG_POST_CORE，内核在 Core Dump 后继续运行。
* 0x2000：只生成并发送 Kernel Panic Log，不生成完整的 Core Dump。

* `artsize`：指定要用于地址解析表（ART）的页数。

* `BootCacheOverride`：BootCache 驱动程序被加载，但从网络启动时不会运行。设置 `BootCacheOverride=1` 可以覆盖此行为。

* `dart`：设置 `dart=0` 会关闭 64 位硬件上的系统 PCI 地址映射器（DART）。DART 在拥有 2GB 以上物理内存的机器上是必需的，但在所有机器上无论内存大小，默认情况下都会启用 DART

* `diag`：启用内核的内置诊断接口及其特定功能。

* `fill`：指定一个整数值，在启动是将会用这个整数填充所有内存。

* `fn`：改变处理器的强制休眠行为。设置 `fn=1` 将关闭强制休眠；设置 `fn=2` 将开启强制休眠。

* `_fpu`：禁用x86上的FPU功能。`_fpu=387` 将禁用 FXSR/SSE/SSE2，而字符串值为 `_fpu=se` 的字符串值将禁用 SSE2。

* `hfile`：休眠文件的名称（这一参数也会修改 sysctl 中的 `kern.hibernatefile`变量）。

* `io`：I/O Kit 驱动调试位。设置为 `0x00200000` （即 kIOLogSynchronous）时会使 `IOLog()` 函数同步执行。

* `novmx=1`：禁用 AltiVec。

* `pcata=0`：禁用板载 PC ATA 驱动器。

* `pmsx=1`：在 OS X 10.4.3 上启用实验性电源管理（PMS）。

* `_router_ip=11.4.5.14`：使用跨局域网内核调试时指定网关 IP。

* `serial=1`，启用串口调试。

* `smbios=1`：在 SMBIOS 驱动中启用详细的日志信息，仅限于 32 位机器。

* `vmdx` 和`pmdx`：内核启动时在内存中创建一个分区，参数格式为 `base.size`，其中 base 是对齐的内存地址、size 是内存页面大小的倍数。vmdx 指虚拟内存、pmdx 指物理内存。创建成功后将会被分别挂载在 `dev/mdx` 和 `dev/emdx` 下。

* `-b`：不执行 `/etc/rc.boot`。

* `-l`：日志中输出内存泄漏相关记录。

* `srv=1`：如果你在 X Servers 或 macOS Server 系统中使用这一参数，macOS 会修改内核的电源和网络参数，提升作为服务器的性能。

* `nvram_paniclog`：将 Kernel Panic 日志写入 NVRAM。

* `acpi`：启用 AppleACPIPlatform 调试。

* `acpi_level`：ACPI 调试等级。

* `idlehalt=1`：无视所有空闲进程、使 CPU 进入低功率模式。

* `platform`：`platform=X86PC` 强制禁用 ACPI 电源管理；`platform=ACPI` 强制启用 ACPI 电源管理。

* `keepsyms=1`：保留 KLD/Address-Symbol 翻译。

### 7.2 kexts的启动参数
因为kexts很多，可以到对应kext的README文件中查看。

因为显卡的最关键，也最复杂，也最需要，所以我来介绍一下whatevergreen支持的启动参数：

* `-wegdbg` to enable debug printing (available in DEBUG binaries).
* `-wegoff` to disable WhateverGreen.
* `-wegbeta` to enable WhateverGreen on unsupported OS versions (12 and below are enabled by default).
* `-wegnoegpu` to disable all external GPUs (or add `disable-gpu` property to each GFX0).
* `-wegnoigpu` to disable internal GPU (or add `disable-gpu` property to IGPU)
* `-wegswitchgpu` to disable internal GPU when external GPU is installed (or add `switch-to-external-gpu`property to IGPU)
* `-radvesa` to disable ATI/AMD video acceleration completely.
* `-rad24` to enforce 24-bit display mode.
* `-raddvi` to enable DVI transmitter correction (required for 290X, 370, etc.).
* `-radcodec` to force the spoofed PID to be used in AMDRadeonVADriver
* `radpg=15` to disable several power-gating modes (see FAQ, required for Cape Verde GPUs).
* `agdpmod=vit9696` disables check for `board-id` (or add `agdpmod` property to external GPU).
* `agdpmod=pikera` replaces `board-id` with `board-ix`
* `agdpmod=ignore` disables AGDP patches (`vit9696,pikera` value is implicit default for external GPUs)
* `ngfxgl=1` boot argument (and `disable-metal` property) to disable Metal support on NVIDIA
* `ngfxcompat=1` boot argument (and `force-compat` property) to ignore compatibility check in NVDAStartupWeb
* `ngfxsubmit=0` boot argument (and `disable-gfx-submit` property) to disable interface stuttering fix on 10.13
* `-ngfxdbg` boot argument to enable NVIDIA driver error logging
* `gfxrst=1` to prefer drawing Apple logo at 2nd boot stage instead of framebuffer copying.
* `gfxrst=4` to disable framebuffer init interaction during 2nd boot stage.
* `igfxframe=frame` to inject a dedicated framebuffer identifier into IGPU (only for TESTING purposes).
* `igfxsnb=0` to disable IntelAccelerator name fix for Sandy Bridge CPUs.
* `igfxgl=1` boot argument (and `disable-metal` property) to disable Metal support on Intel.
* `igfxmetal=1` boot argument (and `enable-metal` property) to force enable Metal support on Intel for offline rendering.
* `igfxpavp=1` boot argument (and `igfxpavp` property) to force enable PAVP output
* `igfxfw=2` boot argument (and `igfxfw` property) to force loading of Apple GuC firmware
* `-igfxvesa` to disable Intel Graphics acceleration.
* `-igfxnohdmi` boot argument (and `disable-hdmi-patches`) to disable DP to HDMI conversion patches for digital sound.
* `-igfxtypec` to force DP connectivity for Type-C platforms.
* `-cdfon` (and `enable-hdmi20` property) to enable HDMI 2.0 patches.
* `-igfxdump` to dump IGPU framebuffer kext to `/var/log/AppleIntelFramebuffer_X_Y` (available in DEBUG binaries).
* `-igfxfbdump` to dump native and patched framebuffer table to ioreg at IOService:/IOResources/WhateverGreen
* `applbkl=0` boot argument (and `applbkl` property) to disable AppleBacklight.kext patches for IGPU. In case of custom AppleBacklight profile- [read here.](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.OldPlugins.en.md)
* `-igfxmlr` boot argument (and `enable-dpcd-max-link-rate-fix` property) to apply the maximum link rate fix.
* `-igfxhdmidivs` boot argument (and `enable-hdmi-dividers-fix` property) to fix the infinite loop on establishing Intel HDMI connections with a higher pixel clock rate on SKL, KBL and CFL platforms.
* `-igfxlspcon` boot argument (and `enable-lspcon-support` property) to enable the driver support for onboard LSPCON chips. [Read the manual](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#lspcon-driver-support-to-enable-displayport-to-hdmi-20-output-on-igpu)
* `-igfxi2cdbg` boot argument to enable verbose output in I2C-over-AUX transactions (only for debugging purposes).
* `igfxagdc=0` boot argument (`disable-agdc` device property) to disable AGDC.
* `igfxfcms=1` boot argument (`complete-modeset` device property) to force complete modeset on Skylake or Apple firmwares.
* `igfxfcmsfbs=` boot argument (`complete-modeset-framebuffers` device property) to specify indices of connectors for which complete modeset must be enforced. Each index is a byte in a 64-bit word; for example, value `0x010203` specifies connectors 1, 2, 3. If a connector is not in the list, the driver's logic is used to determine whether complete modeset is needed. Pass `-1` to disable.
* `igfxonln=1` boot argument (`force-online` device property) to force online status on all displays.
* `igfxonlnfbs=MASK` boot argument (`force-online-framebuffers` device property) to specify indices of connectors for which online status is enforced. Format is similar to `igfxfcmsfbs`.
* `wegtree=1` boot argument (`rebuild-device-tree` property) to force device renaming on Apple FW.
* `igfxrpsc=1` boot argument (`rps-control` property) to enable RPS control patch (improves IGPU performance).
* `-igfxcdc` boot argument (`enable-cdclk-frequency-fix` property) to support all valid Core Display Clock (CDCLK) frequencies on ICL platforms. [Read the manual](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#support-all-possible-core-display-clock-cdclk-frequencies-on-icl-platforms)
* `-igfxdvmt` boot argument (`enable-dvmt-calc-fix` property) to fix the kernel panic caused by an incorrectly calculated amount of DVMT pre-allocated memory on Intel ICL platforms.
* `-igfxblr` boot argument (and `enable-backlight-registers-fix` property) to fix backlight registers on KBL, CFL and ICL platforms.
* `-igfxmpc` boot argument (`enable-max-pixel-clock-override` and `max-pixel-clock-frequency`properties) to increase max pixel clock (as an alternative to patching CoreDisplay.framework).
* `-igfxbls` boot argument (and `enable-backlight-smoother` property) to make brightness transitions smoother on IVB+ platforms. [Read the manual](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#customize-the-behavior-of-the-backlight-smoother-to-improve-your-experience)
* `-igfxdbeo` boot argument (and `enable-dbuf-early-optimizer` property) to fix the Display Data Buffer (DBUF) issues on ICL+ platforms. [Read the manual](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#fix-the-issue-that-the-builtin-display-remains-garbled-after-the-system-boots-on-icl-platforms)
* `applbkl=3` boot argument (and `applbkl` property) to enable PWM backlight control of AMD Radeon RX 5000 series graphic cards [read here.](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.Radeon.en.md)

### 7.3  NVRAM 参数设置

### 7.4 Quirks

## 8.0 PlatformInfo

### 8.1 机型众多，该如何选择？
可以选择的机型有iMac与Macmini，因为iMac pro和Mac Pro没办法驱动核显。虽然iMac机型的性能释放更好，但是它对电能小憩以及休眠极其不友好，所以我换成了Mac mini的机型，只有这样才能完美地睡眠。

### 8.2 Quirks

## 9.0 UEFI

## 10.0 如何定制USB？
macOS 10.14.1+ 的USB端口限制补丁已经失效了，因此无法一次配置所有端口。 [RehabMan](https://github.com/RehabMan)已更新 `USBInjectAll.kext` 并已包含用于排除端口组的引导标志。

1. 将 `USBInjectAll.kext` (用于端口发现) 放入 `EFI/CLOVER/kexts/Other`

2. Clover ▸ DSDT 重命名 (如果需要)

* ☑️ XHC1 ▸ XHC
* ☑️ EHC1 ▸ EH01
* ☑️ EHC2 ▸ EH02

[file:33DD4E91-C38C-4028-84FD-86E15A46D9D1-57229-00004F481F3B8CA1/USBPatch.png?download=1]USBPatch
[file:E5463912-153A-4781-A0BA-E141E92A68B2-57229-00004F482A463880/USBPatch2.png?download=1]USBPatch2

3. 重新启动

4. 运行 Hackintool 然后转到 工具栏 ▸ `已安装` 检查 `USBInjectAll` 是否安装正确

* 如果你看到：`USBInjectAll: Yes (Release-0.7.1)` 就说明没问题了

5. 转到 工具栏 ▸ `USB` 来查看 USB 控制器列表。因为这里需要基于 USB控制器 您可能需要安装额外的 kexts:

* 8086:8CB1 和 macOS (10.11.1) ▸ 请使用 `XHCI-9-series.kext`
* 8086:8D31, 8086:A2AF, 8086:A36D, 8086:9DED ▸ `请使用 XHCI-unsupported.kext`
* 8086:1E31, 8086:8C31, 8086:8CB1, 8086:8D31, 8086:9C31, 8086:9CB1 ▸ `请使用 FakePCIID.kext + FakePCIID_XHCIMux.kext`

6. 如果您缺少了其中一个附加的 kexts，*请完成安装并立即重新启动*，然后再次运行 Hackintool

7. 转到 工具栏 ▸ `USB` 选项

8. *依次选择 USB 端口列表中的各个项目，然后单击“删除”删掉全部项目，完成后再单击“刷新”按钮*

9. 使用 `-uia_exclude_ss uia_include=HS01,HS02`

* 这里的 `HS01` 和 `HS02` 是鼠标和键盘，请根据*自己的设备位置*对其进行更改

10. 运行 Hackintool 并转到 工具栏 ▸ `USB` 选项

```
- [ ] [ ] 用一个 USB 2.0 的设备将所有的2.0端口（通常为黑色）全部插拔一遍
- [ ] [ ] 活动的端口将以`绿色`突出显示
  ![USBOrigin](https://pics.daliansky.net/d/xD0Ar91B/blog/Hackintool-Screen-Shot-for-blog/USBOrigin.png?download=1)
```

11. 删除所有*未*突出显示为绿色的端口，请您也用小本将活动端口的数据记下来，以防万一

12. 删除 `-uia_exclude_ss` 引导标志，并使用 `-uia_exclude_hs` 引导标志重新启动

13. 运行 Hackintool 并转到工具栏 ▸ `USB` 选项

```
- [ ] [ ] 用 USB 3.0 的设备将所有的3.0端口（蓝色_红色_黄色）全部插拔一遍
- [ ] [ ] USB Type-C 接口的设备需要用正反两面对所有的端口进行插拔
- [ ] [ ] 活动的端口将以绿色突出显示
```

14. 删除所有未突出显示为绿色的端口，请您也用小本将活动端口的数据记下来，以防万一
[file:E6CF3323-AEA2-4DDC-AB58-98079344B8A3-57229-00004F48342388EE/USBDEL.png?download=1]USBDEL

15. 使用下拉列表将每个端口设置为适当的接口类型

```
- [ ] [ ] 永久连接设备的USB端口（例如M.2蓝牙卡）应设置为 `Internal` (内建)
- [ ] [ ] 与 USB3 端口相连的 HSxx 端口 (USB2) 应设置为 `USB3`
- [ ] [ ] 内部集线器通常连接到端口PR11和PR21，因此应设置为 `Internal` (内建)
- [ ] [ ] USB Type-C 接口可以是9或10，这取决于硬件如何处理 USB Type-C 型设备/电缆的正反两种可能方向
- [ ] [ ] 如果 USB Type-C 在两个方向上使用相同的 SSxx，则它具有内建切换器，因此应设置为 `TypeC+Sw`
- [ ] [ ] 如果 USB Type-C 在两个方向使用不同的 SSxx，则它没有内建切换器，因此应设置为 `TypeC`
```

```
![USBTest](https://pics.daliansky.net/d/xD0Ar91B/blog/Hackintool-Screen-Shot-for-blog/USBTest.png?download=1)
![USBFinal](https://pics.daliansky.net/d/xD0Ar91B/blog/Hackintool-Screen-Shot-for-blog/USBFinal.png?download=1)
```

16. 使用 `导出` 按钮在桌面上生成 USB 修复文件

```
![USBOutput](https://pics.daliansky.net/d/xD0Ar91B/blog/Hackintool-Screen-Shot-for-blog/USBOutput.png?download=1)
![USBOutput2](https://pics.daliansky.net/d/xD0Ar91B/blog/Hackintool-Screen-Shot-for-blog/USBOutput2.png?download=1)
- [ ] [ ] 复制 `SSDT-EC.aml` (如果有) 到 EFI ▸ CLOVER ▸ ACPI ▸ patched
- [ ] [ ] 接下来的方案请
  2 选 1：
	- [ ] [ ] A) 复制 USBPorts.kext 到 EFI ▸ CLOVER ▸ kexts ▸ Other；或者
	- [ ] [ ] B) 复制 SSDT-UIAC.aml 和 SSDT-USBX.aml (如果有) 到 EFI ▸ CLOVER ▸ ACPI ▸ patched
```

17. 好了，是时候清除不需要的补丁和文件了:

```
- [ ] [ ] 删除 (`-uia_exclude_ss`，`-uia_exclude_hs` 和 `uia_include=x`)
- [ ] [ ] 删除 `USBInjectAll.kext` （如果您使用的是 USB-Ports）
```

18. 重启

19. 运行 Hackintool 然后转到 工具栏 ▸ `USB`

20. 依次选择 USB 端口列表中的各个项目，然后单击“删除”删掉全部项目，完成后再单击“刷新”按钮

```
- [ ] [ ] 您现在可以检查你的 USB 是否全部正常工作
- [ ] [ ] 如果您要更改USB端口类型，请在更改后重新生成修复文件并替换之前的文件
- [ ] [ ] 如果您一不小心删了修补文件，请重新从头来过，或者您可以用您的小本 ⊙﹏⊙∥∣°
```

*FAQ*
Q. 什么是 `USBPorts.kext` ?
A. 它是一个 无代码的核心驱动 用于注入 USB 端口，让所有的USB端口都能正常工作

Q. 我还需要在使用USBPorts.kext 的同时使用 `SSDT-UIAC.aml` 吗？
A. 不，这个方法生成的是一个空壳的无代码的kext驱动，无需同时使用 `SSDT-UIAC.aml`

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

### 11.5 特殊情况
有两个非常特殊的情况：

* 一些主板本身的DSDT就把RTC的地址写错了，我们需要更正。比如，华硕299的主板没有对rtc的0x72以及0x73进行映射，源dsdt中的资源分配如下代码。0x70只加到了0x72之前，从0x74位开始加到了0x78之前，也就是0x72以及0x73被忽略。因此需要官方的ssdt对rtc的资源进行重新分配

```
Device (RTC)
{
    Name (_HID, EisaId ("PNP0B00") _* AT Real-Time Clock *_)   _HID: Hardware ID
    Name (_CRS, ResourceTemplate ()   _CRS: Current Resource Settings
    {
        IO (Decode16,
            0x0070,              Range Minimum
            0x0070,              Range Maximum
            0x01,                Alignment
            0x02,                Length
            )
        IO (Decode16,
            0x0074,              Range Minimum
            0x0074,              Range Maximum
            0x01,                Alignment
            0x04,                Length
            )
        IRQNoFlags ()
            {8}
    })
```

* Smbus是一种可能已经被淘汰的总线方式，此部件在很多新的机型中已经不再被需要。但经证实在过往的一些mac版本中需要激活它才能进入睡眠唤醒，这是一种很奇怪的现象，但是一般来说此补丁不该被需要。请注意原文中的`0x57`应该根据自己的主板来决定。比如我的x299主板应该搜索`x299 intel datasheet`，获得自己主板的`datasheet`后，得知自己Smbus串口的位置为`0xc6`，替换原文中的所有`0x57`为`0xc6`。此SSDT应只用于debug。

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

```
例如（不要直接复制这条命令行）
 mv _Users_xjn_Downloads_OpenCore-0.6.2-RELEASE_Utilities_LogoutHook_LogoutHook.command ~_.LogoutHook.command
```

* 最后在 Terminal（终端）中，输入以下命令按回车：

```
sudo defaults write com.apple.loginwindow LogoutHook _Users_$USER/.LogoutHook.command

 
 
 
```

* 终端会提示要求你输入密码（密码打进去不会显示）。
* 重启，你会在 `ESP/EFI/` 下看到 `nvram.plist`，代表已经成功模拟了。

## 13.0 睡眠综述

### 13.1 准备
So to understand how to fix sleep issues in macOS, we need to first look at what contributes to sleep issues most of the time:

* Incorrectly managed devices(most commonly PCIe based devices)

The reason for this is when devices get an S3 call(or S0 for wake), the driver needs to power down the devices and put into a low state mode(vice versa when waking). Problems arise when such devices don't cooperate with the drivers and the main offenders of these issues are:

* USB Controllers and Devices
* GPUs
* Thunderbolt Controllers and Devices
* NICs(Both Ethernet and Wifi)
* NVMe Drives

And there are others that can cause sleep issues that aren't directly(or obviously) related to PCI/e:

* CPU Power Management
* Displays
* NVRAM
* RTC/System Clocks
* IRQ Conflicts
* Audio
* SMBus
* TSC

And something many people forget are over and under-clocks:

* CPUs
	* AVX often breaks iGPUs and hurt overall stability
* Bad RAM(Both overclocks and mismatched RAM)
	* Even bad/mismatched timings can cause serious issues
* Factory GPU Overclocks
	* OEMs commonly push a card a bit too far with their custom VBIOS
	* Generally these cards will have a physical switch, allowing you to choose a low power VBIOS

#### Preparations
*In macOS*:

Before we get in too deep, we'll want to first ready our system:

```text
sudo pmset autopoweroff 0
sudo pmset powernap 0
sudo pmset standby 0
sudo pmset proximitywake 0
sudo pmset tcpkeepalive 0
```

This will do 5 things for us:

1. Disables autopoweroff: This is a form of hibernation
2. Disables powernap: Used to periodically wake the machine for network, and updates(but not the display)
3. Disables standby: Used as a time period between sleep and going into hibernation
4. Disables wake from iPhone/Watch: Specifically when your iPhone or Apple Watch come near, the machine will wake
5. Disables TCP Keep Alive mechanism to prevent wake ups every 2 hours

*In your config.plist*:

While minimal changes are needed, here are the ones we care about:

* ```
Misc -> Boot -> HibernateMode -> None
```

* We're gonna avoid the black magic that is S4 for this guide

* ```
NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args
```

* `keepsyms=1` - Makes sure that if a kernel panic does happen during sleep, that we get all the important bits from it
* `swd_panic=1` - Avoids issue where going to sleep results in a reboot, this should instead give us a kernel panic log

*In your BIOS*:

* Disable:
	* Wake on LAN
	* Trusted Platform Module
		* Note that if you're using BitLocker in Windows, disabling this will result in all your encryption keys being lost. If you're using BitLocker, either disable or note that it may be a cause for wake issues.
	* Wake on USB(Certain boards may actually require this on to wake, but most will get random wakeup calls with it)
* Enable:
	* Wake on Bluetooth(If using a Bluetooth device for waking like a keyboard, otherwise you can disable)

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

### 13.2Fixing USB
This is the #1 cause of sleep issues on hacks, mainly because Apple's drivers are quite bad at guessing ports and the port limit patches have the ill-effect of creating instability.

* [USB Mapping](https://dortania.github.io/OpenCore-Post-Install/usb/)

This guide also includes some other fixes than just mapping:

* [Fixing USB Power](https://dortania.github.io/OpenCore-Post-Install/usb/misc/power.html)
* [Fixing Shutdown/Restart](https://dortania.github.io/OpenCore-Post-Install/usb/misc/shutdown.html)
* [GPRW/UPRW/LANC Instant Wake Patch](https://dortania.github.io/OpenCore-Post-Install/usb/misc/instant-wake.html)
* [Keyboard Wake Issues](https://dortania.github.io/OpenCore-Post-Install/usb/misc/keyboard.html)

*USB maps with macOS Catalina(10.15) and newer*: You may find that even with USB mapping, your sleep breaks. one possible solution is renaming the IOClass value from `AppleUSBMergeNub` to `AppleUSBHostMergeProperties`. See here for more info: [Changes in Catalina's USB IOClass(opens new window)](https://github.com/dortania/bugtracker/issues/15)

* Note: Some USB devices that do not have proper drivers in macOS can unfortunately result in sleep issues. For example, Corsair water coolers with USB addressable control can prevent the machine from sleeping correctly. For these situations, we recommend users disconnect these troublesome devices when debugging sleep issues.

### 13.3 Fixing GPUs
With GPUs, it's fairly easy to know what might be causing issues. This being unsupported GPUs in macOS. By default, any GPU that doesn't have drivers already provided in the OS will run off very basic drivers known as VESA drivers. These provide minimal display output but also cause a big issue in that macOS doesn't actually know how to properly interact with these devices. To fix this, well need to either trick macOS into thinking it's a generic PCIe device(which it can better handle, ideal for desktops) or completely power off the card(on laptops, desktop dGPUs have issues powering down)

* See here for more info:
	* [Disabling desktop dGPUs(opens new window)](https://dortania.github.io/Getting-Started-With-ACPI/Desktops/desktop-disable)
	* [Disabling laptop dGPUs(opens new window)](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/laptop-disable)

Special notes for iGPU users on 10.15.4 and newer:

* iGPU wake is partially broken due to numerous hacks apple uses in AppleGraphicsPowerManagement.kext with real Macs, to get around this you'll likely need `igfxonln=1` to force all displays online. Obviously test first to make sure you have this issue.
* AAPL,ig-platform-id `07009B3E` may fail for desktop Coffee Lake (UHD 630) users, you can try `00009B3E` instead. `0300923E` is also known to work sometimes.

Other iGPU notes:

* Some systems with iGPUs (e.g. Kaby Lake and Coffee Lake) may cause system instability in lower power states, and can sometimes manifest as NVMe kernel panics. To resolve, you can add `forceRenderStandby=0` to your boot-args to disable RC6 Render Standby. See here for more info: [IGP causes NVMe Kernel Panic CSTS=0xffffffff #1193(opens new window)](https://github.com/acidanthera/bugtracker/issues/1193)
* Certain Ice Lake laptops may also kernel panic on `Cannot allow DC9 without disallowing DC6`due to issues with transitioning states. A work around for this is using either `-noDC9` or `-nodisplaysleepDC6` in your boot-args

Special note for 4k Displays with AMD dGPUs:

* Some displays may fail to wake randomly, mainly caused by AGDC preferences. To fix, apply this to your dGPU in DeviceProperties:
	* `CFG,CFG_USE_AGDC | Data | 00`
	* You can find the PciRoot of your GPU with gfxutil
		* `/path/to/gfxutil -f GFX0`

### 13.4 Fixing Thunderbolt
Thunderbolt is a very tricky topic in the community, mainly due to the complexity of the situation. You really have just 2 paths to go down if you want Thunderbolt and sleep to work simultaneously:

* Disable Thunderbolt 3 in the BIOS
* Attempt to patch Thunderbolt 3:
	* [Thunderbolt 3 Fix(opens new window)](https://osy.gitbook.io/hac-mini-guide/details/thunderbolt-3-fix/)
	* [ThunderboltReset(opens new window)](https://github.com/osy86/ThunderboltReset)
	* [ThunderboltPkg(opens new window)](https://github.com/al3xtjames/ThunderboltPkg/blob/master/Docs/FAQ.md)

Note: Thunderbolt can be enabled without extra work /if/ you're ok without sleep, and vice versa.

### 13.5 Fixing NICs
NICs(network Interface Controllers) are fairly easy to fix with sleep, it's mainly the following:

* Disable

```
WakeOnLAN
```

in the BIOS

* Most systems will enter a sleep/wake loop with this enabled

* Disable

```
Wake for network access
```

in macOS(SystemPreferences -> Power)

* Seems to break on a lot of hacks

### 13.6 Fixing NVMe
So macOS can be quite picky when it comes to NVMe drives, and there's also the issue that Apple's power management drivers are limited to Apple branded drives only. So the main things to do are:

* Make sure the NVMe is on the latest firmware(especially important for [970 Evo Plus drives (opens new window)](https://www.tonymacx86.com/threads/do-the-samsung-970-evo-plus-drives-work-new-firmware-available-2b2qexm7.270757/page-14#post-1960453))
* Use [NVMeFix.kext (opens new window)](https://github.com/acidanthera/NVMeFix/releases)to allow for proper NVMe power management

And avoid problematic drives, the main culprits:

* Samsung's PM981 and PM991 SSDs
* Micron's 2200S

If you however do have these drives in your system, it's best to disable them via an SSDT: [Disabling desktop dGPUs (opens new window)](https://dortania.github.io/Getting-Started-With-ACPI/Desktops/desktop-disable.html). This guide is primarily for dGPU but works the exact same way with NVMe drives(as they're both just PCIe devices)

### 13.7 Fixing CPU Power Management
*For Intel*:

To verify you have working CPU Power Management, see the [Fixing Power Management](https://dortania.github.io/OpenCore-Post-Install/universal/pm.html) page. And if not, then patch accordingly.

Also note that incorrect frequency vectors can result in wake issues, so either verify you're using the correct SMBIOS or adjust the frequency vectors of your current SMBIOS with CPUFriend. Tools like [one-key-cpufriend (opens new window)](https://github.com/stevezhengshiqi/one-key-cpufriend)are known for creating bad frequency vectors so be careful with tools not used by Dortania.

A common kernel panic from wake would be:

```text
Sleep Wake failure in EFI
```

*For AMD*:

Fret not, for their is still hope for you as well! [AMDRyzenCPUPowerManagement.kext (opens new window)](https://github.com/trulyspinach/SMCAMDProcessor)can add power management to Ryzen based CPUs. Installation and usage is explained on the repo's README.md

### 13.8 Displays
So display issues are mainly for laptop lid detection, specifically:

* Incorrectly made SSDT-PNLF
* OS vs firmware lid wake
* Keyboard spams from lid waking it(On PS2 based keyboards)

The former is quite easy to fix, see here: [Backlight PNLF(opens new window)](https://dortania.github.io/Getting-Started-With-ACPI/)

For the middle, macOS's lid wake detection can bit a bit broken and you may need to outright disable it:

```sh
sudo pmset lidwake 0
```

And set `lidwake 1` to re-enable it.

The latter requires a bit more work. What we'll be doing is trying to nullify semi random key spams that happen on Skylake and newer based HPs though pop up in other OEMs as well. This will also assume that your keyboard is PS2 based and are running [VoodooPS2 (opens new window)](https://github.com/acidanthera/VoodooPS2/releases).

To fix this, grab [SSDT-HP-FixLidSleep.dsl (opens new window)](https://github.com/acidanthera/VoodooPS2/blob/master/Docs/ACPI/SSDT-HP-FixLidSleep.dsl)and adapt the ACPI pathing to your keyboard(`_CID`value being `PNP0303`). Once this is done, compile and drop into both EFI/OC/ACPI and under config.plist -> ACPI -> Add.

For 99% of HP users, this will fix the random key spam. If not, see below threads:

* [RehabMan's brightness key guide(opens new window)](https://www.tonymacx86.com/threads/guide-patching-dsdt-ssdt-for-laptop-backlight-control.152659/)

### 13.9 NVRAM
To verify you have working NVRAM, see the [Emulated NVRAM](https://dortania.github.io/OpenCore-Post-Install/misc/nvram.html) page to verify you have working NVRAM. And if not, then patch accordingly.

### 13.10 RTC
This is mainly relevant for Intel 300 series motherboards(Z3xx), specifically that there's 2 issues:

* Be default the RTC is disabled(instead using AWAC)
* The RTC is usually not compatible with macOS

To get around the first issue, see here: [Fixing AWAC(opens new window)](https://dortania.github.io/Getting-Started-With-ACPI/Universal/awac.html)

For the second one, it's quite easy to tell you have RTC issues when you either shutdown or restart. Specifically you'll be greeted with a "BIOS Restarted in Safemode" error. To fix this, we'll need to prevent macOS from writing to the RTC regions causing these issues. There are a couple fixes:

* DisableRtcChecksum: Prevent writing to primary checksum (0x58-0x59), works with most boards

* ```
UEFI -> ProtoclOverride -> AppleRtcRam 
```

* 

```
NVRAM -> Add -> rtc-blacklist
```

* Blacklists certain regions at the firmware level, see [Configuration.pdf (opens new window)](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf)for more info on how to set this up

* RTCMemoryFixup +

```
rtcfx_exclude=
```

* Blacklists certain regions at the kernel level, see README for more info on how to setup

With some legacy boards, you may actually need to patch your RTC: [Z68 RTC(opens new window)](https://www.insanelymac.com/forum/topic/329624-need-cmos-reset-after-sleep-only-after-login/)

### 13.11 IRQ Conflicts
IRQ issues usually occur during bootups but some may notice that IRQ calls can break with sleep, this fix is fairly easy:

* SSDTTime(opens new window)
	* First dump your DSDT in Linux/Windows
	* then select `FixHPET` option

This will provide you with both SSDT-HPET.aml and `patches_OC.plist`, You will want to add the SSDT to EFI_OC_ACPI and add the ACPI patches into your config.plist from the patches_OC.plist

### 13.12 Audio
Unmanaged or incorrectly managed audio devices can also cause issues, either disable unused audio devices in your BIOS or verify they're working correctly here:

* [Fixing Audio](https://dortania.github.io/OpenCore-Post-Install/universal/audio.html)

### 13.13 SMBus
Main reason you'd care about SMBus is AppleSMBus can help properly manage both SMBus and PCI devices like with power states. Problem is the kext usually won't load by itself, so you'll need to actually create the SSDT-SMBS-MCHC.

See here on more info on how to make it: [Fixing SMBus support(opens new window)](https://dortania.github.io/Getting-Started-With-ACPI/Universal/smbus.html)

### 13.13 TSC
The TSC(Time Stamp Counter) is responsible for making sure you're hardware is running at the correct speed, problem is some firmware(mainly HEDT/Server and Asus Laptops) will not write the TSC to all cores causing issues. To get around this, we have 3 options:

* CpuTscSync(opens new window)
	* For troublesome laptops
* VoodooTSCSync(opens new window)
	* For most HEDT hardware
* TSCAdjustReset(opens new window)
	* For Skylake X_W_SP and Cascade Lake X_W_SP hardware

The former 2 are plug n play, while the latter will need some customizations:

* Open up the kext(ShowPackageContents in finder, `Contents -> Info.plist`) and change the Info.plist -> `IOKitPersonalities -> IOPropertyMatch -> IOCPUNumber` to the number of CPU threads you have starting from `0`(i9 7980xe 18 core would be `35` as it has 36 threads total)
* Compiled version can be found here:TSCAdjustReset.kext

The most common way to see the TSC issue:

### 13.14 强制睡眠
睡眠即醒很大程度上跟 USB 的定制相关，一般一个好的 USB 定制就能解决睡眠即醒的问题。当然还有很多无法解决的问题，比如蓝牙不能在 HUB 下进行内建，等等。甚至有些时候我们都不知道为什么黑果会睡不着，那有没有一个办法让黑果强制睡眠呢？答案是有的。经过我的摸索，有几种方法能达到强制睡眠的效果，只是方法不同而已，但主要围绕的还是 `0d/6d` 的数值来做一些工作。这些方法涉及了很多 OC 领域的一些小技巧，我也顺便展示给大家。

> `0d/6d` 补丁是阻止一些部件参与唤醒工作，这其中包括了xhc部件，意味着你无法使用鼠标键盘唤醒，只能用电源键唤醒。但若你有一组除了xhc之外的usb控制器，那把键盘鼠标插在那两个控制器上，可以在使用强制睡眠的情况下用键盘鼠标唤醒电脑。    

- [ ] - - -
#### 13.14.1 分辨0D/06
我们打开之前提取的SSDT，随便搜索五大部件中的一个，比如说 XDCI：
![](hackintosh-Inspiron3670-franksfc/k5jDCx9c3WdJunh.png)截屏2019-11-07下午11.06.02.png

主要是看上图中 `XDCI` 下的 `_PRW` 属性值，可以直接看到 Return 的值为 `GPRW (0x6D, 0x04)`。其中 `6D` 这个数值看主板而定，有些主板叫做 `0D`，而后面 `04`这个值的含义为 S4 级别的电源管理，即休眠甚至关机情况下的唤醒；有些后面的数值是 `03`，代表着 S3 级电源管理。这个我打一个大家比较熟悉的例子， `GLAN` 这个网卡部件的 `PRW` 值也是 `0x04`，为什么要是 `04` 呢？因为这样我们可以使用远程通过网络启动主机功能。

#### 13.14.2 方法一: OC 全局重命名强制睡眠
上一步中已经确认了你的主板是 `0D` 还是 `06`，打开 `OC little` 的 `06/0D` 补丁，选择合适自己主板的补丁集，比如我的是 `Name-6D更名.plist`。将补丁抄入自己的 `config.plist` 后重启生效。

> 全局重命名会导致其他系统无法通过 OC 引导开机，不建议使用。    

- [ ] - - -
#### 13.14.3 方法二: 沿用Clover版本的0D/06补丁&展示TgtBridge在OC下的用法
宪武大大做的 clover 版本的 0d/6d 补丁，其实没啥必要讲，只是有留言问了 tgtbridge 在 oc 下怎么用，那我就展示一下吧。这个补丁原理是一样的，通过重命名的方式改 `_prw`。

直接下载宪武大大的[clover hotpatch]((null))补丁包，打开plist文件。那我们拿出一组数据来讲解怎么把它翻译成oc版本：

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

我总共数了一下，一共有56个_PRW。我们再在主内容栏上按 ⌘ +F 搜索 xhc，直接找到 xhc 的 `_PRW`，刚好我们看到我的 xhc 实在整张表的倒数第 4 个，也就是正数第 53 个：

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
#### 13.14.4 方法三：配合SSDT+重命名的强制睡眠补丁（推荐）
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

### 13.15 笔者的经历：睡眠后无法唤醒Wake failure in EFI
这个问题非常复杂，但也经常出现。是属于疑难杂症的一类，它要求所有黑苹果问题全部解决

主要有这么几类：

* USB定制

* RTC的问题

* CPU不恰当的频率设置，如CPUFriend的定制
* 不恰当的PCIE设备的设置，如显卡
* Bios设置有误
* ACPI错误

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

```
cd ~/desktop
mkdir cpu
cd cpu
git clone https://github.com/corpnewt/CPUFriendFriend.git
git clone https://github.com/acidanthera/CPUFriend.git cp ~/desktop/cpu/CPUFriend/tools/ResourceConverter.sh ~/desktop/cpu/
CpuFriendFriend/CPUFriendFriend.command
```

* 你会看到如图的命令行，这里1 of 4代表第一段睿频的设置，以此类推，数值越大睿频越高，下面要求你填写的是最低的频率值，你想要低一点的800MHZ就填08，高一点的1300MHZ就填0D（注意大小写）
* * 填完前两段后，它会要求你填写EPP值，EPP值越低，性能表现越强。我们是填的前两段的低频率部分，我们可以选择节能型的，比如0x80，如果你想极致性能，可以填0x00。
* 直至你填完所有4段变频需求后，便会生成你的变频plist。我们执行以下命令:

```
cd ~/desktop
mkdir cpu
cd cpu
git clone https://github.com/corpnewt/CPUFriendFriend.git
git clone https://github.com/acidanthera/CPUFriend.git cp ~/desktop/cpu/CPUFriend/tools/ResourceConverter.sh ~/desktop/cpu/
CpuFriendFriend/CPUFriendFriend.command
```

* 我们会在桌面的CPU文件夹中找到你所需要的`ResourceConverter.sh`以及Mac-xxxxxxx.plist两个文件
* 执行以下命令生成你最终需要的`CPUFriendDataProvider.kext`,注意命令行中`Mac-AA95B1DDAB278B95.plist`，请替换成你自己的文件名，这样我们就可以在桌面的CPU文件夹下拿到`CPUFriendDataProvider.kext`。

```
cd ~/Desktop/cpu
./ResourceConverter.sh --kext ~/Desktop/cpu/Mac-AA95B1DDAB278B95.plist
```

* 我们再到CPUFriend的[release](https://github.com/acidanthera/CPUFriend/releases)页面下，下载最新的release版本，得到里面的CPUFriend.kext
* 将`CPUFriendDataProvider.kext`与`CPUFriend.kext`一起放到oc/kexts下,并在config中加载，注意：`CPUFriend.kext`应该放在`CPUFriendDataProvider.kext`的前面。
* 感谢 @请叫我官人 的帮助。完成，自行测试。

## 15.0 休眠与电能小憩

### 15.1 MacOS Yosemite以前
As far as I know and as far as my system is concerned:

darkwake=0 -> Power Nap Disabled

darkwake=1 -> Power Nap Fully Enabled (System fully wakes. Fans on, monitors on, Hourly)

darkwake=2

darkwake=3

darkwake=4

darkwake=5

darkwake=6

darkwake=7

darkwake=8 -> Power Nap Enabled (System Fully wakes, sometimes monitor come on, sometimes not [don't know why])

darkwake=9

darkwake=10 -> Power Nap Enabled (Fans & monitors do not come on, system.log show the times that the computer woke from sleep. Time Machine Backup are performed hourly in sleep mode)

darkwake=11

Hopefully the others can elaborate on the other dark wake states.

### 15.2 Yosemite以来最新的darkwake设置
There are plenty of users who suggest to use _darkwake=8_ or _darkwake=10_ or _darkwake=0_ boot args to solve sleep issues.

Are these suggestions valid at all?

For first we have to clarify something about Darkwake. The DarkWake feature was for first introduced in Mac OS X Lion. This feature allows to wake up certain parts of computer from sleep, while leaving other parts in sleep mode (for example display etc). Darkwake = display stays dark when comp wakes and performs some tasks. When tasks are done, comp should go back to sleep. But on Hack's Darkwake caused several problems, for example comps went into state, where those become inaccessible and forced reboot were needed. On some cases Darkwake caused reboots too and.. Darwake is related to [Power Nap](https://support.apple.com/en-us/HT204032) which is available since OS X Mountain Lion.

- [ ] - - -

Darkwake and XNU

Darkwake is part of [XNU](https://github.com/apple/darwin-xnu/).

> [ ](https://www.insanelymac.com/forum/topic/342002-darkwake-on-macos-catalina-boot-args-darkwake8-darkwake10-are-obsolete/#) QuoteXNU kernel is part of the Darwin operating system for use in macOS and iOS operating systems. XNU is an acronym for X is Not Unix. XNU is a hybrid kernel combining the Mach kernel developed at Carnegie Mellon University with components from FreeBSD and a C++ API for writing drivers called IOKit. XNU runs on x86_64 for both single processor and multi-processor configurations.    
Darkwake behaviours are defined in IOPMrootDomain.cpp.

The easiest way to check XNU version is to use terminal command _uname -av_.

```
uname -av
Darwin MyHack.local 19.2.0 Darwin Kernel Version 19.2.0: Sat Nov  9 03:47:04 PST 2019; root:xnu-6153.61.1~20/RELEASE_X86_64 x86_64
```

According to this macOS Catalina 10.15.2 uses XNU 6153.61.1.

The latest publicly available source code of XNU is 4903.241.1. So we cannot check source code of latest XNU's. Please correct me if I'm wrong.

[Code below](https://github.com/apple/darwin-xnu/blob/master/iokit/Kernel/IOPMrootDomain.cpp#L1099) reveals that boot arg darkwake correlates to enum gDarkWakeFlags.

```c
PE_parse_boot_argn("darkwake", &gDarkWakeFlags, sizeof(gDarkWakeFlags));
```

So lets check the xnu-4903.241.1/iokit_Kernel_IOPMrootDomain.cpp for [Darkwake flags](https://github.com/apple/darwin-xnu/blob/master/iokit/Kernel/IOPMrootDomain.cpp#L308):

```c
// gDarkWakeFlags
enum {
    kDarkWakeFlagHIDTickleEarly      = 0x01, // hid tickle before gfx suppression
    kDarkWakeFlagHIDTickleLate       = 0x02, // hid tickle after gfx suppression
    kDarkWakeFlagHIDTickleNone       = 0x03, // hid tickle is not posted
    kDarkWakeFlagHIDTickleMask       = 0x03,
    kDarkWakeFlagAlarmIsDark         = 0x0100,
    kDarkWakeFlagGraphicsPowerState1 = 0x0200, 
    kDarkWakeFlagAudioNotSuppressed  = 0x0400
};
```

Let's translate these values into decimals and easier to read strings

```html
HID Tickle Early = 1
HID Tickle Late = 2
HID Tickle None = 3
HID Tickle Mask = 3
Alarm Is Dark = 256
Graphics Power State 1 = 512
Audio Not Suppressed = 1024
```

HID = Human-interface devices, such as keyboards, pointing devices, and digitizers like pens and touch pads.

As flags are used for [bitwise operations](https://en.wikipedia.org/wiki/Bitwise_operation), then we can easily notice that combinations 10 and 8 are for sure invalid now. darkwake=8 equals actually to darkwake=0 and darkwake=10 equals to darkwake=2.

If we check older XNU versions, then these values are removed since XNU 2782.1.97 ( = Yosemite ):

```c
    kDarkWakeFlagIgnoreDiskIOInDark  = 0x04, // ignore disk idle in DW
    kDarkWakeFlagIgnoreDiskIOAlways  = 0x08, // always ignore disk idle
    kDarkWakeFlagIgnoreDiskIOMask    = 0x0C
```

According to this boot flags darkwake=8 and darkwake=10 are obsolete if you have Yosemite or newer macOS as related flags are removed from XNU.

What is the default value for darkware boot arg?

According to [XNU source code](https://github.com/apple/darwin-xnu/blob/master/iokit/Kernel/IOPMrootDomain.cpp#L337) the default value of boot arg darkware is 3 (darkwake=3):

```c
static uint32_t         gDarkWakeFlags = kDarkWakeFlagHIDTickleNone;
```

We have to clarify that xnu 4903.221.2 is used since macOS Mojave 10.14.1 and xnu 4903.241.1 is used since macOS Mojave 10.14.3. How about the latest macOS? Sadly there is no source code available.

To figure out which value is defined on latest kernel we have to download the latest available [Kernel Debug Kit](https://developer.apple.com/download/more/?q=Kernel Debug Kit), which is 10.15.1 build 19B77a. By using Hopper Disassembler we see that default value on macOS Catalina 10.15.1 for gDarkWakeFlags is 0x00000003, which equals to 3.

```
                     __ZL14gDarkWakeFlags:        // gDarkWakeFlags
ffffff80012b93b0         dd         0x00000003
```

So by default Darkwake should not post any HID Tickle's. This also reveals the secret why some users encounter issues with frozen peripheral device's on Hack's when Power Nap is enabled. To use Darkwake on Hack's require very well configured USB ports.

- [ ] - - -

Power Nap & Darkwake

If you have Power Nap disabled then computer shouldn't wake automatically. On most cases darkwake boot arg affects how computers should behave on case of Power Nap enabled. If everything is configured properly you do not need define darkwake boot flag at all. Anyhow, there might be motherboards, which benefit from user defined value. But keep in mind that darkwake=8 and darkwake=10 are obsolete since Yosemite.

- [ ] - - -

Which values are valid for darkwake?

As flags are used for bitwise operations, then these values are valid:

darkwake=0
darkwake=1 (darkWakePostTickle behaviour)
darkwake=2 (darkWakePostTickle behaviour)
darkwake=3 (darkWakePostTickle behaviour)
darkwake=256

darkwake=257
darkwake=258
darkwake=259

_.. and so on..._

As I'm not familiar how [PE_parse_boot_argn](https://developer.apple.com/documentation/kernel/1553680-pe_parse_boot_argn?language=objc) function exactly works, I cannot say much about boot arg darkwake=0. According to source code there is no any checks or behaviours defined for darkwake=0. There is a huge chance that using darkwake=0 actually equals to darkwake=3. I hope someone can clarify from source code what exactly happens if darkwake=0 is used trough PE_parse_boot_argn, but it's obvious that darkwake=0 does not equal to darkwake=NO (or darkwake=FALSE). darkwake=0 does not disable power nap, it only affects HID tickle. Please note that darkwake=3 is combination of flags 1 & 2. 1 + 2 = 3. On case we say to the system to do early (1) and later (2) HID tickle both (3), there is no any tickle at all.

Some code samples from IOPMrootDomain:

```c
            else if (!darkWakeMaintenance)
            {
                // Early/late tickle for non-maintenance wake.
                if (((gDarkWakeFlags & kDarkWakeFlagHIDTickleMask) ==
                     kDarkWakeFlagHIDTickleEarly) ||
                    ((gDarkWakeFlags & kDarkWakeFlagHIDTickleMask) ==
                     kDarkWakeFlagHIDTickleLate))
                {
                    darkWakePostTickle = true;
                }
            }
        if (darkWakePostTickle &&
            (kSystemTransitionWake == _systemTransitionType) &&
            (gDarkWakeFlags & kDarkWakeFlagHIDTickleMask) ==
             kDarkWakeFlagHIDTickleLate)
        {
            darkWakePostTickle = false;
            reportUserInput();
        }
        if (powerState > maxPowerState)
        {
            DLOG("> plimit %s %p (%u->%u, 0x%x)\n",
                service->getName(), OBFUSCATE(service), powerState, maxPowerState,
                changeFlags);
            *inOutPowerState = maxPowerState;

            if (darkWakePostTickle &&
                (actions->parameter & kPMActionsFlagIsDisplayWrangler) &&
                (changeFlags & kIOPMDomainWillChange) &&
                ((gDarkWakeFlags & kDarkWakeFlagHIDTickleMask) ==
                 kDarkWakeFlagHIDTickleEarly))
            {
                darkWakePostTickle = false;
                reportUserInput();
            }
        }
void IOPMrootDomain::reportUserInput( void )
{
#if !NO_KERNEL_HID
    OSIterator * iter;
    OSDictionary * matching;

    if(!wrangler)
    {
        matching = serviceMatching("IODisplayWrangler");
        iter = getMatchingServices(matching);
        if (matching) matching->release();
        if(iter)
        {
            wrangler = OSDynamicCast(IOService, iter->getNextObject());
            iter->release();
        }
    }

    if(wrangler)
        wrangler->activityTickle(0,0);
#endif
}
```

As you see from code examples above, there is no any huge mystery about darkwake boot arg and you should use it mostly on case when you really need to manipulate HID tickle behaviour. On most cases it's more important to properly configure you system power management rather than paying with darkwake boot arg, which can be done via terminal command pmset.

- [ ] - - -

Power Management

To check Power Management settings use terminal command:

```
pmset -g
```

Also you can use Hackintool to check power management settings:


Power Management Settings explained:

* displaysleep - display sleep timer; replaces ’dim’ argument in 10.4 (value in minutes, or 0 to disable)
* disksleep - disk spindown timer; replaces ’spindown’ argument in 10.4 (value in minutes, or 0 to disable)
* sleep - system sleep timer (value in minutes, or 0 to disable)
* womp - wake on ethernet magic packet (value = 0/1). Same as "Wake for network access" in the Energy Saver preferences.
* ring - wake on modem ring (value = 0/1)
* powernap - enable/disable Power Nap on supported machines (value = 0/1)
* proximitywake - On supported systems, this option controls system wake from sleep based on proximity of devices using same iCloud id. (value = 0/1)
* autorestart - automatic restart on power loss (value = 0/1)
* lidwake - wake the machine when the laptop lid (or clamshell) is opened (value = 0/1)
* acwake - wake the machine when power source (AC/battery) is changed (value = 0/1)
* lessbright - slightly turn down display brightness when switching to this power source (value = 0/1)
* halfdim - display sleep will use an intermediate half-brightness state between full brightness and fully off (value = 0/1)
* sms - use Sudden Motion Sensor to park disk heads on sudden changes in G force (value = 0/1)
* hibernatemode - change hibernation mode. Please use caution. (value = integer)
* hibernatefile - change hibernation image file location. Image may only be located on the root volume. Please use caution. (value = path)
* ttyskeepawake - prevent idle system sleep when any tty (e.g. remote login session) is ’active’. A tty is ’inactive’ only when its idle time exceeds the system sleep timer. (value = 0/1)
* networkoversleep - this setting affects how OS X networking presents shared network services during system sleep. This setting is not used by all platforms; changing its value is unsupported.
* destroyfvkeyonstandby - Destroy File Vault Key when going to standby mode. By default File vault keys are retained even when system goes to standby. If the keys are destroyed, user will be prompted to enter the password while coming out of standby mode.(value: 1 - Destroy, 0 - Retain)

If you want to disable proximitywake the this command should be used:

```
sudo pmset -a proximitywake 0
```

Recommended settings for Hack's are:

```
sudo pmset -a hibernatemode 0
sudo pmset -a proximitywake 0
sudo pmset -a powernap 0
```

Settings above disable Hibernate, Bluetooth wake by iDevices and Power Nap.

To check what might prevent computer from going into sleep we can [pmset](https://en.wikipedia.org/wiki/Pmset) tool:

```
pmset -g assertions
```

So, before using blindly darkwake boot arg to solve some sleep issues, make instead sure that USB ports and Power Management settings of your Hack are configured properly.

- [ ] - - -

Sleep & Wake

Quite often Hack's users have sleep/wake issues because they don't pay attention to the fact that Apple's macOS is developed for Apple hardware in mind, not regular PC's.

Of course sleep mode isn't something that Apple has invented first. Since December 1996 ACPI superseded APM. ACPI - Advanced Configuration and Power Interface. APM - Advanced Power Management. The ACPI specification defines several states for various hardware components and devices. There are [global "Gx" states and sleep "Sx" states specified](https://en.wikipedia.org/wiki/Advanced_Configuration_and_Power_Interface#OSPM_responsibilities).

|  Gx  |                             Name                             |  Sx  |                         Description                          |
| :--: | :----------------------------------------------------------: | :--: | :----------------------------------------------------------: |
|  G0  |                           Working                            |  S0  | The computer is running and the CPU executes instructions. "Awaymode" is a subset of S0, where monitor is off but background tasks are running |
|  G1  |                           Sleeping                           |  S1  | _Power on Suspend (POS):_ Processor caches are flushed, and the CPU(s) stops executing instructions. The power to the CPU(s) and RAM is maintained. Devices that do not indicate they must remain on may be powered off |
|  S2  |        CPU powered off. Dirty cache is flushed to RAM        |      |                                                              |
|  S3  | commonly referred to as _Standby_, Sleep, or _Suspend to RAM (STR)_: RAM remains powered |      |                                                              |
|  S4  | Hibernation or _Suspend to Disk:_ All content of the main memory is saved to non-volatile memory such as a hard drive, and the system is powered down |      |                                                              |
|  G2  |                           Soft Off                           |  S5  | G2/S5 is almost the same as G3 _Mechanical Off_, except that the power supply unit (PSU) still supplies power, at a minimum, to the power button to allow return to S0. A full reboot is required. No previous content is retained. Other components may remain powered so the computer can "wake" on input from the keyboard, clock, modem, LAN, or USB device |
|  G3  |                        Mechanical Off                        |      | The computer's power has been totally removed via a mechanical switch (as on the rear of a PSU). The power cord can be removed and the system is safe for disassembly (typically, only the real-time clock continues to run using its own small battery) |

Since Mac OS X Lion Apple is using DarkWake, which were wrapped into [Power Nap](https://support.apple.com/en-us/HT204032) on OS X Mountain Lion. To understand and fix macOS sleep issues we also need the knowledge about sleep states which macOS comps may have. To check log of macOS computers sleep_wake we can use __pmset__ tool. Following terminal code prints sleep_wake history.

```
pmset -g log|grep -e " Sleep  " -e " Wake  "
```

I to check more closely sleep history then we can recognise that macOS has at least 3 different sleep "modes":

* Software sleep
* Idle sleep
* Maintenance sleep

Software sleep

Software sleep is trigged by computer user or by some software, which is configured to put comp sleep after certain tasks done.

Idle sleep

Idle sleep is triggered by idle timer. Each time user interacts with computer idle timer is reset. If users doesn't interact within idle timer countdown time, then Idle sleep is triggered.

Maintenance sleep

If user has enabled "Wake for Ethernet network access", then macOS goes from Idle or Software sleep into Maintenance sleep immediately.

Power Nap

If Power Nap is enabled, then computer wakes up automatically after certain period of time, handles certain tasks and goes back to sleep. Apple's documentation reveals that behaviour of power nap doesn't depend only macOS version running on comp but at hardware and it's firmware too. Documentation clearly states that comps made on different time behave differently. which points directly that Power Nap is related to the hardware/firmware too.

> [ ](https://www.insanelymac.com/forum/topic/342002-darkwake-on-macos-catalina-boot-args-darkwake8-darkwake10-are-obsolete/#) QuotePower Nap responds to your battery power state    
> The year your notebook computer was released determines how Power Nap responds to your battery power state.    
> Computers with 2013 or a later year in the model name use Power Nap until the battery is drained. Computers with 2012 or an earlier year in the model name suspend Power Nap if the battery has a charge of 30% or less. Power Nap resumes when you connect to AC power.    
> To increase battery life while using Power Nap, disconnect any USB, Thunderbolt, or FireWire devices that may draw power from the computer. Learn more about maximizing battery life.Power Nap checks for updates at specific intervals    
> When your computer isn't connected to AC power, Power Nap communicates and transfers data for only a few minutes per Power Nap cycle. When connected to AC power, communications and data transfers are continuous.Expand    
According quote above we have to very carefully choose which firmware is emulated on Hack. Changing SMBIOS on Clover/Opencore can help to solve sleep issues or cause them.

An example of sleep log when powernap is enabled:

```
2020-01-05 09:03:23 +0200 Sleep               	Entering Sleep state due to 'Idle Sleep': Using AC (Charge:0%) 21 secs   
2020-01-05 09:04:29 +0200 Sleep               	Entering Sleep state due to 'Maintenance Sleep': Using AC (Charge:0%) 1945 secs 
2020-01-05 09:37:39 +0200 Sleep               	Entering Sleep state due to 'Maintenance Sleep': Using AC (Charge:0%) 3187 secs 
2020-01-05 10:31:33 +0200 Sleep               	Entering Sleep state due to 'Maintenance Sleep': Using AC (Charge:0%) 12467 secs
2020-01-05 14:00:06 +0200 Sleep               	Entering Sleep state due to 'Maintenance Sleep': Using AC (Charge:0%) 11312 secs
2020-01-05 17:08:40 +0200 Wake                	DarkWake to FullWake from Normal Sleep [CDNVA] : due to UserActivity Assertion Using AC (Charge:0%)    
```

An example of sleep log, when ethernet wake and power nap is disabled:

```
2020-01-03 18:15:43 +0200 Sleep               	Entering Sleep state due to 'Idle Sleep':TCPKeepAlive=inactive Using AC (Charge:0%) 3210 secs 
2020-01-03 19:09:13 +0200 Wake                	Wake from Normal Sleep [CDNVA] : due to XDCI CNVW/HID Activity Using AC (Charge:0%) 207 secs  
2020-01-03 19:12:40 +0200 Sleep               	Entering Sleep state due to 'Idle Sleep':TCPKeepAlive=inactive Using AC (Charge:0%) 165903 secs
2020-01-05 17:17:43 +0200 Wake                	Wake from Normal Sleep [CDNVA] : due to XDCI CNVW/HID Activity Using AC (Charge:0%)  
```

As we see from log above, computer stays in sleep without any wakes and no 'Maintenance Sleep'.

So there are several variables which have effect on your computer sleep/wake behaviour and the solution that helped another people, might be useless or even worse on you case.

...

Edited January 5, 2020 by holyfield

### 15.3 darkwake 进一步解释
Before we get deep into configuring darkwake, it would be best to explain what darkwake is. A great in-depth thread by holyfield can be found here: [DarkWake on macOS Catalina(opens new window)](https://www.insanelymac.com/forum/topic/342002-darkwake-on-macos-catalina-boot-args-darkwake8-darkwake10-are-obsolete/)

In its simplest form, you can think of darkwake as "partial wake", where only certain parts of your hardware are lit up for maintenance tasks while others remain asleep(ie. Display). Reason we may care about this is that darkwake can add extra steps to the wake process like keyboard press, but outright disabling it can make our hack wake randomly. So ideally we'd go through the below table to find an ideal value.

Now lets take a look at [IOPMrootDomain's source code (opens new window)](https://opensource.apple.com/source/xnu/xnu-6153.81.5/iokit/Kernel/IOPMrootDomain.cpp.auto.html):

```cpp
// gDarkWakeFlags
enum {
    kDarkWakeFlagHIDTickleEarly      = 0x01, // hid tickle before gfx suppression
    kDarkWakeFlagHIDTickleLate       = 0x02, // hid tickle after gfx suppression
    kDarkWakeFlagHIDTickleNone       = 0x03, // hid tickle is not posted
    kDarkWakeFlagHIDTickleMask       = 0x03,
    kDarkWakeFlagAlarmIsDark         = 0x0100,
    kDarkWakeFlagGraphicsPowerState1 = 0x0200,
    kDarkWakeFlagAudioNotSuppressed  = 0x0400
};
```

Now lets go through each bit:

| Bit  | Name                   | Comment                                                      |
| :--- | :--------------------- | :----------------------------------------------------------- |
| 0    | N/A                    | Supposedly disables darkwake                                 |
| 1    | HID Tickle Early       | Helps with wake from lid, may require pwr-button press to wake in addition |
| 2    | HID Tickle Late        | Helps single keypress wake but disables auto-sleep           |
| 3    | HID Tickle None        | Default darkwake value if none is set                        |
| 3    | HID Tickle Mask        | To be paired with other                                      |
| 256  | Alarm Is Dark          | To be explored                                               |
| 512  | Graphics Power State 1 | Enables wranglerTickled to wake fully from hibernation and RTC |
| 1024 | Audio Not Suppressed   | Supposedly helps with audio disappearing after wake          |

* Note that HID = Human-interface devices(Keyboards, mice, pointing devices, etc)

To apply the above table to your system, it's as simple as grabbing calculator, adding up your desired darkwake values and then applying the final value to your boot-args. However we recommend trying 1 at a time rather than merging all at once, unless you know what you're doing(though you likely wouldn't be reading this guide).

For this example, lets try and combine `kDarkWakeFlagHIDTickleLate` and `kDarkWakeFlagGraphicsPowerState1`:

* `2`= kDarkWakeFlagHIDTickleLate
* `512`= kDarkWakeFlagAudioNotSuppressed

So our final value would be `darkwake=514`, which we can next place into boot-args:

```text
NVRAM
|---Add
  |---7C436110-AB2A-4BBB-A880-FE41995C9F82
    |---boot-args | Sting | darkwake=514
```

The below is more for clarification for users who are already using darkwake or are looking into it, specifically clarifying what values no longer work:

* ```
darkwake=8
```

: This hasn't been in the kernel since Mavericks

* Correct boot-arg would be `darkwake=0`

* ```
darkwake=10
```

: This hasn't been in the kernel sinceMavericks

* Correct boot-arg would be `darkwake=2`

## 16. 修补 DRM

* Note: Safari 14 and macOS 11, Big Sur are currently unsupported by WhateverGreen's DRM patches. Safari 13 in Catalina and older are supported just fine however.
* Note 2: Browsers not using hardware based DRM (ie. Mozilla Firefox or Chromium-based browsers like Google Chrome and Microsoft Edge) will have working DRM without any work both on iGPUs and dGPUs. The below guide is for Safari and other applications using hardware-based DRM.

So with DRM, we have a couple things we need to mention:

* DRM requires a supported dGPU
	* See the [GPU Buyers Guide (opens new window)](https://dortania.github.io/GPU-Buyers-Guide/)for supported cards
* DRM is broken for iGPU-only systems
	* This could be fixed with Shiki (now WhateverGreen) til 10.12.2, but broke with 10.12.3
	* This is due to the issue that our iGPUs don't support Apple's firmware and that our [Management Engine (opens new window)](https://en.wikipedia.org/wiki/Intel_Management_Engine)doesn't have Apple's certificate
* Working hardware acceleration and decoding

### 16.1 Hardware Acceleration and Decoding
So before we can get started with fixing DRM, we need to make sure your hardware is working. The best way to check is by running [VDADecoderChecker (opens new window)](https://i.applelife.ru/2019/05/451893_10.12_VDADecoderChecker.zip):

If you fail at this point, there's a couple things you can check for:

* Make sure your hardware is supported

* See [GPU Buyers Guide(opens new window)](https://dortania.github.io/GPU-Buyers-Guide/)

* Make sure the SMBIOS you're running matches with your hardware

* Don't use a Mac Mini SMBIOS on a desktop for example, as Mac Minis run mobile hardware and so macOS will expect the same

* Make sure the iGPU is enabled in the BIOS and has the correct properties for your setup (

```
AAPL,ig-platform-id
```

and if needed,

```
device-id
```

)

* You can either review the DeviceProperties section from the guide or [WhateverGreen's manual(opens new window)](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md)

* Avoid unnecessary ACPI renames, all important ones are handled in WhateverGreen

* change GFX0 to IGPU
* change PEG0 to GFX0
* change HECI to IMEI
* [etc(opens new window)](https://github.com/dortania/OpenCore-Install-Guide/blob/master/clover-conversion/Clover-config.md)

* Make sure Lilu and WhateverGreen are loaded

* Make sure not to have any legacy graphics patches present as they've been absorbed into WhateverGreen:
	* IntelGraphicsFixup.kext
	* NvidiaGraphicsFixup.kext
	* Shiki.kext

To check if Lilu and WhateverGreen loaded correctly:

```text
kextstat | grep -E "Lilu|WhateverGreen"
```

> Hey one or more of these kexts aren't showing up    
Generally the best place to start is by looking through your OpenCore logs and seeing if Lilu and WhateverGreen injected correctly:

```text
14:354 00:020 OC: Prelink injection Lilu.kext () - Success
14:367 00:012 OC: Prelink injection WhateverGreen.kext () - Success
```

If it says failed to inject:

```text
15:448 00:007 OC: Prelink injection WhateverGreen.kext () - Invalid Parameter
```

Main places you can check as to why:

* Injection order: Make sure that Lilu is above WhateverGreen in kext order
* All kexts are latest release: Especially important for Lilu plugins, as mismatched kexts can cause issues

Note: To setup file logging, see [OpenCore Debugging (opens new window)](https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/debug.html).

Note: On macOS 10.15 and newer, AppleGVA debugging is disabled by default, if you get a generic error while running VDADecoderChecker you can enable debugging with the following:

```text
defaults write com.apple.AppleGVA enableSyslog -boolean true
```

And to undo this once done:

```text
defaults delete com.apple.AppleGVA enableSyslog
```

### 16.2 Testing DRM
So before we get too deep, we need to go over some things, mainly the types of DRM you'll see out in the wild:

FairPlay 1.x: Software based DRM, used for supporting legacy Macs more easily

* Easiest way to test this is by playing an iTunes movie:

FairPlay 1.x test(opens new window)

* FairPlay 1.x trailers will work on any configuration if WhateverGreen is properly set up - including iGPU-only configurations. However, FairPlay 1.x _movies_ will only play on iGPU-only configurations for around 3-5 seconds, erroring that HDCP is unsupported afterwards.

FairPlay 2.x/3.x: Hardware based DRM, found in Netflix, Amazon Prime

* There's a couple ways to test:

* Play a show in Netflix or Amazon Prime

* Play an Amazon Prime trailer:

```
Spider-Man: Far From Home(opens new window)
```

```
- Trailer itself does not use DRM but Amazon still checks before playing
```

* Note: Requires newer AMD GPU to work (Polaris+)

FairPlay 4.x: Mixed DRM, found on AppleTV+

* You can open TV.app, choose TV+ -> Free Apple TV+ Premieres, then click on any episode to test without any trial (you do need an iCloud account)
* Apple TV+ also has a free trial if you want to use it
* Note: Requires either an absent iGPU (Xeon) or newer AMD GPU to work (Polaris+)
	* Possible to force FairPlay 1.x when iGPU is absent

If everything works on these tests, you have no need to continue! Otherwise, proceed on.

### 16.3 Fixing DRM
So for fixing DRM we can go down mainly 1 route: patching DRM to use either software or AMD decoding. Vit made a great little chart for different hardware configurations:

* [WhateverGreen's DRM chart(opens new window)](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.Chart.md)

So how do you use it? First, identify what configuration you have in the chart (AMD represents GPU, not CPU). The SMBIOS listed (IM = iMac, MM = Mac Mini, IMP = iMac Pro, MP = Mac Pro) is what you should use if you match the hardware configuration. If you don't match any of the configurations in the chart, you're out of luck.

Next, identify what Shiki mode you need to use. If there are two configurations for your setup, they will differ in the Shiki flags used. Generally, you want hardware decoding over software decoding. If the mode column is blank, then you are done. Otherwise, you should add `shikigva` as a property to any GPU, using DeviceProperties > Add. For example, if the mode we need to use is `shikigva=80`:

Example of shikigva in Devices Properties

You can also use the boot argument - this is in the mode column.

Here's one example. If we have an Intel i9-9900K and an RX 560, the configuration would be "AMD+IGPU", and we should be using an iMac or Mac Mini SMBIOS (for this specific configuration, iMac19,1). Then we see there are two options for the configuration: one where the mode is `shikigva=16`, and one with `shikigva=80`. We see the difference is in "Prime Trailers" and "Prime/Netflix". We want Netflix to work, so we'll choose the `shikigva=80` option. Then inject `shikigva` with type number/integer and value `80` into our iGPU or dGPU, reboot, and DRM should work.

Here's another example. This time, We have an Ryzen 3700X and an RX 480. Our configuration in this case is just "AMD", and we should be using either an iMac Pro or Mac Pro SMBIOS. Again, there are two options: no shiki arguments, and `shikigva=128`. We prefer hardware decoding over software decoding, so we'll choose the `shikigva=128` option, and again inject `shikigva` into our dGPU, this time with value `128`. A reboot and DRM works.

Notes:

* You can use [gfxutil (opens new window)](https://github.com/acidanthera/gfxutil/releases)to find the path to your iGPU/dGPU.

* `path/to/gfxutil -f GFX0`
* `GFX0`: For dGPUs, if multiple installed check IORegistryExplorer for what your AMD card is called
* `IGPU`: For iGPU

* If you inject `shikigva` using DeviceProperties, ensure you only do so to one GPU, otherwise WhateverGreen will use whatever it finds first and it is not guaranteed to be consistent.

* IQSV stands for Intel Quick Sync Video: this only works if iGPU is present and enabled and it is set up correctly.

* Special configurations (like Haswell + AMD dGPU with an iMac SMBIOS, but iGPU is disabled) are not covered in the chart. You must do research on this yourself.

* [Shiki source (opens new window)](https://github.com/acidanthera/WhateverGreen/blob/master/WhateverGreen/kern_shiki.hpp)is useful in understanding what flags do what and when they should be used, and may help with special configurations.

* For error `VDADecoderCreate failed. err: -12473` in Big Sur, forcing the AMD Decoder(on applicable systems) can help resolve this:

```sh
defaults write com.apple.AppleGVA gvaForceAMDAVCDecode -boolean yes
```

Last Updated

## 六、前置SD卡能不能驱动？
等了许久，前置SD卡终于是有驱动了，RealtekCardReader，但是驱动问题较大，每次睡眠就会意外弹出，并且容易崩溃，暂且不使用，观望中……

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
