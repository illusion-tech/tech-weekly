# WiFi 连接云平台
基本硬件：esp8266-01s,Arduino,电阻，Led灯，温湿度传感器，烟雾浓度传感器，杜邦线，面包板等
这里使用BIGIOT.net与Arduino来实现
esp8266连线：vcc,EN -3v3 GND-GND ,RX-TX TX-RX 
LED连线 6 GND
温湿度传感器连线 Vcc-3v3 GND-GND Data-2
烟雾浓度传感器 Vcc-3v3 GND-GND Ao-A0 Do-4

1.进行烧固操作
使用usb转ttl模块会相对方便一些，使用Arduino连线来进行比较不稳定但是可以实现。
烧录软件推荐使用安信可的：
https://docs.ai-thinker.com/%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B72
需要设置好SPI,Cry,FLASH与下载好的固件包。

使用AT指令：
AT+CWMODE=1来设置应用模式为Station
AT+CWJAP="wcy","123456789" 设置自己需要的wifi与密码
AT+CIPSTART="TCP","www.bigiot.net",8181 用于使用tcp连接云服务器
AT+CIPMODE=1 用于设置为透传模式
AT+CIPSEND 用于进入透传模式
都能够通过后显示ready后，esp8266就可以正常使用了，这里是最大的难点，因为很容易烧录失败。

2.代码运行
详细参考文章末尾

3.可视化显示。
进入BigIot.net，登录后可以在首页配置智能设备与数据接口，如果智能设备处于在线状态就是连接成
功，智能设备的APIKEY与数据接口的ID可以用于接口调用。
在数据查看中显示：
https://ldy-1311016186.cos.ap-beijing.myqcloud.com/29d1c5ae4a82088911a87cc2f5dbe7f.png

4.作为接口使用
点击个人信息申请成为开发者，获取到client_id与client_sercet。
首先获取授权码：接口地址：https://www.bigiot.net/oauth/token POST
参数：client_id(应用id),client_sercet(应用密码),username(用户ID),password(用户apikey),
grant_type(password)
举例：
{
    "access_token": "1561a0fd9c6c990e1197fbcbaf4a5368ab2eb0da",
    "expires_in": 172800,
    "token_type": "Bearer",
    "scope": null,
    "refresh_token": "93d5631f6ebe7f9b26c398324ba691bbc48bda98"
}
token请求一次2天有效，不可频繁获取
而后每次请求资源时需要携带该token
举例：
https://www.bigiot.net/oauth/historydata?
access_token=50f69c98fb079fea603e3659b9b6271493bdfc54&id=23068

AT固件烧录详细：
1.烧录用到的物品:USB转TTL、ESP8266-01S、Arduino（能够外部供电的单片机都可）、杜邦线若干

2:ESP8266-01S有运行模式、下载模式、测试模式的区别，进入这下载模式、运行模式通过拉低或拉高
IO0的电平实现。官方给出的图中ESP8266-01S没有引出GPIO15引脚，只引出了八个引脚，分别是3V3、
RST、EN、TX、RX、IO0、IO2、GND对于EN、RST、IO2这三个引脚安可信已经在电路中拉高拉低

3:USB转TTL:USB转TTL跟ESP8266-01S需要接的引脚有三个，RXT、TXD、GND，其中有一个需要注意的问
题，USB转TTL有3V3、5V和VCC接口，其中3V3与5V是供VCC选择电平的引脚，而不能作为ESP8266-01S的
供电引脚，所以需要用跳线帽来选择VCC电平
TTL:RX-TX,TX-RX,GND_GND
Arduino:3v3-3v3 IO0-GND或悬空 GND-GND
如果使用Esp8266烧录器，就不需要考虑连线问题

以安可信固件烧录为例：
打开烧录软件，会弹出如下图所示界面，然后选择ESP8266  DownloadTool
打开后界面如下图所示，固件包信息选择.66_DOUT_8Mbit打头的bin文件0x00000，CryStalFreq选择
26M,SPISPEED选择40MHZ,SPIMODE选择DOUT,FLASHSIZE选择8Mbit，波特率就选择115200，因为ESP8266
默认波特率是115200，电脑要安装好串口驱动CH340，然后选择自己电脑显示的COM口。
点击start,如果等待变成蓝色则成功。
使用AT指令进入透传模式连接贝壳物联。
//设置WiFi应用模式为Station
AT+CWMODE=1
//连接到WiFi路由器，请将SSID替换为路由器名称，Password替换为路由器WiFi密码
AT+CWJAP="SSID","Password"
//连接贝壳物联服务器
AT+CIPSTART="TCP","www.bigiot.net",8181
//设置为透传模式
AT+CIPMODE=1
//进入透传模式
AT+CIPSEND
如果是V1.0以上的固件则使用
+++
AT
ATE0
AT+RESTORE
AT+CWMODE=3
AT+CWJAP="SSID","Password"
AT+CIPMUX=0
AT+CIPMODE=1
AT+SAVETRANSLINK=1,"121.42.180.30",8181,"TCP"
如果出现乱码是因为Esp8266开始波特率为74880，上电后变成115200。

#include <aJSON.h> //引用库文件
#include <dht11.h> //引用dht11库文件
#include <Wire.h> 
#include <LiquidCrystal_I2C.h>
#define Sensor_AO A0
#define Sensor_DO 4
#define dht11Pin 2   //定义温湿度针脚号为2号引脚
unsigned int sensorValue = 0;
LiquidCrystal_I2C lcd(0x27,16,2);
String DEVICEID = "22412"; // 你的设备ID=======
String APIKEY = "077a64a8a"; // 设备密码==
String INPUTID1 = "23068"; //接口ID1==============
String INPUTID2 = "23070"; //接口ID2==============
String INPUTID3 = "20464";
//=======================================
dht11 dht;    //实例化一个对象
unsigned long lastCheckStatusTime = 0; //记录上次报到时间
unsigned long lastUpdateTime = 0;//记录上次上传数据时间
const unsigned long postingInterval = 40000; // 每隔40秒向服务器报到一次
const unsigned long updateInterval = 5000; // 数据上传间隔时间5秒
unsigned long checkoutTime = 0;//登出时间
void MyPrintLCD(String MyString){
 for (int i=0;i<MyString.length();i++)
 lcd.write(MyString.charAt(i));
  }

void setup() {
  lcd.init(); // 初始化LCD 
  lcd.backlight(); //设置LCD背景等亮 
  
  pinMode(6,OUTPUT);
  pinMode(dht11Pin, INPUT); //通过定义将Arduino开发板上dht11Pin引脚(2号口)的工作模式转化为输出模式
  pinMode(Sensor_DO, INPUT);
  Serial.begin(115200);
  delay(5000);//等一会儿ESP8266
}
void loop() {
  sensorValue = analogRead(Sensor_AO);
      if(sensorValue >50){
    digitalWrite(6,HIGH);//输出5V电压，LED发光
    delay(1000);//发光持续1000ms，即1s
    }else{
      digitalWrite(6,LOW);//0V，熄灭
      delay(1000);//发光持续1000ms，即1s
    }

  //每一定时间查询一次设备在线状态，同时替代心跳
  if (millis() - lastCheckStatusTime > postingInterval) {
    
    
    checkStatus();
  }
  //checkout 50ms 后 checkin
  if ( checkoutTime != 0 && millis() - checkoutTime > 50 ) {
    
    
    checkIn();
    checkoutTime = 0;
  }
  //每隔一定时间上传一次数据
  if (millis() - lastUpdateTime > updateInterval) {/    int tol = dht.read(dht11Pin);    //将读取到的值赋给tol
    float temp = (float)dht.temperature; //将温度值赋值给temp
    float humi = (float)dht.humidity; //将湿度值赋给humi
// float temp = random(230, 250) / 10.0; // 生成30-40之间的随机数
// float humi = random(400, 600) / 10.0; // 生成30-40之间的随机数
 String l1="";
 l1=l1+"temp:"+temp;
 String l2="";
 l2=l2+"humi:"+humi;
lcd.setCursor(0,0);                //设置显示指针
MyPrintLCD(l1); 
lcd.setCursor(0,1); 
MyPrintLCD(l2);
    delay(1000);      //延时1秒
    update3(DEVICEID, INPUTID1, temp, INPUTID2, humi,INPUTID3, sensorValue);
    lastUpdateTime = millis();
  }

  //读取串口信息
  while (Serial.available()) {
    String inputString = Serial.readStringUntil('\n');
    //检测json数据是否完整
    int jsonBeginAt = inputString.indexOf("{");
    int jsonEndAt = inputString.lastIndexOf("}");
    if (jsonBeginAt != -1 && jsonEndAt != -1) {
      //净化json数据
      inputString = inputString.substring(jsonBeginAt, jsonEndAt + 1);
      int len = inputString.length() + 1;
      char jsonString[len];
      inputString.toCharArray(jsonString, len);
      aJsonObject *msg = aJson.parse(jsonString);
      processMessage(msg);
      aJson.deleteItem(msg);
    }
  }
}
//设备登录
//{"M":"checkin","ID":"xx1","K":"xx2"}\n
void checkIn() {
  Serial.print("{\"M\":\"checkin\",\"ID\":\"");
  Serial.print(DEVICEID);
  Serial.print("\",\"K\":\"");
  Serial.print(APIKEY);
  Serial.print("\"}\n");
}
//强制设备下线，用消除设备掉线延时
//{"M":"checkout","ID":"xx1","K":"xx2"}\n
void checkOut() {
  Serial.print("{\"M\":\"checkout\",\"ID\":\"");
  Serial.print(DEVICEID);
  Serial.print("\",\"K\":\"");
  Serial.print(APIKEY);
  Serial.print("\"}\n");
}

//查询设备在线状态
//{"M":"status"}\n
void checkStatus() {
  Serial.print("{\"M\":\"status\"}\n");
  lastCheckStatusTime = millis();
}

//处理来自ESP8266透传的数据
void processMessage(aJsonObject *msg) {
  aJsonObject* method = aJson.getObjectItem(msg, "M");
  if (!method) {
    return;
  }
  String M = method->valuestring;
  if (M == "WELCOME TO BIGIOT") {
    checkOut();
    checkoutTime = millis();
    return;
  }
  if (M == "connected") { 
    checkIn();
  }
}
//同时上传两个接口数据调用此函数
//{"M":"update","ID":"112","V":{"6":"1","36":"116"}}\n
void update3(String did, String inputid1, float value1, String inputid2, float value2,String inputid3,float value3) {
  Serial.print("{\"M\":\"update\",\"ID\":\"");
  Serial.print(did);
  Serial.print("\",\"V\":{\"");
  Serial.print(inputid1);
  Serial.print("\":\"");
  Serial.print(value1);
  Serial.print("\",\"");
  Serial.print(inputid2);
  Serial.print("\":\"");
  Serial.print(value2);
   Serial.print("\",\"");
  Serial.print(inputid3);
  Serial.print("\":\"");
  Serial.print(value3);
  Serial.println("\"}}");
}