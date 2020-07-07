---
title:           STM32CubeMX(四)配置GPIO之按键中断检测
description:     使用STM32CubeMX配置GPIO,以中断方式检测按键
author:          WZR
categories:
    - STM32CubeMX学习
tags:
    - STM32CubeMX
    - GPIO
    - 按键
    - EXTI
    - STM32
---

# 准备

- STM32CubeMX 5.6.1
- MDK V5.28.0
- STM32F103RCT6

# 配置

打开上一篇所配置的CubeMX工程，进入配置界面

## 中断配置

点击芯片视图中相应管脚，选择GPIO_EXTIx

![image-20200705160657259](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705160657.png)

在GPIO配置中配置模式及相应标签

WK_UP下拉，上升沿触发

KEY0，KEY1上拉，下降沿触发

![image-20200705162028116](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705162028.png)

## 中断优先级配置

------

STM32的中断优先级控制通过操作NVIC实现

NVIC为一个8位寄存器，理论上可配置2^8=256级中断，实际上只用了高4位配置，只有16级中断。

ST设置了5个优先级分组分配抢占优先级和子优先级位数：

|      优先级分组      | 抢占优先级位数 | 子优先级位数 |
| :------------------: | :------------: | :----------: |
| NVIC_PriorityGroup_0 |       0        |      4       |
| NVIC_PriorityGroup_1 |       1        |      3       |
| NVIC_PriorityGroup_2 |       2        |      2       |
| NVIC_PriorityGroup_3 |       3        |      1       |
| NVIC_PriorityGroup_4 |       4        |      0       |

默认使用NVIC_PriorityGroup_0

优先级判定规则

- 抢占优先级为主，数字越小，优先级越高
- 抢占优先级相同，判断子优先级，数字越小，优先级越高

------

设置优先级分组规则为NVIC_PriorityGroup_2  

设置3个按键中断的优先级

![image-20200705162905745](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705162906.png)

## 生成源代码

点击GENERATE CODE生成工程源代码，选择Open Project

# 修改用户代码

打开stm32f1xx_it.c

可以找到配置的三个中断的处理函数

```C
/**
  * @brief This function handles EXTI line0 interrupt.
  */
void EXTI0_IRQHandler(void)
{
  /* USER CODE BEGIN EXTI0_IRQn 0 */

  /* USER CODE END EXTI0_IRQn 0 */
  HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_0);
  /* USER CODE BEGIN EXTI0_IRQn 1 */

  /* USER CODE END EXTI0_IRQn 1 */
}

/**
  * @brief This function handles EXTI line[9:5] interrupts.
  */
void EXTI9_5_IRQHandler(void)
{
  /* USER CODE BEGIN EXTI9_5_IRQn 0 */

  /* USER CODE END EXTI9_5_IRQn 0 */
  HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_5);
  /* USER CODE BEGIN EXTI9_5_IRQn 1 */

  /* USER CODE END EXTI9_5_IRQn 1 */
}

/**
  * @brief This function handles EXTI line[15:10] interrupts.
  */
void EXTI15_10_IRQHandler(void)
{
  /* USER CODE BEGIN EXTI15_10_IRQn 0 */

  /* USER CODE END EXTI15_10_IRQn 0 */
  HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_15);
  /* USER CODE BEGIN EXTI15_10_IRQn 1 */

  /* USER CODE END EXTI15_10_IRQn 1 */
}
```

可以发现三个中断处理函数都调用了HAL_GPIO_EXTI_IRQHandler()这个函数，选中右键定位到函数定义处

![image-20200705165235458](https://gitee.com/wziru/BlogPicGo/raw/master/img/20200705165235.png)

跳转到stm32f1xx_hal_gpio.c

```cC
/**
  * @brief  This function handles EXTI interrupt request.
  * @param  GPIO_Pin: Specifies the pins connected EXTI line
  * @retval None
  */
void HAL_GPIO_EXTI_IRQHandler(uint16_t GPIO_Pin)
{
  /* EXTI line interrupt detected */
  if (__HAL_GPIO_EXTI_GET_IT(GPIO_Pin) != 0x00u)
  {
    __HAL_GPIO_EXTI_CLEAR_IT(GPIO_Pin);
    HAL_GPIO_EXTI_Callback(GPIO_Pin);
  }
}
```

该函数先确定中断是否发生，最后调用了HAL_GPIO_EXTI_Callback(GPIO_Pin)

再定位到这个函数

```c
/**
  * @brief  EXTI line detection callbacks.
  * @param  GPIO_Pin: Specifies the pins connected EXTI line
  * @retval None
  */
__weak void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
  /* Prevent unused argument(s) compilation warning */
  UNUSED(GPIO_Pin);
  /* NOTE: This function Should not be modified, when the callback is needed,
           the HAL_GPIO_EXTI_Callback could be implemented in the user file
   */
}
```

这是一个弱定义函数，我们可以重新定义这个函数，并且note注明了请重新在用户文件中实现该函数

## 实现各中断的回调函数

在gpio.c中实现回调函数

```c
/* USER CODE BEGIN 2 */
/**
  * @brief  EXTI line detection callbacks.
  * @param  GPIO_Pin: Specifies the pins connected EXTI line
  * @retval None
  */
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
	switch(GPIO_Pin)			//判断触发中断的Pin脚
	{
		case WK_UP_Pin:			//按下WK_UP键
			HAL_GPIO_WritePin(LED0_GPIO_Port,LED0_Pin,GPIO_PIN_RESET);	//LED0复位，亮起
			break;
		case KEY0_Pin:			//按下KEY0键
			HAL_GPIO_WritePin(LED0_GPIO_Port,LED0_Pin,GPIO_PIN_SET);	//LED0置位，熄灭
			break;
		case KEY1_Pin:			//按下KEY1键
			HAL_GPIO_TogglePin(LED0_GPIO_Port,LED0_Pin);				//LED0翻转电平
			break;
		default:
			break;
	}
}
/* USER CODE END 2 */
```

## mian函数修改

在Code2~Code3段修改代码如下

```c
  /* USER CODE BEGIN 2 */

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
	
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

