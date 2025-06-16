# Eigen Problem

## Review Power Method

### Original Method

$$
Eigenvalue:|\lambda_1|\geq|\lambda_2|...\geq |\lambda_n|\\
Eigenvector:\{\vec v^{(1)}, \vec v^{(2)}...\}\\
\vec x^{(0)}=\Sigma_{j=1}^{n}\beta_j \vec v^{(j)}\\
\vec x^{(1)}=A\vec x=\Sigma_{j=1}^{n}\beta_j A\vec v^{(j)}=\Sigma_{j=1}^{n}\beta_j \lambda_j\vec v^{(j)}\\
...\\
\vec x^{(n)}=A^n\vec x=\Sigma_{j=1}^{n}\beta_j A\vec v^{(j)}=\Sigma_{j=1}^{n}\beta_j \lambda_j^n\vec v^{(j)}\\
=(\lambda_1)^n\Sigma_{j=1}^{n}\beta_j(\frac{\lambda_j}{\lambda_1})^n\vec v^{(j)}\\
\lim _{j\rightarrow \infty}A^j\vec x=\lim_{j\rightarrow \infty}\lambda_1^j\beta_1\vec v^{(1)}\\
\vec x^{(k)} \approx \lambda_1^{k}\beta_1\vec v^{(1)}\\
\lambda_1 \approx \frac{(\vec x^{(k)})_i}{(\vec x^{(k-1)})_i}
$$

### Normalization Power Method

$$
||\vec x^{(k)}||_{\infty}=|\vec x^{(k)}_{p_k}|\\
Normalization:
\vec u^{(k-1)}=\frac{ \vec x^{(k-1)}}{|{\vec x^{(k-1)}_{p_{k-1}}}|}\\
\vec x^{(k)}=A \vec u^{(k-1)}\\
result:\\
\vec u^{(k)}\rightarrow \vec v_1\\
\lambda_1 =\frac{\vec x^{(k)}}{ \vec u^{(k-1)}}=\vec x^{(k)}_{p_{k-1}}
$$



## HSIM 总体思路解析

1. It starts on the **coarsest grid**
2. Then, the hierarchy is **traversed from coarse to fine**
   1. Subspace iterations：
      1. Efficient : Use the eigenvalues computed on one grid to determine a value by which we shift the matrix on the next grid.
      2. Construct the hierarchy :  
         1. vertex sampling (不太理解) -> build prolongation operators -> hierarchy of nested function (关联于vertex hierarchy )
   2. The solution on the previous grid is used as an initialization for the subspace iterations.

## Laplace–Beltrami Eigenproblem

- $Σ$ : Compact and smooth surface  in $R^3$

-  $\phi$: eigenfunction of the Laplace–Beltrami operator $Δ$​ 

- $\lambda$​ : eigenvalue
  $$
  −Δ\phi = λ\phi
  $$

### 构造弱解

- Nultiplying both sides of the equation with  a continuously differentiable function f (两边同时乘一个可微函数,之后进行积分) **这部分化简还有点问题 ,不知道以下理解是否正确**,一次微分函数 $∂Ω$为零 原因还是不很理解）
  $$
  \int_Σ (−Δ\phi f) dA = \int_Σ (λ\phi f)  dA\\
  LEFT=\int_Σ (−div·\nabla\phi f) dA \\
  = \int _Σ-(\sum \frac{\partial(\nabla\phi)}{\partial x} f)dA\\
  =\int_Σ(\sum(\nabla\phi\frac{\partial(\nabla f)}{\partial x}))dA-\int_{\partialΣ}\nabla\phi f dS\\
  =\int_Σ\nabla\phi·\nabla f dA-\int_{\partialΣ}\nabla\phi f dS
  $$


- 化简结果就是文章中的：

  <img src="./Eigen Problem.assets/image-20241206144326578.png" alt="image-20241206144326578" style="zoom:50%;" />

- 问题转化为 ： $\lambda$ 作为 eigenvalue $\phi$ 是 eigenfunction $$\Leftrightarrow$$​ Equation (2) holds for **all continuously differentiable functions f**

-  If Σ is a triangle mesh ,usually the space F of continuous functions that are **linear polynomials over every triangle**. 

- 使用 vector $\Phi=[\Phi_0,...\Phi_n]$​ list function values at all vertices. 用线性插值构造每一个三角形中的函数。

- 构造出两个矩阵 ： stiffness 和 mass matrix  
  $$
  Sij =  ∫ _ Σ  grad φi · grad φj dA\\
  Mij =  ∫ _ Σ  φiφj dA
  $$

### 最终问题形式

$$
S\Phi = \lambda M \Phi
$$

- 这个形式是和之前的积分形式等价。只是这里使用线性来简化函数类型。个人理解如下：

  -  $S\Phi$ 单独取出一行：
    $$
    S_{i,0}\Phi_0+...\\
    =(∫ _ Σ grad φ_i · grad φ_0 dA)\Phi_0+... \\
    \sum grad φ_0·\Phi_0 = grad ~\phi\\
    = ∫ _ Σ \nabla φ_i · \nabla \phi dA
    $$

  - $S\Phi$ 将所有列添加：
    $$
    \sum∫ _ Σ \nabla φ_i · \nabla \phi dA\\
    =\int_Σ\nabla\phi·\nabla f dA
    $$

- 右边同理，但是 for **all continuously differentiable functions f** 所以 ，个人认为在两边也可同时乘 一个 function vector for f

## 问题记录与补充

### Eigenfunctions

**Eigenfunctions** can be expressed as matrices with column vectors and linear operators. An eigenfunction can have infinite dimensions. In mathematics, eigenfunction is defined on any function space as some non-zero function f present in that space with linear operator L acted upon it is only multiplied by its eigenvalue.

### Discrete Laplace-Beltrami Operator

Laplace Operator ：$\Delta = div·\nabla = [\frac{\partial}{\partial x},\frac{\partial}{\partial y},\frac{\partial}{\partial z}][\frac{\partial f_x}{\partial x},\frac{\partial f_y}{\partial y},\frac{\partial f_z}{\partial z}]^T$

### Gauss`s Law \ Divergence Theorem

$$
{\displaystyle \iiint _{\Omega }\left({\frac {\partial P}{\partial x}}+{\frac {\partial Q}{\partial y}}+{\frac {\partial R}{\partial z}}\right)\mathrm {d} v=}\oiint_{\sum}{\displaystyle (P\cos \alpha +Q\cos \beta +R\cos \gamma )\,\mathrm {d} S}\\
\displaystyle \iiint _{\Omega }\mathrm {div} \mathbf {F} \,\mathrm {d} v=\oiint_{\displaystyle \scriptstyle \Sigma }{\displaystyle \mathbf {F} \cdot \mathbf {n} \,\mathrm {d} S.}\\
{\int_{U} \mathrm {div}\mathbf u\mathrm~ {d} x} = \int_{\partial U} \mathbf u· \mathbf n dS
$$

