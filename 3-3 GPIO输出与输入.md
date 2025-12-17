## GPIO使用方法
## GPIO输出
### 初始化时钟 
```c
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIO x, ENABLE);
```
### 定义结构体
```c
GPIO_InitTypeDef GPIO_InitStructure;
```
### 赋值结构体
```c
// 八种输入输出mode，赋值引脚、输出速度Speed常用50MHz
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_12;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
```
输出最常用推挽输出**GPIO_Mode_Out_PP**
输入最常用上拉电平**GPIO_Mode_IPU**
引脚可以用或`|`的方式选择多个引脚
#### 八种GPIO模式
```
GPIO_Mode_AIN - 模拟输入模式  
GPIO_Mode_IN_FLOATING - 浮空输入模式  
GPIO_Mode_IPD - 下拉输入模式  
GPIO_Mode_IPU - 上拉输入模式  
GPIO_Mode_Out_OD - 开漏输出模式  
GPIO_Mode_Out_PP - 推挽输出模式  
GPIO_Mode_AF_OD - 复用开漏模式  
GPIO_Mode_AF_PP - 复用推挽模式
```
![输入图片说明](/imgs/2025-11-07/DHDcH9hQxHCA4Um5.png)

### 外设初始化
`GPIO_Init(GPIOx, &GPIO_InitStructure);`
## GPIO输入
### 8个读取写入函数
#### 读取输入状态：
-   `GPIO_ReadInputDataBit` - **读取单个引脚的电平**（高/低）    
-   `GPIO_ReadInputData` - **读取整个端口所有引脚的电平**
    
#### 读取输出状态：
-   `GPIO_ReadOutputDataBit` - **读取单个引脚的输出寄存器状态**（你设置的值）    
-   `GPIO_ReadOutputData` - **读取整个端口的输出寄存器状态**    

#### 设置输出状态：**
-   `GPIO_SetBits` - **将指定引脚设为高电平**（置1）    
-   `GPIO_ResetBits` - **将指定引脚设为低电平**（清零）    
-   `GPIO_WriteBit` - **设置单个引脚的电平**（根据参数设为高或低）
-   `GPIO_Write` - **一次性写入整个端口的所有引脚**
```c
void Key_Init(void)
{
	// 使能GPIOB的时钟（APB2总线）
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
	// 定义GPIO初始化结构体
	GPIO_InitTypeDef GPIO_InitStructure;
	// 配置GPIO为上拉输入模式（内部上拉电阻，按键按下时接地）
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	// 配置GPIO引脚为Pin1和Pin11
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_1 | GPIO_Pin_11;
	// 配置GPIO速度（对于输入模式，速度设置影响不大）
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	// 根据配置初始化GPIOB
	GPIO_Init(GPIOB, &GPIO_InitStructure);
}

/**
  * @brief  获取按键编号
  * @note   检测两个按键的状态，返回被按下的按键编号（支持按键消抖）
  * @param  无
  * @retval 按键编号：0-无按键，1-按键1(PB1)，2-按键2(PB11)
  */
uint8_t Key_GetNum(void)
{
	uint8_t Key_Num = 0;  // 默认返回0，表示无按键按下
	
	// 检测按键1（PB1）是否按下
	if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0)
	{
		// 延时20ms进行按键消抖，消除机械抖动
		Delay_ms(20);
		
		// 等待按键释放（阻塞式等待）
		// 当按键保持按下时，循环空转；按键松开时退出循环
		while (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0);
		
		// 再次延时20ms，进行松开消抖
		Delay_ms(20);
		
		// 设置返回值为1，表示按键1被按下
		Key_Num = 1;
	}
```
### 模块化编程
- 外围硬件较多，尽量把每个硬件的驱动函数单独提取出来，封装在.c和.h文件中，**有利于简化主函数、方便移植**
- 记得给封装的驱动写注释
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2MjYxNzA4NzZdfQ==
-->