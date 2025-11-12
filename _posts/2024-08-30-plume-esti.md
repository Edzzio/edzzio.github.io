---
layout: post
title: "3D Smoke Plume Estimation"
date: 2024-08-30
subtitle: "Smoke plume modelling and estimation"
mathjax: true
---

<div style="text-align: justify; text-justify: inter-word;">

<p>
I presented a complete pipeline for modeling and estimating the behavior of a three-dimensional smoke plume released into the atmosphere. The plume is represented as a sequence of Gaussian “puffs,” each carrying a fixed mass and spreading according to atmospheric dispersion coefficients. An Extended Kalman Filter (EKF) then fuses sparse concentration measurements to recover each puff’s center position and spread parameters in real time. The entire workflow is implemented in a MATLAB Live Script, enabling interactive visualization of both simulation and estimation results!
</p>

<h2>Objectives</h2>

<ul>
  <li>Develop a 3D Gaussian-puff model that captures transient plume dispersion under varying wind and stability conditions.</li>
  <li>Formulate a nonlinear state-space representation of puff dynamics suitable for recursive estimation.</li>
  <li>Design and tune an EKF to estimate each puff’s standard deviations (horizontal and vertical) and centroid from noisy (simulated) measurements.</li>
  <li>Implement the entire pipeline in a MATLAB Live Script, with interactive plots for dynamic analysis.</li>
  <li>Validate estimator performance against noisy synthetic data and quantify estimation errors.</li>
</ul>

<h2>Modeling Method: Gaussian Puffs</h2>

<p>
The problem with modeling the smoke plume as a single steady-state entity (Gaussian Plume, <em>Fig. 1</em>) is that it cannot capture rapid changes in ambient temperature, humidity, wind direction, etc.
</p>

<figure style="max-width:800px; margin:0 auto; text-align:center;">
  <img src="../assets/img/smoke_plume_vs_puffs.png" width="300" alt="Gaussian Plume vs Puffs">
  <figcaption><em>Figure 1.</em> Gaussian Plume vs. Puff modeling of a smoke plume.</figcaption>
</figure>

<p>
A more dynamic solution is to discretize the plume into individual puffs emitted at fixed intervals. Each puff’s concentration field is:
</p>

<p style="text-align: center;">
$$
C_i(x, y, z, t) =
\frac{Q_{p,i}}{(2\pi)^{3/2}\sigma_h^2\sigma_v}
\exp\!\left[
-\frac{1}{2}
\left(
\frac{(x - x_{c,i}(t))^2 + (y - y_{c,i}(t))^2}{\sigma_h^2}
\right)
\right]
\,
\exp\!\left[
-\frac{1}{2}
\left(
\frac{(z - z_{c,i}(t))^2 + (z + z_{c,i}(t))^2}{\sigma_v^2}
\right)
\right]
$$
</p>

<p>
where:
</p>

<ul style="list-style-type: none; margin-left: 2em;">
  <li><b>\( (x, y, z) \)</b> – evaluation point</li>
  <li><b>\( (x_{c,i}, y_{c,i}, z_{c,i}) \)</b> – puff center</li>
  <li><b>\( \sigma_h, \sigma_v \)</b> – horizontal and vertical spreads (time-varying)</li>
  <li><b>\( Q_{p,i} \)</b> – mass of the puff</li>
</ul>

<p>
Summing over all \( N \) puffs gives the full plume:
</p>

<p style="text-align: center;">
$$
C(x, y, z, t) = \sum_{i=1}^{N} C_i(x, y, z, t)\,.
$$
</p>

<h2>Estimation Method: Extended Kalman Filter</h2>

<p>
Each puff’s state vector is defined as:
</p>

<p>
$$
\mathbf{x}_k = [\,x_c,\,y_c,\,z_c,\,\sigma_h,\,\sigma_v\,]^T
$$
</p>

<p>
and evolves according to:
</p>

<p>
$$
\mathbf{x}_{k+1} = f(\mathbf{x}_k) + \mathbf{w}_k,\quad
\mathbf{y}_k = g(\mathbf{x}_k) + \mathbf{v}_k\,.
$$
</p>

<p>
The EKF alternates between the following steps:
</p>

<p><strong>Predict:</strong></p>

<p>
$$
\hat{\mathbf{x}}_{k+1\mid k} = f(\hat{\mathbf{x}}_{k\mid k}), 
\quad
P_{k+1\mid k} = F_k\,P_{k\mid k}\,F_k^T + Q
$$
</p>

<p><strong>Update:</strong></p>

<p>
$$
K_{k+1} = P_{k+1\mid k}\,G_{k+1}^T\,(G_{k+1}\,P_{k+1\mid k}\,G_{k+1}^T + R)^{-1},
\quad
\hat{\mathbf{x}}_{k+1\mid k+1} = \hat{\mathbf{x}}_{k+1\mid k} + K_{k+1}\bigl(\mathbf{y}_{k+1} - g(\hat{\mathbf{x}}_{k+1\mid k})\bigr)
$$
</p>

<p>
where $$F_k$$ and $$G_{k+1}$$ are the Jacobians of $$f$$ and $$g$$, and $$Q,R$$ represent the process and measurement noise covariances, respectively.
</p>

<h2>Implementation</h2>

<p>
All steps—from puff generation to EKF recursion—are implemented in a single MATLAB Live Script. First, wind speed, stability class, and dispersion coefficients are initialized. A time loop creates new puffs, builds 3D concentration grids, and plots them. The EKF prediction and update use symbolic Jacobians converted into MATLAB functions, and interactive figures visualize true vs. estimated puff centers, spreads, and measured vs. reconstructed concentrations in real time.
</p>

<h2>Results</h2>

<p>
The 3D plume evolution is visualized with successive slice plots (<em>Fig. 2</em>). The EKF-estimated horizontal spread $$\sigma_h$$ tracks the true value closely (<em>Fig. 3</em>), achieving RMSE < 5 %. Estimated concentration curves align with noisy sensor data, reducing bias while keeping each EKF update under 1 ms. The Live Script efficiently processes hundreds of puffs and updates in just a few seconds.
</p>

<figure style="max-width:800px; margin:0 auto 1.5em; text-align:center;">
  <img src="../assets/img/puffs_visualization.png" width="500" alt="Gaussian Puffs evolution">
  <figcaption><em>Figure 2.</em> Puff evolution over time.</figcaption>
</figure>

<figure style="max-width:800px; margin:0 auto 1.5em; text-align:center;">
  <img src="../assets/img/estimation_std.png" width="500" alt="Spread estimation">
  <figcaption><em>Figure 3.</em> Horizontal spread estimation for one puff.</figcaption>
</figure>

<h2>What’s Next?</h2>

<p>
This work will be analyzed and further upgraded by a good friend and fellow PhD student, <a href="https://fr.linkedin.com/in/maya-pivert/en">Maya Pivert</a>. Future work will integrate multiple spatially distributed sensors to improve observability and minimize estimation uncertainty. The Gaussian-puff model will be extended to incorporate buoyancy effects, atmospheric stratification, and basic chemical reactions for reactive plumes to enhance realism. Controlled field experiments are planned to calibrate dispersion coefficients and validate the EKF under real atmospheric conditions. Finally, critical routines will be ported to C/C++ for embedded deployment, enabling on-site, real-time plume monitoring.
</p>

</div>
