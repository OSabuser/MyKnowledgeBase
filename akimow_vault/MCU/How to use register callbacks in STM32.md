---
title: "How to use register callbacks in STM32"
source: "https://community.st.com/t5/stm32-mcus/how-to-use-register-callbacks-in-stm32/ta-p/580499"
author:
published: 2023-08-11
created: 2025-03-13
description: "Introduction In this article, we cover the steps needed to use the Register callback's feature in STM32. The NUCLEO-H503RB (with an STM32H503RBT6"
tags:
  - "clippings"
---

**Introduction**

In this article, we cover the steps needed to use the Register callback's feature in STM32. The **NUCLEO-H503RB** (with an **STM32H503RBT6** microcontroller) board is used, but the steps can be easily tailored to another MCU. All the implementation was done over the **STM32CubeIDE** **v1.13.1** but can be implemented in any other version with minor step changes.

![Figure 1 – NUCLEO-H503RB Source: Adapted from https://www.st.com/en/evaluation-tools/nucleo-h503rb.html](https://community.st.com/t5/image/serverpage/image-id/52103i27B5354C234A13D4/image-size/medium/is-moderation-mode/true?v=v2&px=400 "BMontanari_0-1691433051363.jpeg")Figure 1 – NUCLEO-H503RB Source: Adapted from https://www.st.com/en/evaluation-tools/nucleo-h503rb.html

The **Register** **callback** feature allows you to register a custom callback for interruptions from a specific peripheral. This means you can use a custom function as the callback for any desired peripheral interrupt source in your project.  

In this demonstration, we use the STLINK VCOM Port connected to the MCU's USART3 (figure 2). Furthermore, we use the TIM2 and the board’s user LED (figure 3). Two callbacks are registered: one for the UART\_Tx and the other one for the timer overflow, also known as period elapsed interrupt. By the end of this article, you should have the necessary knowledge to do your own Register callback implementations in any kind of application. So, grab your coffee, settle into your chair, and let us start coding!

![Figure 2 – Connection between MCU’s USART3 and ST-LINK's VCOM Port](https://community.st.com/t5/image/serverpage/image-id/52105i83213FA63ED39F71/image-size/medium/is-moderation-mode/true?v=v2&px=400 "BMontanari_1-1691433051372.png")Figure 2 – Connection between MCU’s USART3 and ST-LINK's VCOM Port

Source: Adapted from [https://www.st.com/content/ccc/resource/technical/layouts\_and\_diagrams/schematic\_pack/group2/94/07/0d/de/30/11/4c/3b/mb-1814-h503rb-b02-schematic/files/mb1814-h503rb-b02-schematic.pdf/jcr:content/translations/en.mb1814-h503rb-b02-schematic.pdf](https://www.st.com/content/ccc/resource/technical/layouts_and_diagrams/schematic_pack/group2/94/07/0d/de/30/11/4c/3b/mb-1814-h503rb-b02-schematic/files/mb1814-h503rb-b02-schematic.pdf/jcr:content/translations/en.mb1814-h503rb-b02-schematic.pdf)

![Figure 3 – User’s LED Connection](https://community.st.com/t5/image/serverpage/image-id/52104i732C2C89BBC24E2E/image-size/medium/is-moderation-mode/true?v=v2&px=400 "BMontanari_2-1691433051374.png")Figure 3 – User’s LED Connection

Source: Adapted from [https://www.st.com/content/ccc/resource/technical/layouts\_and\_diagrams/schematic\_pack/group2/94/07/0d/de/30/11/4c/3b/mb-1814-h503rb-b02-schematic/files/mb1814-h503rb-b02-schematic.pdf/jcr:content/translations/en.mb1814-h503rb-b02-schematic.pdf](https://www.st.com/content/ccc/resource/technical/layouts_and_diagrams/schematic_pack/group2/94/07/0d/de/30/11/4c/3b/mb-1814-h503rb-b02-schematic/files/mb1814-h503rb-b02-schematic.pdf/jcr:content/translations/en.mb1814-h503rb-b02-schematic.pdf)

## **1\. Development**

Create a project for the **STM32H503RB** using the **STM32CubeIDE**. In this project, the following settings are applied:

- USART3: 115200 bits/s - 8N1:
- PA3 as UART\_RX.
- PA4 as UART\_TX.
- PA5 as GPIO Output with USER\_LED label.
- TIM2 as 1 second time base:
- Internal clock as source.
- Prescaler as 24999
- Counter period as 9999
- Autoreload preload enabled.
- NVIC: Enable both USART3’s and TIM2’s ISRs.

![Figure 4 – MCU Project Settings. Source: Author’s screenshot.](https://community.st.com/t5/image/serverpage/image-id/52108iBE11527633DCBA4F/image-size/medium/is-moderation-mode/true?v=v2&px=400 "BMontanari_3-1691433051411.png")Figure 4 – MCU Project Settings. Source: Author’s screenshot.

The next changes are made at the **clock configuration** tab. In this tab, the **HCLK** frequency needs to be changed to 250 MHz (this frequency matches the TIM2 settings for the 1 second time base).

![Figure 5 – Clock configuration settings. Source: Author’s screenshot.](https://community.st.com/t5/image/serverpage/image-id/52106i88D09EE7CF6B9A5B/image-size/medium/is-moderation-mode/true?v=v2&px=400 "BMontanari_4-1691433051417.png")Figure 5 – Clock configuration settings. Source: Author’s screenshot.

The last step, before the code generation, is to enable the **register callback**. For that, the **UART** and **TIM** register callbacks should be enabled under **project manager -> advanced settings** as shown in figure 6.

![Figure 6 – Enabling the Register CallBack for UART and TIM. Source: Author’s screenshot](https://community.st.com/t5/image/serverpage/image-id/52107i78F7C84A74F2995E/image-size/medium/is-moderation-mode/true?v=v2&px=400 "BMontanari_5-1691433051422.png")Figure 6 – Enabling the Register CallBack for UART and TIM. Source: Author’s screenshot

Now, that all the settings are done, the code can be generated by clicking on the respective button or just pressing the shortcut **alt + k**.

In the **main.c** file, we should register the callback functions for the peripherals we want. To apply it for the UART and TIM, the following functions should be called:

```c
HAL_StatusTypeDef HAL_UART_RegisterCallback(UART_HandleTypeDef *huart, HAL_UART_CallbackIDTypeDef CallbackID, pUART_CallbackTypeDef pCallback)
```

Also: 

```c
HAL_StatusTypeDef HAL_TIM_RegisterCallback(TIM_HandleTypeDef *htim, HAL_TIM_CallbackIDTypeDef CallbackID, pTIM_CallbackTypeDef pCallback)
```

Note that we have some parameters to fill. The first one is a pointer to the peripheral handler, &huart3 and &htim2 for the UART3 and TIM2 respectively. The second parameter is a CallBack ID, which points to which the interrupt source calls the given CallBack. A list of these IDs can be found in the respective peripheral HAL driver headers. See figures 7 and 8.

![Figure 7 – List of UART CallBack IDs. Source: Author’s screenshot.](https://community.st.com/t5/image/serverpage/image-id/52110iEDB1033C232883E7/image-size/medium/is-moderation-mode/true?v=v2&px=400 "BMontanari_6-1691433051451.png")Figure 7 – List of UART CallBack IDs. Source: Author’s screenshot.

![Figure 8 – List of TIM CallBack IDs. Source: Author’s screenshot.](https://community.st.com/t5/image/serverpage/image-id/52111i234661D298B3BCCF/image-size/medium/is-moderation-mode/true?v=v2&px=400 "BMontanari_7-1691433051482.png")Figure 8 – List of TIM CallBack IDs. Source: Author’s screenshot.

Finally, the last parameter is a pointer to your custom CallBack function. Just make sure that your function respects the same structure as the HAL excepts. You can find the correct structure in the p\[Peripheral\]\_CallbackTypeDef (like pUART\_CallbackTypeDef). See the examples below:

![Figure 9 – UART Callback structure. Source: Author’s screenshot.](https://community.st.com/t5/image/serverpage/image-id/52109i89A563996AB14FEA/image-size/medium/is-moderation-mode/true?v=v2&px=400 "BMontanari_8-1691433051486.png")Figure 9 – UART Callback structure. Source: Author’s screenshot.

![Figure 10 – TIM Callback structure. Source: Author’s screenshot.](https://community.st.com/t5/image/serverpage/image-id/52112iB5196746A3D43FE7/image-size/medium/is-moderation-mode/true?v=v2&px=400 "BMontanari_9-1691433051490.png")Figure 10 – TIM Callback structure. Source: Author’s screenshot.

Now, that all the preparations for using the Register callback feature are ready, let us dive into an example using that.

          Going back to the **main.c** file, define the following variables:

```c
/* USER CODE BEGIN PV */
uint8_t message[16] = "Period Elapsed\n\r";
FlagStatus periodElapsed = RESET;
/* USER CODE END PV */
```

The message array stores the message that is sent through the UART. The periodElapsed variable is used to control the application flow based on timer interruptions.

Add the function’s prototypes for each custom callback (one for the UART and another for the timer). You can select any name you want, just make sure to follow the function’s arguments and return method as mentioned before.

```c
/* USER CODE BEGIN PFP */
void User_UartCompleteCallback(UART_HandleTypeDef *huart);
void User_TIMPeriodElapsedCallback(TIM_HandleTypeDef *htim);
/* USER CODE END PFP */
```

In the **main** function, we need to associate the callback with the interrupt source by using the Register callback function. This should be done after the MX peripheral initialization, please rely on the USER CODE comment label as a reference.

```cpp
/* USER CODE BEGIN 2 */
  HAL_UART_RegisterCallback(&huart3, HAL_UART_TX_COMPLETE_CB_ID, User_UartCompleteCallback);
  HAL_TIM_RegisterCallback(&htim2, HAL_TIM_PERIOD_ELAPSED_CB_ID, User_TIMPeriodElapsedCallback);
  HAL_TIM_Base_Start_IT(&htim2);
  /* USER CODE END 2 */
```

Note the **HAL\_UART\_TX\_COMPLETE\_CB\_ID** was selected for the UART, to link the **TX complete Interrupt** to the custom function. Additionally, the **HAL\_TIM\_PERIOD\_ELAPSED\_CB\_ID**, for the t**imer period elapsed** **Interrupt**.

After registering the callbacks, the code called a function to start timer 2 as a time base, in interruption mode. This will generate a period elapsed Interrupt every 1 second, as previously set.

In the endless loop, add the following code, which is responsible for transmitting the predefined message by pooling the periodElapsed flag:

```c
/* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
        if(periodElapsed == SET)
        {
               HAL_UART_Transmit_IT(&huart3, message, sizeof(message));
               periodElapsed = RESET;
        }
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
```

Finally, add your custom callback functions code, again, use the USER CODE comment to locate the proper place to add the code snippet.

```c
/* USER CODE BEGIN 4 */
void User_UartCompleteCallback(UART_HandleTypeDef *huart)
{
      HAL_GPIO_TogglePin(USER_LED_GPIO_Port, USER_LED_Pin);
}

void User_TIMPeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
      periodElapsed = SET;
}
/* USER CODE END 4 */
```

Now that you have completed all the steps to implement the Register callback feature, you can build the code, flash it onto your microcontroller, open a terminal and see the application in action.

## **2\. How does this demonstration work?**

This application utilizes timer 2 to generate a 1-second time base, resulting in an interruption being generated every second. This interrupt calls the custom period elapsed callback function, which sets the periodElapsed flag. This flag is used in the infinite while loop in the main function. Once the periodElapsed flag is set, the code within the main loop transmits the defined message through the UART in interruption mode (non-blocking). After the message is queued to be sent, the periodElapsed flag is reset. Finally, once the message transmission is complete, a Tx complete callback is generated, calling the UART custom callback function, which toggles the LED status.

In summary, this application sends a message through the UART every second. The application changes the LED status once the message is fully sent, utilizing the custom callback functions that were created.

![Figure 11 – Demonstration Results in the STM32CubeIDE Serial Terminal.Source: Author’s screenshot.](https://community.st.com/t5/image/serverpage/image-id/52113i37E62B2CA9762279/image-size/medium/is-moderation-mode/true?v=v2&px=400 "BMontanari_10-1691433051495.png")Figure 11 – Demonstration Results in the STM32CubeIDE Serial Terminal.Source: Author’s screenshot.

## **Conclusion**

With this example you can create custom callback Functions and register it as shown above. This can be done for all peripherals which are capable of being managed by the STM32 HAL.

**We hop****e this content was helpful for you!**

**Best wishes for your future developments.**

## **Related links**

[STM32H503RB - High-performance, Arm Cortex-M33, MCU with 128-Kbyte Flash, 32-Kbyte RAM, 250 MHz CPU - STMicroelectronics](https://www.st.com/en/microcontrollers-microprocessors/stm32h503rb.html)

[Description of STM32H5 HAL and low-layer drivers - User manual](https://www.st.com/content/ccc/resource/technical/document/user_manual/group2/cb/9c/c5/f7/e5/1c/48/fa/DM00948527/files/DM00948527.pdf/jcr:content/translations/en.DM00948527.pdf)

[NUCLEO-H503 Schematic](https://www.st.com/content/ccc/resource/technical/layouts_and_diagrams/schematic_pack/group2/94/07/0d/de/30/11/4c/3b/mb-1814-h503rb-b02-schematic/files/mb1814-h503rb-b02-schematic.pdf/jcr:content/translations/en.mb1814-h503rb-b02-schematic.pdf)

[STM32CubeIDE - integrated development environment for STM32 - STMicroelectronics](https://www.st.com/en/development-tools/stm32cubeide.html)



