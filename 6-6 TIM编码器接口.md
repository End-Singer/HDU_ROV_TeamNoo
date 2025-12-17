## 编码器接口简介
![输入图片说明](/imgs/2025-11-15/PZFVeBjZ8S73BIEm.png)
## 正交编码器
![输入图片说明](/imgs/2025-11-15/yBrap96jtiXSeyfq.png)
## 编码器接口基本结构
![输入图片说明](/imgs/2025-11-17/K7ND9HpD9xha02mv.png)
## 工作模式
![输入图片说明](/imgs/2025-11-17/XlJntrz5LKEyAeyy.png)
## 编码器接口配置步骤
1.  开启TIM和GPIO时钟    
2.  配置GPIO为上拉输入模式  
3.  配置定时器时基单元
4.  配置编码器接口模式和使能定时器
```c
// 开启TIM3时钟，TIM3挂载在APB1总线上
RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
// 开启GPIOA时钟，GPIOA挂载在APB2总线上  
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);

// 定义并初始化GPIO结构体
GPIO_InitTypeDef GPIO_InitStructure;
// 设置GPIO模式为上拉输入
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
// 选择引脚6和7
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6 | GPIO_Pin_7;
// 设置GPIO速度
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
// 初始化GPIOA
GPIO_Init(GPIOA, &GPIO_InitStructure);

// 配置TIM3使用内部时钟
TIM_InternalClockConfig(TIM3);

// 定义并初始化定时器时基结构体
TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;
// 设置时钟分频
TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
// 设置计数器向上计数模式
TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;
// 设置自动重装载值
TIM_TimeBaseInitStructure.TIM_Period = 65536 - 1;  //ARR
// 设置预分频器
TIM_TimeBaseInitStructure.TIM_Prescaler = 1-1; //PSC
// 设置重复计数器值
TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
// 初始化TIM3时基单元
TIM_TimeBaseInit(TIM3, &TIM_TimeBaseInitStructure);

// 定义并初始化输入捕获结构体
TIM_ICInitTypeDef TIM_ICInitStructure;
// 将输入捕获结构体赋默认值
TIM_ICStructInit(&TIM_ICInitStructure);
// 配置通道1
TIM_ICInitStructure.TIM_Channel = TIM_Channel_1;
// 设置输入滤波器值
TIM_ICInitStructure.TIM_ICFilter = 0xF;
// 初始化TIM3输入捕获通道1
TIM_ICInit(TIM3, &TIM_ICInitStructure);

// 配置通道2
TIM_ICInitStructure.TIM_Channel = TIM_Channel_2;
// 设置输入滤波器值
TIM_ICInitStructure.TIM_ICFilter = 0xF;
// 初始化TIM3输入捕获通道2
TIM_ICInit(TIM3, &TIM_ICInitStructure);

// 配置编码器接口模式
TIM_EncoderInterfaceConfig(TIM3, TIM_EncoderMode_TI12, TIM_ICPolarity_Rising, TIM_ICPolarity_Rising);
// 使能TIM3计数器
TIM_Cmd(TIM3, ENABLE);
```
### 获取编码器CNT计数函数
```c
// 获取编码器计数值函数
int16_t Get_Encoder(void)
{
	int16_t Temp;  // 定义临时变量存储计数值
    
	Temp = TIM_GetCounter(TIM3);  // 读取TIM3计数器的当前值
	TIM_SetCounter(TIM3, 0);      // 将TIM3计数器清零，为下一次测量做准备
    
	return Temp;  // 返回读取的编码器计数值
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ4MTk1OTU2Ml19
-->