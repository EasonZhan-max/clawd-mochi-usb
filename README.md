# Clawd Mochi USB 控制版

这是一个基于 ESP32-C3 Super Mini 和 ST7789 1.54 寸 240x240 彩屏的小桌面表情屏项目。本仓库当前版本已经改成 USB 串口控制，不再使用 WiFi 热点控制。

本项目基于原版 Clawd Mochi 改造，原项目作者为 yousifamanuel。Claude、Clawd 等名称归其对应权利方所有，本仓库只是个人学习和改造版本。

## 当前版本功能

- ESP32-C3 Super Mini 驱动 ST7789 1.54 寸 240x240 屏幕
- 使用 USB 串口控制，不需要连 WiFi
- 浏览器 HTML 控制页：`usb_controller.html`
- 支持中英文界面切换
- 支持表情按钮、循环表情选择、速度调节、背景色、背光开关
- 支持自定义画布模式，可以在网页上画图并同步到屏幕
- 支持 Terminal 输入

## 文件说明

| 文件/目录 | 作用 |
| --- | --- |
| `clawd_mochi/clawd_mochi.ino` | 烧录到 ESP32-C3 的 Arduino 固件 |
| `usb_controller.html` | 电脑浏览器打开的 USB 控制页 |
| `models/` | 3D 打印外壳模型 |
| `pics/` | 项目图片 |
| `README.md` | 中文使用和修改说明 |

## 硬件清单

| 硬件 | 说明 |
| --- | --- |
| ESP32-C3 Super Mini | 主控板 |
| ST7789 1.54 寸 TFT 屏 | 240x240 SPI 彩屏 |
| 杜邦线/焊线 | 连接屏幕和 ESP32 |
| USB-C 数据线 | 烧录和控制都需要数据线，不只是充电线 |
| 3D 打印外壳 | 可选 |

## 接线方式

| 屏幕引脚 | ESP32-C3 Super Mini |
| --- | --- |
| VCC | 3V3 |
| GND | GND |
| SDA | GPIO 10 |
| SCL | GPIO 8 |
| RES / RST | GPIO 2 |
| DC | GPIO 1 |
| CS | GPIO 4 |
| BL | GPIO 3 |

注意：VCC 接 3.3V，不要接 5V。

## Arduino IDE 烧录教程

1. 安装 Arduino IDE 2.x。
2. 打开 Arduino IDE，进入 `文件 -> 首选项`。
3. 在“其他开发板管理器地址”中加入：

```text
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

4. 进入 `工具 -> 开发板 -> 开发板管理器`，搜索并安装 `esp32 by Espressif Systems`。
5. 进入 `工具 -> 管理库`，安装：

```text
Adafruit GFX Library
Adafruit ST7735 and ST7789 Library
```

6. 打开文件：

```text
clawd_mochi/clawd_mochi.ino
```

7. Arduino IDE 里建议这样设置：

| 设置 | 值 |
| --- | --- |
| Board | ESP32C3 Dev Module |
| USB CDC On Boot | Enabled |
| Upload Speed | 921600 或 115200 |
| Port | 选择 ESP32-C3 对应串口 |

8. 点击上传。

如果提示 `COM3 busy` 或 `port is busy`，先关闭浏览器控制页连接、Arduino 串口监视器，拔插 ESP32 后再上传。

## 使用 HTML 控制页

1. 先把 `clawd_mochi.ino` 烧录到 ESP32-C3。
2. 用 USB 数据线连接电脑和 ESP32-C3。
3. 用 Chrome 或 Edge 打开：

```text
usb_controller.html
```

4. 点击页面上的 `连接 ESP32`。
5. 浏览器弹窗里选择类似下面的串口：

```text
USB JTAG/serial debug unit
```

不要选 `COM1`。

6. 连接成功后，就可以用网页按钮控制表情。

## 控制页功能说明

| 功能 | 说明 |
| --- | --- |
| 正常眼睛 | 左右看和眨眼动画 |
| 眯眼 | 开心眯眼动画 |
| Claude Code | 显示 Claude Code 画面 |
| Logo | 显示 Logo 动画 |
| Hello Xinhua | 打字机显示 Hello / Xinhua |
| Hello Eason | 打字机显示 Hello / Eason |
| Sleep | 黑屏睡觉动画 |
| 状态画面 | 显示当前内部状态，主要用于调试 |
| 重置画面 | 恢复默认红色背景和正常眼睛 |
| 循环表情 | 按勾选项循环播放表情 |
| 速度 | 控制动画速度 |
| 背景色 | 修改表情背景色 |
| 屏幕开/关 | 控制背光 |
| 画布模式 | 在网页上画图并显示到屏幕 |
| Terminal | 在屏幕上显示输入的文字 |

## 串口命令

控制页本质上是通过 USB 串口发送命令，波特率是 `115200`。

常用命令：

```text
/cmd?k=w      正常眼睛
/cmd?k=s      眯眼
/cmd?k=c      Claude Code
/cmd?k=a      Logo
/cmd?k=x      Hello Xinhua
/cmd?k=e      Hello Eason
/cmd?k=z      Sleep 睡觉
/cmd?k=b      眨眼循环开/关
/cmd?k=d      进入 Terminal
/cmd?k=q      退出 Terminal
/status       显示状态画面
/state        返回 JSON 状态
/speed?v=1    慢速
/speed?v=2    正常速度
/speed?v=3    快速
/redraw?bg=#ff0000 设置背景色
/backlight?on=1 打开背光
/backlight?on=0 关闭背光
```

## 怎么修改表情

主要改这个文件：

```text
clawd_mochi/clawd_mochi.ino
```

### 修改眼睛大小和位置

在文件顶部附近找到：

```cpp
#define EYE_W   30
#define EYE_H   60
#define EYE_GAP 120
#define EYE_OX  0
#define EYE_OY  40
```

含义：

| 参数 | 作用 |
| --- | --- |
| `EYE_W` | 眼睛宽度 |
| `EYE_H` | 眼睛高度 |
| `EYE_GAP` | 两只眼睛之间的距离 |
| `EYE_OX` | 整体左右偏移 |
| `EYE_OY` | 整体上下偏移 |

### 修改默认背景色

在 `initColours()` 里找到：

```cpp
C_ORANGE = tft.color565(255, 0, 0);
```

这里当前是红色背景。比如改成蓝色：

```cpp
C_ORANGE = tft.color565(0, 80, 255);
```

### 修改 Hello 文案

找到：

```cpp
void drawXinhuaView()
void drawHelloEasonView()
```

里面的文字可以直接改，例如：

```cpp
drawTypewriterText2("Hello!", "Eason", 4, 4, C_BLACK, animBgColor);
```

### 新增一个表情

1. 在 `.ino` 里新增一个绘制函数，比如：

```cpp
void drawMyFace() {
  termMode = false;
  tft.fillScreen(animBgColor);
  tft.setTextColor(C_BLACK);
  tft.setTextSize(3);
  tft.setCursor(40, 100);
  tft.print("My Face");
}
```

2. 在 `routeCmd()` 里加一个命令：

```cpp
case 'm': drawMyFace(); break;
```

3. 在 `handleUsbCommand()` 的快捷字符列表里加入 `m`：

```cpp
String("wszcdaqtxebm")
```

4. 在 `usb_controller.html` 里增加按钮：

```html
<button data-cmd="m">My Face</button>
```

如果希望它出现在循环表情选择里，再加一个 checkbox：

```html
<label class="emote-check"><input type="checkbox" class="loop-pick" value="m" checked><span>My Face</span></label>
```

## 修改 HTML 控制页

主要改：

```text
usb_controller.html
```

常见修改位置：

| 想改什么 | 搜索关键词 |
| --- | --- |
| 按钮文字 | `data-i18n` 或按钮文本 |
| 增加按钮 | `data-cmd` |
| 循环表情选项 | `loop-pick` |
| 中英文翻译 | `const i18n` |
| 画布逻辑 | `drawCanvas`、`draw/stroke` |
| 串口连接 | `navigator.serial` |

注意：修改 HTML 后，浏览器里按 `Ctrl + F5` 强制刷新。

## 常见问题

### 连接失败或串口打不开

先关闭：

- Arduino 串口监视器
- 其他正在连接 ESP32 的网页
- 其他串口工具

然后拔插 ESP32，再重新连接。

### 上传失败，提示 port is busy

这是串口被占用。关闭控制页连接和 Arduino 串口监视器，再上传。

### HTML 页面没有串口选择弹窗

请使用 Chrome 或 Edge。部分浏览器不支持 Web Serial。

### 画布显示不一致

尽量先点击“画布模式”或直接在画布上开始画。当前版本开始画时会自动清空并同步屏幕画布。

## 许可证

原项目包含 `LICENSE` 文件。本仓库保留原许可证和原作者署名信息。
