
太多这种问题了每次都要我手动敲不好

```
logcat -c 
然后关闭再打开蓝牙
ps |grep bluetooth
logcat |grep (bluetooth的id)



注意没有/sdcard/btsnoop_hci.log的话(sdcard不可用)，
一定要用/data/misc/bluedroid/btsnoop_hci.log，selinux在init.rc就授权
```

```
蓝牙模板hardware
diff --git a/include/vnd_rk30sdk.txt b/include/vnd_rk30sdk.txt
index 16491f6..59c7ec1 100644
--- a/include/vnd_rk30sdk.txt
+++ b/include/vnd_rk30sdk.txt
@@ -4,12 +4,11 @@ UART_TARGET_BAUD_RATE = 1500000
 FW_PATCH_SETTLEMENT_DELAY_MS = 200
 USERIAL_VENDOR_SET_BAUD_DELAY_US = 200000
 LPM_IDLE_TIMEOUT_MULTIPLE = 5
-BTVND_DBG = FALSE
-BTHW_DBG = FALSE
-VNDUSERIAL_DBG = FALSE
-UPIO_DBG = FALSE
+BTVND_DBG = TRUE
+BTHW_DBG = TRUE
+VNDUSERIAL_DBG = TRUE
+UPIO_DBG = TRUE

logcat |grep -e bt_hwcfg -e bt_vendor -e bt_userial_vendor -e bt_upio

```
```
蓝牙模板一：
修改文件/system/etc/bluetooth/bt_stack.conf
diff --git a/bt_stack.conf b/bt_stack.conf
index a0afcb54..14d76f54 100644
--- a/bt_stack.conf
+++ b/bt_stack.conf
@@ -1,12 +1,12 @@
 # Enable BtSnoop logging function
 # valid value : true, false
-BtSnoopLogOutput=false
+BtSnoopLogOutput=true

 # BtSnoop log output file
-BtSnoopFileName=/data/misc/bluetooth/logs/btsnoop_hci.log
+BtSnoopFileName=/sdcard/btsnoop_hci.log

 # Preserve existing BtSnoop log before overwriting
-BtSnoopSaveLog=false
+BtSnoopSaveLog=true
 
关闭再打开蓝牙，每打开一次蓝牙产生新的btsnoop_hci.log
复现问题后，不要再动蓝牙开关(再次打开蓝牙后会产生新的btsnoop_hci.log就看不到问题)提供一下/sdcard/btsnoop_hci.log


logcat -c 
cd /data  到data目录下
然后关闭再打开蓝牙，打开蓝牙后的瞬间，快速运行如下命令
logcat -v time|grep $(ps |grep bluetooth |busybox awk '{print $2}') > logcat_bt.txt &

问题出现后，将logcat_bt.txt提供一下




测量一下LPO引脚有没有32.768k clk提供还有频偏，
PMU 配置LDO提供的 VDDIO引脚的电平，VBAT引脚电平，BT_RTS_N使能引脚电平，还有晶体频偏
检查一下域电压是否一致，IO导通性也测试一下

echo 0 > /sys/class/rfkill/rfkill0/state
echo 1 > /sys/class/rfkill/rfkill0/state
BT_RTS_N引脚有无电平变化

bt_wake引脚在模组出问题的时候（比如蓝牙反复打开，最好保证bt_wake这个引脚是高电平），
不清楚是否有进入lpm模式的fw，最好就是保证这个引脚是高电平，虽然我之前屏蔽过这个脚，
但是没有确认过默认是否是高，可能会导致过一段时间死掉。
 
```

```
蓝牙模板二：
diff --git a/bt_stack.conf b/bt_stack.conf
index 711fe51..0d62aed 100644
--- a/bt_stack.conf
+++ b/bt_stack.conf
@@ -1,6 +1,6 @@
 # Enable BtSnoop logging function
 # valid value : true, false
-BtSnoopLogOutput=false
+BtSnoopLogOutput=true

 # BtSnoop log output file
 BtSnoopFileName=/sdcard/btsnoop_hci.log
@@ -20,19 +20,19 @@ TraceConf=true
 #   BT_TRACE_LEVEL_EVENT   4    ( Debug messages for events )
 #   BT_TRACE_LEVEL_DEBUG   5    ( Full debug messages )
 #   BT_TRACE_LEVEL_VERBOSE 6    ( Verbose messages ) - Currently supported for TRC_BTAPP only.
-TRC_BTM=2
-TRC_HCI=2
-TRC_L2CAP=2
-TRC_RFCOMM=2
-TRC_OBEX=2
-TRC_AVCT=2
-TRC_AVDT=2
-TRC_AVRC=2
-TRC_AVDT_SCB=2
-TRC_AVDT_CCB=2
-TRC_A2D=2
-TRC_SDP=2
-TRC_GATT=2
-TRC_SMP=2
-TRC_BTAPP=2
-TRC_BTIF=2
+TRC_BTM=6
+TRC_HCI=6
+TRC_L2CAP=6
+TRC_RFCOMM=6
+TRC_OBEX=6
+TRC_AVCT=6
+TRC_AVDT=6
+TRC_AVRC=6
+TRC_AVDT_SCB=6
+TRC_AVDT_CCB=6
+TRC_A2D=6
+TRC_SDP=6
+TRC_GATT=6
+TRC_SMP=6
+TRC_BTAPP=6
+TRC_BTIF=6

关闭再打开蓝牙，每打开一次蓝牙产生新的btsnoop_hci.log
复现问题后，不要再动蓝牙开关(再次打开蓝牙后会产生新的btsnoop_hci.log就看不到问题)提供一下/sdcard/btsnoop_hci.log

logcat -c 
cd /data  到data目录下
然后关闭再打开蓝牙，打开蓝牙后的瞬间，快速运行如下命令
logcat -v time|grep $(ps |grep bluetooth |busybox awk '{print $2}') > logcat_bt.txt &

问题出现后，将logcat_bt.txt提供一下



测量一下LPO引脚有没有32.768k clk提供还有频偏，
PMU 配置LDO提供的 VDDIO引脚的电平，VBAT引脚电平，BT_RTS_N使能引脚电平，还有晶体频偏
检查一下域电压是否一致，IO导通性也测试一下

echo 0 > /sys/class/rfkill/rfkill0/state
echo 1 > /sys/class/rfkill/rfkill0/state
BT_RTS_N引脚有无电平变化
```
蓝牙模板三：
```
setprop  persist.bluetooth.btsnoopenable  true

setprop  persist.bluetooth.btsnooppath   /sdcard/btsnoop_hci.log

关闭再打开蓝牙，每打开一次蓝牙产生新的btsnoop_hci.log
复现问题后，不要再动蓝牙开关(再次打开蓝牙后会产生新的btsnoop_hci.log就看不到问题)提供一下/sdcard/btsnoop_hci.log

```
蓝牙模板四：
```
setprop persist.bluetooth.btsnooplogmode full

setprop  persist.bluetooth.btsnooppath   /sdcard/btsnoop_hci.log

关闭再打开蓝牙，每打开一次蓝牙产生新的btsnoop_hci.log
复现问题后，不要再动蓝牙开关(再次打开蓝牙后会产生新的btsnoop_hci.log就看不到问题)提供一下/sdcard/btsnoop_hci.log
```
蓝牙模板五：
```
7.1之后，蓝牙的btsnoop是抓不齐的，要抓vendor可以

diff --git a/include/vnd_rksdk.txt b/include/vnd_rksdk.txt
index 811cf25..967b27c 100644
--- a/include/vnd_rksdk.txt
+++ b/include/vnd_rksdk.txt
@@ -5,10 +5,10 @@ UART_TARGET_BAUD_RATE = 1500000
 FW_PATCH_SETTLEMENT_DELAY_MS = 200
 USERIAL_VENDOR_SET_BAUD_DELAY_US = 200000
 LPM_IDLE_TIMEOUT_MULTIPLE = 5
-BTVND_DBG = FALSE
+BTVND_DBG = TRUE
 BTHW_DBG = TRUE
-VNDUSERIAL_DBG = FALSE
-UPIO_DBG = FALSE
+VNDUSERIAL_DBG = TRUE
+UPIO_DBG = TRUE
 USE_CONTROLLER_BDADDR = FALSE
 FW_AUTO_DETECTION = TRUE
 BT_WAKE_VIA_PROC = TRUE

--- a/bluetooth/1.0/default/vendor_interface.cc
+++ b/bluetooth/1.0/default/vendor_interface.cc
@@ -16,6 +16,8 @@

 #include "vendor_interface.h"

+
+#define LOG_NDEBUG 0
 #define LOG_TAG "android.hardware.bluetooth@1.0-impl"
 #include <cutils/properties.h>
 #include <utils/Log.h>


```


蓝牙模板五：
```
最好用逻辑分析仪抓波形

diff --git a/include/vnd_rksdk.txt b/include/vnd_rksdk.txt
index 811cf25..7fef04e 100644
--- a/include/vnd_rksdk.txt
+++ b/include/vnd_rksdk.txt
@@ -15,8 +15,8 @@ BT_WAKE_VIA_PROC = TRUE
 SCO_PCM_ROUTING = 0
 SCO_PCM_IF_CLOCK_RATE = 2
 SCO_PCM_IF_FRAME_TYPE = 0
-SCO_PCM_IF_SYNC_MODE = 0
-SCO_PCM_IF_CLOCK_MODE = 0
+SCO_PCM_IF_SYNC_MODE = 1
+SCO_PCM_IF_CLOCK_MODE = 1
 PCM_DATA_FMT_SHIFT_MODE = 0
 PCM_DATA_FMT_FILL_BITS = 0
 PCM_DATA_FMT_FILL_METHOD = 0

将模组的PCM_IN PCM_OUT PCM_SYNC PCM_CLK 与声卡的连接断开，然后将模组的PCM_IN PCM_OUT进行短接

再用微信或者app语音测试，进行回环测试，手机上能否自己说话自己听到

用示波器抓一下PCM_SYNC PCM_CLK 两端,是否有clk，是声卡那边提供还是模组那边提供,clk的频率和峰峰值
```
蓝牙更换模板
```
7.1 8723bu

mount -o rw,remount  /system
cp   /storage/usbport1/8723bu/bluetooth_rtk.default.so  /system/lib/hw/bluetooth_rtk.default.so
cp   /storage/usbport1/8723bu/Bluetooth.apk          /system/app/Bluetooth/Bluetooth.apk
cp   /storage/usbport1/8723bu/libbluetooth_jni.so   /system/lib/libbluetooth_jni.so
cp   /storage/usbport1/8723bu/libbt-vendor_usb.so   /system/lib/libbt-vendor_usb.so
cp  /storage/usbport1/8723bu/rtl8723bu_config     /system/etc/firmware/
cp  /storage/usbport1/8723bu/rtl8723b_fw     /system/etc/firmware/

md5sum  /system/lib/hw/bluetooth_rtk.default.so  /system/app/Bluetooth/Bluetooth.apk  /system/lib/libbluetooth_jni.so  /system/lib/libbt-vendor_usb.so

```

```
wifi模板一：

点击关系平板电脑--->开发者模式（连续点击 版本号）退出-->系统-->高级---> 开发者选项-->启动wlan 详细日志记录功能（勾选）
点击关系平板电脑--->开发者模式（连续点击 版本号）退出-->系统-->高级---> 开发者选项-->启动bluetooth hci snoop log 详细日志记录功能（勾选）
./data/media/0/btsnoop_hci.cfa

```


```
wifi模板二：
测量一下LPO 引脚有无32.768k clk 还有频偏，测量WL_REG_ON电平 ，VDDIO电平 ,VBAT电平，还有晶体频偏

echo 0 > /sys/class/rkwifi/power
echo 1 > /sys/class/rkwifi/power

测量一下有无电平变化WL_REG_ON

如果是4.4的内核，那就只有echo 0 > /sys/class/rkwifi/power能生效
有sdio_pwrseq节点，/drivers/core/mmc sdio模块启动会控制拉高WL_REG_ON,如果能拉高，就会控制PMU输出32K
sdio_pwrseq: sdio-pwrseq {
         compatible = "mmc-pwrseq-simple";
         clocks = <&rk808 1>;
         clock-names = "ext_clock";
         pinctrl-names = "default";
         pinctrl-0 = <&wifi_enable_h>;
         reset-gpios = <&gpio0 10 GPIO_ACTIVE_LOW>; /* GPIO0_B2 */
     };

打不开wifi，设置reset-gpios = <&gpio0 10 GPIO_ACTIVE_HIGH>; 测试一下。还不行检查一下引脚导通性或者是否被复用


 cat /d/clk/clk_summary (只有在节点上有clock name的才会显示，没有的就没有，比如设置clock frequency)
 
 find /d/ -name pinmux-pins 
 cat  pinmux-pins| grep mmc (sdio有时就是不知道用哪一个  有时候是sdio节点 有时候又sdmmc节点 这个要看看)
 
 wifi扫描不到经常是模组晶振频偏问题
 
 CONFIG_WIFI_LOAD_DRIVER_WHEN_KERNEL_BOOTUP =n (编译入kernel不是编译模块 去掉module init，extern出去 在sys接口上调用)
 echo 1 > /sys/class/rkwifi/driver
 
 查看打印出来的log看看问题
 

 
```

```
hcq@ubuntu:~/33997.1/kernel/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd$ git diff  Makefile   
diff --git a/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/Makefile b/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/Makefile
index b8a11e7..51d812f 100644
--- a/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/Makefile
+++ b/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/Makefile
@@ -23,6 +23,11 @@ DHDCFLAGS = -Wall -Wstrict-prototypes -Dlinux -DBCMDRIVER -DSDTEST       \
        -Idrivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd \
        -Idrivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/include

+DHDCFLAGS += -DDHD_8021X_DUMP -DDHD_DHCP_DUMP -DDHD_ICMP_DUMP -DDHD_ARP_DUMP
+DHDCFLAGS += -DDHD_TX_DUMP -DDHD_TX_FULL_DUMP
+DHDCFLAGS += -DDHD_RX_DUMP -DDHD_RX_FULL_DUMP
+
+
 DHDOFILES = aiutils.o siutils.o sbutils.o bcmutils.o bcmwifi_channels.o \
        dhd_linux.o dhd_linux_platdev.o dhd_linux_sched.o dhd_pno.o \
        dhd_common.o dhd_ip.o dhd_linux_wq.o dhd_custom_gpio.o \

再提供上层logcat
```
```
sdio配置发一下
32K波形测一下，是否是直的方波
模组是否有接晶振，有的话测一下频偏
sdio是否有接外部上拉，没有就提供内部上拉强度，有的话禁掉内部上拉
io 电压是否一致 io domain配置是否和接的io电压一致
```



