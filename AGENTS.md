# AGENTS.md — FOC_V0-1 (STM32G431RBT6 双路 FOC 电机控制)

## 项目概况
- **MCU:** STM32G431RBT6 (Cortex-M4F, 128MHz, FPU, CORDIC, 128KB Flash, 32KB RAM, LQFP-64)
- **领域:** 双路 PMSM/BLDC 电机 FOC（磁场定向控制）
- **工具链:** STM32CubeMX / STM32CubeIDE 项目（`.ioc` 文件为唯一配置源）

## CubeMX 项目关键约定
- `.ioc` 文件是引脚、时钟和外设配置的**唯一真实来源**。绝不手动编辑 HAL 初始化代码，应通过 CubeMX 重新生成。
- 用户代码写在 `/* USER CODE BEGIN */` / `/* USER CODE END */` 标记之间。在生成文件中标记之外写入代码会在重新生成时丢失。
- CubeMX 通常生成 Makefile、STM32CubeIDE 或 Keil/IAR 工程。生成后构建系统和 IDE 配置会出现在本目录。
- STM32G4xx HAL 库包含此系列专用优化函数（CORDIC 加速器、FMAC、CMSIS-DSP 数学函数）。

## 会话规则
- 本项目使用中文回答。
- 回答必须精炼，避免冗长解释。
- 根据实际情况分析，若用户观点有误须及时指出，不盲从。

## 安全关键注意事项
- FOC 涉及真实硬件上的 PWM 半桥驱动。死区时间、定时器配置或 ADC 采样时序错误可能烧毁 MOSFET。上电前务必对照硬件验证所有 TIM/ADC/DMA 配置。
- 电流采样 ADC 必须与 PWM 中心对齐模式同步，才能正确测量 FOC 电流。
