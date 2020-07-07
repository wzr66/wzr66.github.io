---
title:           STM32CubeMX(三)配置GPIO之按键轮询检测
description:     使用STM32CubeMX配置GPIO，使用轮询方式检测按键
author:          WZR
categories:
    - STM32CubeMX学习
tags:
    - STM32CubeMX
    - GPIO
    - 按键
    - STM32
---

# 准备

- STM32CubeMX 5.6.1
- MDK V5.28.0
- STM32F103RCT6

# 配置

打开上一篇所配置的CubeMX工程，进入配置界面

## 系统Pin配置

查看开发板原理图，查找按键对应管脚

| 按键  |  IO  |
| :---: | :--: |
| WK_UP | PA0  |
| KEY0  | PC5  |
| KEY1  | PA15 |

点击芯片视图中相应管脚，选择GPIO_Input

![image-20200705145612746](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705145613.png)

在GPIO配置中配置模式及相应标签

![image-20200705153310788](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705153311.png)

## 生成源代码

点击GENERATE CODE生成工程源代码，选择Open Project

# 修改用户代码

打开mian.c

找到mian函数，while(1)

在Code2~Code3段修改代码如下

按键检测方法采用轮询方式并加入延时消抖

```c
  /* USER CODE BEGIN 2 */
  HAL_GPIO_WritePin(LED0_GPIO_Port,LED0_Pin,GPIO_PIN_SET);
  HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
	if(HAL_GPIO_ReadPin(WK_UP_GPIO_Port,WK_UP_Pin)==1)					//检测WK_UP是否按下
	{
		HAL_Delay(10);													//消抖
		if(HAL_GPIO_ReadPin(WK_UP_GPIO_Port,WK_UP_Pin)==1)
		{
			HAL_GPIO_WritePin(LED0_GPIO_Port,LED0_Pin,GPIO_PIN_RESET);	//LED0复位，亮起
			while(HAL_GPIO_ReadPin(WK_UP_GPIO_Port,WK_UP_Pin));	 		//检测按键是否松开
		}
	}
	else 
		if(HAL_GPIO_ReadPin(KEY0_GPIO_Port,KEY0_Pin)==0)				//检测KEY0是否按下
		{
			HAL_Delay(10);												//消抖
			if(HAL_GPIO_ReadPin(KEY0_GPIO_Port,KEY0_Pin)==0)
			{
				HAL_GPIO_WritePin(LED0_GPIO_Port,LED0_Pin,GPIO_PIN_SET);//LED0置位，熄灭
				while(!HAL_GPIO_ReadPin(KEY0_GPIO_Port,KEY0_Pin));	 	//检测按键是否松开
			}
		}
		else 
			if(HAL_GPIO_ReadPin(KEY1_GPIO_Port,KEY1_Pin)==0)			//检测KEY1是否按下
			{
				HAL_Delay(10);											//消抖
				if(HAL_GPIO_ReadPin(KEY1_GPIO_Port,KEY1_Pin)==0)
				{
					HAL_GPIO_TogglePin(LED0_GPIO_Port,LED0_Pin);		//LED0翻转电平
					while(!HAL_GPIO_ReadPin(KEY1_GPIO_Port,KEY1_Pin));	//检测按键是否松开
				}
			}
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
	HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);						//翻转LED1
	HAL_Delay(100);
  }
  /* USER CODE END 3 */
```
 点击编译，编译通过无Error

# 烧录代码，观察效果

点击LOAD下载代码

观察到LED1闪烁，

按下WK_UP，LED0亮；按下KEY0，LED0灭；按下KEY1，LED0翻转

