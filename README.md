# hackintosh-Inspiron3670-franksfc
hackintosh-Inspiron3670 by franksfc

机型配置：

| Inspiron 3670 |                                                 |
| :------------ | ----------------------------------------------- |
| 主板          | B360                                            |
| cpu           | i5-9400                                         |
| 显卡          | UHD630(1.05GHZ)                                  |
| 网卡/蓝牙     | AR9565 / 奋威 T919（BCM94360CD）                |
| 声卡          | Realtek ALC867 （layout id: 11)                 |
| 内存          | 8GB                                             |
| 硬盘          | KBG40ZNS256G NVMe KIOXIA 256GB+ ST1000DM010 1TB |
| 显示器        | DELL SE2417HG                                   |

已实现功能
- [x] 网卡蓝牙正常识别使用，支持airdrop 接力
- [x] VDA解码、核显满速
- [x] 声卡驱动，无爆破音
- [x] 正常熄灯睡眠，无噪音
- [x] 定制USB，可以快速充电
- [x] 识别核显接口，支持hdmi输出，利用igfxonln=1解决唤醒后显示屏不亮的问题
- [x] cpu变频，优化cpu性能
- [x] 修补HPET、RTC、TMR以进入系统，睡眠时间正常
- [x] 注入SBUS。摁下关机键跳出提示框
- [x] 加载原生电源管理，开启节能五项
- [x] 原生NVRAM
- [x] 温度传感器正常

未实现功能：
前置SD卡驱动

**注意：因为本机没有USB供电接口，所以T919蓝牙无法使用，配合原来的蓝牙AR9565就可以啦，亦可以实现接力和隔空投送**
**引导版本：Opencore 0.6.5**  
**系统版本：Big Sur beta 11**  
**2021/1/13日更新** 

thannks:
1，https://www.insanelymac.com/forum/topic/345526-opencore-macos-bigsur-on-dell-inspiron-3670/
2，https://www.longzc.cn/index.php/archives/308
