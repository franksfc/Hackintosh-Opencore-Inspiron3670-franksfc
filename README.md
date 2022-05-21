# hackintosh-Inspiron3670-franksfc
hackintosh-Inspiron3670 by franksfc

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
- [x] 声卡驱动，修补HPET、RTC、TMR
- [x] 正常闪灯睡眠
- [x] 定制USB，可以快速充电
- [x] 识别核显接口，支持hdmi输出，利用igfxonln=1解决唤醒后显示屏不亮的问题
- [x] 注入SBUS。摁下关机键跳出提示框
- [x] 加载原生电源管理，开启节能五项
- [x] 加载原生NVRAM
- [x] 温度传感器正常
- [x] 解锁CFGlock，命令：setup_var 0x5BE 0x00（仅使用本机bios）

## 未实现功能：  
VGA接口
hibernation
Darkwake

## 待完善功能：
前置SD读卡器

## 我的折腾成果：  
   
# 1关于Trim     
   这个BG4的固态硬盘在macOS的支持并不好，开机时间十分漫长，需要将SetApfsTrimTimeout设置为0，这时候macOS就会禁止TRIM，即使系统信息中TRIM support依然是支持       
  
# 2关于hidpi     
   hidpi一直都是我头大的问题，解决方法如下：   
   1，首先，开启隐藏的bios选项，开启DVMT64MB     
   2，删除framebuffer-stolenmem，这步十分关键，这个补丁会适得其反    
   3，framebuffer-unifiedmem 设为000000C0 即显存变为3072MB    
   4，用one-key-hidpi，开启hidpi      
  
# 3关于核显接口修补      
   本机型上的UHD630只有VGA和HDMI两个接口。    
   先说HDMI接口，在我的尝试后，con-0或者con-2中设置connector-type为00080000，busid为01，都可以驱动，这个很奇怪，     
   VGA接口，应该是DP转化的，有的品牌可以点亮，但是dell的机器是全网无解的，我也尝试了很久都不能成功      
  
# 4关于睡眠    
   这个机器原来睡眠，经常有睡死的状况，我也不清楚为什么，这本来就是一个黑苹果玄学问题      
   后来我在ACPI中加了DMAC，MEM2，OCWorK-dell，PMCR，PNLF，PPMC，SBUS-MCHC等一系列驱动后，我感觉好像没有问题了，这个还有待观察商榷     
    
# 5关于独立显卡GT730     
   这个显卡，性能不太高，但是还是比核显强了不少，最好加一张RX560比较省事。GT730的显卡在macOS中处于半残状态，不支持Metal2功能集。在最新的Monterey中根本就不支持，虽然可以打补丁把Big Sur中的驱动也打上去。我将太-wegnoegpu禁用了      
   
# 6关于提高核显性能      
  九代核显可以加如下启动参数，增加性能igfxonln=1 -igfxrpsc=1 igfxfw=2     
   
# 7关于T1919网卡     
   因为这个主版是定制的，没有九针USB接口，那就买一个九针转USB的转接头，或者用自带的蓝牙也可以配合使用，不影响隔空投送等功能      
   
# 8关于前置SD卡     
   等了许久，前置SD卡终于是有驱动了，RealtekCardReader，但是驱动问题较大，每次睡眠就会意外弹出，并且容易崩溃，暂且不使用，观望      
   
# 9关于机型选择     
   可以选择的机型有iMac与Macmini，iMac pro和Mac Pro没办法驱动核显，我一开始用的是iMac19,1但不知怎么，macOS的性能释放并没有Windows好。我在尝试机型的时候，发现iMac18,1是核显而且跑分及其高，我选择了它。缺失的功能可以用FeatureUnlock来解决    
    
    
## 如何解锁bios隐藏设置   
1，在EFI文件夹中放入给的EFI-bios文件，重命名为EFI。    
2，输入setup_var 05BE 0x0(关闭CFG lock）   
3，输入setup_var 0x8DC 0x2（DVMT 64MB）  
4，输入setup_var 0x8DD 0x3（DVMT Total MAX）  
5，输入reboot重启。
   
![image](https://github.com/franksfc/hackintosh-Inspiron3670-franksfc/blob/master/CFG%20Lock.jpg)
     
![image](https://github.com/franksfc/hackintosh-Inspiron3670-franksfc/blob/master/DVMT.jpg)
   
*thanks:*  
https://www.insanelymac.com/forum/topic/345526-opencore-macos-bigsur-on-dell-inspiron-3670/   

