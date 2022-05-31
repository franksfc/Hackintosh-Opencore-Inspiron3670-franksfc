# hackintosh-Inspiron3670-franksfc

![image](https://github.com/franksfc/hackintosh-Inspiron3670-franksfc/blob/master/preview.png)

## 机型配置：

| ***Inspiron 3670*** |                                                 |
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
   
## 未实现功能：  
- [ ] VGA接口   
   
## 不稳定功能：
- [ ] 前置SD读卡器  


# Bios设置：
- [x] 关闭Secure Boot
- [x] 关闭Fast Boot
- [x] **SATAmode调整为AHCI**
- [x] 关闭PTT technology
- [x] 关闭wake on USB（实际上不影响USB唤醒）
- [x] 关闭wake on LAN
- [x] 考虑关闭WIFI和Bluetooth，因为系统里不好驱动
   
# 我是如何配置opencore引导文件的
我折腾的过程比较漫长，从下载黑果小兵的镜像用免驱独显，到拔掉独显定制opencore核显，最后不断完善具体的功能，直到黑苹果接近完美，我一步一步地走来，给大家分享一下我的折腾成果，以及我的理由
  
## open core引导配置

 

## 2 显卡  
在充分折腾之后，显卡部分已经基本可以说是完美的了:)
  
### 2.1 关于独立显卡GT730     
这个显卡，性能不太高，但是还是比核显强了不少，最好加一张RX560比较省事。GT730的显卡在macOS中处于半残状态，不支持Metal2功能集。在最新的Monterey中根本就不支持，虽然可以打补丁把Big Sur中的驱动也打上去。我将太-wegnoegpu禁用了  
      
### 2.2 核显插入黑屏如何解决？      
本机型上的UHD630只有VGA和HDMI两个接口。    
先说HDMI接口，在我的尝试后，con-0或者con-2中设置connector-type为00080000，busid为01，都可以驱动，这个很奇怪，     
VGA接口，应该是DP转化的，有的品牌可以点亮，但是dell的机器是全网无解的，我也尝试了很久都不能成功。
最后，由于weg有缺陷，hdmi有几率启动出现黑屏，很不幸，我的机器上有这个毛病，需要添加igfxonln=1就完全解决了。
   
### 2.3 核显性能孱弱，如何进一步提高？      
九代核显UHD630可以加如下启动参数，增加性能-igfxrpsc=1 igfxfw=2 (igfxmetal=1)
   
### 2.4 1080p显示器过于模糊，如何开启hidpi     
hidpi一直都是我头大的问题，一直有1600x900分辨率无法开启的问题。在反复探索后解决办法如下：   
1，首先，开启隐藏的bios选项，开启DVMT64MB     
2，删除framebuffer-stolenmem，这步十分关键，这个补丁会适得其反    
3，framebuffer-unifiedmem 设为000000C0 即显存变为3072MB    
4，用one-key-hidpi，开启hidpi           
     
## 3.如何定制USB

## 4 如何睡眠？
要注意，睡眠的问题主要就是两个，第一个CFG lock，另一个就是USB。所以定制USB十分重要，不要使用USB hub，删掉不用的接口，将蓝牙内建，基本上就能解决了。     
需要注意的是，苹果电脑的休眠和电能小憩功能是与硬件高度绑定的，普通PC不支持这几个功能，我依然在探索当中。但是，目前来看本机器似乎支持手动休眠与电能小憩，没有出现较大的问题。 

### 4.1 如何解锁bios隐藏设置   
1，在EFI文件夹中放入给的EFI-bios文件，重命名为EFI。    
2，输入setup_var 05BE 0x0(关闭CFG lock）   
3，输入setup_var 0x8DC 0x2（DVMT 64MB）  
4，输入setup_var 0x8DD 0x3（DVMT Total MAX）  
5，输入reboot重启。

### 4.2 为何会出现Wake failure in EFI？  
这个问题非常复杂，是属于疑难杂症的一类。可能是CPU的频率问题，可能是RTC的问题，也有很大概率是USB或者显卡的问题。这几种我都见到过。    
我这个机器需要调整bios的设置，才能完美睡眠，否则就会睡死。具体的设置见上文。  

## 三、机型众多，该如何选择？     
可以选择的机型有iMac与Macmini，因为iMac pro和Mac Pro没办法驱动核显。虽然iMac机型的性能释放更好，但是它对电能小憩以及休眠极其不友好，所以我换成了Mac mini的机型，只有这样才能完美地睡眠。  
  
## 四、SSD与Trim      
这个BG4的固态硬盘在macOS的支持并不好，开机时间十分漫长，需要将SetApfsTrimTimeout设置为0，这时候macOS就会禁止TRIM，即使系统信息中TRIM support依然是支持。   
反转了，在关掉Trim之后，会出现”宗卷哈希值不匹配“，甚至出现冻屏、卡死的现象，再三衡量一下，还是吧Trim重新开启，SetApfsTrimTimeout设置为-1
    
## 六、前置SD卡能不能驱动？     
等了许久，前置SD卡终于是有驱动了，RealtekCardReader，但是驱动问题较大，每次睡眠就会意外弹出，并且容易崩溃，暂且不使用，观望         
   
![image](https://github.com/franksfc/hackintosh-Inspiron3670-franksfc/blob/master/CFG%20Lock.jpg)
     
![image](https://github.com/franksfc/hackintosh-Inspiron3670-franksfc/blob/master/DVMT.jpg)
   
**Credits**  
https://www.insanelymac.com/forum/topic/345526-opencore-macos-bigsur-on-dell-inspiron-3670/   

