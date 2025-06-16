# Example-Based Elastic Materials

## Basic概念

- Inhomogeneous：非均质，材料的性质在空间中是不一致的。不同位置的材料可能会有不同的密度、弹性模量等
- Anisotropic：各向异性，材料在不同的方向上性质不同。在一个方向很硬，在另一个方向很软。
- Explicit and Implicit (显示和隐式表示)

  - **显示：**通常直接使用上一帧的信息推导下一帧的位置和速度

    - $a_t = M^{-1}f(x_t,v_t)\\v_{t+Δt}=v_t+Δt⋅at\\x_{t+Δt}=xt+Δt⋅v_{t+Δt}$

  - **隐示：** 不依赖信息，直接求解方程

    - $a_{t+Δt} = M^{-1}f(x_{t+Δt},v_{t+Δt})\\v_{t+Δt}=v_t+Δt⋅a{t+Δt}\\x_{t+Δt}=xt+Δt⋅v_{t+Δt}$
- Strain 应变 ：定量描述物体形变变化（拉伸、压缩、剪切），不考虑整体平移与旋转
- 张量 Tensor ：简单理解就是描述空间中物理量
  - 一阶张量：向量 （力）
  - 二阶张量：矩阵 变形梯度F等
- **Variational Statics ** 变分静力学 ： 力平衡使用势能泛函极小化原则。在弹性体的给定边界条件和外力时，势能最小时达到平衡。


## Introduction

传统物理动画仿真 ：Material Properties $\rightarrow$ Control $\rightarrow$  deformation

本文提出 ： deformation $\rightarrow$ Material , 从物理控制修改成**感知控制**

- **工程仿真：** 先有材料 → 决定变形

- **动画创作：** 先有想象中的变形 → 希望找一种方式能做出这种变形，材料参数是过程中的“帮手”而已

将 Example -based elastic deformation 分成三个步骤

1. Interpolation 插值： 使用多种 姿态 通过插值构成一个空间
   1. 使用 **非线性应变度量（nonlinear strain measure）** **什么是?**ok
   2. 保留旋转不变（**rotation-invariant**） **如何保留？**ok
2. Projection 投影：使用**最小化问题**（找到一个距离某一个形变最靠近的一些 基形变）
3. Simulation ：定义 Elastic Potential 将物体吸引到最佳形变空间 **？如何实现example manifold.**

## Theory

<img src="./Example-Based Elastic Materials.assets/image-20250609200702940.png" alt="image-20250609200702940" style="zoom:50%;" />

**具体算法（暂时对于两个force 还不是很理解）**

<img src="./Example-Based Elastic Materials.assets/image-20250610134417922.png" alt="image-20250610134417922" style="zoom:50%;" />

1. 输入多个 example poses 构建空间
2. 通过最小化 将输入状态映射到目标状态
3. 计算出将 current state 拉到 manifold 的里
4. 加上之前系统的力 继续下一帧计算

### Example Manifold

通过应变来表示这个形变，通过离散化表示：

1. 使用 tet 网格来离散化某一个固体

2. **Deformation gradient** ：计算形变的变化的雅各比矩阵：$X\in R^{3n} ,x\in R_{3n}$ 则从前到后的变化可以表示为 $F(X,x)=\frac{∂x}{∂X}$

3. F是一个**二阶张量** 并且非奇异，根据极分解定理 能够进行分解成：$F=RU$ R 是正交矩阵，于是 $C(X,x)=F^TF$

   1. 是由于 F 中包括了 形变和旋转  因为要保持旋转不变 所以需要分解出去旋转参数

   2. R正交是旋转矩阵原因： $RR^T=I$  , 对于向量v而言 进行旋转之后， $Rv$ 此时 向量长度并不发生变化，并且 同时有一个 $Ru$ 两个向量夹角也不变。

4. 或者换一个忽略旋转的方法 使用 Green strain tensor 构造 $E(X,x)=\frac{1}{2}(C(X,x)-I)$ 

   1. 推导：通过两个距离差来忽略旋转 $\|dx\|^2-\|dX\|^2= dx^Tdx-dX^TdX=dX^T(F^TF-I)dX$

所以 对于每一个形变 都可以使用这个descriptor 表示 （但是不能反过来说） 实现目标 忽略旋转属性的映射。

> [!NOTE]
>
> > 
>
> $$
> E(w)=(1-w)E_1+wE_2
> $$
>
> 通过已知的 pose 进行线性插值, 但是插值产生的描述符并不可实现，当多个tet上的公共点 可能会出现不同的位置 出现矛盾。

所以使用最小二乘法 找到一个距离理想的线性插值最近的一个：
$$
W_I(x_w,w)=\frac{1}{2}\|E(x_w)-E(w)\|^2_F
$$

- $E(w)$ 是线性插值得到的目标描述符（可能不可实现）；
- $E(x_w)$是我们要求的**最近合法应变**；
-  $W_I(x_w,w)$ 称为 Interpolation energy；

### Example Projection

需要将 “当前形变” 转化到 “合法形变” 通过一个力来转化，但是获取力需要一个距离。通过能量函数来近似测地距离。
$$
W(X,x)=\mu|E(X,x)|^2_F+\frac{\lambda}{2}(\frac{V(x)}{V(X)}-1)^2
$$
从 $x$ 到 $x_w$ 的映射 需要满足 最小的 $\min W(x_w,x)$

> [!NOTE]
>
> 还需要求最短距离的原因：
>
> 在插值环节 选择出来了 $E(x_w)$ 到 理想的 $E(w)$ 的差最小。 但是 不能从 $E(x_w)$ 反过来 找到 $x_w$ ,所以会有多个 $x_w$ 这里是在寻找一个 $x$ 作为 pose

$$
\min W(x_w,x) \quad s.t. \quad x_w \in \epsilon\\
min W(x_w,x) \quad s.t. \quad ∇_{x_w}W_I(x_w,w)=0\\
$$

由于上述计算难度过大 于是进行简单退化：
$$
W_p(x_w,w,x)=W(x_w,x)+\gamma|∇_{x_W}(x_w,w)|^2
$$
只需要让这个 变小即可。

## Example-based Simulation

**Variational Statics **
$$
W_{tot}(x_w,X,X)=W_c(x)+W_p(x_w,W,X)+W_{ext}(x)
$$
**Variational Implicit Euler**:
$$
f_{ext}=M(\frac{x_n-x_0}{h^2}-\frac{v_0}{h})+∇_xW_c(x_n)+∇_xW_p(x_w,W,x_n)\\
H=\frac{h^2}{2}(\frac{x_n-x_0}{h^2}-\frac{v_0}{h})^TM(\frac{x_n-x_0}{h^2}-\frac{v_0}{h})+W_c(x_n)+W_p(x_w,W,x_n)
$$
