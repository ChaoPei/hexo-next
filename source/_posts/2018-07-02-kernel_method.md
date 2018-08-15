---
title: 核函数的基本原理
date: 2018-07-02 09:19:44
update: 2018-07-02 09:19:44
categories: 机器学习
tags: [机器学习, 核函数, 非线性映射, 回归]
---

机器学习领域中对于线性不可分的情况通常使用核函数的方法将其映射到高维空间，实现线性可分，之前一直对于核函数理解不到位，容易陷入数学公式推导的深区，本文则通过总结核函数的基本原理摒弃绝大多数数学推导，来对核函数的概念进行归纳总结。

<!--more-->

<script type="text/javascript"src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

假如已经有训练样本\\(x_i\\)，以及其标注 \\(y_i\\)，其中\\(i=1,2,3...N\\)。
现在欲求解一个回归函数\\(f(z)\\)，使得\\(f(x_i) = y_i\\)。

使用线性函数来进行预测，对于一个测试样本，线性回归可以写成如下形式：

$$f(z)=w^Tz$$

线性回归的求解方式有很多种，最常见的就是岭回归的方式，即最小化预测函数与标注的误差，通常还要带上参数的正则项，可以表示如下：

$$min(\lambda||w||^2+\sum_{i}(w^Tx_i-y_i)^2)$$

这个最小化函数有一种闭式解：

$$w=(X^T X+\lambda I)^{-1}X^Ty$$

其中，\\(x_i\\)是\\(X\\)的第\\(i\\)行，\\(y_i\\)是\\(y\\)的第\\(i\\)个元素

但是现实情况中，大部分情况是线性函数无法进行分类预测的，因此引入非线性函数，即\\(f(z)\\)是关于\\(z\\)的非线性函数。非线性函数能够处理线性函数搞不定的分类问题，但是非线性函数的估计求解较难，为了提高效率，引入了核方法。

首先对\\(z\\)进行一个非线性变换，也就是核方法中常说的非线性映射\\(\psi\\)，变换之后，可以找到一个关于\\(z\\)的变换结果的线性函数\\(f(z)\\)将数据进行线性分类，可以表示如下：

$$f(z)=w^T\psi(z)$$

其中\\(w\\)由训练样本的非线性变换的线性组合构成，这个就是核函数的基本假设之一，即：

$$w=\sum_{i}\alpha_i\psi(x_i)$$

二者结合，可以写成：

$$f(z)=\left[ \sum_i\alpha_i\psi(x_i) \right] \psi(z)=\sum_i\left[ \alpha_i\psi(x_i)\psi(z)\right]$$

记\\(\kappa(x_i,x_j)=\psi(x_i)\psi(x_j)\\)，这个就是核函数了，通过这个变换，\\(f(z)\\)可以记作：

$$f(z)=\sum_i\alpha_i\kappa(z,x_i)=\alpha^T\kappa(z)$$

其中\\(\alpha\\)为\\(N\times1\\)矢量，\\(\kappa(z)\\)为\\(N\times1\\)矢量，它的第\\(i\\)个值是训练样本\\(x_i\\)和测试样本\\(z\\)的核函数变换结果。

\\(f(z)\\)虽然是\\(z\\)的非线性函数，却是\\(\kappa(z)\\)的线性函数，可以使用线性函数的求解方法进行求解\\(\alpha\\)

我们称，原始参数\\(w\\)处于原空间prime space中，新参数\\(α\\)则处于对偶空间dual space中。

和真正的线性方法比起来，核方法在估计每一个\\(z\\)的标签时，需要计算z和训练集中每一个样本\\(x_i\\)的核函数\\(\kappa(z, x_i)\\)，因此其复杂度和训练集大小相关。