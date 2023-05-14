# hackintosh-Inspiron3670-franksfc

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

- [ ] VGA接口

## 不稳定功能

- [ ] 前置SD读卡器
- [ ] 开机audiodxe爆音（https://github.com/acidanthera/bugtracker/issues/2073）


## Bios 设置

* 关闭Secure Boot

* 关闭Fast Boot

* SATAmode调整为AHCI

* 关闭PTT technology

* 考虑关闭WIFI和Bluetooth，因为系统里不好驱动

## 如何解锁Bios的CFGlock和DVMT

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

### DELL Bios 提取工具

https://github.com/vuquangtrong/Dell-PFS-BIOS-Assembler

然后将官网下载的BIOS程序拖到Dell_PFS_Extract上，就可以将里面的BIOS提取出来。


### 提取DSDT

尽管提取原始DSDT的方法非常多，我认为 `CLOVER` 的提取方法是最方便并且靠谱的。我们需要一个空的U盘或者空的ESP分区，我的教程是非常偏向小白的，所以这里提取我也会用到windows，以及Diskgenius这个软件，做最简单的示范。

* 进入Windows，插入U盘，打开[DiskGenius](https://www.diskgenius.cn/download.php)，选中我们的U盘，并选择顶部菜单栏的快速分区
  * 分区表类型：`GUID`
  * 不要创建新的 `ESP` 分区
  * 不要创建新的 `MSR` 分区
  * 分区格式为 `FAT32`
* 格式化完成后，放入我从黑果小兵镜像包提取出来的[EFI](https://blog.xjn819.com/tools/EFI.zip)放进去。这是一个 clover 引导，但并不能引导你的系统，只能提取 DSDT。
* 插上U盘，重启，通过U盘引导，看到Clover界面，我们按F4，这样原始的DSDT文件就收集好了。
* 重新通过OC引导进入系统，我们打开U盘，EFI_Clover_ACPI/Orgin下，有我们的原始ACPI内容，我们只需要DSDT.aml这个就行了，保存到安全的地方。


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
