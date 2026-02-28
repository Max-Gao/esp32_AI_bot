# 小智AI ESP32 固件模块结构分析

## 项目概述

本项目是一个基于ESP32系列芯片的AI聊天机器人固件，支持语音交互、物联网控制、多协议通信等功能。采用MIT许可证开源。

---

## 目录结构

```
esp32-main/
├── main/                    # 主程序目录
│   ├── application.cc/h    # 核心应用逻辑
│   ├── main.cc             # 程序入口
│   ├── background_task.h   # 后台任务管理
│   ├── mcp_server.h        # MCP协议服务器
│   ├── ota.h               # OTA远程升级
│   ├── settings.h          # NVS设置存储
│   ├── system_info.h       # 系统信息
│   ├── protocols/          # 通信协议模块
│   ├── audio_processing/   # 音频处理模块
│   ├── audio_codecs/       # 音频编解码器模块
│   ├── display/            # 显示模块
│   ├── led/                # LED控制模块
│   ├── iot/                # 物联网设备模块
│   └── boards/             # 80+种开发板支持
├── scripts/                # 工具脚本
├── docs/                   # 文档资料
├── partitions/             # Flash分区表
└── CMakeLists.txt          # 构建配置
```

---

## 核心模块详解

### 1. 应用核心 (main/application.cc/h)

**功能**：整个机器人的核心状态机和业务逻辑

**状态流转**：
```
starting (启动) → configuring (配置) → idle (空闲) → 
connecting (连接) → listening (聆听) → speaking (说话) → 
upgrading (升级) / idle
```

**主要功能**：
- 语音唤醒词检测
- 音频采集与播放
- 通信协议管理
- 设备状态管理
- 定时器任务(时钟、状态更新)

---

### 2. MCP服务器 (main/mcp_server.h)

**功能**：实现MCP(Model Context Protocol)协议，支持AI大模型控制设备

**核心类**：
- `Property` - 属性定义(布尔/整数/字符串)
- `PropertyList` - 属性列表
- `McpTool` - 工具定义
- `McpServer` - MCP服务器

**常用工具**：
- 音量控制 (volume: 0-100)
- 屏幕亮度 (brightness: 0-100)
- LED颜色/模式
- GPIO控制
- 播放音频

---

### 3. OTA远程升级 (main/ota.h)

**功能**：从远程服务器检查并下载新固件

**功能特性**：
- 版本检查与比对
- 固件下载与烧录
- 激活码验证
- 服务器时间同步
- MQTT/WebSocket配置下发

---

### 4. 设置存储 (main/settings.h)

**功能**：使用ESP32 NVS(非易失性存储)保存配置

**存储内容**：
- WiFi SSID/密码
- 服务器地址
- 设备ID
- 语言设置
- 音量/亮度默认值

---

### 5. 系统信息 (main/system_info.h)

**功能**：获取并显示系统信息

**信息内容**：
- 芯片型号 (ESP32-C3/S3/P4)
- MAC地址
- Flash大小
- 内存使用情况
- 固件版本

---

## 通信协议模块 (main/protocols/)

### 5.1 基础协议 (protocol.h/cc)

定义通用的通信接口和数据结构：
- `AudioStreamPacket` - 音频流数据包
- `BinaryProtocol2/3` - 二进制协议格式
- `ListeningMode` - 聆听模式(自动停止/手动停止/实时)

### 5.2 WebSocket协议 (websocket_protocol.cc)

- 基于WebSocket的全双工通信
- 支持JSON文本和二进制音频
- 自动重连机制
- 心跳保活

### 5.3 MQTT协议 (mqtt_protocol.cc)

- MQTT + UDP混合模式
- 主题订阅/发布
- 消息队列支持
- 低带宽优化

---

## 音频处理模块 (main/audio_processing/)

### 6.1 音频处理器

| 类名 | 说明 |
|------|------|
| `AudioProcessor` | 抽象基类 |
| `AfeAudioProcessor` | AFE语音前处理，支持VAD(语音活动检测)、AEC(回声消除) |
| `NoAudioProcessor` | 空实现(用于无需处理的场景) |

### 6.2 唤醒词检测

| 类名 | 说明 |
|------|------|
| `WakeWord` | 抽象基类 |
| `AfeWakeWord` | 神珍AI离线唤醒词(低功耗) |
| `EspWakeWord` | ESP-IDF内置唤醒词 |
| `NoWakeWord` | 禁用唤醒词 |

### 6.3 音频调试 (audio_debugger.cc)

- 音频数据录制与分析
- 调试服务器实时查看波形

---

## 音频编解码模块 (main/audio_codecs/)

### 7.1 支持的编解码器

| 编解码器 | 芯片 |
|----------|------|
| `es8388_audio_codec` | ES8388 (Box系列) |
| `es8374_audio_codec` | ES8374 |
| `es8311_audio_codec` | ES8311 |
| `box_audio_codec` | ESP-BOX专用 |
| `no_audio_codec` | 无编解码器 |

### 7.2 功能

- I2S音频输入输出
- ADC/DAC配置
- 音量控制
- 采样率配置

---

## 显示模块 (main/display/)

基于LVGL图形库实现

### 8.1 显示驱动

| 驱动 | 支持屏幕 |
|------|----------|
| `oled_display.cc` | OLED (SSD1306) |
| `lcd_display.cc` | LCD (ST7789, ILI9341, GC9A01等) |

### 8.2 UI功能

- **状态栏**：WiFi图标、电量、 mute状态
- **通知显示**：临时消息提示
- **表情显示**：开心、难过、思考等情绪
- **聊天消息**：对话内容展示
- **主题切换**：明暗主题

---

## LED控制模块 (main/led/)

| 类名 | 功能 |
|------|------|
| `SingleLed` | 单个LED指示灯 |
| `GpioLed` | GPIO LED控制 |
| `CircularStrip` | 环形灯带(WS2812) |
| `NoLed` | 无LED |

---

## 物联网模块 (main/iot/)

### 9.1 设备管理 (thing_manager.h)

- 设备注册与发现
- 状态同步
- 命令分发

### 9.2 内置设备 (things/)

| 设备 | 功能 |
|------|------|
| `Speaker` | 扬声器控制 |
| `Screen` | 屏幕控制 |
| `Lamp` | 灯光控制 |
| `Battery` | 电池管理 |

---

## 开发板支持 (main/boards/)

### 10.1 支持的芯片平台

- ESP32
- ESP32-C3
- ESP32-S3
- ESP32-P4

### 10.2 支持的开发板 (80+种)

**主流开发板**：
- 乐鑫 ESP32-S3-BOX3
- M5Stack CoreS3 / AtomS3R
- 立创·实战派 ESP32-S3
- 微雪电子 ESP32-S3-Touch-AMOLED
- LILYGO T-Circle-S3

**特色开发板**：
- ESP-HI 机器狗
- SenseCAP Watcher
- 璀璨·AI吊坠
- 虾哥 Mini C3

---

## 技术特点总结

### 11.1 核心技术

| 技术 | 说明 |
|------|------|
| MCP协议 | AI控制设备的标准化协议 |
| 离线唤醒 | ESP-SR本地语音唤醒 |
| 流式语音 | ASR → LLM → TTS 端到端语音交互 |
| OPUS编解码 | 高效音频压缩(24kHz采样) |
| LVGL UI | 轻量级嵌入式图形库 |

### 11.2 通信方式

- Wi-Fi / 4G (ML307 Cat.1)
- WebSocket + JSON
- MQTT + UDP

### 11.3 扩展能力

- 设备端MCP：控制音量、灯光、电机、GPIO
- 云端MCP：智能家居、PC控制、知识搜索、邮件收发

---

## 相关资源

- 官方文档：[小智AI百科全书](https://ccnphfhqs21z.feishu.cn/wiki/F5krwD16viZoF0kKkvDcrZNYnhb)
- 服务器地址：https://xiaozhi.me
- QQ交流群：575180511
