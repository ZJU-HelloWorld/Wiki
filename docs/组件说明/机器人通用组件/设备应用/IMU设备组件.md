# IMU（Inertial Measurement Unit）设备组件

<img src = "https://img.shields.io/badge/version-1.0.0-green"><sp> <img src = "https://img.shields.io/badge/author-Caikunzhen-lightgrey">

## 理论

惯性测量单元(Inertial measurement unit，简称 IMU)，是测量物体三轴姿态角及加速度的装置。一般IMU包括三轴陀螺仪及三轴加速度计，部分IMU还包括三轴磁力计。

### BMI088

![image-20231212151943](IMU设备组件.assets/image-20231212151943.png)

BMI088 是一款能检测运动与旋转共 6 个自由度的 IMU。其将两个惯性传感器的功能包括在了一个设备之中：一个先进的三轴 16-bit 陀螺仪和一个多功能、先进的三轴 16-bit 加速度计。

BMI088 为满足在剧烈震动环境下高性能消费类设备的所有需求，如比那些安装在无人机和机器人上的设备。该 IMU 是为有效抑制高于几百赫兹的震动而设计的，而这些震动往往因 PCB 或系统结构的谐振而产生。

该 IMU 能够提供三轴的加速度与三轴的角速度，同时还能提供温度测量的功能。可以通过 SPI 或 I2C 的协议与该 IMU 进行通信。

- 组件中仅使用 SPI 协议进行通信

> 以 RoboMaster 开发板 C 型为例，板上已集成一个 BMI088，使用 `SPI1` 进行通信需要的硬件配置方式如下图所示：
> ![image-20231212152356](IMU设备组件.assets/image-20231212152356.png)
> 需要配置其中的 CS_Accel、CS1_Gyro、SPI1_CLK、SPI1_MOSI 与 SPI1_MISO 引脚

### 模块化IMU组件说明

模块化 IMU 组件建立在 BMI088 组件以及 AHRS 组件的基础上，将 IMU 的数据预处理、姿态解算、数据读取等功能封装为一个类。

#### 工作状态

```cpp
enum class ImuStatus : uint8_t {
  kHardwareNotInited,  ///< 硬件未初始化
  kCalcingOffset,      ///< 正在计算零漂
  kWorking,            ///< 正常工作
};
```

`ImuStatus` 枚举类用于表示 IMU 目前的工作状态。

在用户调用 `initHardware()` 前，IMU 处于 `kHardwareNotInited` 状态，不计算姿态，也不能读取相关数据。

用户调用 `initHardware()` 后， IMU 进入`kCalcingOffset` 状态，开始计算零漂，根据当前零漂计算结果处理角速度，并输出姿态角等数据。

零漂计算完成后， IMU 进入 `kWorking` 状态，正常输出各项数据。

组件留有 `resetOffset()` 接口，可以让 IMU 回到 `kCalcingOffset` 状态，重新计算零漂。

#### 数据预处理
在正式工作前， IMU 会计算角速度计零漂，具体方法为：

使用均值递推公式$A_{n}=A_{n-1}+\frac{x_n-A_{n-1}}{n}$计算角速度平均值，共计算 `sample_num` 次，作为零漂计算结果。在后续的计算中减去零漂，得到更加准确的角速度数据。

在零漂计算过程中，默认设备处于静止状态，为保证不受到外部晃动的干扰，程序设置了 `gyro_stationary_threshold` 作为阈值，大于该阈值的数据不计入计算次数。

对于加速度数据，设置了 `acc_threshold` 作为阈值，用于过滤短时撞击。

## 快速开始

组件源码仓库地址：<https://github.com/ZJU-HelloWorld/HW-Components>

### 外设驱动配置

在使用 SPI 与 BMI088 通信时，使用 CubeMX 配置时将对应 SPI 的波特率调整至合理值（10Mbits/s 以下）。例如对于 RoboMaster 开发板 C 型板载的红点激光器，驱动流程如下：

* 对于 SPI1，调整器分频值（Prescaler），使其波特率在 10Mbits/s 以下。
* 需改 SPI1 时钟极性（CPOL）与时钟相位（CPHA）至要求值（CPOL=Low，CPHA=1 Edge或CPOL=High，CPHA=2 Edge）

![image-20231212152746](IMU设备组件.assets/image-20231212152746.png)

* 配置片选信号线的 GPIO

![image-20231212153710](IMU设备组件.assets/image-20231212153710.png)

### 使用前准备

本组件依赖 CMSIS-DSP 运算加速。

使用前需要做以下准备：

* 在 `config.cmake` 文件中设置 `use_hwcomponents_devices_imu` 、 `use_hwcomponents_algorithms_ahrs` 和 `use_hwcomponents_tools` 选项为 `ON`，开启该设备文件的编译，若为 `OFF` 则根驱动配置需求在 `CMakeLists.txt` 文件中重配置为 `ON`。

### 使用示例

#### 1.参数配置

在项目中引用头文件：

```cpp
#include "imu.hpp"
```

实例化一个 `ImuConfig` 结构体，该结构体包含了零漂计算、 `BMI088` 和 `Mahony` 的相关参数。其中大部分参数有默认值，`rot_mat_flatten`和`bmi088_hw_config`为必填项，需要根据实际情况进行修改:

```cpp
static const hello_world::imu::ImuConfig kImuConfig = {
    .rot_mat_flatten = {
      1, 0, 0,
      0, 1, 0,
      0, 0, 1},
    .bmi088_hw_config = { // DM-MC-Board02控制板(h7)
        .hspi = &hspi2,
        .acc_cs_port = GPIOC,
        .acc_cs_pin = GPIO_PIN_0,
        .gyro_cs_port = GPIOC,
        .gyro_cs_pin = GPIO_PIN_3,
    },
};
```

如果使用C板，则将`bmi088_hw_config`项设为：

```cpp
.bmi088_hw_config = { // C板
    .hspi = &hspi1,
    .acc_cs_port = GPIOA,
    .acc_cs_pin = GPIO_PIN_4,
    .gyro_cs_port = GPIOB,
    .gyro_cs_pin = GPIO_PIN_0,
},
```
`ImuConfig`具有默认参数，因此可以对某些参数进行个别修改，但是需要保证结构体成员的前后顺序与结构体中的声明顺序相同，完整配置参数如下：

```cpp
static const hello_world::imu::ImuConfig kImuConfig = {
    /*/< 加速度阈值，用于过滤短时撞击，单位：m/s^2 */
    .acc_threshold = 10.0f,
    /*/< 角速度静止阈值，计算零漂时默认设备处于静止状态，滤去异常数据，单位：rad/s */
    .gyro_stationary_threshold = 0.1f,
    .sample_num = 1000,    ///< 零漂采样次数
    .samp_freq = 1000.0f,  ///< 采样频率
    .kp = 1.0f,            ///< Mahony 比例系数
    .ki = 0.0f,            ///< Mahony 积分系数
    .rot_mat_flatten = {
        1, 0, 0,
        0, 1, 0,
        0, 0, 1},
    .bmi088_hw_config = { // DM-MC-Board02控制板(h7)
        .hspi = &hspi2,
        .acc_cs_port = GPIOC,
        .acc_cs_pin = GPIO_PIN_0,
        .gyro_cs_port = GPIOC,
        .gyro_cs_pin = GPIO_PIN_3,
    },
    .bmi088_config = {
        .acc_range = hello_world::imu::kBMI088AccRange3G,
        .acc_odr = hello_world::imu::kBMI088AccOdr1600,
        .acc_osr = hello_world::imu::kBMI088AccOsr4,
        .gyro_range = hello_world::imu::kBMI088GyroRange1000Dps,
        .gyro_odr_fbw = hello_world::imu::kBMI088GyroOdrFbw1000_116,
    },
};
```

#### 2.实例化
实例化一个IMU组件并放入对应配置进行初始化，如：

```cpp
hello_world::imu::Imu unique_imu = hello_world::imu::Imu(kImuConfig);
```

#### 3.初始化
在正式使用前，调用 `initHardware()` 函数进行硬件初始化。

> 该函数内部会阻塞运行，请不要在中断中调用。

#### 4.数据更新
硬件初始化完成后，定期调用 `update()` 函数更新数据，更新周期务必与 `samp_freq` 中配置的相同：

> 如果之前没有调用initHardware函数，该函数会返回false

#### 5.数据获取
在配置成功后，可调用相关接口函数获取姿态、角速度、加速度、温度数据，相关接口如下：

```cpp
/**
  * @brief      判断 IMU 零漂是否计算完成
  * @retval      bool: IMU 零漂是否计算完成
  * @note        None
  */
bool isOffsetCalcFinished() { return status_ == Status::kWorking; };

// 获取姿态角 (ZYX欧拉角)
float roll() const;
float pitch() const;
float yaw() const;

// 获取角速度 (去零漂)
float gyro_roll() const;
float gyro_pitch() const;
float gyro_yaw() const;

// 获取加速度 (过滤短时撞击)
float acc_x() const;
float acc_y() const;
float acc_z() const;

// 获取温度
float temp() const;
```

## 附录

### BMI088 手册

[BMI088 数据手册](IMU设备组件.assets/Bosch-BMI088.pdf)

### 版本说明

| 版本号                                                         | 发布日期   | 说明            | 贡献者 |
| -------------------------------------------------------------- | ---------- | --------------- | ------ |
| <img src = "https://img.shields.io/badge/version-1.0.0-green"> | 2023.12.12 | IMU 组件（Cpp） | 蔡坤镇 |
| <img src = "https://img.shields.io/badge/version-1.0.1-green"> | 2023.12.13 | 添加旋转配置    | 蔡坤镇 |
| <img src = "https://img.shields.io/badge/version-1.1.0-green"> | 2025.01.08 | 模块化组件      | 金乐天 |