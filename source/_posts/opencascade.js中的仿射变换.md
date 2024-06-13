---
title: opencascade.js中的仿射变换
date: 2024-06-13 13:52:36
tags: 
  - opencascade.js
  - occt
  - 前端
  - 3D
---

## 基本概念
[仿射变换（Affine transformation）](https://zh.wikipedia.org/wiki/%E4%BB%BF%E5%B0%84%E5%8F%98%E6%8D%A2)，又称仿射映射，是指在几何中，对一个向量空间进行一次线性变换并接上一个平移，变换为另一个向量空间。

### 矩阵表示
计算机中，通常使用4*4的矩阵来表示放射变换
<details>
<summary>放射变换矩阵</summary>

#### 平移
$$
\left[
\begin{matrix}
  1 & 0 & 0 & x \\
  0 & 1 & 0 & y \\
  0 & 0 & 1 & z \\
  0 & 0 & 0 & 1 \\
\end{matrix}
\right]
$$

#### 旋转
- 绕X轴
$$
\left[
\begin{matrix}
  1 & 0 & 0 & 0 \\
  0 & cos\theta & -sin\theta & 0 \\
  0 & sin\theta & cos\theta & 0 \\
  0 & 0 & 0 & 1 \\
\end{matrix}
\right]
$$
- 绕Y轴
$$
\left[
\begin{matrix}
  cos\theta & 0 & sin\theta & 0 \\
  0 & 1 & 0 & 0 \\
  -sin\theta & 0 & cos\theta & 0 \\
  0 & 0 & 0 & 1 \\
\end{matrix}
\right]
$$
- 绕Z轴
$$
\left[
\begin{matrix}
  cos\theta & -sin\theta & 0 & 0 \\
  sin\theta & cos\theta & 0 & 0 \\
  0 & 0 & 1 & 0 \\
  0 & 0 & 0 & 1 \\
\end{matrix}
\right]
$$

#### 缩放
$$
\left[
\begin{matrix}
  x & 0 & 0 & 0 \\
  0 & y & 0 & 0 \\
  0 & 0 & z & 0 \\
  0 & 0 & 0 & 1 \\
\end{matrix}
\right]
$$

#### 剪切
$$
\left[
\begin{matrix}
  1 & yx & zx & 0 \\
  xy & 1 & zy & 0 \\
  xz & yz & 1 & 0 \\
  0 & 0 & 0 & 1 \\
\end{matrix}
\right]
$$
</details>

所有复杂的变换都可以通过上述矩阵通过连乘得到

## 变换方法
`OCCT`中有`BRepBuilderAPI_Transform`、`BRepBuilderAPI_GTransform`两种

### BRepBuilderAPI_Transform
- 保持Shape的类型
- 保留变换矩阵，存储在Shape的`Location`中
- 本质上是修改Shape的Location，在Shape本身已有Location变换的情况下，省去了额外的计算
- 速度快

### BRepBuilderAPI_GTransform
- Shape的类型有可能发生改变 (例如Line变成Spline，Plane变成SplineFae)
- 不保留变换矩阵
- 速度慢
