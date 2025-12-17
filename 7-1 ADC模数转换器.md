# ADC
## ADC简介
![输入图片说明](/imgs/2025-11-17/KU0BnRwMmXOfPPbQ.png)
## ADC基本结构
![输入图片说明](/imgs/2025-11-17/k9dZiCYoM7qOYigv.png)
## 输入通道表
![输入图片说明](/imgs/2025-11-17/dhT8rsAqsiH3lxcA.png)
## 转换模式
![输入图片说明](/imgs/2025-11-17/y2wdQvlbBvy65y2O.png)
![输入图片说明](/imgs/2025-11-17/I1XxxHPwjfqaaiKx.png)
![输入图片说明](/imgs/2025-11-17/oEXiWrzp3HORXqtW.png)
![输入图片说明](/imgs/2025-11-17/4Ut7IpDyRIHFbxBK.png)
## 转换时间
![输入图片说明](/imgs/2025-11-17/kT1E9uPUQQmkjwS1.png)
### 校准
![输入图片说明](/imgs/2025-11-17/0zlNFrN1vtCPCOM1.png)
## 单次单通道配置步骤
1.  开启ADC和GPIO时钟，配置ADC时钟分频    
2.  配置GPIO引脚为模拟输入模式   
3.  配置ADC规则组通道（通道、顺序、采样时间）    
4.  初始化ADC参数（转换模式、数据对齐、触发方式、工作模式等）    
5.  使能ADC并进行校准   
**连续转换模式配置说明：**
-   将`ADC_ContinuousConvMode`改为`ENABLE`    
-   在初始化函数最后添加`ADC_SoftwareStartConvCmd(ADC1, ENABLE)`    
-   获取值函数中无需等待转换完成，直接读取转换结果


```c
void AD_Init(void)
{
    // 1. 开启外设时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);  // 使能ADC1时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE); // 使能GPIOA时钟
    RCC_ADCCLKConfig(RCC_PCLK2_Div6);                     // 设置ADC分频因子6，72M/6=12MHz
    
    // 2. 配置GPIO引脚为模拟输入
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;        // 模拟输入模式
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;            // PA0引脚
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;    // 速度设置（模拟输入时此参数无意义）
    GPIO_Init(GPIOA, &GPIO_InitStructure);               // 初始化GPIOA
    
    // 3. 配置ADC规则组通道
    ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_55Cycles5); // ADC1,通道0,采样顺序1,采样时间55.5周期
    
    // 4. 初始化ADC参数
    ADC_InitTypeDef ADC_InitStruct;
    ADC_InitStruct.ADC_ContinuousConvMode = DISABLE;     // 单次转换模式
//    ADC_InitStruct.ADC_ContinuousConvMode = ENABLE;     // 连续转换模式（注释备用）
    ADC_InitStruct.ADC_DataAlign = ADC_DataAlign_Right;  // 数据右对齐
    ADC_InitStruct.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None; // 软件触发
    ADC_InitStruct.ADC_Mode = ADC_Mode_Independent;      // 独立模式
    ADC_InitStruct.ADC_NbrOfChannel = 1;                 // 1个转换通道
    ADC_InitStruct.ADC_ScanConvMode = DISABLE;           // 非扫描模式
    ADC_Init(ADC1, &ADC_InitStruct);                     // 初始化ADC1
    
    // 5. 使能ADC并校准
    ADC_Cmd(ADC1, ENABLE);                               // 使能ADC1
    ADC_ResetCalibration(ADC1);                          // 复位校准寄存器
    while (ADC_GetResetCalibrationStatus(ADC1) == SET);  // 等待复位校准完成
    ADC_StartCalibration(ADC1);                          // 开始ADC校准
    while (ADC_GetCalibrationStatus(ADC1) == SET);       // 等待校准完成
    
//    ADC_SoftwareStartConvCmd(ADC1, ENABLE);            // 连续转换时需要开启（注释备用）
}

uint16_t AD_GetValue(void)
{
    ADC_SoftwareStartConvCmd(ADC1, ENABLE);              // 软件启动转换
    while (ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC) == RESET); // 等待转换结束
    return ADC_GetConversionValue(ADC1);                 // 返回转换结果
}
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NzU2NDM5NjVdfQ==
-->