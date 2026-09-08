# OBD-ESP32-Car-Diagnosis
Add a README file ，可选添加  .gitignore ，选择  C （针对C语言单片机项目）

b# OBD‑ESP32‑Car‑Diagnosis
自制汽车OBD诊断设备，STM32F103C8T6 + TJA1050 + HC‑05，搭配华为鸿蒙平板做维修诊断软件。
实现CAN总线与蓝牙透传，读取汽车故障码、发动机转速、水温、车速等数据流。

> 🎯项目定位：DIY简易汽车维修OBD工具
> - 硬件：STM32F103C8T6主控
> - CAN收发：TJA1050
> - 无线：HC‑05蓝牙SPP
> - 上位机：华为鸿蒙平板 ArkTS App
> - 协议：ISO15765‑4 CAN‑OBD（500Kbps）

## 📦硬件清单
|器件|用途|
|---|---|
|STM32F103C8T6最小系统板|主控，CAN‑蓝牙报文网桥|
|ST‑Link V2|程序烧录调试，**车上使用必须拔掉**|
|TJA1050 CAN收发器|汽车CAN电平转换|
|HC‑05蓝牙模块|STM32 ↔ 鸿蒙平板 SPP串口蓝牙|
|OBD2 16Pin公头线束|车辆OBD诊断插座|
|12V转5V电源模块|汽车12V取电给整套系统供电|
|杜邦线、面包板|硬件调试接线|

## 🔌硬件接线

### STM32 ↔ HC‑05
|STM32F103|HC‑05|
|---|---|
|PA2 USART2_TX | HC‑05 RX|
|PA3 USART2_RX | HC‑05 TX|
|5V | VCC|
|GND | GND|

> HC‑05参数：波特率 9600，数据位8，停止位1，无校验(8N1)

### STM32 ↔ TJA1050
|STM32F103|TJA1050|
|---|---|
|PA11 CAN1_RX | RX|
|PA12 CAN1_TX | TX|
|5V | VCC|
|GND | GND|

### TJA1050 ↔ OBD2‑16PIN
|TJA1050|OBD引脚|
|---|---|
|CAN_H | OBD 6脚|
|CAN_L | OBD 14脚|
|GND | OBD 4 / 5脚车身地|

> OBD16脚：汽车12V输入，接到12V‑5V电源模块

### ST‑Link烧录接线（仅开发阶段）
|ST‑Link|STM32F103|
|---|---|
|SWDIO | PA13|
|SWCLK | PA14|
|GND | GND|

> ❗禁止接3.3V；下载完成上车使用务必拔掉ST‑Link

## 💻STM32CubeIDE工程配置
1. 新建工程选择芯片 `STM32F103C8T6`
2. SYS → Debug：`Serial Wire`（防止芯片锁死）
3. RCC → HSE：外部8M晶振
4. USART2：异步串口，波特率9600，开启NVIC中断
5. CAN1：Normal模式，波特率 **500kbit/s**
6. 时钟树配置 HSE 8M → 系统时钟72MHz
7. 生成HAL工程

> ⚠️STM32CubeIDE不要安装到C盘，会出现图形卡死蓝色界面；建议安装路径 `D:\ST\STM32CubeIDE`

### 固件逻辑
- STM32作为**CAN‑蓝牙透传网桥**
- 蓝牙串口接收平板下发OBD指令，转为CAN报文发送给ECU（请求ID `0x7DF`）
- 接收车辆ECU返回CAN报文（应答ID `0x7E8`），通过蓝牙原样转发给鸿蒙平板
- **报文解析全部放在鸿蒙APP，STM32只负责转发**

## 📱鸿蒙 ArkTS APP
- 开发设备：华为平板 11.5 鸿蒙系统
- 通信方式：蓝牙SPP串口
- 功能：
  1. 扫描、连接HC‑05蓝牙模块
  2. 下发OBD PID命令
  3. 显示原始CAN十六进制报文
  4. 解析故障码、转速、水温、车速等数据
  5. 故障记录查看

> ⚠️鸿蒙模拟器无蓝牙，**必须使用真机平板调试**

## 📋OBD‑II PID命令表(ISO15765‑4)
|命令|功能|计算公式|
|---|---|---|
|03|读取故障码DTC|返回存储故障码|
|04|清除故障码&冻结帧|执行清除|
|01 00|查询支持PID列表|查看车辆支持哪些数据流|
|01 0C|发动机转速|(A*256+B)/4 rpm|
|01 0D|车辆速度|A km/h|
|01 05|冷却液温度|A‑40 ℃|
|01 0A|燃油压力|A*3 kPa|
|01 11|节气门位置|A*100/255 %|
|01 42|系统电压|(A*256+B)/100 V|

> CAN请求ID：`0x7DF`；ECU应答ID：`0x7E8`

## ⚠️重要避坑
1. ST‑Link仅烧录使用，上车**必须拔掉**，否则干扰CAN总线
2. TJA1050必须共地，OBD GND一定要接好，地不共会CAN乱码无通信
3. HC‑05波特率严格9600，STM32串口波特率保持一致
4. 部分老车CAN速率250Kbps，APP后期可增加波特率切换功能
5. 先面包板回环测试CAN收发，硬件调试完成后再接真实汽车OBD口
6. 本项目为DIY学习项目，不用于专业汽修商用设备

## 📂仓库目录结构
ash

# OBD‑ESP32‑Car‑Diagnosis
自制汽车OBD诊断设备，STM32F103C8T6 + TJA1050 + HC‑05，搭配华为鸿蒙平板做维修诊断软件。
实现CAN总线与蓝牙透传，读取汽车故障码、发动机转速、水温、车速等数据流。

> 🎯项目定位：DIY简易汽车维修OBD工具
> - 硬件：STM32F103C8T6主控
> - CAN收发：TJA1050
> - 无线：HC‑05蓝牙SPP
> - 上位机：华为鸿蒙平板 ArkTS App
> - 协议：ISO15765‑4 CAN‑OBD（500Kbps）

## 📦硬件清单
|器件|用途|
|---|---|
|STM32F103C8T6最小系统板|主控，CAN‑蓝牙报文网桥|
|ST‑Link V2|程序烧录调试，**车上使用必须拔掉**|
|TJA1050 CAN收发器|汽车CAN电平转换|
|HC‑05蓝牙模块|STM32 ↔ 鸿蒙平板 SPP串口蓝牙|
|OBD2 16Pin公头线束|车辆OBD诊断插座|
|12V转5V电源模块|汽车12V取电给整套系统供电|
|杜邦线、面包板|硬件调试接线|

## 🔌硬件接线

### STM32 ↔ HC‑05
|STM32F103|HC‑05|
|---|---|
|PA2 USART2_TX | HC‑05 RX|
|PA3 USART2_RX | HC‑05 TX|
|5V | VCC|
|GND | GND|

> HC‑05参数：波特率 9600，数据位8，停止位1，无校验(8N1)

### STM32 ↔ TJA1050
|STM32F103|TJA1050|
|---|---|
|PA11 CAN1_RX | RX|
|PA12 CAN1_TX | TX|
|5V | VCC|
|GND | GND|

### TJA1050 ↔ OBD2‑16PIN
|TJA1050|OBD引脚|
|---|---|
|CAN_H | OBD 6脚|
|CAN_L | OBD 14脚|
|GND | OBD 4 / 5脚车身地|

> OBD16脚：汽车12V输入，接到12V‑5V电源模块

### ST‑Link烧录接线（仅开发阶段）
|ST‑Link|STM32F103|
|---|---|
|SWDIO | PA13|
|SWCLK | PA14|
|GND | GND|

> ❗禁止接3.3V；下载完成上车使用务必拔掉ST‑Link

## 💻STM32CubeIDE工程配置
1. 新建工程选择芯片 `STM32F103C8T6`
2. SYS → Debug：`Serial Wire`（防止芯片锁死）
3. RCC → HSE：外部8M晶振
4. USART2：异步串口，波特率9600，开启NVIC中断
5. CAN1：Normal模式，波特率 **500kbit/s**
6. 时钟树配置 HSE 8M → 系统时钟72MHz
7. 生成HAL工程

> ⚠️STM32CubeIDE不要安装到C盘，会出现图形卡死蓝色界面；建议安装路径 `D:\ST\STM32CubeIDE`

### 固件逻辑
- STM32作为**CAN‑蓝牙透传网桥**
- 蓝牙串口接收平板下发OBD指令，转为CAN报文发送给ECU（请求ID `0x7DF`）
- 接收车辆ECU返回CAN报文（应答ID `0x7E8`），通过蓝牙原样转发给鸿蒙平板
- **报文解析全部放在鸿蒙APP，STM32只负责转发**

## 📱鸿蒙 ArkTS APP
- 开发设备：华为平板 11.5 鸿蒙系统
- 通信方式：蓝牙SPP串口
- 功能：
  1. 扫描、连接HC‑05蓝牙模块
  2. 下发OBD PID命令
  3. 显示原始CAN十六进制报文
  4. 解析故障码、转速、水温、车速等数据
  5. 故障记录查看

> ⚠️鸿蒙模拟器无蓝牙，**必须使用真机平板调试**

## 📋OBD‑II PID命令表(ISO15765‑4)
|命令|功能|计算公式|
|---|---|---|
|03|读取故障码DTC|返回存储故障码|
|04|清除故障码&冻结帧|执行清除|
|01 00|查询支持PID列表|查看车辆支持哪些数据流|
|01 0C|发动机转速|(A*256+B)/4 rpm|
|01 0D|车辆速度|A km/h|
|01 05|冷却液温度|A‑40 ℃|
|01 0A|燃油压力|A*3 kPa|
|01 11|节气门位置|A*100/255 %|
|01 42|系统电压|(A*256+B)/100 V|

> CAN请求ID：`0x7DF`；ECU应答ID：`0x7E8`

## ⚠️重要避坑
1. ST‑Link仅烧录使用，上车**必须拔掉**，否则干扰CAN总线
2. TJA1050必须共地，OBD GND一定要接好，地不共会CAN乱码无通信
3. HC‑05波特率严格9600，STM32串口波特率保持一致
4. 部分老车CAN速率250Kbps，APP后期可增加波特率切换功能
5. 先面包板回环测试CAN收发，硬件调试完成后再接真实汽车OBD口
6. 本项目为DIY学习项目，不用于专业汽修商用设备

## 📂仓库目录结构

# 本地文件夹打开git bash
git init
git add .
git commit -m "初始提交OBD项目代码"
git remote add origin https://github.com/daneang515-ai/OBD-ESP32-Car-Diagnosis
git push -u origin main


OBD‑ESP32‑Car‑Diagnosis/
├── src/                # 源码，ESP32/STM32代码
├── docs/               # 文档：OBD协议说明、接线图、调试笔记
├── schematics/          # 电路图、PCB截图
├── README.md           # 项目介绍、硬件清单、使用说明
└── .gitignore           # 忽略编译生成的bin、build文件夹
