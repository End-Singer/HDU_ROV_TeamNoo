# 🚀 STM32 SGA Demo 工程 —— 三分钟搞懂结构，十分钟驱动你的电机

**基于 STM32F1 / STM32L4，Keil uVision5 开发环境**

----------

# 1. 工程总体结构简介（务必理解）

根据 README 的描述，SGA 工程分为三层：

```arduino
Apply（应用层）
│── Logic（用户上层逻辑）
│── Task  （用户外设初始化、中断等）
BSP（板级支持包）
Driver（硬件驱动封装）
Hardware（HAL 底层库）
```

其中你真正要改的只有：

-   **Apply/Logic/usercode.c** —— 写你的整体逻辑
    
-   **Apply/Task/task_userinit.c** —— 初始化外设（PWM、串口、电机接口）
    
-   **Apply/Logic/config.c** —— 配置你所用的 GPIO/TIM/UART 句柄
    

Driver 与 BSP 里已经封装了 HAL 驱动，不建议改。

----------

# 2. DEMO 工程里 main() 的执行流程

来自 `main.c`：

```c
Task_Sys_Init();   // 初始化 HAL & SGA 驱动库（必须）
Task_UserInit();   // 用户外设初始化（你应该在这里添加 PWM 初始化）
UserLogic_Code();  // 主逻辑循环（你的控制逻辑写这里）
```

因此你要驱动电机，必须：

1.  **在 config.c 中定义一个 PWM 句柄（TIM + 通道）**
    
2.  **在 Task_UserInit() 中调用驱动初始化函数**
    
3.  **在 UserLogic_Code() 中写循环并输出占空比**
    

下面分别展开。

----------

# 3. PWM 电机控制——你要知道哪里写什么

SGA 的 PWM 驱动属于 Driver 层，使用非常简单（类似 HAL）：

-   初始化：`Drv_PWM_Init()`
    
-   设置占空比：`Drv_PWM_SetDuty()`
    

只需搞定 3 个文件：
| 文件位置 |作用|
|--|--|
|Apply/Logic/config.c |写 PWM 因子：定时器、通道、频率、GPIO 配置|
|Apply/Task/task_userinit.c|调用 PWM 初始化函数|
|Apply/Logic/usercode.c|设置 PWM 占空比，驱动电机|

下面给你一个可以直接复制的模板。
----------

# 4. 第一步：在 config.c 中添加 PWM 句柄

在你的 `config.c` 最末尾，仿照已有的 GPIO、UART 示例，加一个 PWM 配置。

假设你的电机驱动板使用：

-   TIM3 CH1 输出 PWM
    
-   对应引脚 PA6（F103 默认）
    

可添加如下句柄（你可以改成你需要的定时器/引脚）：

```c
/* PWM 电机驱动示例（TIM3 CH1） */
tagPWM_T MotorPWM =
{
    .tPWMHandle.Instance = TIM3,        // 使用 TIM3
    .tPWMHandle.Init.Prescaler = 72-1,  // 72MHz / 1MHz
    .tPWMHandle.Init.Period = 20000-1,  // PWM 频率 50Hz（舵机），或换成更高频率控制电机
    .ucChannel = TIM_CHANNEL_1,         // 通道 1
    .tGPIO.tGPIOInit.Pin   = GPIO_PIN_6,
    .tGPIO.tGPIOInit.Mode  = GPIO_MODE_AF_PP,
    .tGPIO.tGPIOInit.Pull  = GPIO_NOPULL,
    .tGPIO.tGPIOInit.Speed = GPIO_SPEED_FREQ_HIGH,
    .tGPIO.tGPIOPort       = GPIOA,
};

```

如果你是驱动 **直流电机（PWM + 方向控制）**，Period 可以设为 **20 kHz** 左右，例如：

```c
.tPWMHandle.Init.Period = 2000-1;   // 1 MHz / 2000 = 500 Hz
```

可以根据你的电调/驱动器自己改。

----------

# 5. 第二步：在 Task_UserInit() 中初始化 PWM

`Apply/Task/task_userinit.c`：

```c
void Task_UserInit(void)
{
    // 初始化 PWM
    Drv_PWM_Init(&MotorPWM);

    // 其他外设初始化
}
```

这样 TIM3 就会自动启动 PWM 输出。

----------

# 6. 第三步：在 usercode.c 中控制电机 PWM

`Apply/Logic/usercode.c`：

```c
extern tagPWM_T MotorPWM;

/* 用户逻辑代码 */
void UserLogic_Code(void)
{
    printf("SGA_DEMO: Motor PWM Test\r\n");

    while(1)
    {
        // 电机加速
        for(int duty = 0; duty < 1000; duty += 10)
        {
            Drv_PWM_SetDuty(&MotorPWM, duty); // duty = 0~Period
            Drv_Delay_Ms(10);
        }

        // 电机减速
        for(int duty = 1000; duty > 0; duty -= 10)
        {
            Drv_PWM_SetDuty(&MotorPWM, duty);
            Drv_Delay_Ms(10);
        }
    }
}
```

如果 Period 是 2000-1，则 duty 区间就是 `0 ~ 1999`。

----------

# 7. 如何扩展到多个推进器（水下机器人）

水下机器人一般 4~6 个推进器（PWM 输出）。

你可以像上面一样，复制多个句柄：

`tagPWM_T MotorPWM[4];`

每个配置不同的：

-   TIMx   
-   CHx 
-   GPIO 引脚
    

例如：
| 推进器 | 定时器 | 通道 | 引脚 |
|--|--|--|--|
| 前左 | TIM3 | CH1 | PA6 |
| 前右 | TIM3 | CH2 | PA7 |
| 后左 | TIM2 | CH1 | PA0 |
| 后右 | TIM2 | CH2 | PA1 |

然后：

### config.c

定义多个 PWM。

### task_userinit.c

```c
Drv_PWM_Init(&MotorPWM[0]);
Drv_PWM_Init(&MotorPWM[1]);
Drv_PWM_Init(&MotorPWM[2]);
Drv_PWM_Init(&MotorPWM[3]);
```

### usercode.c

写你的混控：

```c
Drv_PWM_SetDuty(&MotorPWM[0], FL);
Drv_PWM_SetDuty(&MotorPWM[1], FR);
Drv_PWM_SetDuty(&MotorPWM[2], BL);
Drv_PWM_SetDuty(&MotorPWM[3], BR);
```

这样你就能写出推进器控制逻辑，例如：

-   前进
-   后退  
-   旋转   
-   上浮 / 下潜（通过垂直推进器）
    

----------

# 8. 加入手柄或传感器（可选）

Demo 中还包含：

-   **PS2 手柄驱动**（Dev_PS2）
    
-   **姿态传感器 JY901**
    
-   **压力传感器 MS5837**
    

你可以在 usercode 中写：

```c
uint8_t key = Dev_PS2_DataKey(&myPs2);
uint8_t ly = Dev_PS2_AnologData(PSS_LY);
```

然后将摇杆数据映射到电机 PWM，做出“遥控版水下机器人”。

如果需要，我还能帮你写完整的**遥控控制代码模板**。

----------

# 9. 总结：你的电机驱动主线就是三步

|步骤|文件|内容|
|--|--|--|
|1|config.c|添加 PWM 句柄、GPIO、TIM 配置|
|2|task_userinit.c|调用 PWM 初始化函数|
|3|usercode.c|写你的控制逻辑，通过设置 PWM 占空比驱动电机|

你完全可以基于 Demo 工程，快速扩展成完整的水下机器人运动控制程序。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2OTY4NDE1MTksLTE4MTYzNDk2NjNdfQ
==
-->