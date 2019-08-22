---
layout: post
title:  "GraphSLAM formulation"
date:   2019-08-22 09:00:01
categories: research
---


# GraphSLAM

## Problem Definition

Given $$x_0$$, $$u_{1:t}$$, $$z_{1:t}$$, $$c_{1:t}$$, $$R_{1:t}$$, $$Q_{1:t}$$,

- $$x_0$$: initial pose
- $$u_{1:t}$$: prediction input 
- $$R_{1:t}$$: prediction uncertainty
- $$z_{1:t}$$: measurements
- $$Q_{1:t}$$: measurement uncertainty

Full SLAM posterior of the trajectory:

$$
\begin{aligned}
& p(y_{0:t}|z_{1:t}, u_{1:t}, c_{1:t}) \\\\
& y_{0:t} = \begin{bmatrix} x_0 \\ x_1 \\ \vdots \\ x_t \\ m \end{bmatrix}, \quad
y_t = \begin{bmatrix} x_0 \\ m \end{bmatrix} \\\\
& \mu_{0:t} = \begin{bmatrix} x_0 \\ x_1 \\ \vdots \\ x_t \\ m \end{bmatrix}, \quad
M_t = \begin{bmatrix} x_0 \\ m \end{bmatrix}
\end{aligned}
$$

## Expanding the posterior

$$
\begin{aligned}
& f(y_{0:t}) = p(y_{0:t}|z_{1:t}, u_{1:t}, c_{1:t}) \\
& = p(y_{0:t}|z_{1:t-1}, z_t, u_{1:t}, c_{1:t}) \\
& = \eta \ p(z_t|y_{0:t}, z_{1:t-1}, u_{1:t}, c_{1:t})p(y_{0:t}|z_{1:t-1}, u_{1:t}, c_{1:t}) \\
& = \eta \ p(z_t|y_{0:t}, z_{1:t-1}, u_{1:t}, c_{1:t})p(y_{0:t}|z_{1:t-1}, u_{1:t}, c_{1:t}) \\
& = \eta \ p(z_t|y_t, c_t)p(x_t, y_{0:t-1}|z_{1:t-1}, u_{1:t}, c_{1:t}) \\
& \quad\quad\quad (p(x, y|z)=p(x|y, z)p(y|z)) \\
& = \eta \ p(z_t|y_t, c_t)p(x_t|y_{0:t-1}, z_{1:t-1}, u_{1:t}, c_{1:t})p(y_{0:t-1}| z_{1:t-1}, u_{1:t}, c_{1:t}) \\
& = \eta \ p(z_t|y_t, c_t)p(x_t|x_{t-1}, u_t)p(y_{0:t-1}| z_{1:t-1}, u_{1:t}, c_{1:t}) \\
& \quad\quad\quad p(y_{0:t-1}| z_{1:t-1}, u_{1:t}, c_{1:t})=f(y_{0:t-1}) \\
& = \eta \ p(y_0) \prod_{t} \begin{bmatrix} p(x_t|x_{t-1}, u_t) p(z_t|y_t, c_t) \end{bmatrix} \\
& = \eta \ p(x_0)p(m) \prod_{t} \begin{bmatrix} p(x_t|x_{t-1}, u_t) p(z_t|y_t, c_t) \end{bmatrix} \\
& = \eta' \ p(x_0) \prod_{t} \begin{bmatrix} p(x_t|x_{t-1}, u_t) p(z_t|y_t, c_t) \end{bmatrix} \text{(no prior for map)} \\
\end{aligned}
$$



The goal of GraphSLAM:
$$
\begin{aligned}
& \widehat{y}_{0:t} = arg \max_{y_{0:t}} p(y_{0:t}|z_{1:t-1}, z_t, u_{1:t}, c_{1:t}) \\
& = arg \min_{y_{0:t}} -\log p(y_{0:t}|z_{1:t-1}, z_t, u_{1:t}, c_{1:t}) \\
& = arg \min_{y_{0:t}} -\log \eta' - \log p(x_0) - \sum_t \log p(x_t|x_{t-1}, u_t) - \sum_t \sum_i \log p(z_t^i|y_t, c_t^i) \\
& = arg \min_{y_{0:t}} J_{Graph}
\end{aligned} \\
$$



The reason to apply log to the posterior

- As log is monotonically increasing function, $$\widehat{x} = arg \min_{x} f(x) = arg \min_{x} \log f(x)$$
- Since motion and measurement models are formulated by Gaussian distribution (exponential function), applying log makes the equation simpler



## Calculate the objective function

$$
J_{Graph} = {1 \over 2} \begin{Bmatrix} x_0^T \Sigma x_0 + \sum_t \begin{bmatrix} x_t - g(x_{t-1}, u_t) \end{bmatrix}^T R_t^{-1} \begin{bmatrix} x_t - g(x_{t-1}, u_t) \end{bmatrix} \\
+ \sum_t \sum_i \begin{bmatrix} z_t^i - h(y_t, c_t^i) \end{bmatrix}^T Q_t^{-1} \begin{bmatrix} z_t^i - h(y_t, c_t^i) \end{bmatrix} + \text{const.} \\
\end{Bmatrix}
$$

Solving $$\widehat{y}_{0:t}$$ is a nonlinear least-square problem ($$g()$$, $$h()$$ functions are nonlinear over $$y_{0:t}$$)

Approximate nonlinear functions to the first order Talyor expansion

$$
\begin{align}
& g(x_{t-1}, u_t) \approx g(\mu_{t-1}, u_t) + G_t (x_{t-1} - \mu_{t-1}) \\
& G_t = {\partial \over \partial x_{t-1}} g(x_{t-1}, u_t) \mid_{x_{t-1} = \mu_{t-1}} 
= {\partial \over \partial \mu_{t-1}} g(\mu_{t-1}, u_t) \\
& h(y_t, c_t^i) \approx h(M_t, c_t^i) + H_t^i (y_t - M_t) \\
& H_t^i = {\partial \over \partial y_t} h(y_t, c_t^i) \mid_{y_t = M_t} 
= {\partial \over \partial M_t} h(M_t, c_t^i)
\end{align}
$$

$$
J_{Graph} = {1 \over 2} \left\{ x_0^T \Sigma x_0 
+ \sum_t \begin{bmatrix} x_t - g(\mu_{t-1}, u_t) - G_t (x_{t-1} - \mu_{t-1}) \end{bmatrix}^T R_t^{-1} \\
\begin{bmatrix} x_t - g(\mu_{t-1}, u_t) - G_t (x_{t-1} - \mu_{t-1}) \end{bmatrix} \\
+ \sum_t \sum_i \begin{bmatrix} z_t^i - h(M_t, c_t^i) - H_t^i (y_t - M_t) \end{bmatrix}^T Q_t^{-1} \\ 
\begin{bmatrix} z_t^i - h(M_t, c_t^i) - H_t^i (y_t - M_t) \end{bmatrix} + \text{const.} \right\} \\
$$

$$
J_{Graph} = {1 \over 2} \left\{ x_0^T \Sigma x_0 
+ \sum_t \left[ (x_t - G_t x_{t-1})^T R_t^{-1} (x_t - G_t x_{t-1}) \\
- 2(x_t - G_t x_{t-1})^T R_t^{-1} (g(\mu_{t-1}, u_t) - G_t \mu_{t-1}) + \text{const.} \right] \\
+ \sum_t \sum_i \left[ (-H_t^i y_t)^T Q_t^{-1} (-H_t^i y_t) \\
+ (-H_t^i y_t)^T Q_t^{-1} (z_t^i - h(M_t, c_t^i)) + H_t M_t) + \text{const.} \right] \right\}
$$

$$
J_{Graph} = {1 \over 2} x_0^T \Sigma x_0 
+ {1 \over 2} \sum_t \begin{bmatrix} x_{t-1}^T & x_t^T \end{bmatrix} 
\begin{bmatrix} -G_t^T \\ I \end{bmatrix} R_t^{-1} 
\begin{bmatrix} -G_t & I \end{bmatrix} 
\begin{bmatrix} x_{t-1} \\ x_t \end{bmatrix} \\
- \sum_t \begin{bmatrix} x_{t-1}^T & x_t^T \end{bmatrix} 
\begin{bmatrix} -G_t^T \\ I \end{bmatrix} R_t^{-1} 
\begin{bmatrix} g(\mu_{t-1}, u_t) - G_t \mu_{t-1} \end{bmatrix} \\
+ {1 \over 2} \sum_t \sum_i y_t^T {H_t^i}^T  Q_t^{-1} H_t^i y_t \\
- \sum_t \sum_i y_t^T {H_t^i}^T Q_t^{-1} (z_t^i - h(M_t, c_t^i)) + H_t M_t) + \text{const.}
$$


In vector-matrix form, the linearized equation becomes
$$
J_{Graph} = {1 \over 2} y_{0:t}^T \Omega y_{0:t} - \xi^T y_{0:t} + \text{const.}
$$



## Nonlinear Optimization

1. Set an initial guess $$\mu_{0:t}$$
2. Compute $$\Omega$$ and $$\xi$$ in $$J_{Graph}$$ with $$\mu_{0:t}$$
3. Update $$\mu_{0:t} \leftarrow \Omega^{-1}\xi$$
4. Iterate 2~3 until convergence

# Transformation Between Point Clouds

Given two point clouds $$\mathbf{p}=\left\{ p_i \right\}_{i=1:N}$$, $$\mathbf{q}=\left\{ q_i \right\}_{i=1:N}$$ where $$p_i$$ corresponds to $$q_i$$ in a different coordinate system.  

The goal is to find $$R, t$$ which minimizes $$\sum_i \begin{Vmatrix} Rp_i+t - q_i \end{Vmatrix}$$



## Algorithm

1. Translate point clouds for their center to be at the origin

   $$\mathbf{p'} = \left\{p_i - \mathbf{\bar{p}}\right\}_{i=1:N}, \quad \mathbf{q'} = \left\{q_i - \mathbf{\bar{q}}\right\}_{i=1:N}$$

2. Create rotation motion matrix

   $$\mathbf{m} = \left\{p_i' - q_i'\right\}_{i=1:N}, \quad M=\begin{bmatrix} m_0 \\ m_1 \\ ... \\m_N \end{bmatrix}$$

3. SVD on $$M$$ to find null vector of $$M$$

   $$M = U \Sigma V^T$$: (NxN) x (Nx3) x (3x3)

4. The last column of $$V$$ corresponding to the least singular value $$\sigma_3$$ is the rotational axis

5. Compute rotation angle

   $$cos \theta = {1 \over N} \sum_i {p \cdot q \over \begin{vmatrix} p \end{vmatrix} \begin{vmatrix} q \end{vmatrix}}$$

6. Compute rotation matrix

   1. angle, axis -> quaternion -> rotation matrix
   2. angle, axis -> twist -> rotation matrix (Rodrigues' formula)

7. Compute translation vector

   $$t = {1 \over N} \sum_i \left\{ q_i - Rp_i \right\}$$



