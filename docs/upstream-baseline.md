# 原版基线档案：BEETLE（CircuitDigest）

> 用途：W1 任务书与系统图的依据；也是“可复现性”评分的证据。
> 核对日期：2026-11-19。核对人：<姓名>。

## 1. 来源与版本

| 类型 | 链接 | 说明 |
|---|---|---|
| 图文教程 | <https://circuitdigest.com/microcontroller-projects/esp32-wall-climbing-robot> | 结构、电路、装配、控制、实测数据 |
| 代码与 3D | <https://github.com/Circuit-Digest/BEETLE-ROBOT> | 仓库仅 3 个文件 |
| 外壳 STL | <https://www.thingiverse.com/thing:7387540> | 另见仓库内 `BEETLE 3D FILES.zip` |

- **Commit SHA**：`dfad0b544459b171927d50707d58ace401587524`（2026-07-21，main 分支最新）
- **许可证**：README 末尾写 “MIT”，但仓库中**没有 LICENSE 文件**，GitHub 识别为无许可证。
  引用时按此事实注明；本组代码独立重写，不复用其源码。
- **仓库内容**：`README.md`、`wall_climbing.ino`（362 行）、`BEETLE 3D FILES.zip`

## 2. 原版器件（2S 方案）

| 类别 | 型号 / 规格 | 数量 | 备注 |
|---|---|---|---|
| 主控 | 经典 ESP32 开发板 | 1 | 任意支持 `ledc` 的型号 |
| 吸附动力 | 4300KV 无刷电机 | 1 | 经 ESC 驱动 |
| 螺旋桨 | 3 英寸 | 1 | 与电机、ESC 配套 |
| 电调 | 2S 6–8A ESC | 1 | 信号 50Hz PWM |
| 行走电机 | N20 减速电机 100RPM | 2 | 左右各一 |
| 电机驱动 | MX1508 双路 H 桥 | 1 | IN1/IN2 控制正反转 |
| 电池 | 2S 7.4V 350mAh LiPo | 1 | 动力源 |
| 降压 | MP1584 | 1 | 7.4V → 5V |
| 轮 | 轻质轮 + 橡胶胎面 | 4 | 后轮直连电机轴，前轮 2mm 轴 |

## 3. 引脚与信号

| 功能 | ESP32 引脚 | 说明 |
|---|---|---|
| ESC 信号 | GPIO4 | `ledc` 50Hz / 16bit，脉宽 1000–2000µs |
| 左电机 IN1 / IN2 | GPIO16 / GPIO17 | MX1508 |
| 右电机 IN1 / IN2 | GPIO18 / GPIO19 | MX1508 |

- 串口：115200，连上 WiFi 后打印 IP。
- 上电流程：`ledcWrite(1000)` → `delay(5000)` 完成 ESC arm → `WiFi.begin` → 注册路由 → `server.begin`。
- 网页接口：`/`、`/speed?value=`、`/bldcOn`、`/bldcOff`、`/forward`、`/backward`、`/left`、`/right`、`/stop`。

## 4. 关键指标（作者实测）

- 整机质量：约 **137 g**（含电池、ESP32、BLDC、ESC、驱动电机）。
- 实测吸附力：**360 g**；作者目标 ≥ 2 × 自重（274 g），实际约 **2.6 ×**。
- 续航估算（本组计算，待实测）：350mAh 在 6–8A 下约 **2.6–3.5 min**。

## 5. 代码结构（362 行）

| 部分 | 行号 | 作用 |
|---|---|---|
| 配置 | 1–21 | WiFi、引脚、PWM 频率与分辨率 |
| `usToDuty` / `setThrottle` | 23–43 | 微秒→占空比；0–100% → 1000–2000µs |
| 运动函数 | 49–91 | stop / forward / backward / left / right，直接写 GPIO |
| 网页 | 93–263 | 内嵌 HTML，`fetch` 轮询 |
| 路由 | 269–356 | 速度、吸附开关、方向按钮 |
| 主流程 | 299–362 | setup：arm → WiFi → 路由；loop：`handleClient` |

## 6. 已识别的缺口（转为 Issue）

| # | 缺口 | 后果 | 对应任务 |
|---|---|---|---|
| 1 | 无失联停车、无命令超时 | 断网或息屏后机器人继续跑 | W2 断联测试 |
| 2 | 无电压/电流/压差遥测 | W4 无法测输入功率与吸附力 | W4 测量 |
| 3 | 无软启动与限幅 | 桨瞬间满转 | W2 / W5 |
| 4 | 复位即重新 arm，WiFi 阻塞无超时 | 复位后桨可能再转；连不上永远不进 server | W2 |
| 5 | 网页无鉴权 | 同网段任何人可控 | W5 改型 |

## 7. 与原版改型（V5.0）的差异

| 项 | 原版 BEETLE | V5.0 改型 | 结论 |
|---|---|---|---|
| 动力 | 2S 7.4V + 3″ 桨 + 6–8A ESC | 3S + 50mm 涵道 + 40A ESC | **不直接兼容**，需重新设计匹配 |
| 控制 | 经典 ESP32 | ESP32-S3 | 换型需重新适配 PWM |
| 行走 | N20 + MX1508 | 可更换 | 必须复测电流 |

> 结论：首版按 2S 原版走，改型放到 W5，并在 BOM 中写“对标件—差异—风险—验证方式”。
