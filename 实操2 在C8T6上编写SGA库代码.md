# C8T6与VCT6代码复用
## 详细解决步骤

### 方法一：重新创建/配置工程（推荐）

这是最彻底的解决方案：

1.  **在Keil中更改设备型号**：
    
    text
    
    Project → Options for Target → Device
    
    将 `STM32F103VC` 改为 `STM32F103C8`，Keil会自动选择正确的启动文件和内存配置。
    
2.  **修改下载算法**：
    
    text
    
    Project → Options for Target → Debug → Settings → Flash Download
    
    -   删除原有的下载算法
        
    -   点击 `Add`，选择：
        
        text
        
        STM32F10x Medium-density Flash
        
        -   起始地址：`0x08000000`
            
        -   大小：`0x10000` (64KB)
            
3.  **确认内存配置**：
    
    text
    
    Project → Options for Target → Target
    
    -   **IRAM1 (RAM)**：`Start: 0x20000000`，`Size: 0x5000` (20KB)
        
    -   **IRAM2**：保持为空
        
    -   **IROM1 (Flash)**：`Start: 0x8000000`，`Size: 0x10000` (64KB)

#  点灯
1. 先在config.c里配置
```c
tagGPIO_T ledGPIO[] =
{
	[0] =
	{ 
		.tGPIOInit.Pin 		= GPIO_PIN_0,				/* PB0引脚 */
		.tGPIOInit.Mode 	= GPIO_MODE_OUTPUT_PP,		/* 推挽输出模式 */
		.tGPIOInit.Pull 	= GPIO_NOPULL,				/* 无上下拉 */
		.tGPIOInit.Speed 	= GPIO_SPEED_FREQ_LOW,		/* 低速即可 */
		.tGPIOPort 			= GPIOB,					/* GPIOB分组 */
	},
};
```
每个 `tagGPIO_T` 结构体只能控制一个GPIO引脚！
不要忘记在config.h里声明`extern tagGPIO_T ledGPIO[];`

2. 然后在task_userinit.c里初始化
```c
void Task_UserInit(void)
{
	// 初始化GPIO
	Drv_GPIO_Init(ledGPIO, 1); 
	// 初始化其他外设（PWM、串口、电机接口）...
}
```
1代表数量，如果上面配置了0，1，2，3四个ledGPIO
就可以写`Drv_GPIO_Init(ledGPIO, 4)`

3. 然后在usercode.c里调用
```c
Drv_GPIO_Set(&ledGPIO[0]);
```
不用改main.c

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU0NjUwNTYzOCwtMjA1MDkzMjM4NSwtNz
AxMTAxNjQ0XX0=
-->