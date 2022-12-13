CMSIS-DSP 使用指南

<img src = "https://img.shields.io/badge/version-1.0.0-green"> <sp> <img src = "https://img.shields.io/badge/author-dungloi-lightgrey">

## 简介

CMSIS-DSP 软件库是一套用于基于 Cortex-M 和 Cortex-A 处理器的设备的通用计算处理函数。该库分为多个功能，每个功能涵盖一个特定类别，如：

- 基本数学函数
- 快速数学函数
- 复杂的数学函数
- 过滤功能
- 矩阵函数
- 变换函数
- 电机控制功能
- 统计功能
- 支持功能
- 插值函数
- 支持向量机函数 (SVM)
- 贝叶斯分类器函数
- 距离函数
- 四元数函数

该库通常具有单独的函数来对 8 位整数、16 位整数、32 位整数和 32 位浮点值进行操作。

将 CMSIS-DSP 库搭配具有 FPU 的处理器，将能够提升运算效率。Cortex-M4 内核便具有单精度浮点单元 (FPU)，支持所有 Arm 单精度数据处理指令和所有数据类型。它还实现了全套 DSP（数字信号处理）指令和增强应用程序安全性的内存保护单元 (MPU)。

> 关于 CMSIS 的更多说明，请参考[官方文档](https://arm-software.github.io/CMSIS_5/latest/General/html/index.html). 选择其中的 [CMSIS-DSP 标签页](https://arm-software.github.io/CMSIS_5/latest/DSP/html/index.html) 以查看 CMSIS-DSP 的相关信息。
>
> 请注意，CMSIS-DSP V1.10.1 及其之后的版本迁移至了独立的新仓库，[新文档地址](https://arm-software.github.io/CMSIS-DSP/latest/index.html)



## 软件包下载

[基于 v1.14.2 的裁剪版本（推荐）](https://g6ursaxeei.feishu.cn/wiki/wikcnOU966oKwhSHbul9Tp9pufg)

[最新版本](https://github.com/ARM-software/CMSIS-DSP/releases)



## 使用说明

### 主要版本差异

建议使用最新版本的 CMSIS-DSP 源码加入工程进行编译. 当前最新发布版本为 ![GitHub release (latest by date)](https://img.shields.io/github/v/release/ARM-software/CMSIS-DSP)

* V1.10.1 及之后的版本迁移到新的仓库
* V1.10.0 添加了 `atan2` 支持
* V1.9.0 添加了矩阵向量相乘的功能，添加复数矩阵转置，支持四元数，添加对 `f16` 类型的更多支持
* V1.8.0 添加了支持向量机、贝叶斯函数等等新函数
* V1.6.0 更改了 DSP 文件夹结构
* V1.5.3 删除预编译宏 `__FPU_USED`, `__DSP_PRESENT`

### 引入源码并配置 CMake  

在我们提供的 [CMakeLists模板](https://zju-helloworld.github.io/Wiki/%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/CubeMX%2BVSCode%2BOzone%E9%85%8D%E7%BD%AESTM32%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%28Windows%E7%B3%BB%E7%BB%9F%29/#cmakelists) 中有以下内容：

```cmake
# ########################## USER CONFIG SECTION ##############################
# set up proj
project(your_proj_name C CXX ASM) # TODO
set(CMSISDSP your_dsp_path) # if using DSP, modify your_dsp_path here
			                # e.g. set(CMSISDSP Drivers/CMSIS/DSP)
```

请在这里填写项目名称，以及工程根目录下所使用的 CMSIS-DSP 源码文件夹的相对路径。

> 例如你将 CMSIS-DSP 源码文件夹放在了 `./Drivers/CMSIS/` 目录下，且源码文件夹名为 `CMSIS-DSP-1.14.2`，那么请将 `your_dsp_path` 改为  `Drivers/CMSIS/CMSIS-DSP-1.14.2`

```cmake
# ! rebuild or use command line `cmake .. -D` to switch option
# floating point settings
option(ENABLE_HARD_FP "enable hard floating point" OFF) # TODO
option(ENABLE_SOFT_FP "enable soft floating point" OFF) # TODO
option(USE_NEW_VERSION_DSP "DSP version >= 1.10.0" ON) # TODO
```

请在这里设置浮点选项。若使用具有 FPU 的处理器，将 `ENABLE_HARD_FP` 选项修改为 `ON`，删除 `build` 目录后重新编译；或在 `build` 目录下使用 `cmake .. -DENABLE_HARD_FP=ON` 配置命令行，启用浮点运算单元。

> `ENABLE_SOFT_FP` 则用于兼容没有 FPU 的处理器。

然后选择是否使用 >= 1.10.0 版本的 CMSIS-DSP，选项的修改方式与上文所述类似。新版本的文件包含路径、预编译指令将与老版本有所差异，就像这样：

```cmake
# add inc and src here	
include_directories(
  if(ENABLE_HARD_FP)
  if(USE_NEW_VERSION_DSP)
  ${CMSISDSP}/Include/dsp
  ${CMSISDSP}/Include
  ${CMSISDSP}/PrivateInclude
  else()
  ${CMSISDSP}/Include
  endif()
  endif()
)
```

以上一段配置是 CMSIS-DSP 需要的头文件包含路径，不需要改动。

```cmake
# !! Keep only sub folders required to build and use CMSIS-DSP Library.
# !! If DSP version >= 1.10, for all paths including DSP folders, plz add [^a] to filter DSP files.
# !! e.g. your_dsp_path = Drivers/CMSIS/DSP, use "Drivers/[^a]*.*" "${CMSISDSP}/[^a]*.*" 
file(GLOB_RECURSE SOURCES
  "Core/*.*"
  "Drivers/*.*"
	
  # "${CMSISDSP}/*.*" # uncomment this line when using DSP
  # TODO
)

# #############################################################################
```

最后需要注意的是源文件的选取。我们建议将软件包裁剪后加入工程，可使用上文提供的已裁剪版本，裁剪规则参考源码仓库的 `README.md`：

![image-20221214012731420](CMSIS-DSP%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97.assets/image-20221214012731420.png)

需要注意，对于这样裁剪得到的软件包，直接编译会有大量 `WARNING`，具体原因可参考 [这条经验](https://g6ursaxeei.feishu.cn/wiki/wikcnvTNsHomNrfLE0PVHN5VWhc?field=fldrk77lHy&record=recDLg4nf3&table=tbl5nghP4qHQIiZ5&view=vewlyW2exr)。因此，当添加  CMSIS-DSP 文件夹内的所有文件，及包含有 CMSIS-DSP 文件夹的父目录下的所有文件时，可以加上正则表达式 `[^a]` 来解决 `WARNING`. 

> 例如你之前将 `your_dsp_path` 改为 `Drivers/CMSIS/CMSIS-DSP-1.14.2`，意味着 `Drivers/` 是CMSIS-DSP 文件夹的父目录，那么你可以这样筛选源文件：
>
> ```cmake
>  "Drivers/[^a]*.*"
>  "${CMSISDSP}/[^a]*.*"
> ```



## 附录

### 版本说明

| 版本号                                                | 发布日期   | 说明                           | 贡献者 |
| ----------------------------------------------------- | ---------- | ------------------------------ | ------ |
| ![hh](https://img.shields.io/badge/version-1.0.0-green) | 2022.12.14 | 首次发布                       | 薛东来 |
