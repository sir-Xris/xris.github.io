---
layout: post
title: 傅里叶变换
category: mathematics
tags: [ 数学, 傅里叶变换 ]
comments: true
---

前排警告，公式恐惧症勿入……

本来打算就说说离散傅里叶变换的，结果从傅里叶级数开始全部说了过去……不会讲具体怎么求一个函数的傅里叶变换，虽然有公式，但是主要因为没说狄利克雷-若尔当判别法，所以很大一部分函数都没法求。如果感兴趣的话，直接搜吧。

![](https://upload.wikimedia.org/wikipedia/commons/2/2b/Fourier_series_and_transform.gif)
这是傅里叶变换的一个直观例子，将时域的一个方波“拆”成频域的一列收敛的值，也就是三角级数。傅里叶变换就是将一个函数用三角级数来逼近，且由于傅里叶级数的性质，使其在各个学科有广泛的应用。

以下常提到的时域和频域，一个对应着转换前的以时间为参数的函数（数列），一个对应着转换后的以频率为参数的函数（数列）。但是实际上它们的涵义已经不只是以时间、频率为参数了，只是还用这个名字而已，约定俗成，我也就不改了。

还有几个例子

![](https://upload.wikimedia.org/wikipedia/commons/0/0a/Synthesis_square.gif)
<center>方波</center>

![](https://upload.wikimedia.org/wikipedia/commons/b/bb/Synthesis_triangle.gif)
<center>三角波</center>

![](https://upload.wikimedia.org/wikipedia/commons/d/d4/Synthesis_sawtooth.gif)
<center>锯齿波</center>

四张图片均来自维基百科。

<!-- more -->

{::nomarkdown}以下几个记号的规定，按照维基百科的来，\(x(t), \bar x(t),x[n],x_n\)分别指的是时域的非周期函数、周期函数、无穷数列、有穷数列；\(X(f),\bar X(f),X[k],X_k\)分别指的是频域的非周期函数、周期函数、无穷数列、有穷数列。其余有穷数列无穷数列表示方法基本一致。时域的函数参数均使用\(t\)，数组下标均使用\(n\)；频域的函数参数均使用\(f\)(frequency)，数组下标均使用\(k\)。有穷数列的长度会在离散傅里叶变换那里说明。{:/nomarkdown}

## 傅里叶级数（Fourier Series, FS）

{::nomarkdown}
我们已经熟悉使用求一个多项式来拟合函数的方法，使用泰勒展开可以得到一个多项式。不考虑不可导的情况，泰勒公式拓展一下可以得到泰勒级数$$f(x)=\sum_{k\in\mathbb Z_0^+}\frac{f^{(k)}(x_0)}{k!}(x-x_0)^k$$

除了幂级数，还有一个很重要的函数项级数，是由三角函数组成的级数，称为三角级数。三角级数旨在描述周期性的函数，一个周期函数总可以表示成正弦、余弦函数的叠加，亦即$$
\begin{align*}
f(x)&=A[0]+\sum_{k\in\mathbb Z^+}A[k]\sin(k\omega x+\varphi[k])\\
&=A[0]+\sum_{k\in\mathbb Z^+}A[k](\sin\varphi[k]\cos k\omega x+\cos\varphi[k]\sin k\omega x)\\
\end{align*}
$$

然后，令\(a[0]=2A[0],\ a[k]=A[k]\sin\varphi[k],\ b[k]=A[k]\cos\varphi[k],\ t=\omega x\)，得到$$f\left(\frac t\omega\right)=\frac{a[0]}2+\sum_{k\in\mathbb Z^+}(a[k]\cos kt+b[k]\sin kt)$$

上式即为三角级数，数列\(a[k]\)和\(b[k]\)为其系数。
{:/nomarkdown}

### 欧拉-傅里叶系数公式

{::nomarkdown}
自然，我们开始寻找求三角级数的求法，注意到三角函数系在任意一个周期的正交性，举例而言在一个周期\([-\pi,\pi]\)内，\(\forall n,m\in\mathbb Z_0^+:\)$$
\begin{align*}
&\int_{-\pi}^\pi\sin nx\cos mx\,\mathrm dx=0\\
&\int_{-\pi}^\pi\sin nx\sin mx\,\mathrm dx=0&n\not=m\\
&\int_{-\pi}^\pi\cos nx\cos mx\,\mathrm dx=0&n\not=m\\
\end{align*}
$$

所以我们分别用\(\cos nt\)和\(\sin nt\)乘以一个周期为\(2\pi\)的时域上的周期函数\(\bar x(t)\)，并在任意一个周期内积分，这里是\([-\pi,\pi]\)，如下$$
\begin{align*}
&\int_{-\pi}^\pi\bar x(t)\cos nt\,\mathrm dt\\
=&\int_{-\pi}^\pi\left[\frac{a[0]}2+\sum_{m\in\mathbb Z^+}(a[m]\cos mt+b[m]\sin mt)\right]\cos nt\,\mathrm dt\\
=&\frac{a[0]}2\int_{-\pi}^\pi\cos nt\,\mathrm dt+\sum_{m\in\mathbb Z^+}\left(a[m]\int_{-\pi}^\pi\cos mt\cos nt\,\mathrm dt+b[m]\int_{-\pi}^\pi\sin mt\cos nt\,\mathrm dt\right)\\
=&a[n]\int_{-\pi}^\pi\cos^2nt\,\mathrm dt=\pi a[n]\\
\\
&\int_{-\pi}^\pi\bar x(t)\sin nt\,\mathrm dt\\
=&\int_{-\pi}^\pi \left[\frac{a[0]}2+\sum_{m\in\mathbb Z^+}(a[m]\cos mt+b[m]\sin mt)\right]\sin nt\,\mathrm dt\\
=&\frac{a[0]}2\int_{-\pi}^\pi\sin nt\,\mathrm dt+\sum_{m\in\mathbb Z^+}\left(a[m]\int_{-\pi}^\pi\cos mt\sin nt\,\mathrm dt+b[m]\int_{-\pi}^\pi \sin mt\sin nt\,\mathrm dt\right)\\
=&b[n]\int_{-\pi}^\pi\sin^2nt\,\mathrm dt=\pi b[n]\\
\end{align*}
$$

化简可得$$
a[k]=\frac1\pi\int_{-\pi}^\pi\bar x(t)\cos kt\,\mathrm dt\\
b[k]=\frac1\pi\int_{-\pi}^\pi\bar x(t)\sin kt\,\mathrm dt\\
$$

我们称如上的欧拉-傅里叶公式形式的三角级数为以\(2\pi\)为周期的函数\(\bar x(t)\)的傅里叶级数。
{:/nomarkdown}

傅里叶级数收敛的条件是函数满足狄利克雷-若尔当判别法，这里不提。

### 傅里叶级数的复数形式

{::nomarkdown}
根据欧拉公式，\(\mathrm e^{\mathrm i\theta}=\cos\theta+\mathrm i\sin\theta\)，代入傅里叶级数可得$$
\begin{align*}
\bar x(t)&=\frac{a[0]}2+\sum_{k\in\mathbb Z^+}\left(a[k]\frac{\mathrm e^{\mathrm ikt}+\mathrm e^{-\mathrm ikt}}2-\mathrm ib[k]\frac{\mathrm e^{\mathrm ikt}-\mathrm e^{-\mathrm ikt}}2\right)\\
&=\frac{a[0]}2+\sum_{k\in\mathbb Z^+}\left(\frac{a[k]-\mathrm ib[k]}2\mathrm e^{\mathrm ikt}+\frac{a[k]+\mathrm ib[k]}2\mathrm e^{-\mathrm ikt}\right)\\
\end{align*}
$$

这里，再令\(X[k]=(a[k]-\mathrm ib[k])/2,X[0]=a[0]/2,X[-k]=(a[k]+\mathrm ib[k])/2\)，可以将\(\bar x(t)\)继续化简为$$\bar x(t)=\sum_{k\in\mathbb Z}X[k]\mathrm e^{\mathrm ikt}$$

上式和原式等价，均为傅里叶级数。然后求系数，将欧拉-傅里叶系数公式回代入\(X[0], X[k], X[-k]\)可归纳得$$X[k]=\frac1{2\pi}\int_{-\pi}^\pi\bar x(t)\mathrm e^{-\mathrm ikt}\,\mathrm dt\qquad k\in\mathbb Z$$
{:/nomarkdown}

嘛，方(tou)便(lan)起见，后面就基本都用复数表示法了。

### 任意周期函数的傅里叶级数展开

{::nomarkdown}
如果一个函数以\(T_0\)为周期，我们只需要对其进行放缩使周期成为\(2\pi\)，求出傅里叶级数再放回来即可，实际上只是做一次变量代换，可以求出$$\bar x(t)=\sum_{k\in\mathbb Z}X[k]\mathrm e^{-\frac{2\pi}{T_0}\mathrm ikt}$$

这里参数为$$X[k]=\frac1{T_0}\int_{T_0}\bar x(t)\mathrm e^{-\frac{2\pi}{T_0}\mathrm ikt}\,\mathrm dt$$
{:/nomarkdown}

### 周期延拓

{::nomarkdown}如果一个函数只在某段区间有定义，我们只需要将其定义域扩充至\(\mathbb R\)，且，就可以以周期函数的方法求出其傅里叶级数，最后再将其傅里叶级数定义域限制为原定义域即可，这个方法叫做周期延拓。{:/nomarkdown}

## 连续傅里叶变换（Continuous Fourier Transform, CFT）

上面讲的都是周期函数的傅里叶变换，如果一个函数并不是周期的，也可以对其进行傅里叶变换，这种傅里叶变换被称为连续傅里叶变换。

{::nomarkdown}
这时候我们可以认为一个非周期函数的周期等于无穷，对原傅里叶级数取极限，可得$$
\begin{align*}
x(t)&=\lim_{T_0\to+\infty}\bar x(t)\\
&=\lim_{T_0\to+\infty}\sum_{k\in\mathbb Z}\left[\frac1{T_0}\mathrm e^{\frac{2\pi}{T_0}\mathrm ikt}\int_{T_0}\bar x(u)\mathrm e^{-\frac{2\pi}{T_0}\mathrm iku}\,\mathrm du\right]\\
&=\lim_{f_0\to0}f_0\sum_{k\in\mathbb Z}\left[\mathrm e^{2\pi\mathrm ikf_0t}\int_{1/f_0}\bar x(u)\mathrm e^{-2\pi\mathrm ikf_0u}\,\mathrm du\right]\\
&=\int_\mathbb R\left[\int_\mathbb R x(u)\mathrm e^{-2\pi\mathrm ifu}\,\mathrm du\right]\mathrm e^{2\pi\mathrm ift}\,\mathrm df\\
\end{align*}
$$

其中\(T_0f_0=1\)，关于最后一步的推导，参考黎曼积分的定义，整个和式作为一个关于\(kf_0\)的函数。然后取出中间方括号部分，我们得到频域上的普通频率形式的傅里叶变换公式$$X(f)=\int_\mathbb Rx(t)\mathrm e^{-2\pi\mathrm ift}\,\mathrm dt$$其逆傅里叶变换公式就是外层的$$x(t)=\int_\mathbb RX(f)\mathrm e^{2\pi\mathrm ift}\,\mathrm df$$
{:/nomarkdown}

其实这是傅里叶变换的其中一个定义（直接取中间方括号部分为傅里叶变换），不过这个是最简单常用的，而且有很多优美的性质，这里就用这个了。

## 离散时间傅里叶变换（Descrete Time Fourier Tranform, DTFT）

在实际的生活中我们并不总能拿到一个函数的表达式，也许只能取样以获取其特征。如何从样本中获取它傅里叶变换后的连续频域函数，这就需要离散时间傅里叶变换。

{::nomarkdown}
然后给个公式，这里的\(T\)是时域数列的采样时间间隔，于是取\(t=Tn,\ x[n]=x(Tn)\)，求和得$$
\begin{align*}
\bar X(f)&=\sum_{n\in\mathbb Z}x(Tn)\mathrm e^{-2\pi\mathrm iTfn}\\
&=\sum_{n\in\mathbb Z}x[n]\mathrm e^{-2\pi\mathrm iTfn}\\	
\end{align*}\\
$$

容易证明其为周期$$
\begin{align*}
\bar X\left(f+\frac1T\right)&=\sum_{n\in\mathbb Z}x[n]\mathrm e^{-2\pi\mathrm i\left(f+\frac1T\right)n}\\
&=\sum_{n\in\mathbb Z}x[n]\mathrm e^{-2\pi\mathrm ift-2\pi\mathrm in}\\
&=\sum_{n\in\mathbb Z}x[n]\mathrm e^{-2\pi\mathrm ift}=\bar X(f)\\
\end{align*}
$$

反过来求逆离散时间傅里叶变换公式，其实就是在求傅里叶级数。因为\(\bar X(f)\)的公式其实就是傅里叶级数的逆，所以求\(x[n]\)的过程也就是在求傅里叶级数的过程，直接套公式即可。$$
x[n]=T\int_{1/T}\bar X(f)\mathrm e^{2\pi\mathrm iTfn}\,\mathrm df
$$
{:/nomarkdown}

可以用从傅里叶级数拓展到连续傅里叶变换同样的方法将离散时间傅里叶变换推导到连续傅里叶变换。

## 离散傅里叶变换（Descrete Fourier Transform, DFT）

其实这个才是本系列的重点……然后基本上已经被讲完了。这个用于计算机科学更多，因为离散傅里叶变换处理的时域和频域都是有限长的离散数列。

其实离散傅里叶变换里的时域数列和频域数列周期延拓后分别是傅里叶级数中的频域数列和离散时间傅里叶变换中的时域数列，或者也可以说，它的时域频域数列均为连续傅里叶变换的采样，所以大概没什么好说的，这里直接上公式了。

{::nomarkdown}
公式里，\(N\)为离散数列的长度。
$$
\begin{align*}
x_n&=\frac1N\sum_{k=0}^{N-1}X_k\mathrm e^{\frac{2\pi}N\mathrm ikn}&n\in\{0,1,2,\cdots,N-1\}\\\
X_k&=\sum_{n=0}^{N-1}x_n\mathrm e^{-\frac{2\pi}N\mathrm ikn}&k\in\{0,1,2,\cdots,N-1\}\\
\end{align*}
$$
{:/nomarkdown}

## 傅里叶变换家族的关系

{::nomarkdown}

连续傅里叶变换（CFT）$$
\begin{align*}
X(f)&=\int_\mathbb Rx(t)\mathrm e^{-2\pi\mathrm ift}\,\mathrm dt\\
x(t)&=\int_\mathbb RX(f)\mathrm e^{2\pi\mathrm ift}\,\mathrm df\\
\end{align*}
$$

傅里叶级数（FS）$$
\begin{align*}
X[k]&=\frac1{T_0}\int_{T_0}\bar x(t)\mathrm e^{-\frac{2\pi}{T_0}\mathrm ikt}\,\mathrm dt\\
\bar x(t)&=\sum_{k\in\mathbb Z}X[k]\mathrm e^{\frac{2\pi}{T_0}\mathrm ift}\\
\end{align*}
$$

离散时间傅里叶变换（DTFT）$$
\begin{align*}
\bar X(f)&=\sum_{n\in\mathbb Z}x[n]\mathrm e^{-2\pi\mathrm iTfn}\\
x[n]&=T\int_{1/T}\bar X(f)\mathrm e^{2\pi\mathrm iTfn}\,\mathrm df\\
\end{align*}
$$

离散傅里叶变换（DFT）$$
\begin{align*}
X_k&=\sum_{n=0}^{N-1}x_n\mathrm e^{-\frac{2\pi}N\mathrm ikn}&k\in\{0,1,2,\cdots,N-1\}\\
x_n&=\frac1N\sum_{n=0}^{N-1}X_k\mathrm e^{\frac{2\pi}N\mathrm ikn}&n\in\{0,1,2,\cdots,N-1\}\\
\end{align*}
$$
{:/nomarkdown}

很明显的可以看出来，CFT的逆和CFT是相互对称的，DFT的逆和DFT是相互对称的，而FS和DTFT的逆，DTFT和FS的逆是相互对称的。实际上，四者分别可以通过离散化、周期化或者对周期取极限来实现相互转换：

CFT对于频域离散化、也就是对于时域周期化可以推导出FS

CFT对于时域离散化、也就是对于频域周期化可以推导出DTFT

FS再对于时域离散化、也就是对于频域周期化，时域频域均取一段周期内的有限数列得到DFT

DTFT再对于频域离散化、也就是对于时域周期化，时域频域均取一段周期内的有限数列得到DFT

反过来，对于被周期化了的，可以使周期趋向于无穷重新推导出上一个。

### 狄拉克\\(\\delta\\)函数

然后对于离散化、周期化也是有其对应的函数可以实现的，这里简要介绍一些有趣的性质，不再推导。

{::nomarkdown}
狄拉克\(\delta\)函数$$\delta(x)=\frac1{2\pi}\int_\mathbb R\mathrm e^{\mathrm ixt}\,\mathrm dt$$

更直观，但是更不严谨的定义是一个$$\delta(x)=\left\{\begin{align*}&+\infty&x=0\\&0&x\not=0\end{align*}\right.$$且满足$$\int_\mathbb R\delta(x)\,\mathrm dx=1$$
{:/nomarkdown}

<img src="https://upload.wikimedia.org/wikipedia/commons/4/48/Dirac_distribution_PDF.svg" width="50%" />

第一个可能不好一下子看明白，第二个大概就可以理解了并脑补出其图形了。这东西根本就不是通常意义下的函数，不过不管它了，主要看性质。

{::nomarkdown}
然后先给个狄拉克\(\delta\)函数的傅里叶变换，这里\(\hat\delta(x)\)表示变换后的函数$$
\begin{align*}
\hat\delta(x)&=\int_\mathbb R\delta(s)\mathrm e^{-2\pi\mathrm ixs}\,\mathrm ds\\
&=\int_\mathbb R\frac1{2\pi}\mathrm e^{-2\pi\mathrm ixs}\int_\mathbb R\mathrm e^{\mathrm ist}\,\mathrm dt\,\mathrm ds\\
&=\int_\mathbb R\frac1{2\pi}\int_\mathbb R\mathrm e^{\mathrm is(t-2\pi x)}\,\mathrm dt\,\mathrm ds\\
&=\int_\mathbb R\delta(s)\,\mathrm ds\\
&=1
\end{align*}
$$

再定义一下狄拉克梳子$$
\Delta_T(x)=\sum_{n\in\mathbb Z}\delta(x-Tn)
$$
{:/nomarkdown}

也被称为采样函数，因为其与连续函数的积就等于采了样，实数域上的积分是采到样本的和。很像是给离散数列求和是吧？所以实际上它和函数的积是给函数采样，而其与函数卷积是周期化。

<img src="https://upload.wikimedia.org/wikipedia/commons/4/49/Dirac_comb.svg" width="50%" />

{::nomarkdown}
于是在笛卡尔坐标系上出现了一排排间距为\(T\)的无限高的针，可得其周期为\(T\)，求出其傅里叶级数为$$
\begin{align*}
\Delta_T(x)&=\sum_{k\in\mathbb Z}\mathrm e^{\frac{2\pi}T\mathrm ikx}\frac1T\int_T\Delta_T(t)\mathrm e^{-\frac{2\pi}T\mathrm ikt}\,\mathrm dt\\
&=\frac1T\sum_{k\in\mathbb Z}\mathrm e^{\frac{2\pi}T\mathrm ikx}\int_T\delta(t)\mathrm e^{-\frac{2\pi}T\mathrm ikt}\,\mathrm dt\\
&=\frac1T\sum_{k\in\mathbb Z}\mathrm e^{\frac{2\pi}T\mathrm ikx}\hat\delta\left(\frac kT\right)\\
&=\frac1T\sum_{k\in\mathbb Z}\mathrm e^{\frac{2\pi}T\mathrm ikx}\\
\end{align*}
$$

瞬间简单很多，这也是其另一种等价的表现方式。然后我们就用这个来求其傅里叶变换$$
\begin{align*}
\hat\Delta_T(x)&=\int_\mathbb R\mathrm e^{-2\pi\mathrm ixt}\frac1T\sum_{k\in\mathbb Z}\mathrm e^{\frac{2\pi}T\mathrm ikt}\,\mathrm dt\\
&=\frac1T\sum_{k\in\mathbb Z}\int_\mathbb R\mathrm e^{2\pi\mathrm it\left(\frac kT-x\right)}\,\mathrm dt\\
&=\frac1T\sum_{k\in\mathbb Z}\delta\left(x-\frac kT\right)\\
&=\frac1T\Delta_{1/T}(x)=\Delta(Tx)\\
\end{align*}
$$

对于狄拉克\(\delta\)函数和梳子，我的理解是强行给了离散的值积分的定义，让数列可以按照函数方式处理。这样我们就可以给一个函数取样了。
{:/nomarkdown}

除此之外，它与函数的卷积，实际上就是取一部分的定义域并进行周期延拓。

本懒鬼考试回来了，接着打。

真正让傅里叶变换发挥作用的，是卷积定理和快速傅里叶变换（FFT），还请期待。
