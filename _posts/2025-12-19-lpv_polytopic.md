---
title: "Polytopic LPV for SRMs"
categories: 제어
tags: Linear-algebra, Control-Theory
date: 2025-12-19 10:00:00
permalink: /posts/2025/12/polyLPVSRM/
excerpt: "Linear parameter varying, switched reluctance cmotors."
---
# SRM Polytopic (Quasi-)LPV Model Summary for Python Verification

This document summarizes the LPV/polytopic modeling decisions we made so far, so you can implement Python code to compare:
1) the **original (nonlinear/quasi-LPV) SRM dynamics**, and  
2) the **polytopic LPV approximation** constructed over excited-current operating regions.

We assume **unipolar phase currents** and **90° periodicity** of inductance profiles.

---

## 0. Notation

Phases: \(j \in \{a,b,c\}\)

Given functions (from SRM magnetic model / identification):
- \(L_j(\theta)\): phase inductance
- \(L'_j(\theta)=\dfrac{\partial L_j(\theta)}{\partial\theta}\)

Motor parameters:
- \(R\): phase resistance
- \(J_m\): inertia
- \(B_m\): viscous friction coefficient

Inputs:
- Phase voltages \(v_a,v_b,v_c\)
- Disturbance torque \(T_d\)

---

## 1. “Reference” (Original) Quasi-Nonlinear SRM State Model (for simulation baseline)

### 1.1 State, input, output
State:
\[
x=\begin{bmatrix} i_a & i_b & i_c & \omega & \theta \end{bmatrix}^\top
\]
Input:
\[
u=\begin{bmatrix} v_a & v_b & v_c & T_d \end{bmatrix}^\top
\]
Output (measured currents):
\[
y=\begin{bmatrix} i_a & i_b & i_c \end{bmatrix}^\top,\qquad
C=\begin{bmatrix} I_{3\times3} & 0_{3\times2} \end{bmatrix}
\]

### 1.2 Dynamics (baseline model)
Electrical (per phase):
\[
\dot i_j
= -\frac{R}{L_j(\theta)}\,i_j
-\frac{L'_j(\theta)}{L_j(\theta)}\,i_j\,\omega
+\frac{1}{L_j(\theta)}\,v_j
\]

Mechanical:
\[
\dot\omega
= -\frac{B_m}{J_m}\omega
+\sum_{j\in\{a,b,c\}}\frac{1}{J_m}L'_j(\theta)\,i_j^2
-\frac{1}{J_m}T_d
\]

Kinematics:
\[
\dot\theta=\omega
\]

> This model is “ground truth” for comparison.  
> If you use phase offsets (e.g., \(L_a(\theta)=L(\theta-\phi_a)\)), implement them consistently in the functions \(L_j(\theta)\), \(L'_j(\theta)\).

---

## 2. Goal: Polytopic LPV Approximation with Vertex-LMI-Friendly Constant Matrices

We wanted a polytopic form where **each vertex has constant \(A\) matrices**, to improve feasibility of vertex-wise LMI conditions.

### Key modeling idea (B):
Use a **local affine approximation** of the term \(\dfrac{R}{L_j(\theta)}\) so that part of it becomes a \(\theta\)-state coupling term (i.e., populating \(A(1:3,5)\)).

---

## 3. Scheduling / Periodicity

### 3.1 90° periodicity
Use the wrapped angle:
\[
\bar\theta=\mathrm{mod}\!\left(\theta,\frac{\pi}{2}\right)\in\left[0,\frac{\pi}{2}\right]
\]

### 3.2 Theta grid for polytopic partition (9 points)
\[
\theta_k=\frac{k-1}{8}\cdot\frac{\pi}{2},\quad k=1,\dots,9
\]

### 3.3 Theta weights \(\zeta_k(\bar\theta)\) (hat/linear interpolation)
If \(\bar\theta\in[\theta_m,\theta_{m+1}]\):
\[
\zeta_m=\frac{\theta_{m+1}-\bar\theta}{\theta_{m+1}-\theta_m},\qquad
\zeta_{m+1}=\frac{\bar\theta-\theta_m}{\theta_{m+1}-\theta_m}
\]
\[
\zeta_k=0\ (k\neq m,m+1)
\]
Properties:
\[
0\le \zeta_k\le 1,\qquad \sum_{k=1}^9\zeta_k=1
\]

---

## 4. Current Vertices (Unipolar) and “Excited-only” Polytope

We assume:
\[
i_a,i_b,i_c \in [0,I_{\max}]
\]
and we construct current vertices from \(\{0,I_{\max}\}\) for each phase:
\[
\sigma=(\sigma_a,\sigma_b,\sigma_c)\in\{0,1\}^3
\]
Vertex currents:
\[
i_a^{(\sigma)}=\sigma_a I_{\max},\quad
i_b^{(\sigma)}=\sigma_b I_{\max},\quad
i_c^{(\sigma)}=\sigma_c I_{\max}
\]

### 4.1 Excluding the (0,0,0) vertex
We **exclude** \(\sigma=000\) to focus on excited conditions.
Define:
\[
\mathcal S=\{001,010,011,100,101,110,111\}
\]
So **7 current vertices** remain.

### 4.2 Current barycentric weights (per phase)
For each phase:
\[
\mu_{j,0}(i_j)=1-\frac{i_j}{I_{\max}},\qquad
\mu_{j,1}(i_j)=\frac{i_j}{I_{\max}}
\]
Then for each \(\sigma\in\{0,1\}^3\):
\[
w_\sigma(i)=\mu_{a,\sigma_a}(i_a)\,\mu_{b,\sigma_b}(i_b)\,\mu_{c,\sigma_c}(i_c)
\]
and \(\sum_\sigma w_\sigma = 1\).

Weight for excluded vertex:
\[
w_{000}=\mu_{a,0}\mu_{b,0}\mu_{c,0}
\]

### 4.3 Normalized weights over excited vertices
For \((i_a,i_b,i_c)\neq(0,0,0)\), define:
\[
\tilde w_\sigma(i)=\frac{w_\sigma(i)}{1-w_{000}(i)},\qquad \sigma\in\mathcal S
\]
Then:
\[
0\le \tilde w_\sigma\le 1,\qquad \sum_{\sigma\in\mathcal S}\tilde w_\sigma = 1
\]

> Practical note: to avoid numerical issues when currents are tiny, define an “excited region”
\[
i_a+i_b+i_c \ge I_{\min} > 0
\]
and only use the excited-polytope model there.

---

## 5. Final Polytopic Weights (Total 9×7 = 63 vertices)

Define combined weights:
\[
\Xi_{k,\sigma}(x)=\zeta_k(\bar\theta)\,\tilde w_\sigma(i)
\qquad (k=1,\dots,9,\ \sigma\in\mathcal S)
\]
Then:
\[
0\le \Xi_{k,\sigma}\le 1,\qquad
\sum_{k=1}^9\sum_{\sigma\in\mathcal S} \Xi_{k,\sigma}=1
\]

---

## 6. Vertex Matrix Construction: \(A_{k,\sigma}\), \(B_k\), \(C\)

State and input remain:
\[
x=\begin{bmatrix} i_a&i_b&i_c&\omega&\theta\end{bmatrix}^\top,\quad
u=\begin{bmatrix}v_a&v_b&v_c&T_d\end{bmatrix}^\top
\]
\[
C=\begin{bmatrix}I_{3\times3} & 0_{3\times2}\end{bmatrix}
\]

### 6.1 Precompute inductance terms at theta grid points
At each theta vertex \(\theta_k\):
\[
L_{j,k}=L_j(\theta_k),\qquad L'_{j,k}=L'_j(\theta_k)
\]

Define:
\[
g_{j,k}=\frac{L'_{j,k}}{L_{j,k}}
\]

### 6.2 Local affine approximation of \(f_j(\theta)=\frac{R}{L_j(\theta)}\)
We approximate near \(\theta_k\):
\[
f_j(\theta)\approx a_{j,k}+b_{j,k}\theta
\]
where:
\[
b_{j,k}=f'_j(\theta_k)=\frac{d}{d\theta}\Big(\frac{R}{L_j(\theta)}\Big)\Big|_{\theta_k}
= -R\frac{L'_{j,k}}{L_{j,k}^2}
\]
\[
a_{j,k}=f_j(\theta_k)-b_{j,k}\theta_k=\frac{R}{L_{j,k}}-b_{j,k}\theta_k
\]

This yields:
\[
-\frac{R}{L_j(\theta)}i_j \approx -a_{j,k} i_j - b_{j,k} i_j\,\theta
\]
so we get a constant \(\theta\)-column at each current vertex by replacing \(i_j\) with \(i_j^{(\sigma)}\).

### 6.3 Vertex \(A_{k,\sigma}\in\mathbb{R}^{5\times 5}\)
Use vertex current values:
\[
i_a^{(\sigma)}=\sigma_a I_{\max},\quad
i_b^{(\sigma)}=\sigma_b I_{\max},\quad
i_c^{(\sigma)}=\sigma_c I_{\max}
\]

Nonzero entries of \(A_{k,\sigma}\) (state order \([i_a,i_b,i_c,\omega,\theta]\)):

**(i) “Resistance” part (diagonal via affine approx)**
\[
A_{k,\sigma}(1,1)=-a_{a,k},\quad
A_{k,\sigma}(2,2)=-a_{b,k},\quad
A_{k,\sigma}(3,3)=-a_{c,k}
\]

**(ii) Omega coupling in current equations (from \(-\frac{L'}{L}i\omega\))**
\[
A_{k,\sigma}(1,4)=-g_{a,k}\,i_a^{(\sigma)},\quad
A_{k,\sigma}(2,4)=-g_{b,k}\,i_b^{(\sigma)},\quad
A_{k,\sigma}(3,4)=-g_{c,k}\,i_c^{(\sigma)}
\]

**(iii) Theta-column coupling (from affine approx of \(-\frac{R}{L}i\))**
\[
A_{k,\sigma}(1,5)=-b_{a,k}\,i_a^{(\sigma)},\quad
A_{k,\sigma}(2,5)=-b_{b,k}\,i_b^{(\sigma)},\quad
A_{k,\sigma}(3,5)=-b_{c,k}\,i_c^{(\sigma)}
\]

**(iv) Mechanical torque term (using original torque model without 1/L)**
We use the identity \(L' i^2 = (L'i)\,i\) and replace one \(i\) by vertex value.
\[
A_{k,\sigma}(4,1)=\frac{1}{J_m}L'_{a,k}\,i_a^{(\sigma)},\quad
A_{k,\sigma}(4,2)=\frac{1}{J_m}L'_{b,k}\,i_b^{(\sigma)},\quad
A_{k,\sigma}(4,3)=\frac{1}{J_m}L'_{c,k}\,i_c^{(\sigma)}
\]
Friction:
\[
A_{k,\sigma}(4,4)=-\frac{B_m}{J_m}
\]

**(v) Kinematics**
\[
A_{k,\sigma}(5,4)=1
\]

All other entries are zero.

### 6.4 Vertex \(B_k\in\mathbb{R}^{5\times4}\)
We keep input matrices depending only on \(\theta_k\):
\[
B_k(1,1)=\frac{1}{L_{a,k}},\quad
B_k(2,2)=\frac{1}{L_{b,k}},\quad
B_k(3,3)=\frac{1}{L_{c,k}},\quad
B_k(4,4)=-\frac{1}{J_m}
\]
All other entries are zero.

### 6.5 Output matrix
\[
C=\begin{bmatrix}I_{3\times3} & 0_{3\times2}\end{bmatrix}
\]

---

## 7. Final Polytopic LPV Model (63 vertices)

Dynamics:
\[
\dot x
=
\sum_{k=1}^9\sum_{\sigma\in\mathcal S}\Xi_{k,\sigma}(x)\,A_{k,\sigma}\,x
\;+\;
\sum_{k=1}^9 \zeta_k(\bar\theta)\,B_k\,u
\]
Output:
\[
y=Cx
\]

---

## 8. How to Use This in Python Comparison

You will likely implement two simulators:

### 8.1 Baseline simulator (original model)
- Use the equations in Section 1 with your exact \(L_j(\theta),L'_j(\theta)\).

### 8.2 Polytopic simulator
At each time step:
1) Wrap angle: \(\bar\theta=\mathrm{mod}(\theta,\pi/2)\)
2) Compute \(\zeta_k(\bar\theta)\)
3) Compute current weights \(\mu_{j,0},\mu_{j,1}\)
4) Compute unnormalized \(w_\sigma\), normalize to \(\tilde w_\sigma\) over \(\sigma\in\mathcal S\)
5) Compute \(\Xi_{k,\sigma}=\zeta_k \tilde w_\sigma\)
6) Evaluate:
   - \(A_\text{poly}=\sum_{k,\sigma}\Xi_{k,\sigma} A_{k,\sigma}\)
   - \(B_\text{poly}=\sum_k \zeta_k B_k\)
7) Integrate:
\[
\dot x = A_\text{poly}x + B_\text{poly}u
\]

### Suggested metrics
- Trajectory error: \(\|x_\text{baseline}-x_\text{poly}\|\)
- Current RMSE per phase
- Speed and angle error over excited intervals
- Sensitivity vs. \(I_{\min}\) threshold (if used)

---

## 9. Implementation Notes / Caveats

1) **Excluded vertex normalization:** The normalization \(\tilde w_\sigma\) is only safe when \(1-w_{000}\) is not tiny. Use an excited threshold \(I_{\min}\) or fallback model near zero current.

2) **Vertex count:** 63 constant vertices = feasible for LMI but still sizable. If your switching strategy implies only 1–2 phases conduct at a time, you can optionally exclude unrealistic current vertices (e.g., 111).

3) **Affine approximation accuracy:** The affine approximation of \(\frac{R}{L(\theta)}\) around each \(\theta_k\) is a modeling approximation. Its mismatch is part of what you will quantify in simulation.

4) **Theta periodicity:** Ensure consistent use of \(\bar\theta\) in the polytopic weights and inductance evaluation.

---

## 10. Checklist of What You Need to Provide in Code

- Functions:
  - `L_a(theta)`, `L_b(theta)`, `L_c(theta)`
  - `dL_a(theta)`, `dL_b(theta)`, `dL_c(theta)`
- Constants: `R, Jm, Bm, Imax`
- Input trajectory: `v_a(t), v_b(t), v_c(t), Td(t)`
- Initial conditions: `i_a(0), i_b(0), i_c(0), w(0), theta(0)`

---

End of document.
