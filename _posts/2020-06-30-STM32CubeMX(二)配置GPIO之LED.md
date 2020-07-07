---
title:           STM32CubeMX(二)配置GPIO之LED
description:     使用STM32CubeMX配置GPIO点亮LED实现流水灯
author:          WZR
categories:
    - STM32CubeMX学习
tags:
    - STM32CubeMX
    - GPIO
    - STM32
---

# 准备

- STM32CubeMX 5.6.1
- MDK V5.28.0
- STM32F103RCT6

# 配置

新建工程，选择型号STM32F103RCTx，开始配置

## 系统Pin配置

### SYS配置

SW模式

![image-20200705134045442](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705134101.png)

### RCC配置

选择外部晶振

![image-20200705134227052](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705134227.png)

### GPIO配置

查看开发板上LED灯对应IO口

| LED  |  IO  |
| :--: | :--: |
| LED0 | PA8  |
| LED1 | PD2  |

点击配置界面中芯片视图的相应管脚，选择GPIO_Output

![image-20200705135621470](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705135621.png)

点击GPIO配置界面中出现的GPIO项，配置输出模式，速度，用户标签等

![image-20200705140501883](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705140502.png)

## 时钟配置

STM32F103RCT6主频最高72MHz，这里设置为72MHz

![image-20200705140920953](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705140921.png)

## 工程配置

### Project

设置工程名，路径，IDE为MDK……

![image-20200705141212158](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705141212.png)

### Code

选择添加必要的hal库，每个外设生成单独的.c/.h文件……

![image-20200705150511716](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705150512.png)

## 生成源代码

点击GENERATE CODE生成工程源代码，选择Open Project

# 添加用户代码

打开mian.c

找到mian函数，while(1)

在Code2与Code3段添加代码如下

```c
  /* USER CODE BEGIN 2 */
  HAL_GPIO_WritePin(LED0_GPIO_Port,LED0_Pin,GPIO_PIN_SET);
  HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
	HAL_GPIO_TogglePin(LED0_GPIO_Port,LED0_Pin);
	HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
	HAL_Delay(400);
  }
  /* USER CODE END 3 */
```
 点击编译，编译通过无Error

点击魔术棒，设置debug，选择所使用的工具，选择SW模式

![image-20200705142446631](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705142447.png)

# 烧录代码，观察效果

点击LOAD下载代码

观察到两个LED来回闪烁，实现流水灯效果

