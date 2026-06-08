# HPM6E80IVM1 数据手册简略骨架

> 基于《HPM6E00 系列高性能微控制器数据手册 Rev0.10》提取
> 注：本文档仅摘录与机器人信号板设计相关关键参数，完整数据以官方数据手册为准

---

## 1. 芯片概述

HPM6E80IVM1 属于 **HPM6E00 系列**，是面向**实时以太网和运动控制**的 RISC-V 双核高性能 MCU。

- **内核**: 双核 32 位 RISC-V @ 600MHz，支持 RV32-IMAFDCBP（含位操作扩展）
- **性能**: 5.6 CoreMark/MHz
- **封装**: 289BGA (14×14mm, 0.8mm pitch) / 196BGA (12×12mm, 0.8mm pitch)
- **工作温度**: -40°C ~ +105°C（环境温度）
- **特色**: 内置千兆以太网 TSN 交换机、EtherCAT 从站控制器、四轴运动控制

---

## 2. 关键资源

### 2.1 内核与系统

| 项目 | 规格 |
|------|------|
| CPU | 双核 RISC-V @ 600MHz |
| L1 I-Cache / D-Cache | 各 32KB |
| ILM / DLM (每核) | 各 256KB |
| 中断 | PLIC: 8 级优先级，中断嵌套/向量扩展 |
| DMA | XDMA(32ch) + HDMA(32ch) + DMAMUX |
| 核间通信 | 2x 邮箱 MBX |
| 协处理器 | FFA（FFT + FIR 加速，4096点） |

### 2.2 内部存储器

| 类型 | 容量 | 说明 |
|------|------|------|
| ILM0/DLM0 (CPU0) | 256KB+256KB | 指令/数据本地存储器 |
| ILM1/DLM1 (CPU1) | 256KB+256KB | 指令/数据本地存储器 |
| AXI SRAM0 | 512KB | 高速片上 SRAM |
| AXI SRAM1 | 512KB | 高速片上 SRAM |
| AHB SRAM | 32KB | 低延迟 DMA 访问 |
| **片上 SRAM 总计** | **2080KB** | |
| Boot ROM | 128KB | 启动代码 + Flashloader |
| OTP | 4096 位 | 出厂信息/密钥/启动配置 |
| 内置 Flash | 可选型号（HPM6E8Y/6E84 含 4MB） | HPM6E80IVM1 **无内置 Flash** |

### 2.3 实时以太网系统 ⭐

| 模块 | 说明 |
|------|------|
| **EtherCAT 从站控制器 (ESC)** | 3 端口，8 FMMU，8 SyncManager，60KB ProcessDataRAM，64位分布时钟 |
| **TSN 交换机 (TSW)** | 4 端口（3 外部 + 1 内部）100/1000Mbps，支持 TSN 协议族 |
| **以太网控制器 (ENET)** | 10/100/1000M，RGMII/RMII/MII，IEEE1588 时间戳 |
| **内置以太网 PHY** | 2×100M PHY（仅 HPM6E8Y 型号，HPM6E80IVM1 **不含**） |

### 2.4 通讯外设 ⭐

| 外设 | 数量 | 说明 |
|------|------|------|
| **UART** | **17 路** (含 1x PUART 低功耗) | 最高 80MHz 时钟 |
| **SPI** | **8 路** | 最高 80MHz 时钟 |
| **I2C** | **8 路** | 标准/快速/快速+ (100k/400k/1Mbps) |
| **CAN-FD** | **8 路 原生** ⭐ | 支持 CAN2.0B + CAN-FD (8Mbps)，自带时间戳 |
| **USB2.0 OTG** | **1 路** | 集成 HS-PHY |
| 精确时间协议 PTPC | 1 路 | 64位计数器，连接 CAN 模块 |

> **关键改进**: HPM6E80 提供 **8 路原生 CAN-FD**，无需外扩，大大简化设计！

### 2.5 运动控制系统 ⭐

| 外设 | 数量 | 说明 |
|------|------|------|
| PWM (PWMV2) | 4 模块 × 8 通道 = **32 路** | 100ps 高精度，互补输出，死区插入 |
| 正交编码器输入 QEIV2 | 4 路 | |
| 正交编码器输出 QEOV2 | 4 路 | 模拟编码器信号输出 |
| 串行编码器接口 SEI | 4 路 | 支持多种绝对值编码器协议 |
| 旋转变压器解码 RDC | **2 路** ⭐ | 支持旋变传感器 |
| 运动管理控制器 MTG | 2 路 | 轨迹规划 |
| 坐标变换器 VSC | 2 路 | Park/Clarke 变换 |
| 环路计算器 CLC | 2 路 | PID 闭环控制 |
| Σ∆ 信号接收 SDM | 2×4 通道 | 8 通道 Σ∆ 数字滤波 |
| 可编程逻辑 PLB | 1 路 | 自定义逻辑 |
| 互联管理器 TRGM | 1 路 | 模块间信号互联 |

### 2.6 通用定时器

| 外设 | 数量 | 说明 |
|------|------|------|
| GPTMR | 9 组 × 4 通道 | 32 位，含 1x PTMR 电源管理域 |
| 看门狗 | 5 个 | 含 1x PWDG 电源管理域 |
| RTC | 1 个 | 电池备份域 |

### 2.7 模拟外设

| 外设 | 数量 | 说明 |
|------|------|------|
| 16 位 ADC | **4 个** | 16 输入通道，2MSPS（12 位模式 4MSPS），ENOB 12bit |
| 比较器 ACMP | **8 个** | 轨到轨，内置 8 位 DAC，高速模式 6.5ns |

### 2.8 音频外设

| 外设 | 说明 |
|------|------|
| I2S | 2 组，TDM 最多 16 通道 |
| PDM | 数字麦克风，8 通道 |
| DAO | 数字音频输出，Class-D |

### 2.9 外部存储器接口

| 接口 | 说明 |
|------|------|
| XPI (1 个) | SPI NOR Flash / NAND / HyperBus / PSRAM，最高 166MHz DDR |
| FEMC | SDRAM/LPSDRAM 8/16/32bit @166MHz + SRAM 控制器 |
| PPI | 可编程并口总线，最高 32bit，4 片选，支持同步/异步 |

#### XPI0 Channel A 引脚 (SPI NOR Flash，ALT14)

| 信号 | 289BGA 引脚 | BGA 位置 | 说明 |
|------|------------|----------|------|
| XPI0_CA_CS0 | **PB30** | A5 | 片选 0 |
| XPI0_CA_SCLK | **PB29** | B4 | 时钟 |
| XPI0_CA_D_0 | **PB27** | B3 | 数据 0 (MOSI) |
| XPI0_CA_D_1 | **PB28** | A4 | 数据 1 (MISO) |
| XPI0_CA_D_2 | **PB26** | A3 | 数据 2 (WP#) |
| XPI0_CA_D_3 | **PB31** | A5 | 数据 3 (HOLD#) |
| XPI0_CA_DQS | **PB25** | B2 | 数据选通 (DDR 模式) |
| XPI0_CA_CS1 | **PB24** | A2 | 片选 1 (可选) |

> **注意**: XPI0 引脚在 **PB24~PB31** 组 (B04 分组)，不是 PF 引脚！

### 2.10 信息安全

AES-128/256/SM4、SHA-1/256/SM3、真随机数发生器 RNG、安全启动、EXIP 在线解密、密钥管理器 KEYM

### 2.11 GPIO

- **206 个 GPIO** (289BGA) / **148 个 GPIO** (196BGA)
- 支持 3.3V / 1.8V 模式，分组供电
- 开漏、内部上下拉、驱动能力调节、施密特触发器
- 电源管理域 (PYx) 和电池备份域 (PZx) 支持低功耗保持

---

## 3. 电源

### 3.1 供电架构

单一 3.0~3.6V 输入（DCDC_IN + VPMC），内置 DCDC Buck + LDO：

| 电源轨 | 类型 | 说明 |
|--------|------|------|
| **DCDC_IN** (8 pins) | 3.0~3.6V 输入 | → 片上 Buck DCDC → **VDD_SOC** (0.9~1.3V) |
| **DCDC_LP** (5 pins) | 开关节点(LX) | 接 L1(4.7μH) → VDD_SOC 网络 |
| **VDD_SOC** (11 pins) | **片上 DCDC 输出** | 核心供电，**严禁外部供电** |
| **VPMC** (U4) | 3.0~3.6V 输入 | → 片上 LDOPMC → **VDD_PMCCAP**(~1.1V) |
| | | → 片上 LDOOTP → **VDD_OTPCAP**(~1.1V) |
| **VDD_USB** (M12) | **片上 LDO 输出 (~1.0V)** | USB 核心供电，**严禁外部供电，严禁接3.3V！** |
| **VBAT** (U8) | 2.4~3.6V 输入 | RTC/备份域（可直连 3.3V 或经 LDO 供电） |
| **VIO_Bxx** | 3.3V / 1.8V 输入 | IO 分组供电 |
| **VANA** (M11) | 3.0~3.6V 输入 | 模拟供电（经 FB 滤波） |
| **VPLLVUSB** | PLL + USB 供电 | 含 VPLL(G12) + VUSB(H12) 两个电源焊球 |

> **⚠️ 关键区别**: 与 HPM6454 不同，HPM6E80 **无 VDD_BATCAP/LDOBAT**，**VDD_USB 是内部 LDO 输出（非电源输入）**

### 3.2 DCDC 外围元件

| 位号 | 参考值 |
|------|--------|
| L1 (DCDC 电感) | 2.2~10μH，典型 4.7μH |
| C1 (DCDC_IN) | 33~66μF |
| C2 (VDD_SOC) | 0.1μF |
| C3 (片上 DCDC 内部节点) | 0.1μF |
| C4 (VDD_PMCCAP) | 4.7μF + 0.1μF |
| C5 (VDD_OTPCAP) | 4.7μF + 0.1μF |
| VDD_PHYxCAP | 10μF + 0.1μF（内置 PHY 型号） |

### 3.3 DCDC 电气特性

| 参数 | 最小值 | 典型值 | 最大值 | 单位 |
|------|--------|--------|--------|------|
| 输入电压 | 3.0 | 3.3 | 3.6 | V |
| 输出电压 | 0.6 | - | 1.375 | V |
| 输出精度 (Run) | -3% | - | +3% | |
| 过流保护 | - | 4 | - | A |

### 3.4 工作模式

| 模式 | CPU0 | CPU1 | VDD_SOC | VPMC | VBAT |
|------|------|------|---------|------|------|
| 等待 | 开 | 可选 | 开 | 开 | 开 |
| 停止 | 可选 | 可选 | 开 | 开 | 开 |
| 休眠 | 关 | 关 | 关 | 开 | 开 |
| 关机 | 关 | 关 | 关 | 关 | 开 |

### 3.5 典型功耗 (DCDC_IN=3.3V, 双核全开)

| 条件 | 频率 | 外设 | 25°C | 85°C |
|------|------|------|------|------|
| VDD_SOC=1.25V | 600MHz | 全开 | 362mA | 395mA |
| VDD_SOC=1.25V | 600MHz | 全关 | 184mA | 219mA |
| VDD_SOC=1.15V | 480MHz | 全开 | 277mA | 293mA |
| VDD_SOC=1.15V | 200MHz | 全开 | 158mA | 178mA |

---

## 4. 时钟系统

| 时钟源 | 频率 | 说明 |
|--------|------|------|
| XTAL24M (OSC24M) | 24MHz | 主晶振，PLL 时钟源 |
| XTAL32K (OSC32K) | 32.768kHz | RTC 晶振 |
| RC24M | 24MHz | 内部 RC (±15% 未校准) |
| RC32K | 32kHz | 内部 RC (±10% 未校准) |
| PLL | VCO 400~1000MHz | **3 个 PLL**，小数分频+展频 |

### 关键外设时钟上限

| 时钟 | 最大值 | 对应外设 |
|------|--------|----------|
| 定时器 TMR | 100MHz | PWM/GPTMR |
| UART | 80MHz | 串口 |
| SPI | 80MHz | SPI |
| I2C | 80MHz | I2C |
| CAN | 80MHz | CAN-FD |

---

## 5. 引脚功能 (变化较大！)

### 5.1 JTAG 调试接口 (与旧芯片不同)

| 引脚 | 功能 (ALT24) | 说明 |
|------|-------------|------|
| PA04 | JTAG_TDO | 复位后高阻 |
| PA05 | JTAG_TDI | 复位后输入上拉 |
| PA06 | JTAG_TCK | 复位后输入下拉 |
| PA07 | JTAG_TMS | 复位后输入上拉 |
| PA08 | JTAG_TRST | 复位后输入上拉 |

### 5.2 启动配置 (与旧芯片不同！)

- BOOT_MODE[0:1] = **PA02:PA03**（旧芯片为 PZ07:PZ06）
  - 00: XPI NOR 启动
  - 01: ISP/串行启动（UART0/USB0）
  - 10: ISP/串行启动
  - 11: 保留

### 5.3 特殊功能引脚

| 引脚名称 | 描述 |
|----------|------|
| XTAL_IN/OUT | 24MHz 主时钟 |
| RTC_XTAL_IN/OUT | 32.768kHz RTC 时钟 |
| RESET_N (RSTN) | 复位，低电平 ≥300μs |

### 5.4 内置以太网 PHY 引脚（如适用型号）

- ENETPHY0: PA20(RD_N), PA21(RD_P), PA22(TD_N), PA23(TD_P)
- ENETPHY1: PA16(RD_N), PA17(RD_P), PA18(TD_N), PA19(TD_P)
- PA27/PA28: RBIAS+LED
- PA30/PA31: MDIO/MDC

### 5.5 ⚠️ 关键引脚复用冲突表（机器人信号板 PINMUX 必看）

以下冲突直接影响 8 路 CAN-FD + JTAG + UART0 + TSN 的同时使用：

| 冲突组 | 引脚 | 功能 A | 功能 B | 严重程度 | 处理策略 |
|--------|------|--------|--------|----------|----------|
| **JTAG vs MCAN1** | PA04 | JTAG_TDO (ALT24) | MCAN1_RXD (ALT7) | 🔴 严重 | 量产固件后释放 JTAG，MCAN1 仅限量产态使用 |
| **JTAG vs MCAN1** | PA05 | JTAG_TDI (ALT24) | MCAN1_TXD (ALT7) | 🔴 严重 | 同上 |
| **JTAG vs MCAN2** | PA08 | JTAG_TRST (ALT24) | MCAN2_TXD (ALT7) | 🔴 严重 | 同上 |
| **UART0 vs MCAN0** | PA00 | UART0_TXD (ALT2) | MCAN0_TXD (ALT7) | 🟡 中等 | Debug 阶段用 UART0，量产释放给 CAN0 |
| **UART0 vs MCAN0** | PA01 | UART0_RXD (ALT2) | MCAN0_RXD (ALT7) | 🟡 中等 | 同上 |
| **BOOT vs MCAN0 STBY** | PA02 | BOOT_MODE1 | MCAN0_STBY (ALT7) | 🟢 低 | 启动后复用为 CAN0 STBY，启动时已锁存 |
| **BOOT vs MCAN1 STBY** | PA03 | BOOT_MODE0 | MCAN1_STBY (ALT7) | 🟢 低 | 同上 |
| **TSW vs MCAN4** | PA16/17/18 | TSW0_P1 端口 | MCAN4 全部引脚 | 🟡 中等 | 用 CAN4 则 TSW P1 不可用 |
| **TSW vs MCAN5** | PA20/21 | TSW0_P1 RXD_3/RXCK | MCAN5 RXD/TXD | 🟡 中等 | 用 CAN5 则 TSW P1 部分不可用 |
| **TSW vs MCAN6** | PA24/25/26 | TSW0_P1 TXCK/TXD_0/TXD_1 | MCAN6 全部引脚 | 🟡 中等 | 用 CAN6 则 TSW P1 不可用 |
| **TSW vs MCAN7** | PA30/31 | TSW0_P1 MDC/MDIO | MCAN7 RXD/TXD | 🟡 中等 | 用 CAN7 则 TSW P1 管理总线不可用 |
| **ETH0 MDIO vs MCAN7** | PA30 | ETH0_MDIO (ALT18) | MCAN7_RXD (ALT7) | 🟡 中等 | 外部 PHY 管理与 CAN7 冲突 |

> **建议方案**:
> - **开发阶段**: 保留 JTAG (PA04~PA08) + UART0 (PA00/PA01)，牺牲 MCAN0~MCAN2，使用 MCAN3~MCAN7 (5路)
> - **量产阶段**: JTAG 引脚释放给 MCAN1/MCAN2，UART0 释放给 MCAN0，实现全部 8 路 CAN-FD
> - **TSN 交换机**: 若使用 TSN，则 MCAN4~7 不可用（引脚被 TSW 占用），最多用 MCAN0~3 (4路)
> - **以太网 PHY MDIO**: 使用 PA30/PA31 做 MDIO/MDC 时，MCAN7 不可用

---

## 6. 电气特性 (关键参数)

### 6.1 绝对最大值

| 参数 | 最小值 | 最大值 |
|------|--------|--------|
| DCDC_IN / VPMC | -0.3V | 3.6V |
| VBAT | -0.3V | 3.6V |
| VIO_Bxx (3.3V) | -0.3V | 3.6V |
| VIO_Bxx (1.8V) | -0.3V | 1.98V |
| ESD HBM | - | 2000V |
| ESD CDM | - | **750V** (旧500V) |
| 存储温度 | -40°C | 150°C |

### 6.2 正常工作条件

| 参数 | 最小值 | 典型值 | 最大值 |
|------|--------|--------|--------|
| DCDC_IN | 3.0V | 3.3V | 3.6V |
| VPMC | 3.0V | 3.3V | 3.6V |
| VDD_SOC (性能 600MHz) | 1.25V | 1.275V | 1.30V |
| VDD_SOC (平衡 480MHz) | 1.15V | 1.175V | 1.30V |
| VDD_SOC (节能 400MHz) | 1.05V | 1.075V | 1.30V |
| T_A (环境温度) | -40°C | - | 105°C |

### 6.3 ADC 特性 (4个 16位 ADC)

- 采样率: **2MSPS**（12 位模式 4MSPS）
- ENOB: **12 bit** (旧芯片 12.3bit)
- SINAD: 74dB
- 输入通道: 16/个，共 **32 路模拟输入引脚**
- VREFH: 2.4~VDDA

---

## 7. 封装

| 封装 | 尺寸 | 间距 | GPIO |
|------|------|------|------|
| BGA289 | 14×14mm | 0.8mm | 206 |
| BGA196 | 12×12mm | 0.8mm | 148 |
| θJA (BGA289) | 31.6°C/W (±5%) | | |

---

## 8. 订购信息

| 型号 | CPU | 主频 | 内置Flash | 内置PHY | 封装 |
|------|-----|------|-----------|---------|------|
| **HPM6E80IVM1** ⭐ | 双核 | 600MHz | **无** | **无** | BGA289 |
| HPM6E8YIVM1 | 双核 | 600MHz | 4MB | 2×百兆 | BGA289 |
| HPM6E84IVM1 | 双核 | 600MHz | 4MB | 无 | BGA289 |
| HPM6E70IVM1 | 双核 | 600MHz | 无 | 无 | BGA289 |

> **HPM6E80IVM1** 无内置 Flash 和 PHY，需外挂 SPI NOR Flash + 外部千兆 PHY
