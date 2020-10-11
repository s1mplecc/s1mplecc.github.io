---
title: LaTeX 数学公式语法
tags: []
categories: [Languages]
date: 2020-10-11 10:05:13
mathjax: true
---
## 特殊格式

|符号|代码|描述|
|:----:|:----:|:----:|
| $x^n$ | `x^n` | 上标符号 |
| $x_1$ | `x_1` | 下标符号 |
| $v_{\mbox{初始}}$ | `V_{\mbox{初始}}` | 汉字形式 |
| $\displaystyle \frac{x+y}{y+z}$ | `\displaystyle \frac{x+y}{y+z}` | 字体控制 |
| $\underline{x+y}$ | `\underline{x+y}` | 下划线符号 |
| $\overbrace{a+b+c+d}^{2.0}$ | `\overbrace{a+b+c+d}^{2.0}` | 上大括号 |
| $a+\underbrace{b+c}_{1.0}+d$ | `a+\underbrace{b+c}_{1.0}+d` | 下大括号 |
| $x \stackrel{f} \rightarrow y$ | `x \stackrel{f} \rightarrow y` | 上位符号 |
| ${n+1 \choose k}={n \choose k}+{n \choose k-1}$ | `{n+1 \choose k}={n \choose k}+{n \choose k-1}` | 组合公式 |

## 算数运算

|符号|代码|描述|
|:----:|:----:|:----:|
| $\pm$ | `\pm` | 加减运算 |
| $\mp$ | `\mp` | 减甲运算 |
| $\times$ | `\times` | 乘法运算 |
| $\cdot$ | `\cdot` | 点乘运算 |
| $\ast$ | `\ast` | 星乘运算 |
| $\div$ | `\div` | 除法运算 |
| $\frac{x+y}{y+z}$ | `\frac{x+y}{y+z}` 或 `{x+y} \over {y+z}` | 分式运算 |
| $\overline{x}$ | `\overline{x}` | 平均数运算 |
| $\sqrt x$ | `\sqrt x` | 开根号运算 |
| $\sqrt[n]{x+y}$ | `\sqrt[n]{x+y}` | 开方运算 |
| $\log_2 x$ | `\log_2 x` | 对数运算 |
| $\lim \limits_{n \to \infty}x_n$ | `\lim \limits_{n \to \infty}x_n` | 极限运算 |
| $\sum \limits_{i=1}^n{x_i}$ | `\sum \limits_{i=1}^n{x_i}` | 求和运算 |
| $\prod \limits_{i=1}^n{x_i}$ | `\prod \limits_{i=1}^n{x_i}` | 连乘运算 |
| $\int_{a}^{b} e^x\, dx$ | `\int_{a}^{b} e^x\, dx` | 积分运算 |
| $\iint \limits_D\, dx\,dy$ | `\iint \limits_D\, dx\,dy` | 双重积分 |
| $\oint_{L} x+y\, dx\,dy$ | `\oint_{L} x+y\, dx\,dy` | 闭合曲线积分 |
| $\partial x$ | `\partial x` | 微分运算 |

## 逻辑运算

|符号|代码|描述|
|:----:|:----:|:----:|
| $\geq$ | `\geq` | 大于等于 |
| $\leq$ | `\leq` | 小于等于 |
| $\neq$ | `\neq` | 不等于 |
| $\ngeq$ | `\ngeq` | 不大于等于 |
| $\not\geq$ | `\not\geq` | 不大于等于 |
| $\nleq$ | `\nleq` | 不小于等于 |
| $\not\leq$ | `\not\leq` | 不小于等于 |
| $\approx$ | `\approx` | 约等于 |
| $\equiv$ | `\equiv` | 恒定等于 |
| $\bigodot$ | `\bigodot` | 同或 |
| $\bigotimes$ | `\bigotimes` | 同与 |
| $\bigoplus$ | `\bigoplus` | 异或 |

## 集合运算

|符号|代码|描述|
|:----:|:----:|:----:|
| $\in$ | `\in` | 属于 |
| $\notin$ | `\notin` | 不属于 |
| $\subset$ | `\subset` | 子集 |
| $\supset$ | `\supset` | 子集 |
| $\subseteq$ | `\subseteq` | 真子集 |
| $\subsetneq$ | `\subsetneq` | 非真子集 |
| $\supseteq$ | `\supseteq` | 真子集 |
| $\supsetneq$ | `\supsetneq` | 非真子集 |
| $\not\subset$ | `\not\subset` | 非子集 |
| $\not\supset$ | `\not\supset` | 非子集 |
| $\cup$ | `\cup` | 并集 |
| $\cap$ | `\cap` | 交集 |
| $\setminus$ | `\setminus` | 差集 |
| $\mathbb{R}$ | `\mathbb{R}` | 实数集合 |
| $\mathbb{Z}$ | `\mathbb{Z}` | 自然数集合 |
| $\emptyset$ | `\emptyset` | 空集 |

## 数学符号

|符号|代码|描述|
|:----:|:----:|:----:|
| $\infty$ | `\infty` | 无穷 |
| $\imath$ | `\imath` | 虚数 |
| $\jmath$ | `\jmath` | 虚数 |
| $\hat{a}$ | `\hat{a}` | 数学符号 |
| $\check{a}$ | `\check{a}` | 数学符号 |
| $\breve{a}$ | `\breve{a}` | 数学符号 |
| $\tilde{a}$ | `\tilde{a}` | 数学符号 |
| $\bar{a}$ | `\bar{a}` | 数学符号 |
| $\vec{a}$ | `\vec{a}` | 矢量符号 |
| $\acute{a}$ | `\acute{a}` | 数学符号 |
| $\grave{a}$ | `\grave{a}` | 数学符号 |
| $\mathring{a}$ | `\mathring{a}` | 数学符号 |
| $\dot{a}$ | `\dot{a}` | 一阶导数符号 |
| $\ddot{a}$ | `\ddot{a}` | 二阶导数符号 |
| $\uparrow$ | `\uparrow` | 上箭头 |
| $\Uparrow$ | `\Uparrow` | 上箭头 |
| $\downarrow$ | `\downarrow` | 下箭头 |
| $\Downarrow$ | `\Downarrow` | 下箭头 |
| $\leftarrow$ | `\leftarrow` | 左箭头 |
| $\Leftarrow$ | `\Leftarrow` | 左箭头 |
| $\rightarrow$ | `\rightarrow` | 右箭头 |
| $\Rightarrow$ | `\Rightarrow` | 右箭头 |
| $\longrightarrow$ | `\longrightarrow` | 向右长箭头 |
| $\Longrightarrow$ | `\Longrightarrow` | 向右长箭头 |
| $A \xleftarrow{n=0} B \xrightarrow[T]{n>0} C$ | `A \xleftarrow{n=0} B \xrightarrow[T]{n>0} C` | 上下方可输入公式的箭头 |
| $\triangleq$ | `\triangleq` | 定义为 |
| $\because$ | `\because` | 因为 |
| $\therefore$ | `\therefore` | 所以 |
| $\forall$ | `\forall` | 任意 |
| $\exists$ | `\exists` | 存在 |
| $\ldots$ | `\ldots` | 底端对齐的省略号 |
| $\cdots$ | `\cdots` | 中线对齐的省略号 |
| $\vdots$ | `\vdots` | 竖直对齐的省略号 |
| $\ddots$ | `\ddots` | 斜对齐的省略号 |

## 其他

**矩阵表示**

```
\left[ \begin{matrix} 1 &2 &\cdots &4\\5 &6 &\cdots &8 \\ \vdots &\vdots &\ddots &\vdots \\ 13 &14 &\cdots &16\end{matrix} \right]
```

$$\left[ \begin{matrix} 1 &2 &\cdots &4\\5 &6 &\cdots &8 \\ \vdots &\vdots &\ddots &\vdots \\ 13 &14 &\cdots &16\end{matrix} \right]$$

**大括号公式**

```
F(n)= \begin{cases} 1 & \text{n = 0 或 1}\\ F(n-1)+F(n-2) & \text{n > 1} \end{cases}
```

$$F(n)= \begin{cases} 1 & \text{n = 0 或 1}\\ F(n-1)+F(n-2) & \text{n > 1} \end{cases}$$

## 希腊字母

| 大写 | 代码         | 小写 | 代码         |
|:---:|:----------:|:---:|:----------:|
| A | `A` | α | `\alhpa` |
| B | `B` | β | `\beta` |
| Γ | `\Gamma` | γ | `\gamma` |
| Δ | `\Delta` | δ | `\delta` |
| E | `E` | ϵ | `\epsilon` |
| Z | `Z` | ζ | `\zeta` |
| H | `H` | η | `\eta` |
| Θ | `\Theta` | θ | `\theta` |
| I | `I` | ι | `\iota` |
| K | `K` | κ | `\kappa` |
| Λ | `\Lambda` | λ | `\lambda` |
| M | `M` | μ | `\mu` |
| N | `N` | ν | `\nu` |
| Ξ | `\Xi` | ξ | `\xi` |
| O | `O` | ο | `\omicron` |
| Π | `\Pi` | π | `\pi` |
| P | `P` | ρ | `\rho` |
| Σ | `\Sigma` | σ | `\sigma` |
| T | `T` | τ | `\tau` |
| Υ | `\Upsilon` | υ | `\upsilon` |
| Φ | `\Phi` | ϕ | `\phi` |
| X | `X` | χ | `\chi` |
| Ψ | `\Psi` | ψ | `\psi` |
| Ω | `\v` | ω | `\omega` |