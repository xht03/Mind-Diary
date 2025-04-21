---
title: "6 Z变换"
author: "xht03"
date: 2025-04-21 11:34:50
tags:
- notes
categories:
- Digital Signal Processing
header-includes:
- \usepackage{xeCJK}
---

## Z变换

一个序列 $x(n)$ 的离散时间傅里叶变换（DTFT）为：

$$
X(e^{j\omega}) = \sum_{n=-\infty}^{+\infty} x(n)e^{-j\omega n}
$$

上式收敛条件之一是序列 $x(n)$ 必须是绝对可和的，即：

$$
\sum_{n=-\infty}^{+\infty} |x(n)| < +\infty
$$

然而在实际应用中，许多序列不是绝对可求和的，因此不能进行离散时间傅里叶变换。为此，我们引入了 Z 变换，用以处理非绝对可求和的序列。

给定 $x(n)$ ，其 Z 变换定义如下：

- 双边 Z 变换：

  $$
  X(z) = \sum_{n=-\infty}^{+\infty} x(n)z^{-n}
  $$

- 单边 Z 变换：

  $$
  X(z) = \sum_{n=0}^{+\infty} x(n)z^{-n}
  $$

令 $z = re^{j\omega}$ ，那么：

$$
X(z) = \sum_{n=-\infty}^{+\infty} x(n)r^{-n}e^{-j\omega n} = \sum_{n=-\infty}^{+\infty} [x(n)r^{-n}]e^{-j\omega n}
$$

我们可以将 $x(n)r^{-n}$ 看作一个新的序列 $y(n)$ ，则原序列 $x(n)$ 的 Z 变换可以看作是序列 $y(n)$ 的离散时间傅里叶变换（DTFT）。

**由 DTFT 的收敛条件可知，$X(z)$ 收敛的条件是序列 $y(n)=x(n)r^{-n}$ 必须是绝对可和的。**

---

## 收敛域

对任意给定序列 $x(n)$ ，使其 Z 变换收敛的所有 Z 值的结合称为收敛域（ROC）。

Z 变换存在的条件是**级数绝对可和**，即：

$$
\sum_{n=-\infty}^{+\infty} |x(n)z^{-n}| < +\infty
$$

满足上述不等式的 $z$ 的取值范围称为收敛域（ROC）。

---

## 零点与极点

常用的 Z 变换是有理函数，我们可以将其写为两个多项式之比：

$$
X(z) = \frac{P(z)}{Q(z)}
$$

- 零点：$P(z)$ 的根。

- 极点：$Q(z)$ 的根。

我们注意到，在极点处 Z 变换不存在，因此收敛域不包括极点，收敛域总是用极点限定其边界的。

---

**例 1**

考虑如下序列的 Z 变换及其收敛域：

$$
x[n] = \alpha^n u[n]
$$

则 Z 变换为：

$$
X(z) = \sum_{n=0}^{+\infty} \alpha^n z^{-n} = \sum_{n=0}^{+\infty} (\frac{\alpha}{z})^n
$$

当 $|\frac{\alpha}{z}| < 1$ 时，级数收敛，$X(z) = \frac{1}{1-\alpha z^{-1}} $ ，收敛域为 $|z| > |\alpha|$ 。

![](https://ref.xht03.online/202504211816989.png)

---

**例 2**

考虑如下序列的 Z 变换及其收敛域：

$$
x[n] = -\alpha^n u[-n-1]
$$

则 Z 变换为：

$$
X(z) = -\sum_{n=-\infty}^{-1} \alpha^n z^{-n} = -\alpha^{-1} z \sum_{n=0}^{+\infty} \alpha^{-n} z^{n} 
$$

当 $|\alpha^{-1}z| < 1$ 时，级数收敛，

$$
X(z) = -\frac{-\alpha^{-1}z}{1-\alpha^{-1}z} = \frac{1}{1-\alpha z^{-1}}
$$

收敛域为 $|z| < |\alpha|$ 。

![](https://ref.xht03.online/202504211829908.png)

从例 1 和例 2 中我们可以看出，不同的 $x(n)$ 可能有相同的 Z 变换，但收敛域不同。为了保证由逆 Z 变换求出的序列是唯一的，我们必须指明收敛域。

---

