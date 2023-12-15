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

* 在使用 STM32CubeMX 生成项目时，请在 `Code Generator` 界面 `Enable Full Assert`，来帮助断言设备驱动中的错误；在 `main.c` 中修改 `assert_failed` 函数以指示断言结果，如添加 `while(1);`
* 在 `config.cmake` 文件中设置 `use_hwcomponents_devices_imu` 选项为 `ON`，开启该设备文件的编译

### 示例

在项目中引用头文件：

```cpp
#include "imu.hpp"
```

实例化一个 BMI088 硬件配置结构体并进行相关配置，同时根据安装方式设定旋转矩阵，如：

```cpp
imu::BMI088HWConfig hw_config = {
  .hspi = &hspi1,
  .acc_cs_port = GPIOA,
  .acc_cs_pin = GPIO_PIN_4,
  .gyro_cs_port = GPIOB,
  .gyro_cs_pin = GPIO_PIN_0,
};

float rot_mat_flatten[9] = {
    1, 0, 0,
    0, 1, 0,
    0, 0, 1};
```

实例化一个 BMI088 组件并放入对应硬件配置进行初始化，如：

```cpp
namespace imu = hello_world::devices::imu;

imu::BMI088* imu_ptr = nullptr;
imu_ptr = new imu::BMI088(hw_config, rot_mat_flatten);
```

当有特殊需求时，还可以调整 BMI088 的配置，如：

```cpp
imu::BMI088Config config = {
    .acc_range = imu::kBMI088AccRange3G,
    .acc_odr = imu::kBMI088AccOdr1600,
    .acc_osr = imu::kBMI088AccOsr4,
    .gyro_range = imu::kBMI088GyroRange1000Dps,
    .gyro_odr_fbw = imu::BMI088GyroOdrFbw1000_116,
};

imu_ptr = new imu::BMI088(hw_config, rot_mat_flatten, config);
```

- `BMI088Config` 具有默认参数，因此可以在需要对某些默认参数进行修改时进行个别修改，但是需要保证前后结构体成员前后的复制顺序需要与结构体中的声明顺序相同

然后调用方法 `init` 对 BMI088 的进行配置, 如：

```cpp
while (imu_ptr->init() != imu::kBMI088ErrStateNoErr) {
}
```

在配置成功后，可调用方法 `getData` 获取传感器数据，如：

```cpp
float gyro_data[3], acc_data[3], temp;
imu_ptr->getData(acc_data, gyro_data, &temp);
```

- 可以传入空指针 `nullptr` 作为参数，此时不会读取对应的数据

### 组件说明

#### `BMI088HWConfig` 结构体

BMI088 硬件配置

| 名称           | 类型       | 示例值 | 描述                                 |
| :------------- | :--------- | :----- | :----------------------------------- |
|`hspi`|`SPI_HandleTypeDef*`|`&hspi1`|IMU 对应的 SPI 句柄的指针|
|`acc_cs_port`|`GPIO_TypeDef*`|`GPIOA`|加速度计片选端口|
|`acc_cs_pin`|`uint32_t`|`GPIO_PIN_4`|加速度计片选端口|
|`gyro_cs_port`|`GPIO_TypeDef*`|`GPIOB`|陀螺仪片选端口|
|`gyro_cs_pin`|`uint32_t`|`GPIO_PIN_0`|陀螺仪片选引脚|

#### `BMI088Config` 结构体

BMI088 硬件配置

| 名称           | 类型       | 示例值 | 描述                                 |
| :------------- | :--------- | :----- | :----------------------------------- |
|`acc_range`|`BMI088AccRange`|`kBMI088AccRange3G`|BMI088 加速度计量程，单位：$\rm g$|
|`acc_odr`|`BMI088AccOdr`|`kBMI088AccOdr1600`|BMI088 加速度计输出频率（Output Data Rate），单位：$\rm{Hz}$|
|`acc_osr`|`BMI088AccOsr`|`kBMI088AccOsr4`|加速度计过采样率（Oversampling Rate）|
|`gyro_range`|`BMI088GyroRange`|`kBMI088GyroRange1000Dps`|BMI088 陀螺仪量程，单位：$\rm{deg/s}$|
|`gyro_odr_fbw`|`BMI088GyroOdrFbw`|`kBMI088GyroOdrFbw1000_116`|BMI088 陀螺仪输出频率（Output Data Rate）与滤波器带宽（Filter Bandwidth），单位：$\rm{Hz}$|

#### `BMI088` 类

##### public

属性

| 名称                    | 类型            | 示例值      | 描述                                                      |
| :---------------------- | :-------------- | :---------- | :-------------------------------------------------------- |
|`kHspi_`|`SPI_HandleTypeDef* const`|`&hspi1`|IMU 对应的 SPI 句柄的指针|
|`kAccCsPort_`|`GPIO_TypeDef* const`|`GPIOA`|加速度计片选端口|
|`kAccCsPin_`|`const uint32_t`|`GPIO_PIN_4`|加速度计片选端口|
|`kGyroCsPort_`|`GPIO_TypeDef* const`|`GPIOB`|陀螺仪片选端口|
|`kGyroCsPin_`|`const uint32_t`|`GPIO_PIN_0`|陀螺仪片选引脚|

方法

| 名称 <img width=250/>            | 参数说明                                                              | 描述                                                                                                 |
| :--------------- | :-------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
|`BMI088`|`hw_config`: 硬件配置</br>`config`: 设备配置</br>`rot_mat_flatten`: 旋转矩阵（展平）[$r_{0,0}$ $r_{0,1}$ $r_{0,2}$ $r_{1,0}$ $r_{1,1}$ $r_{1,2}$ $r_{2,0}$ $r_{2,1}$ $r_{2,2}$]，使用后可释放|BMI088 初始化|
|`~BMI088`|/|BMI088 析构函数|
|`setInput`|`input`: 发给电调的期望值|设定发给电调的期望值|
|`init`|`self_test`: 是否开启传感器自检测</br>返回: 错误状态|进行 BMI088 芯片配置，该函数内部会阻塞运行，请不要在中断中调用|
|`getData`|`acc_data`: 加速度计三轴数据，[$a_x$ $a_y$ $a_z$]，单位：$\rm{m/s^2}$</br>`gyro_data`: 陀螺仪三轴数据，[$\omega_x$ $\omega_y$ $\omega_z$]，单位：$\rm{rad/s}$</br>`temp_ptr`: 温度数据指针，单位：℃|获取传感器数据，传入 `null_ptr` 则不获取对应数据|

##### private

属性

| 名称                    | 类型            | 示例值      | 描述                                                      |
| :---------------------- | :-------------- | :---------- | :-------------------------------------------------------- |
|`config_`|`BMI088Config`|/|BMI088 设备配置|
|`rot_mat_flatten_`|`float*`|/|展平的旋转矩阵|
|`rot_mat_`|`arm_matrix_instance_f32`|/|旋转矩阵|

方法

| 名称             | 参数说明                                                              | 描述                                                                                                 |
| :--------------- | :-------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
|`accCsL`|/|加速度计片选信号拉低|
|`accCsH`|/|加速度计片选信号拉高|
|`gyroCsL`|/|陀螺仪片选信号拉低|
|`gyroCsH`|/|陀螺仪片选信号拉高|
|`accInit`|`self_test`: 是否进行自检测</br>返回: 错误状态|加速度计配置|
|`gyroInit`|`self_test`: 是否进行自检测</br>返回: 错误状态|陀螺仪配置|
|`accSelfTest`|返回: 错误状态|加速度计自检测|
|`gyroSelfTest`|返回: 错误状态|陀螺仪自检测|
|`getGyroData`|`gyro_data`: 陀螺仪三轴数据，[$\omega_x$ $\omega_y$ $\omega_z$]，单位：$\rm{rad/s}$|获取陀螺仪数据|
|`getAccData`|`acc_data`: 加速度计三轴数据，[$a_x$ $a_y$ $a_z$]，单位：$\rm{m/s^2}$|获取加速度计数据|
|`getTemp`|返回: 温度，单位：℃|获取温度数据|
|`gyroWrite`|`mem_addr`: 寄存器地址</br>`value`: 待写入数据|向陀螺仪寄存器写入数据|
|`读取陀螺仪寄存器数据`|`mem_addr`: 寄存器地址</br>返回: 读取数据|读取陀螺仪寄存器数据|
|`gyroMultiRead`|`start_mem_addr`: 寄存器起始地址</br>`len`: 带读取数据长度</br>`rx_data`: 读取得到的数据|陀螺仪多寄存器数据读取|
|`accWrite`|`mem_addr`: 寄存器地址</br>`value`: 待写入数据|向加速度计寄存器写入数据|
|`accRead`|`mem_addr`: 寄存器地址</br>返回: 读取数据|读取加速度计寄存器数据|
|`accMultiRead`|`start_mem_addr`: 寄存器起始地址</br>`len`: 带读取数据长度</br>`rx_data`: 读取得到的数据|加速度计多寄存器数据读取|

#### 内部函数
| 名称 <img width=250/>            | 参数说明                                                              | 描述                                                                                                 |
| :--------------- | :-------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
|`DelayUs`|`us`: 需要延时的时间，单位：$\rm{\mu s}$|微秒级延时|

## 附录

### BMI088 手册

[BMI088 数据手册](IMU设备组件.assets/Bosch-BMI088.pdf)

### 版本说明

| 版本号                                                       | 发布日期   | 说明               | 贡献者 |
| ------------------------------------------------------------ | ---------- | ------------------ | ------ |
| <img src = "https://img.shields.io/badge/version-1.0.0-green"> | 2023.12.12 | IMU 组件（Cpp） | 蔡坤镇 |
| <img src = "https://img.shields.io/badge/version-1.0.1-green"> | 2023.12.13 | 添加旋转配置 | 蔡坤镇 |