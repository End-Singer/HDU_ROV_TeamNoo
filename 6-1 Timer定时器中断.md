# 定时器
定时器可以对输入的时钟进行计数
## 定时器类型
![输入图片说明](/imgs/2025-11-09/YOpdj7TgW2I9QMnj.png)
不同32型号定时器资源不同，eg：f103只有tim1、2、3、4
### 基本定时器
- **预分频器PSV**：PSV的值与分频系数差1，比如0实际是除1，1是除2
- **自动重载寄存器**：存目标值的
- **更新中断**：当PSV=自动重载值，触发中断叫更新中断，还产生一个更新事件，接着清零计数器，中断接NVIC
- **主模式触发DAC**：就是硬件自动每隔一段时间触发一次DAC输出一段波形，不用软件参与硬件自动化，节省中断和CPU资源
---
三种计次方式
>向上自增：增到目标值，中断并清零
>向下自减：从目标值减少，减到0中断并重置目标值
>中央对齐：向上自增与向下自减交替
>基本定时器只支持向上自增；通用和高级支持三种方式
## 定时中断基本结构
![输入图片说明](/imgs/2025-11-09/6JmeCfi2A7HcnS0M.png)
## 定时器中断配置流程
1. **使能定时器时钟**
2. #### **选择时钟源**
3. #### **配置时基参数（核心配置）**
> **定时时间计算公式：定时时间 = (Prescaler + 1) × (Period + 1) / TIMx_CLK**
> 定时时间（秒） = (预分频器值 + 1) × (周期值 + 1) ÷ 定时器时钟频率(Hz)
4. **清除中断标志**
5. **使能定时器中断**
6. **配置NVIC优先级分组**
7. **配置NVIC中断通道**
8. **使能定时器**
```c
// 定时器初始化函数
void Timer_Init(void){
    // 1. 使能定时器时钟
    // TIM2属于APB1总线上的外设，需要先使能其时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);
    
    // 2. 配置定时器时钟源
    // 选择内部时钟作为定时器时钟源（默认选择）
    TIM_InternalClockConfig(TIM2);
    
    // 3. 配置定时器时基参数
    TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;
    // 3.1 设置时钟分频（用于数字滤波器，DIV1表示不分频）
    TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
    // 3.2 设置计数器计数模式：向上计数模式
    TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;
    // 3.3 设置自动重装载值（周期值）
    // 计数到10000-1后产生更新事件，然后从0重新开始计数
    TIM_TimeBaseInitStructure.TIM_Period = 10000 - 1;
    // 3.4 设置预分频器值
    // 对定时器时钟进行7200分频，用于调整定时周期
    TIM_TimeBaseInitStructure.TIM_Prescaler = 7200 - 1;
    // 3.5 设置重复计数器值（高级定时器使用，基本定时器设为0）
    TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
    // 应用时基配置到TIM2定时器
    TIM_TimeBaseInit(TIM2, &TIM_TimeBaseInitStructure);
    
    // 4. 清除定时器更新标志位（避免首次使能就立即进入中断）
    TIM_ClearFlag(TIM2, TIM_FLAG_Update);
    
    // 5. 使能定时器更新中断
    // 当计数器溢出时产生更新事件，触发中断
    TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);
    
    // 6. 配置NVIC中断优先级分组
    // 设置系统中断优先级分组为组2（2位抢占优先级，2位响应优先级）
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    
    // 7. 配置NVIC中断控制器
    NVIC_InitTypeDef NVIC_InitStructure;
    // 7.1 选择TIM2中断通道
    NVIC_InitStructure.NVIC_IRQChannel = TIM2_IRQn;
    // 7.2 设置抢占优先级为2
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 2;
    // 7.3 设置响应优先级为1
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
    // 7.4 使能该中断通道
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    // 应用NVIC配置
    NVIC_Init(&NVIC_InitStructure);
    
    // 8. 使能定时器
    // 启动TIM2定时器，计数器开始计数
    TIM_Cmd(TIM2, ENABLE);
}

// TIM2中断服务函数模板
void TIM2_IRQHandler(void){
    // 检查是否是TIM2的更新中断
    if(TIM_GetITStatus(TIM2, TIM_IT_Update) == SET){
        // 用户中断处理代码写在这里
        
        // 清除TIM2更新中断标志位（必须清除，否则会持续进入中断）
        TIM_ClearITPendingBit(TIM2, TIM_IT_Update);
    }
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQwMTMzMTIzN119
-->
