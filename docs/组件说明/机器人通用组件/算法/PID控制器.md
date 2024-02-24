# PID 控制器

## 标准 PID

PID（_Proportional-Integral-Derivative_）控制器是一种线性控制器，根据被控对象给定值 $r(t)$ 和实际值 $y(t)$ 的控制偏差

$$
e(t)=r(t)-y(t)
$$

构成控制律

$$
u(t)=K_c[e(t)+{1 \over {T_i}}\int ^t_0e(t){\mathrm{d}}t+T_d{\frac{\mathrm de(t)}{\mathrm dt}}]
$$

或以传递函数表示

$$
G(s)=\frac{U(s)}{E(s)}=K_c(1+\frac1{T_is}+T_ds)
$$

其中积分分量能够提升系统的稳态性能；微分分量能够改善系统的动态性能。

在数字 PID 控制中，使用的是离散化的 PID 控制器。在模拟 PID 的理论基础上，以一系列采样时刻点 $kT$ 代表连续时间 $t$，以矩形法数值积分近似代替积分，以一阶后向差分近似替代微分，可得离散 PID 表达式

$$
\begin{align}
u(k)&=K_p[e(k)+\frac{1}{T_i}\sum^k_{j=0}e(j)T+T_d\frac{e(k)-e(k-1)}{T}]\\
&=K_pe(k)+K_i\sum^k_{j=0}e(j)T+K_d\frac{e(k)-e(k-1)}{T}
\end{align}
$$

在标准的数字 PID 控制器的基础上，还可以引入一系列优化算法，形成非标准控制算法，以改善系统品质，满足不同控制系统的需要。

## PID 优化

### 积分项优化

#### 无扰动操作（Bumpless Operation）

无扰动操作包括参数初始化过程和变参数的情况。考虑在控制器工作过程中改变积分增益参数，可能引起输出的较大变化，不利于控制。因此将积分项改进为积分增益与误差乘积的累积，而不是误差累积后再乘以积分增益。积分项表示为：

$$
u_i(k)=\sum^k_{j=0}K_ie(j)T
$$

#### 梯形积分（Trapezoidal Rule）

为尽量减小余差，提高积分项的运算精度，可将矩形积分改为梯形积分。积分项表示为：

$$
u_i(k)=\sum^k_{j=0}K_i{e(j)+e(j-1)\over 2}T \\
$$

#### 抗积分饱和（Anti-windup）

积分饱和现象指若系统存在一个方向的偏差，控制器输出由于积分作用的不断累加而持续增大，若超出执行机构正常运行范围便进入了饱和区。一旦系统出现反向偏差，控制器输出逐渐从饱和区退出，进入饱和区越深则退出所需时间越长。在这段时间内，执行机构仍停留在极限位置，而不能随偏差反向立即做出相应的改变，此时将造成控制性能恶化。抗积分饱和法在计算控制器输出 $u(k)$ 时，首先判断上一时刻 $u(k-1)$ 是否已超出限制范围：若 $u(k-1) > \epsilon_u$，只累加负偏差；若 $u(k-1) < \epsilon_l$，只累加正偏差。积分项表示为：

$$
\begin{align}
u_i(k)=\sum^{k-1}_{j=0}K_ie(j)T+\alpha K_i e(k)T\\
其中\ \alpha=
\begin{cases}
0, &u(k-1)\cdot e(k)>0\ ,u(k-1) \notin [\epsilon_l,\epsilon_u]\\
1, &else
\end{cases}
 \end{align}
$$

其中阈值区间 $[\epsilon_l,\epsilon_u]$ 根据实际情况人为指定。

#### 积分分离（Integration Separation）

积分环节的作用主要为消除静差，提高控制精度。但在短时间系统输出产生较大偏差时，可能会造成积分过度积累，使控制量过大，引起系统超调甚至振荡。此时需要引入积分分离，当被控量与给定值偏差较大时，取消积分作用；接近时引入积分控制。积分项表示为：

$$
\begin{align}
u_i(k)=\beta \sum^k_{j=t_0}K_ie(j)T \\
其中\ \beta=
\begin{cases}
1, & e(k) \in [\epsilon_l,\epsilon_u] \\
0, & else
\end{cases}
  \end{align}
$$

其中 $t_0$ 表示引入积分控制的时刻，阈值区间 $[\epsilon_l,\epsilon_u]$ 根据实际情况人为指定。

#### 变速积分（Changing Rate）

基于积分分离优化的思想，根据系统偏差大小改变积分的速度，偏差越大，积分越慢，反之则越快，形成连续的变化过程。为此，设置系数 $f[e(k)]$，当 $|e(k)|$ 增大时，$f$ 减小，反之增大，其值在 $[0,1]$ 变化。积分项表示为：

$$
u_i(k)=\sum^{k-1}_{j=0}K_ie(j)T+f[e(k)]\cdot K_i  e(k)T\\
$$

系数 $f$ 与当前偏差 $e(k)$ 的关系可以是线性的，可设为

$$
f[e(k)]=
\begin{cases}
1, &|e(k)| {\leqslant} \epsilon_l\\
\frac{\epsilon_u - |e(k)| }{\epsilon_u-\epsilon_l},& \epsilon_l< |e(k)|{\leqslant} \epsilon_u\\
0, &|e(k)| >\epsilon_u
\end{cases}
$$

其中阈值 $ 0<= \epsilon_l < \epsilon_u $ 根据实际情况人为指定。

### 微分项优化

#### 只对输出微分（Derivative of the process variable）

为避免由于给定值频繁升降（尤其是阶跃）而引起的系统振荡，可采用只对输出微分的优化算法，其特点是只对系统输出进行微分。这样，在改变给定值时，微分项输出仅与被控量的变化相关，而这种变化通常是比较缓和的，从而能够明显改善系统的动态特性。此方案也称为“微分先行”。

简单的离散形式优化的微分项表示为：

$$
u_d(k)=K_d\frac{y(k)-y(k-1)}{T_i}\\
$$

也可以通过权重值将误差微分和输出微分结合起来：

$$
u_d(k)=K_d\frac{ (1-w_0) \cdot (e(k)-e(k-1))   + w_0 \cdot (y(k)-y(k-1))}{T_i}\\
$$

其中 $w_0 \in [0,1] $。

#### 微分滤波（Derivative Filter）

微分项可改善系统的动态特性，但也易引进高频干扰，在误差扰动突变时尤其显出微分项的不足。因此，可以在控制算法中加入一阶惯性环节（低通滤波器），可使系统性能得到改善。可将滤波器直接加在微分环节上或控制器输出上。此方案也称为“不完全微分”。

事实上，还可以针对系统的频域特性设计合适的滤波器，这里不做更深入的探究。

对微分环节输出进行一阶滤波，微分项表示为：

$$
u_d(k)=
\begin{cases}
  (1-w_0)\cdot u_d(k) +w_0 \cdot u_d(k-1),& u_d(k-1) \notin [\epsilon_l,\epsilon_u] \\
  u_d(k), & others
\end{cases}
$$

其中系数 $w_0\in [0,1]$ 和阈值区间 $[\epsilon_l,\epsilon_u]$ 根据实际情况人为指定。

### 其他优化

#### 时间间隔自动采样（Automatic Time Interval Sampling）

理论上离散 PID 的采样时间间隔是一致的，但实际应用中，由于资源有限或任务过重，PID 每次计算之间的时间间隔可能不是一致的。因此，在调用 PID 进行计算时，将自动获取和记录计算时刻，来计算每次的采样间隔 $T_i$ ，并用于积分项计算和微分项计算。

#### 角度最小差值（Period-Sub）

由于角度是周期性的，所以从角度 $A$ 到角度 $B$ 可以有两种路径，两种路径往往一长一短，其中较短路径是令人感兴趣的。由于角度分弧度值和角度值以及其周期性的特性，这种最短路径特性可以扩大到任意周期上。因此，周期性由圆来代表。周期的大小用圆上刻度来表示，当周期大小为0，即圆上没有刻度时，采用直接相减的方式。

对于角度的控制，需要考虑是否选择最短的变换路径。在 PID 的计算过程中，这一点可以通过更改误差值 $e(j)$ 的计算方式来实现：

$$
e(k) = \begin{cases}
 r(k)-y(k) & \text{ if } period=0 \\
 (r(k)-y(k)) \mod{period}  & \text{ if } period \ne 0
\end{cases}
$$

#### 带死区（Dead Band）

为避免控制作用过于频繁，消除由于频繁动作所引起的振荡，必要时可采用带死区的 PID 控制算法，即判断控制偏差是否小于给定阈值，若小于则不输出。当阈值过大时，在阈值点的输出会从 0 突变，对输出造成干扰。更一般的，给定阈值区间 $[\epsilon_l,\epsilon_u]$，当控制偏差在阈值区间之外时，正常输出；当控制偏差在 $[\epsilon_l+ \delta, \epsilon_u -\delta)$ 时，输出为 $({\epsilon_l+\epsilon_u})/{2} $；当控制偏差在 $[\epsilon_l,\epsilon_l+ \delta) \cup  [\epsilon_u -\delta, \epsilon_u]$ 时，进行插值处理。控制器输出表示如下：

$$
u(k)=
\begin{cases}
u(k), & \epsilon_u<=e(k)\\
\frac{\epsilon_l+\epsilon_u}{2} +\frac{u(k)- \epsilon_u +\delta }{\delta}\frac{\epsilon_u-\epsilon_l}{2},& \epsilon_u-\delta \le e(k)<\epsilon_u \\
({\epsilon_l+\epsilon_u})/{2} ,&  \epsilon_l+ \delta \le e(k) < \epsilon_u -\delta\\
\frac{\epsilon_l+\epsilon_u}{2} +\frac{u(k) - \epsilon_l -\delta }{\delta}\frac{\epsilon_u-\epsilon_l}{2} , &  \epsilon_l \le e(k) < \epsilon_l + \delta\\
u(k), & e(k) < \epsilon_l
\end{cases}
$$

其中阈值区间 $[\epsilon_l, \epsilon_u]$ 根据实际情况人为指定，$\delta=(\epsilon_u-\epsilon_l)/10$。

#### 给定值平滑（Setpoint Ramping）

当给定值出现较大的阶跃变化，很容易引起超调。使用线性斜坡函数或一阶滤波为给定值安排过渡过程，使其从其旧值逐渐变化到新值，以避免阶跃变化产生的不连续性，进而使对象运行平稳，适用于高精度伺服系统的位置跟踪。

实际应用中，一般给出最大阶跃范围。若超出该范围，对给定值进行一阶滤波处理。给定值表示如下：

$$
r(k)=
\begin{cases}
r(k), &|e(k)| {\leqslant} \epsilon_2\\
w_1\cdot r(k)+(1-w_1)r(k-1), &|e(k)| {>} \epsilon_2
\end{cases}
$$

其中阈值 $\epsilon_2$ 及系数 $w_1\in (0,1]$ 根据实际情况人为指定。

#### 前馈校正（Feed-forward）

在前馈控制（开环）中考虑系统的已知信息，再将输出加到 PID 控制器（闭环）的控制输出，形成复合校正，能够进一步提升整体的系统性能。前馈校正通路由于不受反馈的影响，不会造成系统的振荡，从而能够在不影响稳定性的情况下改善系统的响应。前馈量通常可以单独提供控制器输出的主要部分；PID 控制器则用来补偿给定值和实际值之间的误差。

前馈量依可量测扰动或给定量来产生，构成按扰动补偿和按输入补偿两种复合控制形式。

<figure markdown>
![按扰动补偿的前馈](./PID控制器.assets/按扰动补偿.png){width=60%}
<figcaption>(a)按扰动补偿的前馈</figcaption>
</figure>

<figure markdown>
![按输入补偿的前馈](./PID控制器.assets/按输入补偿.png){width=60%}
<figcaption>(b)按输入补偿的前馈</figcaption>
</figure>



我们重点关注按输入补偿的复合控制系统设计。由上图 (b) ，系统输出为
$$
C(s)=\frac{[G_1(s)+G_r(s)]G_2(s)}{1+G_1(s)G_2(s)}R(s)
$$
如果选择前馈补偿装置传函 $G_r(s)=\frac1{G_2(s)}$，则有
$$
C(s)=R(s)
$$
使得系统输出复现输入，具有理想的时间响应特性。实际这种全补偿难以实现，因此一般采用部分补偿，常用的方法有取输入信号的一阶导数作为前馈补偿信号，即：
$$
G_r(s)=\lambda_1s
$$
此方案原理可参考资料[2]第283页。实际中由于控制器工作频率可能高于给定信号频率，此时若采用后向差分则不能获得平滑的前馈量。可以考虑采用跟踪微分器。


## 组件构成

PID 组件提供 `BasicPid` 和 `MultiNodesPid`  两个类来实现 PID 控制器。

<figure markdown>
![BasicPid 的结构](./PID控制器.assets/BasicPid.png){width=60%}
<figcaption>BasicPid 的结构</figcaption>
</figure>

`BasicPid` 集成了多种优化方法，可以单独使用。具体实现的方式如图所示。可以根据实际情况，配置 `BasicPid` 的参数来获取满足要求的控制效果。

<figure markdown>
![MultiNodesPid 串行计算时的数据传输方式](./PID控制器.assets/串行.png){width=60%}
<figcaption>(a)MultiNodesPid 串行计算时的数据传输方式</figcaption>
</figure>

<figure markdown>
![MultiNodesPid 并行计算时的数据传输方式](./PID控制器.assets/并行.png){width=60%}
<figcaption>(b)MultiNodesPid 并行计算时的数据传输方式</figcaption>
</figure>


`MultiNidesPid` 内含以 `BasicPid` 作为节点的容器，通过不同的数据传输方式，可以实现串行或并行计算，具体如图所示。用户可以自行对 `BasicPid` 进行二次开发。


## 快速开始

### BasicPid

#### 实例化一个 PID 控制器

创建一个使用默认参数的 PID 控制器实例：

```cpp
pid::BasicPid pid;
```

创建一个指定 $K_p$ , $K_i$ , $K_d$ 而其他参数默认的 PID 控制器实例：

```cpp
pid::BasicPid pid(0.6f, 0.1f, 0.0f);
```

创建一个指定所有参数的 PID 控制器实例：

```cpp
const pid::BasicPid::Params kBasicPidParams={  
  .auto_reset =false, // 不开启自动清零  
  .kp = 0.6f,  
  .ki = 0.0f,  
  .kd = 0.0f,  
  .max_interval_ms = 5, // 两次调用间隔超过5ms，认为采样异常，若此时开启自动清零，则会将 PID 动态数据重置为初始值  
  .out_limit = pid::OutLimit(true, -100.0f, 100.0f),  // 开启输出限幅，限幅范围为[-100, 100]  
  // 其他优化项目
};
```

为了减少内存占用，建议使用常量或局部变量。
尽管 `BasicPid::Params` 类型与 `BasicPidParams` 类型相同，但还是建议使用 `BasicPid::Params` 类型。

#### 修改实例化后的参数

提供了修改参数的接口 `params()`。

```cpp
// 单个参数的修改
pid.params().kp = 10; // 设置比例系数为 10  
pid.params().auto_reset = true; // 开启自动清零  
pid.params().inte_anti_windup.setParams(true ,-0.5, 0.5); // 开启积分抗饱和优化  
// 一次性修改所有参数  
const pid::BasicPidParams::Params params = ...;  
pid.params() = params;   
```

> 注意：一次性修改所有参数的用法中，如果 pid 中已经修改了部分参数，但 params 中未作对应的修改，即 params 采用默认值，那么 pid 的参数将会被覆写成默认值

#### 调用 `calc` 函数进行计算

 ```cpp
 namespace pid = hello_world::pid;  
 pid::BasicPid pid = ...;  
 float ref = ...; // 参考值  
 float fdb = ...; // 反馈值  
 float ffd = ...; // 前馈值  
 float out = ...; // 存储输出值  
 // 不使用前馈值进行计算  
 pid.calc(&ref, &fdb, nullptr, &out);  
 // 使用前馈值进行计算  
 pid.calc(&ref, &fdb, &ffd, &out);  
```

#### 监视数据

提供了数据查看接口 `datas()`。

```cpp
 namespace pid = hello_world::pid;  
 pid::BasicPid pid = ...;  
 pid::BasicPid::Datas debug_datas;  
 void Task(){  
   pid.calc(...);  
   debug_datas=pid.datas();  
 };
```

### MultiNodesPid

`MultiNodesPid` 依赖 `std::list` 实现，提供部分容器操作接口。为更好使用该类，请优先阅读`std::list` 的相关事项。

#### 实例化与参数配置

提供了四种构造函数，通用参数为多节点 PID 控制器的类型和输出限幅，不同的是指定节点数量和初始化数据的方式。其中，多节点 PID 控制器的输出限幅与其内部节点 PID 的输出限幅完全无关，即，其内部节点 PID 的输出限幅需要单独设置。

```cpp
namespace pid = hello_world::pid;
// 实例化三节点的串行 PID 控制器，多节点 PID 控制的输出限制为 -16000 ~ 16000，内部节点参数均为默认值
pid::MultiNodesPid cascade_pid(pid::MultiNodesPidType::kCascade, pid::OutLimit(true，-16000, 16000), 3);
// 节点参数修改
multi_nodes_pid.paramsAt(0).kp = 0.6;
multi_nodes_pid.paramsAt(1).kp = 0.6;
multi_nodes_pid.paramsAt(2) = {
  .auto_reset = false,
  .out_limit = pid::OutLimit(true,-7, 7), 
};
// 实例化三节点的并行 PID 控制器，多节点 PID 控制的输出限制为 -16000 ~ 16000，内部节点参数在初始化时指定
const pid::MultiNodesPid::Params params_arr[3] = {{...},{...},{...}};
pid::MultiNodesPid parallel_pid(pid::MultiNodesPidType::kParallel, pid::OutLimit(true，-16000, 16000),pid::MultiNodesPid::ParamsList(params_arr,params_arr+3));
```

#### 计算

```cpp
 
namespace pid = hello_world::pid;  
pid::MultiNodesPid two_nodes_cascade_pid = ...;  
pid::MultiNodesPid two_nodes_parallel_pid = ...;  
void CascadeCalc(){  
  float ref[1] = ...; // 参考值，请注意参考值的数组大小需要大于 1 ，否则会出现数组越界  
  float fdb[2] = ...; // 反馈值，请注意参考值的数组大小需要大于等于节点数量，否则会出现数组越界  
  float ffd = ...; // 前馈值  
  float out = ...; // 存储输出值  
  // 不使用前馈值进行计算  
  two_nodes_cascade_pid.calc(ref, fdb, nullptr, &out);  
  // 使用前馈值进行计算  
  two_nodes_cascade_pid.calc(ref, fdb, &ffd, &out);  
  // 或者使用接口: cascadeCalc  
};

void ParallelCacl(){
  float ref[2] = ...; // 参考值，请注意参考值的数组大小需要大于等于节点数量，否则会出现数组越界    
  float fdb[2] = ...; // 反馈值，请注意参考值的数组大小需要大于等于节点数量，否则会出现数组越界  
  float ffd = ...; // 前馈值  
  float out = ...; // 存储输出值  
  // 不使用前馈值进行计算  
  two_nodes_parallel_pid.calc(ref, fdb, nullptr, &out);  
  // 使用前馈值进行计算  
  two_nodes_parallel_pid.calc(ref, fdb, &ffd, &out);  
  // 或者使用接口: parallelCalc  
};
```

#### 数据监视

```cpp
 
namespace pid = hello_world::pid;  
pid::MultiNodesPid multi_nodes_pid = ...;  
pid::MultiNodesPid::Datas debug_datas;  
void Task(){  
  multi_nodes_pid.calc(...);  
  debug_datas = multi_nodes_pid.datasAt(0);  
};  
```

注意：由于使用 `std` 标准库链表，在 Ozone 中，不能通过数据监视窗口查看链表数据。


## 历史版本
