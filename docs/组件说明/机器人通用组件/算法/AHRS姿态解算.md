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

* 在 `config.cmake` 文件中设置 `use_hwcomponents_algorithms_ahrs` 选项为 `ON`，开启该设备文件的编译，若为 `OFF` 则根驱动配置需求在 `CMakeLists.txt` 文件中重配置为 `ON`。

### 示例

在项目中引用头文件：

```cpp
#include "ahrs.hpp"
```

实例化一个 AHRS 姿态解算并根据需要进行配置：

```cpp
namespace hw_ahrs = hello_world::ahrs;

hw_ahrs::Ahrs* ahrs_ptr = nullptr;
float samp_freq = 1000.0f, kp = 1.0f, ki = 0.0f;
ahrs_ptr = new hw_ahrs::Mahony(samp_freq, kp, ki);
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

## 附录

### 版本说明

| 版本号                                                       | 发布日期   | 说明               | 贡献者 |
| ------------------------------------------------------------ | ---------- | ------------------ | ------ |
| <img src = "https://img.shields.io/badge/version-1.0.0-green"> | 2023.12.15 | 发布 AHRS 组件（Cpp） | 蔡坤镇 |