## 中断系统
中断允许CPU暂停当前的任务，去处理更紧急的临时事件，处理完后还能无缝衔接回来。
### 中断优先级
STM32的中断优先级由一个名为 **NVIC** 的模块管理。你可以给每个中断源分配一个优先级数字。**数字越小，优先级越高**。通过配置NVIC，你就能精确控制当多个中断同时发生时，谁先被处理，以及谁能打断谁。
### 中断嵌套
STM32的中断优先级通常被分为**抢占优先级**和**响应优先级**。

-   **抢占优先级：** 决定了中断之间是否能相互嵌套。高抢占优先级可以打断低抢占优先级。（**强行打断**）
- **响应优先级：** 当两个或多个中断的抢占优先级相同，并且它们同时到来时，CPU先响应哪一个的顺序。（**插队**）

### NVIC基本结构
![NVIC结构](/imgs/2025-11-06/tGB2ePkferiEYmui.png)

-每个中断通道有16个可编辑优先级（16个等级），用4个二进制数可以表示0~15的十进制数，这个十进制数对应16个优先级
-该十进制数的数值越小，优先级越高
-4个二进制数可以进行切分，5种切分方法，如下图：

![优先级分组](/imgs/2025-11-06/p5xZfAbHETtKZAdO.png)
##EXTI 外部中断
- EXTI可以监测指定GPIO口的电平信号，当指定GPIO产生电平变化时，EXTI向NVIC发出中断申请，NVIC裁决后中断CPU主程，使CPU执行EXTI对应的中断程序。
#### 触发方式：
- 上升沿（低电平到高电平瞬间）
- 下降沿（高电平到低电平瞬间）
- 双边沿（电平高低上升下降瞬间均可）
- 软件触发（程序执行代码）
---
- 支持所有GPIO口，**但相同Pin不能同时触发中断**
- 通道数：16个GPIO_Pin + PVD输出、RTC闹钟、USB唤醒、以太网唤醒
- 触发响应方式：中断响应（CPU执行）、事件响应（无需CPU参与，其他外设响应）
- EXTI基本结构
![输入图片说明](/imgs/2025-11-06/N6joMw5pmdmXepn4.png)
### AFIO 超级接线板
#### AFIO 的三大核心功能
-   **GPIO引脚复用功能选择**：同一外设的信号可映射到多个不同引脚，AFIO用于精确配置具体由哪个引脚承载该复用功能信号。
    
-   **外部中断线配置**：每条EXTI中断线一次只能连接一个GPIO引脚，通过配置AFIO的EXTICR寄存器来选择具体的信号源引脚。
    
-   **调试端口重新映射**：当默认调试引脚被占用时，可通过AFIO将SWJ调试接口功能切换到备用引脚组。
### 需要用到外部中断的硬件外设
如旋转编码器输出信号、红外遥控接收头的输出等
#### 旋转编码器
有A相、B相两个输出
作用是为了	
1. **判断方向**  AB相输出的方波有相位差，可以辨别旋转方向
2. **倍频**  提高精度
3. **消抖** 消除机械触点抖动产生的毛刺信号
### 外部中断的配置流程
参考**EXTI基本结构**
1. 配置RCC，把涉及的外设时钟都打开
2. 配置GPIO，选择端口为输入模式
3. 配置AFIO，选择用的GPIO连接到EXTI
4. 配置EXTI，选择触发方式和响应方式
5. 配置NVIC，给中断选择一个合适的优先级
## 外部中断配置流程
1. **时钟使能配置**
2. **GPIO引脚配置**
3. **EXTI引脚映射配置**
4. **外部中断(EXTI)配置**
5. **NVIC中断优先级分组配置**
6. **NVIC中断控制器配置**
7. **编写中断服务函数**
```c
// 计数器变量：用于存储计数传感器的计数值
uint16_t CountSenSor_Count;

// 计数传感器初始化函数
void CountSensor_Init(void){
    // 使能GPIOB时钟，因为传感器引脚连接到GPIOB
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE); // GPIOB是APB2的外设
    // 使能AFIO(复用功能IO)时钟，用于配置外部中断引脚映射
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE); // AFIO也是APB2的外设
    // EXTI和NVIC的时钟是一直打开的不用开启
    
    // 配置GPIO引脚
    GPIO_InitTypeDef GPIO_InitStructure;
    // 设置引脚为上拉输入模式，默认高电平，当传感器触发时拉低
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    // 设置引脚速度
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    // 选择PB14引脚作为计数传感器输入
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_14;
    // 初始化GPIOB的PB14引脚
    GPIO_Init(GPIOB, &GPIO_InitStructure);
    
    // 将PB14引脚连接到EXTI14外部中断线
    GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource14);
    
    // 配置外部中断(EXTI)
    EXTI_InitTypeDef EXTI_InitStructure;
    // 选择外部中断线14（对应PB14引脚）
    EXTI_InitStructure.EXTI_Line = EXTI_Line14;
    // 使能外部中断线
    EXTI_InitStructure.EXTI_LineCmd = ENABLE;
    // 设置为中断模式（区别于事件模式）
    EXTI_InitStructure.EXTI_Mode = EXTI_Mode_Interrupt;
    // 设置为下降沿触发（从高电平变为低电平时触发中断）
    EXTI_InitStructure.EXTI_Trigger = EXTI_Trigger_Falling;
    // 应用EXTI配置
    EXTI_Init(&EXTI_InitStructure);
    
    // 设置NVIC中断优先级分组为组2
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    
    // 配置NVIC中断控制器
    NVIC_InitTypeDef NVIC_InitStructure;
    // 选择EXTI15~10中断通道（PB14属于这个中断向量）
    NVIC_InitStructure.NVIC_IRQChannel = EXTI15_10_IRQn;
    // 使能该中断通道
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    // 设置抢占优先级为1
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
    // 设置响应优先级为1
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
    // 应用NVIC配置
    NVIC_Init(&NVIC_InitStructure);
}

// 获取当前计数值函数
uint16_t CountSenSor_Get(void){
    // 返回当前的计数值
    return CountSenSor_Count;
}

// EXTI15~10中断服务函数（处理PB10~PB15引脚的外部中断）
void EXTI15_10_IRQHandler(void){
    // 检查是否是EXTI14线产生的中断（即PB14引脚）
    if (EXTI_GetITStatus(EXTI_Line14) == SET){
        // 消抖处理：再次读取引脚电平，确认确实是低电平
        if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_14) == 0)
        {
            // 确认有效触发，计数值自增一次
            CountSenSor_Count ++;
        }
        // 清除EXTI14线的中断挂起位，否则会持续进入中断
        EXTI_ClearITPendingBit(EXTI_Line14);
    }
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzM5NzA1MzEwXX0=
-->