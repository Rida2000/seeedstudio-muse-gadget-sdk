# Seeed XIAO ESP32-S3 1.47 英寸触摸屏

[English](seeed-xiao-s3-147.md)

[Seeed Studio XIAO 1.47 英寸 IPS 触摸屏（ESP32-S3）](https://www.seeedstudio.com/1-47-Inch-Touch-Display-Powered-by-XIAO-ESP32-S3-Plus-p-6996.html)由 XIAO ESP32-S3 Plus 和一块小显示扩展板组成，带 172×320 IPS 屏幕、电容触摸、PDM 麦克风和两个用户按键。

<img src="https://files.seeedstudio.com/wiki/Display_Gadgets/imgs/147_ESP32S3Plus_display_hardware_heroNEW.jpg" width="480" alt="Seeed Studio XIAO ESP32-S3 Plus 1.47 英寸 IPS 触摸屏">

| 硬件 | 规格 |
|---|---|
| 主控 | XIAO ESP32-S3 Plus，双核 ESP32-S3，最高 240 MHz |
| 存储 | 16 MB Flash、8 MB 八线 PSRAM |
| 屏幕 | 1.47 英寸 172×320 IPS LCD，JD9853A，SPI 接口 |
| 输入 | 电容触摸（AXS5106L）、PDM 麦克风、USR1 和 USR2 |
| 无线 | 2.4 GHz Wi-Fi、蓝牙 LE |
| USB | 芯片自带的 USB Serial/JTAG（`303a:1001`） |

## 接入 Muse 后能做什么

Muse 头像、触摸设置、蓝牙配对、Wi-Fi、USR1 按住说话、中日文字幕，以及 USR2 息屏和唤醒。Muse 的回复以文字显示在屏幕上。

板子有麦克风、没有喇叭，所以按"只有麦克风"的设备运行：Settings > Sound 里只有麦克风增益和电平，回复始终以文字显示在屏幕上。板上的 LSM6DS3 IMU、MicroSD 卡槽和电池接口暂未使用；这个配置关闭了 OTA，固件通过 USB 更新。

已在一台设备上验证：开机、用 USR1 确认蓝牙配对、Wi-Fi、按住说话并在屏幕上显示回复、触摸设置、USR2 息屏和唤醒，以及在设置里关机后按 USR1 唤醒。在安静环境、20 cm 以内说话时识别效果最好。

## 引脚

引脚来自 [Seeed 官方入门指南](https://wiki.seeedstudio.com/getting_started_1.47_inch_touch_display_esp32s3/)、[XIAO ESP32-S3 Plus 引脚图](https://wiki.seeedstudio.com/xiao_esp32s3_getting_started/)和 [Seeed_GFX2](https://github.com/Seeed-Studio/Seeed_GFX2)（JD9853A 初始化序列、AXS5106L 触摸报告和 `0519_DashBoard_147_ESP32` PDM 示例）。改编的屏幕初始化序列保留了 Seeed 的 [MIT 许可声明](licenses/Seeed_GFX2.txt)。

| 功能 | XIAO 引脚 | GPIO |
|---|---|---|
| PDM 时钟／数据 | D0／D1 | 1／2 |
| LCD CS／DC | D2／D3 | 3／4 |
| LCD SCK／MOSI | D8／D10 | 7／9 |
| LCD 与触摸复位／背光 | D17／D18 | 13／12 |
| 触摸 SDA／SCL／INT | D4／D5／D7 | 5／6／44 |
| USR1 说话／USR2 息屏 | D19／D15 | 11／42 |
| SD 片选，保持高电平 | D6 | 43 |

JD9853A 使用 ST7789 兼容驱动加厂商初始化序列：MADCTL `0x48`（MX+BGR），X 偏移 34，Y 偏移 0，不反色。触摸的 X 坐标随之镜像。屏幕和触摸共用一根复位线，开机时在两者启动前拉一次。

PDM 麦克风在 I2S0 上，这是 S3 唯一能接收 PDM 的 I2S，采样率 16 kHz。板子负责去掉 PDM 滤波器带来的直流偏置，增益由 Muse 按 Settings > Sound 里的设置统一处理（默认 30 dB）。串口控制台走芯片自带的 USB，因为 UART0 的 GPIO43/44 用作 SD 片选和触摸中断。

## 编译和烧录

使用 ESP-IDF v6.0.1。在 `esp32/` 目录下：

```sh
tools/muse/board.sh build xiao147
idf.py -B build-muse-seeed-xiao-s3-147 menuconfig   # ESP32 Device SDK > Muse Gadgets SDK token
tools/muse/board.sh build xiao147
tools/muse/board.sh flash xiao147
```

同时插着多块芯片自带 USB 的板子时，在 `flash` 后面加上这块板的 USB 序列号（即它的 MAC）或串口。

## 配对和使用

1. 在 Muse app 里打开 **Settings > Devices > Developer mode**，点 **Add Device**，选择屏幕上显示的 `MuseGadget-XXXXXX`。
2. 按 **USR1** 确认配对，然后选择一个 2.4 GHz Wi-Fi。
3. 按住 **USR1** 说话，松开发送，回复显示在屏幕上。
4. 按 **USR2** 息屏，再按一次唤醒。
5. 在头像上向左滑打开设置，上下滑动翻页。

在 172 px 宽的屏幕上，设置页每一行的名称和数值分两行显示。
