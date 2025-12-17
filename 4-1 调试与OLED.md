## 常用调试方法
![输入图片说明](/imgs/2025-11-06/HcOGIJrAmDOFpYHa.png)
## OLED屏幕
![输入图片说明](/imgs/2025-11-06/9NlhdgYyv6Bll9Zi.png)
## OLED驱动函数
![输入图片说明](/imgs/2025-11-06/CNNK5gdSvrwBaIXR.png)
![输入图片说明](/imgs/2025-11-06/6UpkUXaKfqk65lSf.png)
## OLED驱动模块
`江科大科协\程序源码\STM32Project-有注释版\1-4 OLED驱动函数模块`
包含 **OLED.c、OLED.h、OLED_Font.h** 三个文件
## OLED引脚配置与引脚初始化
```c
/* 引脚配置 - 定义OLED I2C通信的SCL和SDA引脚控制宏 */
// SCL(时钟线)控制：PB8引脚，x为0或1，表示输出低电平或高电平
#define OLED_W_SCL(x)		GPIO_WriteBit(GPIOB, GPIO_Pin_8, (BitAction)(x))
// SDA(数据线)控制：PB9引脚，x为0或1，表示输出低电平或高电平  
#define OLED_W_SDA(x)		GPIO_WriteBit(GPIOB, GPIO_Pin_9, (BitAction)(x))

/* 引脚初始化 - 初始化OLED I2C通信所需的GPIO引脚 */
void OLED_I2C_Init(void)
{
    // 使能GPIOB的时钟，因为SCL和SDA引脚都在GPIOB上
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
    // 定义GPIO初始化结构体变量
    GPIO_InitTypeDef GPIO_InitStructure;
    
    // 配置GPIO为开漏输出模式，I2C通信通常使用开漏输出
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_OD;
    // 设置GPIO输出速度为50MHz
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    
    // 初始化SCL引脚(PB8)
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_8;
    GPIO_Init(GPIOB, &GPIO_InitStructure);
    
    // 初始化SDA引脚(PB9)  
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;
    GPIO_Init(GPIOB, &GPIO_InitStructure);
    
    // 初始化后将SCL和SDA引脚置为高电平，这是I2C总线的空闲状态
    OLED_W_SCL(1);
    OLED_W_SDA(1);
}
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA0NjY3NjUxNl19
-->