# STM32G431CBU6 引脚分配评估

**封装:** UFQFPN-48 (48 脚, 约 37 个可用 GPIO)

---

## 引脚需求统计

| 功能 | 引脚数 | 说明 |
|------|--------|------|
| 三路互补 PWM (TIM1) | 6 | CH1/CH1N, CH2/CH2N, CH3/CH3N |
| 电流采样 ADC | 2 | Ia, Ib（两电阻方案） |
| 母线电压 ADC | 1 | Vbus 分压 |
| 温度检测 ADC | 1 | NTC 温度 |
| 栅极驱动使能 | 1 | EN 控制 |
| 栅极驱动故障 | 1 | FLT 信号输入 |
| SPI 磁编码器 | 4 | SCK / MISO / MOSI / CS |
| UART 调试 | 2 | TX / RX |
| SWD 调试 | 2 | SWCLK / SWDIO |
| **合计** | **20** | |

---

## 评估结论: **够用，非常充裕**

UFQFPN-48 有 ~37 个可用 GPIO，20 个核心引脚仅占约 **54%**，剩余 **17 个引脚** 可扩展：

- **CAN** (2 脚) — 可加 SN65HVD230
- **USB-CDC** (2 脚) — 无需额外芯片
- **Hall 传感器** (3 脚) — 预留
- **第三路电流 Ic** (1 脚) — 三电阻方案
- **额外 GPIO** — LED / 按键 / 额外 SPI

---

## 建议 CubeMX 分配（TIM1 中心对齐 + ADC 注入组）

| 引脚 | 功能 | 备注 |
|------|------|------|
| PA8 | TIM1_CH1 | U 相上桥 |
| PA13 | TIM1_CH1N | U 相下桥（SWD 复用，注意调试冲突） |
| PA9 | TIM1_CH2 | V 相上桥 |
| PB0 | TIM1_CH2N | V 相下桥 |
| PA10 | TIM1_CH3 | W 相上桥 |
| PB1 | TIM1_CH3N | W 相下桥 |
| PA0 | ADC1_IN1 | Ia 电流 |
| PA1 | ADC1_IN2 | Ib 电流 |
| PA2 | ADC1_IN3 | Vbus 电压 |
| PA3 | ADC1_IN4 | NTC 温度 |
| PA5 | SPI1_SCK | 编码器 |
| PA6 | SPI1_MISO | 编码器 |
| PA7 | SPI1_MOSI | 编码器 |
| PA4 | GPIO (SPI1_NSS) | 编码器 CS |
| PA15 | GPIO | Gate EN |
| PB12 | GPIO | Gate FLT |
| PA11 | USART1_CTR/RT_TX? No... | → 调试 UART |
| PA12 | USART1_RX? No... | → 调试 UART |
| PB6 | LPUART1_TX | 调试 UART |
| PB7 | LPUART1_RX | 调试 UART |
| PA13 | SWDIO | 调试 |
| PA14 | SWCLK | 调试 |

> **关于 PA13 冲突:** TIM1_CH1N 默认在 PA13，与 SWDIO 复用。调试模式下 CH1N 功能会被 SWD 占用。解决方案：
> - 调试时用 TIM1_CH1N 的**重映射 AF** 到其他引脚（如 PB13），或
> - 调试时临时禁用 CH1N（仅用上桥 PWM，死区由栅极驱动处理），或
> - PA13 在调试连接后由 CubeMX 自动重映射为 SWD，但运行时失去 PWM 功能

**推荐:** 将 TIM1_CH1N 重映射到 PB13（备选），PA13 留给 SWDIO 专用。CubeMX 中 TIM1 有多个 remap 选项。

### 优化后的简洁分配

| 引脚 | 功能 |
|------|------|
| PA8 | TIM1_CH1 (UH) |
| PB13 | TIM1_CH1N (UL) ← 远离 PA13 |
| PA9 | TIM1_CH2 (VH) |
| PB14 | TIM1_CH2N (VL) ← 备选 |
| PA10 | TIM1_CH3 (WH) |
| PB15 | TIM1_CH3N (WL) ← 备选 |
| PA0 | Ia ADC |
| PA1 | Ib ADC |
| PA2 | Vbus ADC |
| PA4 | 编码器 SPI1_NSS |
| PA5 | SPI1_SCK |
| PA6 | SPI1_MISO |
| PA7 | SPI1_MOSI |
| PC4 | Gate EN |
| PB12 | Gate FLT |
| PB0 | NTC ADC (ADC1_IN15) |
| PB6 | LPUART1_TX |
| PB7 | LPUART1_RX |
| PA13 | SWDIO |
| PA14 | SWCLK |

---

## 结论

**48 脚完全够用**，无需升级到更高脚数封装。上述分配用了 ~20 脚，还剩大量 GPIO 可扩展。唯一注意点是 **TIM1_CH1N 避免复用 PA13**（SWD 冲突），CubeMX 中通过 TIM1 重映射解决。
