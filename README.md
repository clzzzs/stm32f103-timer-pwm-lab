# STM32F103C8T6 定时器与 PWM 呼吸灯实验

用**同一个 TIM2** 完成两件事：任务一用**更新中断**让 LED 以 2 s 周期亮灭；任务二把 **TIM2_CH1 配成 PWM 输出**，通过动态修改比较寄存器 `CCR1` 实现呼吸灯，并用 Keil 软件仿真中的逻辑分析仪对占空比变化做定量分析。

- 芯片：STM32F103C8T6（Blue Pill 最小系统板）
- 工具链：STM32CubeMX（STM32Cube FW_F1 V1.8.7）+ Keil MDK-ARM V5.32 + ST-Link V2
- 时钟：HSE 8 MHz → PLL ×9 → SYSCLK 72 MHz（APB1 36 MHz，APB1 上的定时器时钟为 72 MHz）

## 硬件接线

| 引脚 | 复用功能 | 连接 | 有效电平 |
| --- | --- | --- | --- |
| PA0 | TIM2_CH1 | 红色 LED + 220 Ω 限流电阻到 GND | 高电平点亮 |

两个任务复用同一套接线，只改软件配置。

## 关键参数

| 参数 | 任务一（计数方式） | 任务二（PWM 方式） |
| --- | --- | --- |
| Prescaler (PSC+1) | 7200 → 计数 10 kHz | 72 → 计数 1 MHz |
| Counter Period (ARR+1) | 10000 → 更新 1 s | 1000 → PWM 1 ms（1 kHz） |
| 通道 / 中断 | 无通道，使能更新中断 | Channel1 = PWM Generation，不使能中断 |
| 现象 | 亮 1 s / 灭 1 s，周期 2 s | 占空比 0→100%→0，呼吸周期 2 s |

定时时间：`T = (PSC+1)(ARR+1)/f_TIM = 7200 × 10000 ÷ 72 MHz = 1.000 s`；PWM 频率：`72 MHz ÷ 72 ÷ 1000 = 1 kHz`。

## 主要结果

- 任务一：LED 亮灭周期 2 s，占空比 50%，误差只取决于晶振精度与中断响应延迟。
- 任务二：1000 级调光，CCR1 每 1 ms 变化 1，一个完整呼吸周期 2 s。
- 仿真测量：`TIM2_CCR1` 呈 0→999→0 三角波，游标跨两个周期 Δ = 4.000716 s，折算单周期 **2.0004 s**（与设计偏差约 0.02%）。
- 仿真局限：Keil 的 STM32 模型只模拟寄存器、不驱动引脚电平，`porta.0` 恒为 0；`TIM2_SR` 的 UIF 置位后无人清除，不能用于测周期。

## 目录结构

```
.
├─ 实验3-定时器与PWM实验.md     报告正文（Markdown，含 22 张插图）
├─ output/
│  ├─ 实验3-定时器与PWM实验.pdf  报告 PDF（A4，19 页）
│  └─ zip/                       两个任务的 Keil 工程压缩包
├─ code/
│  ├─ task1/                     任务一：定时器中断控制 LED（CubeMX + Keil 工程）
│  └─ task2/                     任务二：PWM 呼吸灯（CubeMX + Keil 工程）
└─ images/                       报告插图（22 张图/动图 + 同名 mp4）
```

## 如何编译下载

1. 用 Keil 打开 `code/task1/MDK-ARM/task1.uvprojx` 或 `code/task2/MDK-ARM/task2.uvprojx`
2. F7 编译，F8 下载（工具链需已安装 STM32F1xx 器件支持包）

若要用 STM32CubeMX 修改配置，打开对应目录下的 `.ioc` 文件即可；重新生成时代码写在 `USER CODE` 区域内的部分会保留。

## 仓库地址

<https://github.com/clzzzs/stm32f103-timer-pwm-lab>

## 博客

（待填写）
