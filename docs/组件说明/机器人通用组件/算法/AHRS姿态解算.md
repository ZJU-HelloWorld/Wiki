# AHRS 姿态解算
 
<img src = "https://img.shields.io/badge/version-1.0.0-green"> <sp> <img src = "https://img.shields.io/badge/author-Caikunzhen-lightgrey"> 
 
## 理论
AHRS（Attitude and Heading Reference System）俗称航姿参考系统，AHRS由加速度计，磁场计，陀螺仪构成，能够为飞行器提供航向（yaw），横滚（roll）和侧翻（pitch）信息，这类系统用来为飞行器提供准确可靠的姿态与航行信息。

### Mahony 算法
Mahony 算法是常见的姿态融合算法，将加速度计，磁力计，陀螺仪共九轴数据，融合解算出机体四元数。

具体理论推导可查看文章 [Mahony姿态解算算法笔记（一）](https://zhuanlan.zhihu.com/p/342703388)

## 快速开始

组件源码仓库地址：<https://github.com/ZJU-HelloWorld/HW-Components>

### 使用前准备
 
本组件依赖 CMSIS-DSP 运算加速。
 
使用本组件前需要做以下准备：

* 在使用 STM32CubeMX 生成项目时，请在 Code Generator 界面 Enable Full Assert，来帮助断言设备驱动中的错误；在 `main.c` 中修改 `assert_failed` 函数以指示断言结果，如添加 `while(1);`
* 在 `config.cmake` 文件中设置 `use_hwcomponents_algorithms_ahrs` 选项为 `ON`，开启该设备文件的编译

### 示例

在项目中引用头文件：

```cpp
#include "ahrs.hpp"
```

实例化一个 AHRS 姿态解算并根据需要进行配置：

```cpp
namespace ahrs = hello_world::algorithms::ahrs;

ahrs::Ahrs* ahrs_ptr = nullptr;
float samp_freq = 1000.0f, kp = 1.0f, ki = 0.0f;
ahrs_ptr = new ahrs::Mahony(samp_freq, kp, ki);
```

以固定频率调用 `update` 方法以计算当前姿态，足以调用频率需要与初始化时提供的 `samp_freq` 一致。

```cpp
/* acc_data: 加速度数据 gyro_data: 角速度数据 */
ahrs_ptr->update(acc_data, gyro_data);
```

可以通过 `getEulerAngle` 方法获取姿态对应的欧拉角（Z-Y-X）：

```cpp
float euler_angle[3]; // [roll pitch yaw]
ahrs_ptr->getEulerAngle(euler_angle);
```

### 组件说明

#### `Ahrs` 虚基类

##### public

| 名称<img width=250/> | 参数说明                                                     | 描述                                  |
| :------------------ | :----------------------------------------------------------- | ------------------------------------- |
|`Ahrs`|/|AHRS 初始化|
|`Ahrs`|`quat_init`: 初始化四元数，[$q_w$ $q_x$ $q_y$ $q_z$]|AHRS 初始化，初始化四元数需要为单位四元数|
|`~Ahrs`|/|AHRS 析构函数|
|`update`|`acc_data`: 加速度计三轴数据，[$a_x$ $a_y$ $a_z$]，无单位要求</br>`gyro_data`: 陀螺仪三轴数据，[$\omega_x$ $\omega_y$ $\omega_z$]，单位：$\rm{rad/s}$|根据反馈数据进行姿态更新，加速度计三轴数据需包含重力加速度项|
|`getQuat`|`quat`: 当前姿态对应的四元数|获取当前姿态对应的四元数|
|`getEulerAngle`|`euler_angle`: 当前姿态对应的欧拉角（Z-Y-X），[roll pitch yaw]，单位：$\rm{rad}$|获取当前姿态对应的欧拉角（Z-Y-X）|

##### protected

属性

| 名称          | 类型         | 示例值    | 描述               |
| :------------ | :----------- | :-------- | :----------------- |
| `quat_`       | `float*`    | [1 0 0 0] | 当前姿态对应的四元数 |

方法

| 名称<img width=250/> | 参数说明                                                     | 描述                                  |
| :------------------ | :----------------------------------------------------------- | ------------------------------------- |
|`invSqrt`|`x`: 数（>0）|快速计算 $\cfrac{1}{\sqrt x}$|

#### `Mahony` 类（继承：`Ahrs`）

##### public

| 名称<img width=250/> | 参数说明                                                     | 描述                                  |
| :------------------ | :----------------------------------------------------------- | ------------------------------------- |
|`Mahony`|`samp_freq`: 采样频率，单位：$\rm{Hz}$</br>`kp`: 比例系数（>=0）</br>`ki`: 积分系数（>=0）|Mahony 初始化|
|`Mahony`|`quat_init`: 初始化四元数，[$q_w$ $q_x$ $q_y$ $q_z$]</br>`samp_freq`: 采样频率，单位：$\rm{Hz}$</br>`kp`: 比例系数（>=0）</br>`ki`: 积分系数（>=0）|Mahony 初始化|
|`Ahrs`|`quat_init`: 初始化四元数，[$q_w$ $q_x$ $q_y$ $q_z$]|AHRS 初始化，初始化四元数需要为单位四元数|
|`~Mahony`|/|Mahony 析构函数|
|`update`|`acc_data`: 加速度计三轴数据，[$a_x$ $a_y$ $a_z$]，无单位要求</br>`gyro_data`: 陀螺仪三轴数据，[$\omega_x$ $\omega_y$ $\omega_z$]，单位：$\rm{rad/s}$|根据反馈数据进行姿态更新，加速度计三轴数据需包含重力加速度项|

##### private

属性

| 名称          | 类型         | 示例值    | 描述               |
| :------------ | :----------- | :-------- | :----------------- |
| `samp_freq_`       | `float`    | 1000.0f | 采样频率 |
| `dbl_kp_`       | `float`    | 2.0f | 2 * kp |
| `dbl_ki_`       | `float`    | 0.0f | 2 * ki |
| `i_out_`       | `float*`    | / | 积分项输出 |

## 附录

### 版本说明

| 版本号                                                       | 发布日期   | 说明               | 贡献者 |
| ------------------------------------------------------------ | ---------- | ------------------ | ------ |
| <img src = "https://img.shields.io/badge/version-1.0.0-green"> | 2023.12.15 | 发布 AHRS 组件（Cpp） | 蔡坤镇 |