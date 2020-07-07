---
title:           STM32CubeMX(一)初识CubeMX
description:     STM32CubeMX了解及配置流程
author:          WZR
categories:
    - STM32CubeMX学习
tags:
    - STM32CubeMX
    - STM32
---

# STM32CubeMX介绍

STM32CubeMX 是一种图形化工具，它允许通过分步过程对 STM32 微控制器和微处理器进行非常简单的配置，以及生成相应的 ARM Cortex-M 内核初始化 C 代码或用于 Cortex-A 内核的部分 Linux 设备树。

## 主要特点

- 直观的 STM32 微控制器和微处理器选择
- 丰富易于使用的图形用户界面，可配置：
  - 具有自动冲突解决的引脚
  - 外设和中间件功能模式，动态验证 Arm Cortex-M 内核的参数约束
  - 具有配置动态验证的时钟树
  - 具有估计功耗结果的功率序列
- 生成初始化 C 代码项目，符合 IAR™、Keil 和 GCC 编译器，用于 Arm Cortex-M 内核
- 为 Cortex-A 内核生成部分 Linux 设备树 （STM32 微处理器）
- 作为在 Windows、Linux 和 macOS 上运行的独立软件

# STM32CubeMX配置流程

## CubeMX配置

- ## 新建工程，选择芯片或开发板型号

  打开CubeMX，点击File–>New Project

  搜索芯片型号，点击Start Project进入配置

  ![image-20200704180850158](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200704180905.png)

- ## Pinout & Configuration（Pin脚初始化）
  
  配置所需外设及相应IO
  
  ![image-20200704181137590](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200704181137.png)
  
  - ### System Core（系统）
  
    配置DMA，GPIO，IWDG，NVIC，RCC，SYS，WWDG
  
    一般只配置两项：SYS及RCC
  
    SYS配置调试模式，
  
    - 一般选Serial Wire，即常用的SW模式
  
    ![image-20200704202034125](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200704202034.png)
  
    RCC配置时钟源，
  
    - 如果选择使用外部高速时钟（HSE），则需要在System Core中配置RCC；
    - 如果使用默认内部时钟（HSI），这一步可以略过；
  
  - ### Analog（模拟）
  
    配置ADC，DAC
  
  - ### Timers(定时器)
  
    配置RTC及TIM定时器
  
  - ### Connectivity（通信)
  
    配置CAN，I2C，SDIO，SPI，UART，USART，USB等通讯协议外设
  
  - ### Multimedia（音频）
  
    配置I2S等外设
  
  - ### Computing（计算）
  
    配置CRC
  
  - ### Middleware（中间件）
  
    配置FATFS（文件系统），FreeRTOS（操作系统），USB_DEVICE等ST提供的中间件
  
- ## Clock Configuration（时钟初始化）

  配置单片机时钟树，一般将HCLK配置为最大频率72MHz

  - HSE，LSE为外部高速、低速时钟源，即通常所用的8MHz与32.768kHz晶振所提供的时钟信号
  - HSI RC，LSI RC为内部时钟源

  ![image-20200704213602763](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200704213603.png)

- ## Project Manager（工程管理）

  - ### Project

    配置工程名，路径，选择生成工程的IDE等工程相关设置

    ![image-20200704215154872](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200704215155.png)

  - ### Code Generator

    hal库设置，生成文件设置等

    ![image-20200704215408408](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200704215408.png)

  - Advanced Settings

- Tools

- ## 生成工程代码

  点击右上角GENERATE CODE生成工程源代吗，弹出窗口，选择直接用IDE打开工程，开始开发

## 修改源代码

在用户代码注释“Begin”与“End”中间加入用户代码

```c
	/* USER CODE BEGIN WHILE */
	while (1)
  	{
    	/* USER CODE END WHILE */

    	/* USER CODE BEGIN 3 */
  	}
  	/* USER CODE END 3 */
```

## 增加外设

如果之后要添加一些外设，返回STM32CubeMX界面进行配置外设，然后在Project Manager的Code Generator中确保选择当重新生成源代码时保持用户代码，重新生成工程源代码，再次进入IDE修改源代码完善工程。
