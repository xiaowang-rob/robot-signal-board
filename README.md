# 机器人信号板 (Robot Signal Board)

> 基于 **HPM6E80IVM1**（双核 RISC-V @ 600MHz）的多总线通信汇聚板
> 实现机器人控制系统中的多总线接入、边缘通信管理及上位机数据汇聚

---

## 项目概述

机器人信号板是面向工业机器人/移动机器人控制系统的通信汇聚与边缘管理硬件平台，将分散的传感器、驱动器总线（CAN-FD、RS485、UART）统一接入，通过 USB2.0 上报上位机，同时提供千兆以太网和 WiFi（HLK-7628N）双通道上行。

### 核心接口

| 接口 | 数量 | 说明 |
|------|------|------|
| 千兆以太网 (RGMII) | 1 路 | 10/100/1000M，YT8531C PHY |
| CAN-FD | 7 路原生 + 1 路桥接 | HPM6E80 自带 + MCP2518FD SPI 外扩 |
| RS485 | 8 路 | 半双工，工业级隔离 |
| UART 串口 | 4 路 | 含 1 路 Debug |
| HLK-7628N WiFi 模块 | 1 路 | 2.4G WiFi + 5 口百兆交换 |
| USB 2.0 | 1 路 | Device 模式，向上位机输出 |

### 数据流

```
传感器/驱动器 → CAN-FD×8, RS485×8, UART×4 → HPM6E80IVM1 → USB2.0 → 上位机
                                                    ↕
                                              HLK-7628N (局域网/无线)
```

## 硬件规格

| 项目 | 参数 |
|------|------|
| MCU | HPM6E80IVM1（双核 RISC-V @ 600MHz, BGA-289） |
| 存储 | SPI NOR Flash（XPI0 接口，建议 16MB+） |
| 以太网 | YT8531C 千兆 PHY（RGMII, QFN-40）+ 集成磁性 RJ45 |
| 电源输入 | USB 5V + DC Jack 5~12V（SS34 OR-ing） |
| 板级电源 | VCC5V 总线 + DCDC 5V→3.3V（≥3A）+ LDO 3.0V（VBAT） |
| PCB 层数 | 6 层板 |
| 封装 | 0405 为主，连接器 GH1.25 / 板对板 / 软排线 |

## 文档索引

| 文档 | 说明 |
|------|------|
| [`docs/设计方案.md`](docs/机器人信号板_设计方案.md) | 整体设计方案、接口分配、数据流 |
| [`docs/原理图指南.md`](docs/机器人信号板_原理图指南.md) | 详细原理图设计指导（含引脚连接表、配置说明） |
| [`docs/PCB指南.md`](docs/机器人信号板_PCB指南.md) | PCB 布局布线指导（叠层、阻抗、分区） |
| [`docs/软件指南.md`](docs/机器人信号板_软件指南.md) | 固件开发指南、外设配置、启动流程 |
| [`docs/器件选型清单.md`](docs/机器人信号板_器件选型清单.md) | BOM、选型建议、用量 |
| [`docs/引脚复用总表.md`](docs/HPM6E80IVM1_引脚复用总表.md) | HPM6E80IVM1 完整引脚复用分配 |
| [`docs/设计方案_核对清单.md`](docs/机器人信号板_设计方案_核对清单.md) | 设计审查核对清单 |

### 数据手册

| 文件 | 器件 |
|------|------|
| [`datasheets/HPM6E80IVM1.pdf`](datasheets/HPM6E80IVM1.pdf) | MCU 数据手册 |
| [`datasheets/YT8531C_规格书.pdf`](datasheets/YT8531C_规格书.pdf) | 千兆以太网 PHY |
| [`datasheets/HLK-7628N_规格书.pdf`](datasheets/HLK-7628N_规格书.pdf) | WiFi 模块 |
| [`datasheets/pinmux_pages.pdf`](datasheets/pinmux_pages.pdf) | 引脚复用参考页 |

## 设计状态（Rev 2.0）

| 模块 | 状态 |
|------|------|
| 电源架构 | ✅ 设计完成 |
| HPM6E80 最小系统 | ✅ 设计完成 |
| 千兆以太网 (YT8531C) | ✅ 原理图指南完成 |
| CAN-FD ×8 | ✅ 设计完成 |
| RS485 ×8 | ✅ 设计完成 |
| HLK-7628N WiFi | ✅ 设计完成 |
| USB 2.0 | ✅ 设计完成 |
| PCB 布局指南 | 🔄 以太网部分待补充 |
| KiCad 原理图 | ❌ 尚未创建 |

## 许可

本项目所有设计文档和原理图以 **CC BY-NC-SA 4.0** 许可发布（如无特别声明）。
