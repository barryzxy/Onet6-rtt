#开关量#
&emsp;&emsp;在嵌入式系统中的都会不同的方式将系统运行状态传送给用户，LED可以说是嵌入式系统中最简单的方式之一，本章简单介绍如何设计LED接口电路，对比应用操作系统与不用的异同。

&emsp;&emsp;本节主要目的介绍有无操作系统程序实现细节的差别。
在本章中以LED显示为例介绍RT-Thread操作系统线程管理相关的内容，并给出线程设计模型，以供设计者使用

书中所用开发套件关于开关量有三个外设可以进行演示，分类是LED、蜂鸣器、键盘。
##STM32 IO硬件介绍
STM32F系统处理器GPIO随着型号不同个数也不经相同 。以STM32F103VC 100引脚的为例,在100个引脚中除电源，时钟，模拟电路输入参考源之外所有的80个引脚均可配置成为不同的模式进行工作。
在STM32中每个脚可通过配置成不同的模式，进行功能在图 3 3 GPIO端口位的基本结构中表明每个引脚/端口输入输出中可选配置选项。可以通过配置寄存器行设置，比较繁琐，在ST官方提供的库中已对各功能进行划分，在库中以宏定义的形式给出。下面进行介绍。
![GPIO端口位的基本结构](/figures/ch03-gpio-bit.png)
**GPIO主要特性**

+ 输出状态可选推挽、开漏、上拉/下拉
+ 每个I/O速度可配置
+ 输入状态可选悬空、上位/下拉、模拟
+ I/O引脚功能复用
+ 可对GPIO每个引脚进行位操作


&emsp;&emsp;输入浮空、输入上拉、输入下拉、模拟输入、开漏输出、推挽式输出、推挽式复用功能、开漏复用功能；工作模式GPIO_Mode在STM32库中以枚举型定义如下：
~~~{.c}
/* stm32f10x_gpio.h */
typedef enum
{ 
	GPIO_Mode_AIN = 0x00,         //模拟输入
	GPIO_Mode_IN_FLOATING = 0x04, //浮空输入
	GPIO_Mode_IPD = 0x28,         //下拉输入
    GPIO_Mode_IPU = 0x48,         //上拉输入
    GPIO_Mode_Out_OD = 0x14,      //开漏输出
    GPIO_Mode_Out_PP = 0x10,      //推挽式输出
    GPIO_Mode_AF_OD = 0x1C,       //推挽式复用功能
    GPIO_Mode_AF_PP = 0x18        //开漏复用功能
}GPIOMode_TypeDef;
~~~
&emsp;&emsp;GPIO最大输出速度根据需要可配置为2M、10M、50M进行工作，最大输出速度在STM32库中以枚举型定义如下：
~~~{.c}
/* stm32f10x_gpio.h */
typedef enum
{ 
    GPIO_Speed_10MHz = 1,
    GPIO_Speed_2MHz, 
    GPIO_Speed_50MHz
}GPIOSpeed_TypeDef;

~~~
##外围器件介绍

###LED基本知识
发光二极管（Light Emitting Diode），是一种能够将电能转化为可见光的固态半导休器件，当电流流过LED时就会产生可视的光，LED的发光强度与通过LED的电流成正比如图5-1所示。目前常用的LED有红色、黄色、蓝色等。
![LED工作原理](/figures/led.png)
LED电路介绍

![led电路图](/figures/led-sch.png)
**按键**

**蜂鸣器**

##电路设计
**LED电路图**



##GPIO硬件相关函数
```c
/* stm32f10x_gpio.c */
void GPIO_Init(GPIO_TypeDef* GPIOx, GPIO_InitTypeDef* GPIO_InitStruct)
```
**功能** 
根据GPIO_InitStruct中指定的参数进行初始化外设GPIOx寄存器。
************
**参数**
**************************
GPIOx		        通过x来选择GPIO外设,如GPIOA、GPIOE等
GPIO_InitStruct 	指向结构GPIO_InitTypeDef的指针，包含外设GPIO的配置信息
*********************
GPIO_InitTypeDef    中可以指定引脚、GPIO工作速度和工作模式。
********************************
**返回值**  无

**范例**
```c
/* rt_hw_led.c */
void rt_hw_led_init(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;
    .
    GPIO_InitStructure.GPIO_Pin   = GPIO_Pin_5;       	  //指定第五引脚
    GPIO_InitStructure.GPIO_Mode  = GPIO_Mode_Out_PP;     //工作模式
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;     //工作速度
    GPIO_Init(GPIOE, &GPIO_InitStructure);           	  //初始化
}
```
注：引脚选中是通过`GPIO_Pin_x`选定，选定多个`GPIO_Pin_5|GPIO_Pin_6`表示指定第5位和每六位，`x`可为`0`到`15`
 
**GPIO置位操作**
```c
/* stm32f10x_gpio.c */
void GPIO_SetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
```
##驱动模型
下图显示了led模块的程序接口，给应用程序提供3个接口函数，`led_init()、led_on()、led_off()`分别实现对led1、led2、led3、led4的初始化和led点亮和熄灭操作。
![led驱动抽象框图](/figures/led-control-mode.png)
`rt_hw_led_init()`完成对led1、led2、led3、led4硬件的初始化；在实例中，应用

