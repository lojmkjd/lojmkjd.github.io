---
title: 3DGS 原理学习笔记 
date: 2025-3-16 12:00:00 +0800
categories: [Leaning Notes, 3DGS]
tags: [Principle]
---
# 3DGS 原理学习总结

*以下的内容是基于个人学习后的感悟和理解，若有错误之处，请大家不吝赐教，鞭策本人*

**一句话概括**

个人认为，在3DGS算法中，本质是计算机图像学中原本的三角形图元被替换成了3D Gaussian图元

## 多维高斯函数

多维维高斯分布的形式为：
$$
f_{\mu,\Sigma}(x)=\frac{\exp(-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu))}{\sqrt{(2\pi)^D |\Sigma|}}
$$
$f_{\mu,\Sigma}(x)$ 是表示多维高斯分布的概率密度函数，其中 $x$ 是一个 $D$ 维向量（$x\in R^D$）

$D$ 是输入数据的维数，表示向量 $x$ 包含 $D$ 个随机变量

$\mu$ 是一个$D$ 维向量，表示多维高斯分布的均值向量

$\Sigma$ 是一个 $D\times D$ 的协方差矩阵，表示不同维度之间的协方差关系



## 椭球函数与三维高斯函数的关系

### 椭球

椭球函数的表示形式为：
$$
Ax^2+By^2+Cz^2+2Dxy+2Exz+2Fyz=1
$$

### 高斯函数的理解

参考：[【较真系列】讲人话-3d gaussian splatting全解(原理+代码+公式)【1】 捏雪球_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1zi421v7Dr?spm_id_from=333.788.videopod.sections&vd_source=588ffc06ee8b77d388509da6c839599e)

当高斯函数的指数为一个常数时变化如下：

![image-20250317152715390](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250317152715390.png)

可见当D=2时，随机变量为一个椭圆；当D=3时，随机变量绘制为一个椭球。

![image-20250317153006098](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250317153006098.png)

又因为高斯函数的取值范围为 $[0,1]$ ，所以常熟 $constant \in [0,C]，C为任意正实数$， 由以上可知，3维高斯函数的随机变量在空间中为一个**实心椭球**。



## 椭球的形状控制

![image-20250317153857873](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250317153857873.png)

由图可以知道，当三个维度的随机变量相互独立，即协方差矩阵的斜对角为0时，为球体；三个随机变量不相互独立的时候，为椭球。

所以可以通过控制协方差矩阵来控制这个椭球的形状，使用均值向量来控制球体在三维空间中的位置。

那么问题就转换到如何控制协方差矩阵上了。

### 协方差矩阵控制椭球形状的原理

![image-20250317155058260](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250317155058260.png)

假设一个三维的高斯分布如上，又已知高斯函数遵循仿射变换，即线性变换，所以：

![image-20250317154929502](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250317154929502.png)

因为协方差矩阵是一个正定矩阵，所以通过特征值分解可得：
$$
\Sigma=Q \Lambda Q^T
$$
所以任意高斯分布的仿射变换为：
$$
w=Ax+b \\
w \sim N(A\mu+b,AQ \Lambda(AQ)^T)
$$
当源高斯分为为标准高斯分布时，即

![image-20250317161136170](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250317161136170.png)

那么任意的高斯分布都可以由标准高斯分布获得：
$$
w \sim N(\mu,\Sigma) \\
w \sim N(A{\mu}_0+b,AIA^T)
$$
得：
$$
\Sigma=A I A^T
$$
所以，如果我们能够控制矩阵 $A$ 就可以控制协方差矩阵。

### 如何计算A矩阵 

通过仿射变换的性质我们可以知道：

矩阵 $A$ 是用于表达旋转和缩放的，即
$$
A=RS \\
R是旋转矩阵，S是缩放矩阵
$$
这样公式就转化为：

![image-20250317162536766](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250317162536766.png)

所以，如果原高斯分布不是标准高斯时，其实 $\Lambda$ 也是一个对角矩阵，可以看作由缩放矩阵平方得到：
$$
\Lambda=S S^T
$$

------

> [!NOTE]
>
> 通过如上的公式推导，我们就可以通过控制输入数据的协方差矩阵和均值向量去构造一个椭球，而较为繁琐的协方差又可以通过对服从标准高斯分布的球体的旋转矩阵和缩放矩阵生成，均值向量也可以通过球体的移动向量生成。
>

> 需要构建的工具函数功能为：
>
> 输入：缩放矩阵、旋转矩阵
> 输出：协方差矩阵



## 坐标转换

参考：[【较真系列】讲人话-3d gaussian splatting全解(原理+代码+公式)【2】 抛雪球_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1cz421872F?spm_id_from=333.788.player.switch&vd_source=588ffc06ee8b77d388509da6c839599e)

![image-20250317172609522](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250317172609522.png)

> [!NOTE]
>
> 计算机图形学(Computer Graphics，简称CG)

### 模型变换

参考：[机器人学—清华大学出版社—第三章_](http://www.tup.tsinghua.edu.cn/upload/books/yz/090429-01.pdf)

模型变换的本质是将所有的物体从局部坐标系变换到统一的世界坐标系，即将所有东西放置在同一个坐标系下，用一套统一的坐标系描述。

#### 初始状态下的关系

在三维图形学和计算机图形学中，**初始状态下，物体的局部坐标系（模型坐标系）默认与世界坐标系是重合的**。

- 局部坐标系是物体自身的坐标系，原点通常定义在物体的中心或特定顶点（如模型的几何中心）。
- 世界坐标系是全局的参考系，用于描述场景中所有物体的位置关系。
- **默认重合**：当未对物体施加任何变换（平移、旋转、缩放）时，局部坐标系的原点和轴方向与世界坐标系完全一致。此时，物体局部坐标与世界坐标值相同

#### 应用模型变换后的变化

当对物体应用模型变换（平移、旋转、缩放）时：

- **局部坐标系保持不变**：物体的顶点在局部坐标系中的坐标值不会改变。例如，一个立方体顶点在局部坐标系中的坐标始终是(-0.5, -0.5, -0.5)到(0.5, 0.5, 0.5)，无论其在世界中的位置如何。
- **世界坐标系发生变化**：通过模型矩阵（Model Matrix）将局部坐标变换到世界坐标系后，物体的位置、方向、大小会相对于世界坐标系重新定义

#### 变换公式

![image-20250318172725109](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250318172725109.png)

如图，初始状态下局部坐标系喝全局坐标系为重合状态，此时物体上的任何一个点 $c$ 在局部坐标系 $\{object\}$ 中的坐标为 $P_c$ ，将该物体进行缩放、旋转、平移后摆放到世界坐标系下，此时这个物体所在的局部坐标系的原点在世界坐标系下的坐标为 $_{world}^{object}P$ ，物体上的点 $c$ 在世界坐标系下的坐标为 $P_w$ ，那么变换公式为（非齐次坐标格式）：
$$
_{}^{world}P =_{world}^{object}R_{3 \times 3} \cdot (S \cdot _{}^{object}P) +_{world}^{object}P
$$
这样就可以实现将任何的物体摆放到世界坐标系下，实现所有物体在坐标上关系上的统一

以上公式也可以写为（齐次坐标格式）：
$$
P_w=_{world}^{object}T \cdot _{world}^{object}R \cdot S \cdot P_c
$$

> [!NOTE]
>
> 这里建议使用**先旋转再平移**的方法，可以避免平移引起的旋转误差。
> $$
> 先平移后旋转:
> R \cdot T=
> \begin{bmatrix}
> r & 0 \\
> 0 & 1
> \end{bmatrix}
> \begin{bmatrix}
> I & t \\
> 0 & 1
> \end{bmatrix}=
> \begin{bmatrix}
> r & rt \\
> 0 & 1
> \end{bmatrix}\\
> 先旋转后平移：
> T \cdot R =
> \begin{bmatrix}
> I & t \\
> 0 & 1
> \end{bmatrix}
> \begin{bmatrix}
> r & 0 \\
> 0 & 1
> \end{bmatrix}=
> \begin{bmatrix}
> r & t \\
> 0 & 1
> \end{bmatrix}\\
> $$



### 观测变换

参考：[Lecture 04 Transformation Cont._哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1X7411F744?spm_id_from=333.788.videopod.episodes&vd_source=588ffc06ee8b77d388509da6c839599e&p=4)

观测变换本质是从世界坐标系变换到相机坐标系，将相机作为原点坐标系，当情景变化时，可以当作物体发生运动而相机不进行运动。

#### 变换目的

![image-20250318201231187](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250318201231187.png)

如图观测变换的目的是将观测坐标系 $\{view\}$ 的原点变为固定原点，使其他所有的物体和物点的坐标关系基于观测坐标系 $\{view\}$ 进行描述。

#### 观测坐标系的构建

在构建观测坐标系 $\{view\}$ 的时候，规定坐标系的 $-Z$ 方向指向被观测的物点， $+Y$ 方向指向观测器的上方， $+X$ 轴遵守右手法则构建。

设从世界坐标系变换到观测坐标系的变换矩阵为：
$$
M_{view}=R_{view}T_{view}
$$

#### 公式推导

由模型变换可知：
$$
_{}^{world}P=_{world}^{view}T \cdot _{world}^{view}R \cdot _{}^{view}P
$$

> [!CAUTION]
>
> 因为在世界坐标系中，所有的物体已经完成了缩放，相互对应的缩放大小是对应不变的，所以这里无进行做法，仅进行坐标变换。

所以：
$$
_{}^{view}P=_{world}^{view}R^{-1} \cdot _{world}^{view}T^{-1} \cdot _{}^{world}P
$$
综合以上公式可得
$$
R_{view}=_{world}^{view}R^{-1}
=\begin{bmatrix}
r & 0 \\
0 & 1
\end{bmatrix}^{-1}
=\begin{bmatrix}
r^T & 0 \\
0 & 1
\end{bmatrix}
$$

$$
T_{view}=_{world}^{view}T
=\begin{bmatrix}
I & t \\
0 & 1
\end{bmatrix}^{-1}
=\begin{bmatrix}
I & -t \\
0 & 1
\end{bmatrix}
$$

> [!NOTE]
>
> 到此，已经实现了所有的物体摆放是基于观测相机的视角描述了，当观测相机姿态发生变换时，实际观测到的状态会变成周围物体相对于观测相机发生变化了。



### 投影变换

参考：[Lecture 04 Transformation Cont._哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1X7411F744/?p=4&spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=588ffc06ee8b77d388509da6c839599e)

![image-20250318221047284](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250318221047284.png)

这里介绍了两种投影模型：透视投影（perspective projection）和正交投影（orthographic projection）

- 透视投影：正常人眼所看到的投影，符合近大远小的视觉认知。
- 正交投影：相比于透视投影，正交投影可以看作将相机置于无限远的地方。

#### 正交投影

![image-20250319002651167](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250319002651167.png)

1. 定义立方体 $[l,r] \times [b,t] \times [f,n]$ :

   - $[l,r]$ 是立方体在 $x$ 轴上的范围，$[b,t]$ 是立方体在 $y$ 轴上的范围，$[f,n]$ 是立方体在 $z$ 轴上的范围

2. 将立方体平移到原点，变换矩阵为：
   $$
   T_{ortho}=
   \begin{bmatrix}
   1 & 0 & 0 & -\frac{r+l}{2} \\
   0 & 1 & 0 & -\frac{t+b}{2} \\
   0 & 0 & 1 & -\frac{n+f}{2} \\
   0 & 0 & 0 & 1 \\
   \end{bmatrix}
   $$

3. 将立方体缩放到 $[-1,1]^3$ （立方体边长为2），变换矩阵为：
   $$
   R_{ortho}=
   \begin{bmatrix}
   \frac{2}{r-l} & 0 & 0 & 0 \\
   0 & \frac{2}{t-b} & 0 & 0 \\
   0 & 0 & \frac{2}{n-f} & 0 \\
   0 & 0 & 0 & 1 \\
   \end{bmatrix}
   $$

由此可见，正交变换是一种 **仿射变换** ，变换矩阵定义为：
$$
M_{ortho}=R_{ortho}T_{ortho}
$$

#### 透视变换

##### Frustum 到 Cuboid

![image-20250319124946615](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250319124946615.png)

##### 变换关系

![image-20250319144848890](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250319144848890.png)

如上图，取空间中的一个范围 $[f,n]$ 生成Frustum，取Frustum空间中任意一点 $(x,y,z)$ ，根据压缩规则，我们可以知道压缩前后的坐标变化如下：

1. $x$ 变换到 $x'$，且 $x'=x_n$ 
2. $y$ 变换到 $y'$ ，且 $y'=y_n$ 
3. $z$ 变换到 $z'$ ，$z'$ 不定，即 $z'=unknown \in[f,n]$ 

> [!NOTE]
>
> 由上面的变换示意图我们可以发现，原本在 $Frustum$ 空间中的视线变换到 $Cuboid$ 空间时，变短了，所以在视线上的点坐标在 $z$ 轴方向应该也是变得更靠近 $n$ 面，这里我们并不知道是多少，所以为 $unknown$ 。

至此我们得到以下公式：
$$
\frac{y}{y_n}=\frac{z}{n}=\frac{y}{y'}
$$
得：
$$
y'=\frac{ny}{z}
$$
同理：
$$
x'=\frac{nx}{z}
$$
所以：
$$
\begin{bmatrix}
x'\\y'\\z'\\1
\end{bmatrix}
=\begin{bmatrix}
\frac{nx}{z}\\\frac{ny}{z}\\unknown\\1
\end{bmatrix}
$$

##### 变换矩阵推导

设
$$
\begin{bmatrix}
x'\\y'\\z'\\1
\end{bmatrix}
=\begin{bmatrix}
\frac{nx}{z}\\\frac{ny}{z}\\unknown\\1
\end{bmatrix}=\frac{1}{z}M_{Frustum}^{Cubiod}
\begin{bmatrix}
x\\y\\z\\1
\end{bmatrix}
$$
其中 $M \in R^{4 \times 4}$ 。

> [!TIP]
>
> 这里可以将 $(x',y',z')$ 放大 $z$ 倍，用以方便计算。
> $$
> \begin{bmatrix}
> nx\\ny\\unknown \cdot z\\z
> \end{bmatrix}
> =M_{Frustum}^{Cubiod}
> \begin{bmatrix}
> x\\y\\z\\1
> \end{bmatrix}
> $$

根据以上公式可以确认：
$$
M_{Frustum}^{Cubiod}=
\begin{bmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & A & B \\
0 & 0 & 1 & 0 \\
\end{bmatrix}
$$
当 $(x,y,z)$ 取到 $n$ 面上的点时， $z=n$ ，那么：
$$
M
\begin{bmatrix}
x\\y\\n\\1
\end{bmatrix}
=
\begin{bmatrix}
nx\\ny\\unknown \cdot n\\n
\end{bmatrix}
$$
因为 $unknown \in[f,n]$ ，所以取  $unknown=n$ ，则
$$
\begin{bmatrix}
nx\\ny\\n^2\\n
\end{bmatrix}
=
\begin{bmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & A & B \\
0 & 0 & 1 & 0 \\
\end{bmatrix}
\begin{bmatrix}
x\\y\\n\\1
\end{bmatrix}
$$
得：
$$
An+B=n^2
$$
同理，当 $(x,y,z)$ 取到 $f$ 面上的点时， $z=f$ ，同时因为 $unknown \in[f,n]$ ，所以取  $unknown=f$​ ，则：
$$
\begin{bmatrix}
nx\\ny\\f^2\\f
\end{bmatrix}
=
\begin{bmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & A & B \\
0 & 0 & 1 & 0 \\
\end{bmatrix}
\begin{bmatrix}
x\\y\\f\\1
\end{bmatrix}
$$
得：
$$
Af+B=f^2
$$
联立方程解得：
$$
A=n+f,B=-nf
$$
所以
$$
M_{Frustum}^{Cubiod}=
\begin{bmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & n+f & -nf \\
0 & 0 & 1 & 0 \\
\end{bmatrix}
$$
最终：
$$
\begin{bmatrix}
x'\\y'\\z'\\1
\end{bmatrix}
=\frac{1}{z}
\begin{bmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & n+f & -nf \\
0 & 0 & 1 & 0 \\
\end{bmatrix}
\begin{bmatrix}
x\\y\\z\\1
\end{bmatrix}
=\begin{bmatrix}
\frac{nx}{z}\\\frac{ny}{z}\\n+f-\frac{nf}{z}\\1
\end{bmatrix}
$$

> [!CAUTION]
>
>  $\frac{1}{z}$ 也可以看作 对应点实时生成的 缩放矩阵 $S(z)$ 

##### 雅可比近似矩阵

因为透视变换是一个非仿射变换，所以引进雅可比近似矩阵，使变量在微分的尺度上近似仿射变换。

因为
$$
\begin{bmatrix}
x'\\y'\\z'\\1
\end{bmatrix}
=\begin{bmatrix}
\frac{nx}{z}\\\frac{ny}{z}\\n+f-\frac{nf}{z}\\1
\end{bmatrix}
$$
所以雅可比矩阵为：
$$
J=\begin{bmatrix}
\frac{n}{z} & 0 & -\frac{nx}{z^2}\\
0 & \frac{n}{z} & -\frac{ny}{z^2}\\
0 & 0 & \frac{nf}{z^2}
\end{bmatrix}
$$


### 视口变换

![image-20250319195541914](C:\Users\78094\AppData\Roaming\Typora\typora-user-images\image-20250319195541914.png)

视口变换很好理解，就是一个仿射变换+平行投影，这个过程与 $z$ 轴坐标无关。

在投影变换后，我们最终得到一个 $[-1,1]^3$ 的标准立方体，对该立方体进行平移和缩放，得到一个 $[0,w] \times[0,h]$ 的图像，得变换矩阵为：
$$
M_{viewport}=
\begin{bmatrix}
\frac{w}{2} & 0 & 0 & \frac{w}{2} \\
0 & \frac{h}{2} & 0 & \frac{h}{2} \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

### 栅格化

参考：[Lecture 06 Rasterization 2 (Antialiasing and Z-Buffering)_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1X7411F744?spm_id_from=333.788.videopod.episodes&vd_source=588ffc06ee8b77d388509da6c839599e&p=6)

在进行栅格化时，需要消除锯齿，所以先进行滤波，再进行栅格化

滤波的目的是是图像信息的频率降低。使得栅格化函数的频率大于等于图像频率的两倍（香农采样定理），从而消除锯齿。详细分析图像的见傅里叶变换。

> [!CAUTION]
>
> 至此，完成了对一个高斯椭球的观测成像。



## 雅可比矩阵的影响

对于高斯椭圆来说，椭圆的位置是一个点，可以进行非线性变换，但是协方差矩阵描述了一个高斯的分散程度，它描述的不是一个点，而是一种形状，在非线性变换的过程中，协方差矩阵可能会被拉伸，这种拉伸会导致3D高斯在投影到2D平面后不再是一个高斯，这是我们不希望发生的。

所以使用雅可比矩阵对高斯椭圆的协方差矩阵进行变换：
$$
V=J\Sigma J^T
$$
