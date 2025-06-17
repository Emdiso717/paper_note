# XPBD: Position-Based Simulation of Compliant Constrained Dynamics

## Introduction

- PBD Problem : 随着迭代次数和时间步数的变化，会影响刚度。直观：迭代次数越多、时间步数（迭代的时间间隔），就会导致刚度上升
  - 在处理多物体交互问题时：会出现由于某些需要增加刚度而影响了软物体
  - 对于处理单一物体的问题时：拉伸和收缩刚度在同一个物体上是存在区别的
- PBD 由于没定义 约束力 的概念 所以主要应用于 低精度的速度优先的 场景。

- 本文提出的方法 **XPBD** ：
  - 解决了迭代次数与刚度的问题 引入拉格朗日算子
  - 将 PBD 约束扩展为可直接对应明确的弹性与耗散能量势函数。
  - 与牛顿迭代器进行对比 验证高效性

### Background 

参照 PBD note 中内容，能够发现这个迭代对于刚度限制并不好，（这个刚度系数和时间步长有关系但是方法中没有设置 ，但是这个刚度最好和时间和迭代次数无关），并且多个约束互相影响，会导致无法收敛。

提出的解决方法：**说白了 就是不想让时间步长影响这个修正偏移量**

- 使用一个正则化的约束 （消除约束彼此影响从而不能收敛的情况）
- 约束希望来自于真实的物理能量模型 （**为什么PBD并不符合真实物理模型**）
- 实现与时间步长和迭代次数无关的结果

> [!TIP]
>
> 什么是正则化约束：是一种柔性的约束，不是向PBD中必须满足 $C(x)=0$这样的强硬约束 ，而是允许可以不等于 0 但是必须存在一定的代价。

### XPBD

- 为什么 这个方法对于 时间步长 和迭代次数并不敏感？

> [!TIP]
>
> 暂时理解：对于传统 PBD 来说 ，第一步进行一个显示欧拉，根据不同步长获得一个新的点，如果步长大则会导致 x 差距较大，此时再通过限制函数 强制拉回，会导致形变量巨大，直观看刚度比较大。
>
> $E_k+C(x)$ 是通过这样进行最小化限制的 
>
> 对于 XPBD 而言，将限制修改成正则化 这样对于步长和限制进行了中和，没有强制要求等于0 解决了这个问题
>
> $Ma+C(x)\rightarrow E_k+U_k$ 最小化改成了能量最小化，此时就没有强制要求限制函数必须为0

**具体步骤**

不同于PBD 这里建立在力的基础上 再进行位置迭代：
$$
Ma=-\bigtriangledown U^T(x)
$$
在这里用隐式采样的到：
$$
M(\frac{x^{n+1}-2x^n+x^{n-1}}{\Delta t^2})=-\bigtriangledown U^T(x^{n+1})
$$
在这能量定义时通常使用二次定义：
$$
U=\frac{1}{2}C^T(x)\alpha^{-1}C(x)
$$
定义一个拉格朗日乘数 $\lambda=-\hat\alpha^{-1}C(x)$在这里 $\hat\alpha=\frac{\alpha}{\Delta t^2}$

所以此时得到两个等式：
$$
M(x^{n+1}-\hat{x})-\bigtriangledown C(x^{n+1})^T\lambda^{n+1}=0\\
C(x^{n+1})+\hat{\alpha}\lambda^{n+1}=0
$$
这里第一个式子表示了能量最小原则，第二个式子代表了限制

此时将两个式子定义为 $g,h$ ,通过泰勒展开来简化求解：
$$
\begin{pmatrix}
K&-\bigtriangledown C^T(x_i)\\
\bigtriangledown C(x_i)&\hat{\alpha}\\
\end{pmatrix}
\begin{pmatrix}
\Delta x\\
\Delta \lambda
\end{pmatrix}
=
-\begin{pmatrix}
g(x_i,\lambda_i)\\
h(x_i,\lambda_i)
\end{pmatrix}
$$
通过一系列简化之后的到最终需要求解的式子：
$$
\begin{pmatrix}
M&-\bigtriangledown C^T(x_i)\\
\bigtriangledown C(x_i)&\hat{\alpha}\\
\end{pmatrix}
\begin{pmatrix}
\Delta x\\
\Delta \lambda
\end{pmatrix}
=
-\begin{pmatrix}
0\\
h(x_i,\lambda_i)
\end{pmatrix}
$$


