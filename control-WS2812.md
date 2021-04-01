# 基于Arduino Nano控制灯

## 软件下载

- Arduino IDE：https://pan.baidu.com/s/1cq5_qrXknBx6vrirxOtXhA 提取码：aesv
- CH340串口驱动：https://pan.baidu.com/s/1CihpBwMU65Q5nRXrnmvfPA   提取码：j18l 

## Arduino IDE配置

- FastLED.h库下载：打开Arduino IDE，菜单栏中选择<font color=yellow>“工具”-->“管理库...”</font>搜索对应的<font color=pink>"FastLED.h"</font>j进行下载即可。
- 开发板的版本设置：本次使用的开发板为<font color=yellow>**Arduino Nano**</font>,所以菜单栏中选择<font color=yellow>“工具”-->“开发板</font>选择对应型号版本开发板，处理器选择<font color=yellow>**ATmega328P**</font>。<font color=green>**注：购买Arduino Nano时注意商家提醒，是否处理器选择为ATmega328P，有的旧版本的开发板引脚接线并没有更正，这时候需要选择ATmega328P（Old Bootloader）**</font>

## 程序分析

<font color=yellow>(将所有代码合并即为完整代码)</font>

导入控制LED的库

~~~c++
#include <FastLED.h>
~~~

一些参数的定义

~~~c++
#define LED_PIN     3 //定义开发板引脚
#define NUM_LEDS    5 //定义LED灯数量
#define BRIGHTNESS  64 //亮度
#define LED_TYPE    WS2812 //LED灯芯片型号
#define COLOR_ORDER GRB //GRB模式
CRGB leds[NUM_LEDS];


CRGBPalette16 currentPalette;
TBlendType    currentBlending;
extern CRGBPalette16 myRedWhiteBluePalette;
extern const TProgmemPalette16 myRedWhiteBluePalette_p PROGMEM;

//定义用来接收串口发出字符串的数量
static int number;
int num = 0;
~~~

函数初始化（可以理解为python中的**__init__**方法）

~~~c++
void setup() {
    Serial.begin(9600); //设置串口波特率9600
    FastLED.addLeds<LED_TYPE, LED_PIN, COLOR_ORDER>(leds, NUM_LEDS).setCorrection( TypicalLEDStrip );//设置灯的型号以及参数等
    FastLED.setBrightness(  BRIGHTNESS );
    //设置呼吸灯的颜色
    CRGB red = CRGB::Red;
    CRGB green  = CRGB::Green;
    CRGB purple  = CRGB::Purple;
    CRGB blue  = CRGB::Blue;
    CRGB black  = CRGB::Black;
    //设置呼吸灯循环的颜色顺序
    currentPalette = CRGBPalette16(
                      green,  green,  black,  black,
                      red, red, black,  black,
                      blue,  blue,  black,  black,
                      purple, purple, black,  black );
    currentBlending = LINEARBLEND;
}
~~~

之后进入主循环loop()

~~~c++
void loop()
{   
    code();
}
~~~

其中code()函数主要分为两个部分：Serial 通信以及灯光效果，一下为code()代码：

- 第一部分为Serial 通信：

- ~~~c++
      while(Serial.available()){//判断是否有消息接收
        mySerial.read();//读取一个字符，并将其pop出
        num = num + 1;//记录字节长度
        number = num;//number定义为全局变量，他不会随着loop循环而改变，为了实现在妹有数据的情况下，灯一直循环演示上一个数据的指令。直到重新接收新指令。
      }
      Serial.println(num);
  ~~~

- 第二部分为灯光效果：

- ~~~c++
  	delay(20);//延时20ms保证灯不会损坏
      if(number > 0){
        switch (number){
        case 1:{ flash1(); break;}
        case 2:{ flash2(); break;}
        case 3:{ flash3(); break;}
        case 4:{ flash4(); break;}
        case 5:{ flash5(); break;}
        }
     }
    num = 0;//执行完一次指令num重置。但是number没有重置，如果没有数据传入，他会继续执行上一次接收到的指令。
  ~~~

- flash1()：<font color=yellow>**呼吸灯**</font>

- ~~~c++
  static uint8_t startIndex = 0;//定义为调色板颜色序号
  for(int j=0;j<50;j++){//遍历调色板
  	startIndex = startIndex + 1;
  	for(int i=0;i < NUM_LEDS; i++){//遍历5个灯
  		leds[i] = ColorFromPalette( currentPalette, startIndex, 255, currentBlending);//设置灯的颜色、调色板的序号、亮度、变换方式
           FastLED.show();//灯亮
           FastLED.delay(1000 / 250); //灯亮的时间延时
           }
    }
  ~~~

- flash2()：<font color=yellow>**单灯循环**</font>

- ~~~c++
  for(int i=0;i < NUM_LEDS; i++){//灯从1-5
      delay(100);
      if(i>0){
      	leds[i-1] = CRGB::Black;
      }
      leds[i] = CRGB::Blue;
      FastLED.show();
      FastLED.delay(1000 / 250); 
   }
   leds[4]=CRGB::Black;
   for(int i=4;i >= 0; i--){//灯从5-1
   	delay(100);
      if(i<4){
      	leds[i+1] = CRGB::Black;
      }
      leds[i] = CRGB::Blue;
      FastLED.show();
      FastLED.delay(1000 / 250); 
    }
    leds[0]=CRGB::Black;
  ~~~

- flash3()：<font color=yellow>**双色灯循环**</font>

- ~~~c++
      for(int i=0;i < NUM_LEDS; i++){//第一种颜色的灯逐个点亮
      delay(100);
      leds[i] = CRGB::Blue;
      FastLED.show();
      FastLED.delay(1000 / 250); 
      }
      for(int i=4;i >= 0; i--){//第二种颜色的灯反向逐个点亮
      delay(100);
      leds[i] = CRGB::Yellow;
      FastLED.show();
      FastLED.delay(1000 / 250);
      }
  ~~~

- flash4()：<font color=yellow>**警示灯**</font>

- ~~~c++
      for(int i=0;i < NUM_LEDS; i++){//以很快的速度闪烁红光
      leds[i] = CRGB::Red; 
      }
      FastLED.show();
      FastLED.delay(1000 / 250);
      delay(100);
      for(int i=0;i < NUM_LEDS; i++){
      leds[i] = CRGB::Black; 
      }
      FastLED.show();
      FastLED.delay(1000 / 250);
      delay(100);
  ~~~

- flash5()：<font color=yellow>**关灯**</font>

- ~~~c++
      for(int i=0;i < NUM_LEDS; i++){//关灯
      leds[i] = CRGB::Black; 
      }
      FastLED.show();
      FastLED.delay(1000 / 250);
      break;
  ~~~

## 引脚接法

LED灯分别为VCC，GND以及LED。VCC为+3.3V或者+5V，LED为数据口，接Arduino Nano的D3引脚。通信通过RX，TX进行，RX接TX，TX接RX。

### 注意事项

- 输入的指令要尽量有些间隔，不然有时输入过快会导致数据丢失。
- 小心短接！LED靠近线的那个灯为LED[0]

