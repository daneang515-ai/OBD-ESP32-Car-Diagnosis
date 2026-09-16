# 🚀 快速上手指南 (QUICKSTART)

> 从零开始，30 分钟跑通整套 DIY 汽车 OBD 诊断设备

---

## 📋 目录

- [硬件准备](#硬件准备)
- [第一步：STM32 固件烧录](#第一步stm32-固件烧录)
- [第二步：硬件接线](#第二步硬件接线)
- [第三步：鸿蒙 App 安装](#第三步鸿蒙-app-安装)
- [第四步：蓝牙配对](#第四步蓝牙配对)
- [第五步：接车测试](#第五步接车测试)
- [调试建议](#调试建议)

---

## 硬件准备

参考主 README [硬件清单](../README.md#-硬件清单) 备齐器件。

**核心 7 件套：**

```
STM32F103C8T6 + ST-Link V2 + TJA1050 + HC-05 + OBD2线束 + 12V→5V模块 + 杜邦线/面包板
```

> 💰 总计约 **¥90**

---

## 第一步：STM32 固件烧录

### 1.1 安装工具链

- 下载安装 **STM32CubeIDE** (≥1.14.0)
  - ⚠️ **不要装 C 盘**，建议 `D:\ST\STM32CubeIDE`
- 安装 **STM32CubeProgrammer**（备用烧录工具）

### 1.2 导入工程

```bash
git clone https://github.com/daneang515-ai/OBD-ESP32-Car-Diagnosis.git
cd OBD-ESP32-Car-Diagnosis
```

1. 打开 STM32CubeIDE
2. `File → Import → Existing Projects into Workspace`
3. 选择 `src/` 目录
4. 确认工程加载成功

### 1.3 关键配置（STM32CubeIDE）

| 配置项 | 设置 |
|--------|------|
| 芯片 | STM32F103C8T6 |
| SYS → Debug | Serial Wire |
| RCC → HSE | 外部 8M 晶振 |
| USART2 | 异步，9600，开启 NVIC 中断 |
| CAN1 | Normal 模式，500kbit/s |
| 时钟树 | HSE 8M → SYSCLK 72MHz |

> ⚠️ **务必开启 Serial Wire**，否则可能锁死芯片无法再次烧录。

### 1.4 编译

- 右键工程 → `Build Project`
- 无报错后生成 `.elf` 文件

### 1.5 烧录

**方式 A：ST-Link（推荐）**

1. ST-Link 连接 STM32：
   - SWDIO → PA13
   - SWCLK → PA14
   - GND → GND
2. 点击 🐞 `Debug` 按钮烧录
3. 烧录完成后 **断开 ST-Link**

**方式 B：串口/其他**

参考 [tools/stlink_flash.bat](../tools/) 修改适配。

---

## 第二步：硬件接线

### 接线清单

**STM32 ↔ HC-05**

| STM32 | HC-05 |
|-------|-------|
| PA2 (TX) | RX |
| PA3 (RX) | TX |
| 5V | VCC |
| GND | GND |

**STM32 ↔ TJA1050**

| STM32 | TJA1050 |
|-------|---------|
| PA11 (CAN_RX) | RX |
| PA12 (CAN_TX) | TX |
| 5V | VCC |
| GND | GND |

**TJA1050 ↔ OBD2**

| TJA1050 | OBD Pin |
|---------|---------|
| CAN_H | Pin 6 |
| CAN_L | Pin 14 |
| GND | Pin 4/5 |

**供电**

```
OBD Pin 16 (12V) → 12V→5V 模块 → STM32 5V 引脚
OBD Pin 4/5 (GND) → 模块 GND → STM32 GND
```

### ⚠️ 接线要点

1. **务必共地**：所有 GND 连在一起（STM32、TJA1050、OBD）
2. **CAN_H/CAN_L 别接反**
3. **ST-Link 上车前必须拔掉**
4. **HC-05 波特率 9600**，与 STM32 USART2 一致

### 🔧 面包板回环测试（强烈建议先做）

**目的**：不接车，验证 STM32 + TJA1050 的 CAN 收发是否正常。

```c
// 在 main() 中加入回环测试
hcan.Instance->BTR |= CAN_BTR_LBKM;  // 开启回环模式
// 发送一帧，看是否能收到 → 验证硬件 OK
```

测试通过后再接真实车辆 OBD 口。

---

## 第三步：鸿蒙 App 安装

### 3.1 环境准备

- 安装 **DevEco Studio** (≥4.0)
- 准备 **华为平板 11.5**（鸿蒙 4.0+）

### 3.2 编译运行

```bash
cd harmony_app/
# 用 DevEco Studio 打开
# 选择设备 → 华为平板 → Run
```

### 3.3 真机调试配置

1. 平板开启「开发者选项」→「USB 调试」
2. USB 连接电脑
3. DevEco Studio 选择设备 → 运行

> ⚠️ **鸿蒙模拟器无蓝牙**，必须用真机调试！

---

## 第四步：蓝牙配对

1. 平板进入 **系统设置 → 蓝牙**
2. 搜索设备，找到 `HC-05`
3. 点击配对，密码 `1234`
4. 配对成功后打开 OBD App
5. App 内点击「扫描并连接」

---

## 第五步：接车测试

### 5.1 连接 OBD

- 车辆点火开关打到 **ON**（不启动发动机）
- 将 OBD 接头插入车辆诊断口（通常在方向盘下方）

### 5.2 App 操作

1. 打开 App → 点击「扫描并连接」
2. 连接成功后，点击「读转速」
3. 看到转速数据返回 ✅

### 5.3 验证步骤

| 步骤 | 命令 | 预期结果 |
|------|------|----------|
| 1 | `0100` | 返回支持 PID 列表 |
| 2 | `010C` | 返回发动机转速 |
| 3 | `010D` | 返回车速 |
| 4 | `0105` | 返回水温 |
| 5 | `03` | 返回故障码 |

> 📖 PID 详细说明见 [OBD_PID_REFERENCE.md](OBD_PID_REFERENCE.md)

---

## 调试建议

推荐调试流程（**自下而上，逐层验证**）：

```
Step 1: 面包板回环测试 CAN（不接车）
    ↓ 验证 STM32 + TJA1050 硬件
Step 2: STM32 ↔ HC-05 串口回环
    ↓ PC 串口助手模拟平板，验证蓝牙链路
Step 3: 接真实车辆 OBD，先读 0100
    ↓ 验证车辆 CAN 通信
Step 4: 逐项测试 PID 读取
    ↓ 验证各数据流解析
Step 5: 鸿蒙 App 真机联调
    ↓ 完整体验
```

### PC 端调试工具

```bash
# 用 can_sniffer.py 嗅探 CAN 数据（通过串口）
python tools/can_sniffer.py COM3
```

---

## 🎉 完成！

全部跑通后，你应该能看到：

- ✅ App 显示实时转速、水温、车速
- ✅ 实时曲线动态更新
- ✅ 可读取/清除故障码

**遇到问题？** 查看 [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

---

**祝你玩得开心！🛠️🚗**
