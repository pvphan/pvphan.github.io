---
layout: post
title: "Primer on camera calibration, part 2/2"
author: "Paul Vinh Phan"
categories: journal
image: schoolofathens.jpeg
tags: [camera,calibration,intrinsic,extrinsic,optimization,levenberg-marquardt]
---

{:centeralign: style="text-align: center;"}

School of Athens by Raphael ([Musei Vaticani](https://www.museivaticani.va/content/museivaticani/en/collezioni/musei/stanze-di-raffaello/stanza-della-segnatura/scuola-di-atene.html)).
{: centeralign }

In [Part 1]({% post_url 2022-03-27-camera-calibration-1 %}), we defined the calibration parameters $$\textbf{A}, \textbf{k}, \textbf{W}$$ and sum-squared projection error, $$E$$.
We now move on to how to estimate and refine these calibration parameters so we can reason spatially with images.
For a more complete walkthrough of each step of Zhang's method, here's a link to the [tutorial paper by Burger](https://www.researchgate.net/profile/Wilhelm-Burger/publication/303233579_Zhang's_Camera_Calibration_Algorithm_In-Depth_Tutorial_and_Implementation/links/5eaad8c9a6fdcc70509c3c9b/Zhangs-Camera-Calibration-Algorithm-In-Depth-Tutorial-and-Implementation.pdf) again.

Table of Contents:
* TOC
{:toc}


# What is 'Zhang's method'?

Currently, the most popular method for calibrating a camera is **Zhang's method** published in [A Flexible New Technique for Camera Calibration](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr98-71.pdf) by Zhengyou Zhang (1998).
Older methods typically required a precisely made 3D calibration target or a mechanical system to precisely move the camera or target.
In contrast, Zhang's method requires only a 2D calibration target and only loose requirements on how the camera or target moves.
This means that anyone with a desktop printer and a little time can accurately calibrate their camera!

The general strategy of Zhang's method is to impose naïve assumptions as constraints to get an **initial guess** of calibration parameter values with singular value decomposition (SVD), then release those constraints and **refine** those guesses with non-linear least squares optimization.

The ordering of steps for Zhang's method are:
1. Use the 2D-3D point associations to compute the per-view set of homographies $$\textbf{H}$$.
2. Use the homographies $$\textbf{H}$$ to compute an *initial guess* for the **intrinsic matrix**, $$A_{init}$$.
3. Using the above, compute an *initial guess* for the **distortion parameters**, $$\textbf{k}_{init}$$.
4. Using the above, compute an *initial guess* **camera pose** (per-view) in target coordinates, $$\textbf{W}_{init}$$.
5. Initialize **non-linear optimization** with the *initial guesses* above and then **iterate** to minimize **projection error**, producing $$A_{final}$$, $$\textbf{k}_{final}$$, and $$\textbf{W}_{final}$$.


# The steps of Zhang's method

## Zhang.1) Homography estimation using Direct Linear Transformation (DLT)

Compute initial intrinsic matrix, A

First, we need to use the 2D-3D point associations to compute the homographies $$\textbf{H} = [H_1, H_2, ..., H_n]$$, for each of the $$n$$ views in the dataset.

To do this we employ two tricks:
- Trick #1: Assume there is **no distortion** in the camera (7.a). This is of course not true, but it will let us compute approximate values.
- Trick #2: We define the world coordinate system so that it's $$z = 0$$ plane is the **plane of the calibration target**.
This allows us to drop the $$z$$ term of the 3D world points (7.b).

Trick #1 simplifies our projection equation (5) to a distortion-less [**pinhole camera model**](https://hedivision.github.io/Pinhole.html):

$$
\begin{equation}
u_{ij}
=
hom^{-1}
(
\textbf{A}
\cdot
\Pi \cdot W_i \cdot {}^wX_{ij}
)
\tag{7.a}\label{eq:7.a}
\end{equation}
$$

And after expanding, trick #2 allows us to drop some dimensions from this expression:

$$
\begin{equation}
\begin{pmatrix}
u\\
v\\
1\\
\end{pmatrix}
_{ij}
=
hom^{-1}
\left(
\textbf{A}
\cdot
\begin{pmatrix}
1 & 0 & 0 & 0\\
0 & 1 & 0 & 0\\
0 & 0 & 1 & 0\\
\end{pmatrix}
\begin{pmatrix}
|     & |     & |     & t_x\\
r_{x} & r_{y} & r_{z} & t_y\\
|     & |     & |     & t_z\\
0 & 0 & 0 & 1\\
\end{pmatrix}
_{i}
\begin{pmatrix}
x_w\\
y_w\\
0\\
1\\
\end{pmatrix}
_{ij}
\right)
\tag{7.b}\label{eq:7.b}
\end{equation}
$$

Now we can strike out the 3rd column of $$W_i$$ and 3rd row of $${}^wX_{ij}$$, and rewrite the $$hom^{-1}(\cdot)$$ as a scale factor $$s$$.
We'll call the product of $$\textbf{A}$$ and that 3-by-3 matrix our

$$
\begin{equation}
s
\begin{pmatrix}
u\\
v\\
1\\
\end{pmatrix}
_{ij}
=
\underbrace{
    \textbf{A}
    \cdot
    \begin{pmatrix}
    |     & |     & t_x\\
    r_{x} & r_{y} & t_y\\
    |     & |     & t_z\\
    \end{pmatrix}
    _{i}
}_{H_i}
\begin{pmatrix}
x_w\\
y_w\\
1\\
\end{pmatrix}
_{ij}
\tag{8}\label{eq:8}
\end{equation}
$$

We've now arrived at an expression for our homography $$H_i$$ which relates a 2D point on the target plane to a 2D point on our image plane for the $$i$$-th view. We'll also introduce a change in notation for the homogeneous image point which we'll need to solve for $$H_i$$:

$$
\begin{equation}
s
\begin{pmatrix}
u\\
v\\
1\\
\end{pmatrix}
_{ij}
=
\begin{pmatrix}
\hat{u}\\
\hat{v}\\
\hat{w}\\
\end{pmatrix}
=
H_i
\begin{pmatrix}
x_w\\
y_w\\
1\\
\end{pmatrix}
_j
\tag{9}\label{eq:9}
\end{equation}
$$

Next, we'll use the Direct Linear Transformation (DLT) technique to rewrite this expression in homogeneous form $$M \cdot \textbf{h} = 0$$.
After that, we can apply Singular Value Decomposition (SVD) to solve for $$\textbf{h}$$, and then reshape it into $$H$$.
For readability, we'll assume the following equations refer to the $$i$$-th view and drop the $$i$$.
To support a reformulation of (9), we define $$H$$ and $$\textbf{h}$$ as:

$$
\begin{equation}
H
=
\begin{pmatrix}
h_{11} & h_{12} & h_{13} \\
h_{21} & h_{22} & h_{23} \\
h_{31} & h_{32} & h_{33} \\
\end{pmatrix}
\tag{10}\label{eq:10}
\end{equation}
$$

$$
\begin{equation}
\textbf{h}
=
\begin{pmatrix}
h_{11} & h_{12} & h_{13} & \
h_{21} & h_{22} & h_{23} & \
h_{31} & h_{32} & h_{33}
\end{pmatrix}
^\top
\tag{11}\label{eq:11}
\end{equation}
$$

### Reformulate using DLT

Now we'll reformulate (9) so that we can solve for the values of $$H$$.
We desire an expression in terms of $$u$$, $$v$$, $$x_w$$, and $$y_w$$ (which are known) with respect to unknown values of $$\textbf{h}$$ (which we will solve for).

Relating the first and second terms from (9), we can rewrite them as a pair of homogeneous equations. Note that $$s = \hat{w}$$:

$$
\begin{equation}
\left\{
\begin{aligned}
u \hat{w} - \hat{u} = 0 \\
v \hat{w} - \hat{v} = 0

\end{aligned} \right.
\tag{12}\label{eq:12}
\end{equation}
$$

And relating the second and third terms from (9) with the elements of $$H_i$$ distributed:

$$
\begin{equation}
\begin{split}

\hat{u} = h_{11} x_w + h_{12} y_w + h_{13} \\
\hat{v} = h_{21} x_w + h_{22} y_w + h_{23} \\
\hat{w} = h_{31} x_w + h_{32} y_w + h_{33} \\

\end{split}
\tag{13}\label{eq:13}
\end{equation}
$$

### Solve for H using SVD

For a Python example of this, you can look at [linearcalibrate.py: estimateHomography](https://github.com/pvphan/camera-calibration/blob/main/src/linearcalibrate.py#L24).


## Zhang.2) Compute initial intrinsic matrix, A

For a Python example of this, you can look at [linearcalibrate.py: computeIntrinsicMatrix](https://github.com/pvphan/camera-calibration/blob/main/src/linearcalibrate.py#L93).


## Zhang.3) Compute initial distortion vector, k


## Zhang.4) Compute initial extrinsic parameters, W


## Zhang.5) Refine A, k, W using non-linear optimization

Until now, we've been estimating $$\textbf{A}, \textbf{k}, \textbf{W}$$ to get a good initialization point for our optimization.
In non-linear optimization, it's often impossible to arrive at a good solution unless the initialization point for the variables was at least somewhat reasonable.

1. Express the projection equation symbolically (e.g. `sympy`).
1. Take the partial derivatives of the projection expression with respect to the calibration parameters.
1. Arrange these partial derivative expressions into the Jacobian matrix $$J$$ for the projection expression.

Now we are ready to run our non-linear optimization algorithm. Levenberg-Marquardt is a popular choice as it works well in practice.

1. Start by setting the *current* calibration parameters $$\textbf{P}_{curr}$$ to the initial guess values computed in Zhang.1 - Zhang.3.
1. Use $$\textbf{P}_{curr}$$ to project the input world points $${}^wX_{ij}$$ to thier image coordinates $$u_{ij}$$
1. Evaluate the Jacbobian $$J$$ for all input points at the *current* calibration parameter values.

Below, green crosses are the measured 2D marker points and magenta crosses are the projection of the associated 3D points using the 'current' camera parameters.
This gif plays through the iterative refinement of the camera parameters (step #5 of Zhang's method).
(Generation of this gif is part of the github repo linked at the top of this post.)

![](assets/img/reprojection.gif)
{: centeralign }


# Final remarks

In part 2, we went into the theory of Zhang's popular calibration method for computing intial values for $$\textbf{A}, \textbf{k}, \textbf{W}$$ and their refinement.
Camera calibration can be daunting due to assumed knowledge in camera projection models, linear algebra, and optimization.
We walked through a common camera projection model and then stepped through Zhang's method, introducing numerical methods as needed.

Thanks for reading!


# Appendix

### Singular Value Decomposition (SVD)

SVD decomposes a matrix $M$ ($m$,$n$) to three matrices $U$ ($m$,$m$), $\Sigma$ ($m$,$n$), and $V^\top$ ($n$,$n$), such that $$M = U \cdot \Sigma \cdot V^\top$$.
The properties of the resulting three matrices are like that of Eigenvalue and Eigenvector decomposition.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c8/Singular_value_decomposition_visualisation.svg/206px-Singular_value_decomposition_visualisation.svg.png)
{: centeralign }
Visualization of SVD from Wikipedia.
{: centeralign }

The properties of this decomposition have many uses, one of which is **solving homogeneous linear systems** of the form
$$M \cdot x = 0$$.

A solution for the value of $x$ for this linear system is the smallest eigenvector of $V^\top$.
Using numpy, solving looks like this:

```python
# M * x = 0, where M (m,n) is known and we want to solve for x (n,1)
U, Σ, V_T = np.linalg.svd(M)
x = V_T[-1]
```

Additional links:
- [(Wikipedia) More applications of the SVD](https://en.wikipedia.org/wiki/Singular_value_decomposition#Applications_of_the_SVD)
- [(blog) Explanation of SVD by Greg Gunderson](https://gregorygundersen.com/blog/2018/12/10/svd/)


### Non-linear least squares optimization (Levenberg-Marquardt)

Non-linear optimization is the task of computing a set of parameters which **minimizes a non-linear value function**.
This is a huge topic of its own that I'm not quite ready to articulate.
Maybe I'll do a post in the future and update this one with a link.

![](https://i.stack.imgur.com/gdJ3v.gif)
{: centeralign }

Visualization of Gauss-Newton optimization.
{: centeralign }

Additional links:
- [Algorithms for Optimization (Kochenderfer & Wheeler)](https://mitpress.mit.edu/books/algorithms-optimization) (yay, Tim!)


Substituting the definition of predicted position $$u_{ij}$$ from (5):

$$
\begin{equation}
E
=
\sum\limits_{i}^{n} \sum\limits_{j}^{m}

||
z_{ij}
-
hom^{-1}
(
    \textbf{A}
    \cdot
    hom(
        distort(
            hom^{-1}(
                \Pi \cdot W_i \cdot {}^wX_{ij}
            ),
            \textbf{k}
        )
    )
)
||^2
\tag{6.b}\label{eq:6.b}
\end{equation}
$$

