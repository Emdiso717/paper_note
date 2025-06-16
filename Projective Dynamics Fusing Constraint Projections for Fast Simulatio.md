# Projective Dynamics: Fusing Constraint Projections for Fast Simulatio

> [!TIP]
>
> 这篇文章是一个方法的整理总结，作为background 背景知识了解比较好

## Implicit Euler Solver

这里对于 变分（variational）的形式进行了清晰推导以及解释：

对于 mesh 所有 vertices 需要存储的内容是此时的位置 $q \in R^{m\times 3}$ 和速度信息 $v \in R^{m\times 3}$  ,定义了一个势能函数，由位置决定每一个位置的势能。$W_i(q)$ 于是获得内力的表达式 $f_{int}(q)=-\sum∇W_i(q)$  ，此时通过 隐式欧拉 获得：
$$
q_{n+1}=q_{n} + hv_{n+1}\\
$$

$$
v_{n+1}=v_n+hM^{-1}(f_{int}(q_{n+1})+f_{ext})
$$

将 (1) 带入 （2）中获得一个 类似于牛顿第二定理的结果等式：
$$
M（q_{n+1}-q_n-hv_n)=h^2(f_{int}(q_{n+1})+f_{ext})
$$

$$
\frac{1}{h^2}M(q_{n+1}-q_{n}-hv_n-h^2M^{-1}f_{ext})-f_{int}(q_{n+1})=0\\
\frac{1}{h^2}M(q_{n+1}-s_n)+\sum∇W_i(q_{n+1})=0
$$

可以发现最后最后式子是一个最优解形式，可以视为一阶导数等于0的情况，则积分之后原式就是：
$$
\min_{q_{n+1}}\frac{1}{2h^2}\|M^{\frac{1}{2}}(q_{n+1}-s_n)\|^2_F+\sum W_i(q_{n+1})
$$
之后使用牛顿迭代法求解上述等式，但是势能函数的二阶导数 和一阶导数每一次迭代都需要重新求解，并且求解线性系统消耗很大，于是这个势能函数需要进行优化，这种非线形的不好用。

### Nonlinear Elasticity

对于之前的问题需要定义一个合适的势能函数 $W$ ，通常使用离散元素strain$E(q)$ 在前一篇文章note中解析了 Green strain的具体推导。这里会把 $E(q)=0$视为一个流形，则能量就可以视为 到这个流形到距离。在这里希望把这两个概念分开，解偶求解流形和距离。
$$
W(q,p)=d(q,p)+\delta_E(p)
$$
$\delta_E(p)$ 只是一个指示函数 需要保证p点落在 流形 上，此时最小化这个投影距离，作为势能函数，这样能有效降低计算压力。

在这里使用二次距离来优化，第一能够视为一个最小二乘问题，同时在求解二阶导数更加方便：
$$
W(q,p)=\frac{w}{2}\|Aq-Bp\|^2_F+\delta_c(p)
$$

### PBD 具体流程

**The First Step** : explicit Euler 忽略内力 :$q_{n+1}=q_n+hv_{n}$

**The second step**:对最小势能计算:$\frac{1}{2}\sum\|M^{0.5}(S_iq-p)\|^2_F+\delta c_i(p_i)$

这个是用一个近似于牛顿希尔德的算法进行最优计算，由于有多个限制条件则需要保证每一个被限制。

对其使用拉格朗日乘数：
$$
\frac{1}{2}\sum\|M^{0.5}(S_iq-p)\|^2_F+\lambda(C(q)+tr(∇c((q)^T)\Delta q))
$$
之后为了保证是线形则对C做了一把泰勒展开 但是只保留了前两项。

由于是需要同时满足所有的C这里是视为一个线形方程组求解了。

求偏导之后 求解出 $\Delta q=-M^{-1}∇C(q)\frac{C(q)}{\|M^{-\frac{1}{2}}∇C(q)\|^2_F}$

**The thrid Step** 根据 偏移量 更新下一步的$q_{n+1}$ 和 $v_{n+1}$

 