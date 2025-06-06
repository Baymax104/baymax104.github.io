---
layout: post
title: 《机器学习》——支持向量机
categories:
- 人工智能
- 机器学习
tags:
- 人工智能
- 机器学习
- SVM
- 支持向量机
typora-root-url: ./
math: true
---

## 基本型

每个样本可以对应到样本空间中的一个点，对赢样本分类也就是找到一个超平面，将不同类别的样本划分开

超平面使用线性方程来表示
$$
\omega^Tx+b=0
$$
假设超平面能够将样本正确分类，则有如下条件
$$
\begin{align}\left\{\begin{aligned}
\omega^Tx+b&\ge+1,&y_i&=1\\
\omega^Tx+b&\le-1,&y_i&=-1
\end{aligned}\right.\end{align}
$$
距离超平面最近的几个使上式等号成立的样本点称为支持向量，异类支持向量到超平面的距离之和称为间隔
$$
\gamma=\frac{2}{\vert\vert\omega\vert\vert}
$$
在满足约束条件的情况下，找到最大间隔，即最小化$\vert\vert\omega\vert\vert$，等价于最大化$\vert\vert\omega\vert\vert$，SVM基本型如下
$$
\begin{align}
\min\limits_{\omega,b}\quad&{1\over2}\vert\vert\omega\vert\vert^2\\
s.t.\quad &y_i\cdot(\omega^Tx_i+b)\ge1,\quad i=1,2,...,m
\end{align}
$$

## 核函数

在现实样本中，原始样本空间中也许不存在一个超平面能够将样本正确分类，此时可将样本映射到更高维的空间中，样本在高维空间中线性可分，此时存在一个可将样本正确分类的超平面
