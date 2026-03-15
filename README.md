# 🔥 STM32 学习与折腾记录

> "单片机一通，万物皆可亮灯。"  
> —— 某大二自动化/电子信息系同学的人生感悟

![芯片](https://img.shields.io/badge/MCU-STM32F103C8-blue?style=flat-square&logo=stmicroelectronics)
![内核](https://img.shields.io/badge/Core-Cortex--M3-green?style=flat-square)
![IDE](https://img.shields.io/badge/IDE-Keil%20MDK-orange?style=flat-square)
![库](https://img.shields.io/badge/Library-SPL%20V3.5.0-purple?style=flat-square)
![状态](https://img.shields.io/badge/状态-努力学习中-red?style=flat-square)
![进度](https://img.shields.io/badge/LED-已点亮-brightgreen?style=flat-square)

---

## 📖 项目简介

本仓库是一名大二在读生从 **2025年12月** 开始踏入 STM32 嵌入式世界的学习记录与代码仓库。

第一步永远是点灯——不是因为点灯简单，而是因为**点不亮才是真的痛苦**。  
在这里，你可以看到一个普通大学生如何从"寄存器是什么"到"哦原来就是内存映射的地址"的心路历程。

> 警告：本仓库存在大量注释掉的寄存器操作代码，请勿惊慌，那是作者在用生命理解底层原理。

---

## 🛠️ 开发环境

| 项目 | 内容 |
|------|------|
| **开发板** | STM32F103C8T6（江湖人称"蓝色小药丸" / Blue Pill） |
| **内核** | ARM Cortex-M3 @ 72MHz（本工程配置为 12MHz，够用） |
| **Flash** | 64KB（够写几千个 `while(1)`） |
| **RAM** | 20KB |
| **IDE** | Keil MDK uVision（ARMCC V5.06） |
| **外设库** | STM32 标准外设库 SPL V3.5.0 |
| **调试器** | ST-Link（总比 printf 调试优雅一点） |
| **操作系统** | 裸机（FreeRTOS？下辈子的事） |

---

## 📁 工程结构

```
Stm32-study or test/
└── 2-1 stm32工程模板/          ← 第一个 Keil 工程，神圣的起点
    ├── Project.uvprojx          ← Keil 工程主文件，双击它，人生开始
    ├── Project.uvoptx           ← 调试/仿真选项（别乱动）
    │
    ├── start/                   ← 启动层：芯片上电后做的第一件事
    │   ├── startup_stm32f10x_md.s   ← 汇编启动文件（堆栈初始化、跳 main）
    │   ├── core_cm3.c / .h          ← CMSIS Cortex-M3 内核接口
    │   ├── stm32f10x.h              ← 寄存器地址全家桶（看一眼头晕正常）
    │   └── system_stm32f10x.c / .h  ← SystemInit()，时钟配置在这里
    │
    ├── Library/                 ← STM32 标准外设库（23 个外设，一个不少）
    │   ├── stm32f10x_gpio.c/h   ← GPIO（本工程主角）
    │   ├── stm32f10x_rcc.c/h    ← 时钟控制（开时钟忘了？LED永远不亮）
    │   ├── stm32f10x_usart.c/h  ← 串口（printf 调试的希望）
    │   ├── stm32f10x_tim.c/h    ← 定时器（PWM/延时的核心）
    │   ├── stm32f10x_adc.c/h    ← ADC（电压测量）
    │   ├── stm32f10x_i2c.c/h    ← I²C（OLED 屏的入场券）
    │   ├── stm32f10x_spi.c/h    ← SPI（W25Q128 们的最爱）
    │   ├── stm32f10x_dma.c/h    ← DMA（摸鱼专用，让硬件自己搬数据）
    │   └── ...                  ← 其余 15 个外设模块（备而不用，备而安心）
    │
    ├── User/                    ← 用户代码，这里才是真正写代码的地方
    │   ├── main.c               ← 主程序（目前功能：让 LED 灭掉）
    │   ├── stm32f10x_conf.h     ← 库配置（include 所有外设头文件）
    │   ├── stm32f10x_it.c / .h  ← 中断服务程序（目前全是空的）
    │
    ├── Objects/                 ← 编译产物（.axf 可执行文件在这里）
    └── Listings/                ← 汇编列表文件（调试用）
```

---

## 🚀 当前进度：工程模板 `2-1`

### 功能说明

操控板载 LED（**PC13**，低电平点亮）。

```c
int main(void)
{
    // Step 1: 开 GPIOC 的时钟
    // 忘了这一步? LED 永远不亮，调试两小时后才发现没开时钟（不是我说的）
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);

    // Step 2: 配置 PC13 为推挽输出
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode  = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin   = GPIO_Pin_13;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOC, &GPIO_InitStructure);

    // Step 3: 输出高电平 → LED 灭（低电平亮，反逻辑，习惯就好）
    GPIO_SetBits(GPIOC, GPIO_Pin_13);

    while (1) { }   // 大道至简
}
```

> 代码里还保留了等效的**寄存器直接操作版本**（注释状态），  
> 因为理解了寄存器，你才真正理解了单片机——而不是在调用别人的魔法函数。

### 知识点清单

- [x] 理解 GPIO 的时钟门控机制（RCC）
- [x] 掌握 `GPIO_InitTypeDef` 结构体初始化方式
- [x] 了解推挽输出 vs 开漏输出的区别
- [x] 理解寄存器操作与标准库函数的对应关系
- [x] 成功点亮（或熄灭）板载 LED
- [ ] 让 LED 闪起来（下一步）
- [ ] 按键控制 LED（再下一步）
- [ ] 串口打印 Hello World（人生新高度）
- [ ] OLED 显示屏（装逼新境界）

---

## 📚 学习路线规划

```
GPIO 基础
    ↓
外部中断 (EXTI)
    ↓
定时器 + PWM
    ↓
串口通信 (USART)
    ↓
I²C / SPI 总线
    ↓
ADC / DAC
    ↓
DMA 数据传输
    ↓
... (FreeRTOS？也许吧)
```

---

## 💡 写给未来迷路的自己

1. **编译报错先看第一个**，后面的往往是连锁反应，别一个一个看到崩溃。
2. **忘记开时钟**是新手三大死穴之首，另外两个是：引脚配置错、中断没使能。
3. **HardFault** 出现时深呼吸，大概率是数组越界或空指针。
4. `while(1)` 是最孤独也是最坚定的死循环，MCU 不跑完这辈子就在这里了。
5. 看不懂的寄存器，翻 **Reference Manual**，STM32F103 的 RM 是你最好的朋友（共 1096 页）。

---

## 🗓️ 更新日志

| 日期 | 内容 |
|------|------|
| 2025-12-08 | 项目创建，人生第一个 STM32 工程模板，PC13 LED 控制 |

---

## 🎓 作者

**WYC** — 大二在读，正在被单片机教做人。  
立志从亮灯开始，一路走向……还不知道哪里，但肯定比现在厉害。

> _"纸上得来终觉浅，绝知此事要烧板。"_

