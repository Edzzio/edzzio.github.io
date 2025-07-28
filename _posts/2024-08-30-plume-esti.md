---
layout: post
title: "3D Smoke Plume Estimation"
date: 2024-08-30
subtitle: "Smoke plume modelling and estimation"
---

<head>
  <style>
    /* Apply full justification to all paragraphs */
    p {
      text-align: justify;
    }
  </style>
</head>

<h2>Summary</h2>
<p>
  I presented a complete pipeline for modeling and estimating the behavior of a three‑dimensional smoke plume released into the atmosphere. The plume is represented as a sequence of Gaussian “puffs”, each carrying a fixed mass and spreading according to atmospheric dispersion coefficients. An Extended Kalman Filter (EKF) then fuses sparse concentration measurements to recover each puff’s center position and spread parameters in real time. The entire workflow is implemented in a MATLAB Live Script, enabling interactive visualization of both simulation and estimation results !
</p>

<h2>Objectives</h2>
<p>

  - Develop a 3D Gaussian‑puff model that captures transient plume dispersion under varying wind and stability conditions.
  - Formulate a nonlinear state‑space representation of puff dynamics suitable for recursive estimation.
  - Design and tune an EKF to estimate each puff’s standard deviations (horizontal and vertical) and centroid from noisy measurements (simulated).
  - Implement the entire pipeline in a MATLAB Live Script, with interactive plots for dynamic analysis.
  - Validate estimator performance against noisy synthetic data and quantify estimation errors.
</p>

<h2>Modeling Method : Gaussian Puffs</h2>
<p>
  The problem with modeling the a smoke plume as a singular entity (Gaussian Plume, Fig. 1) is in its steady-state property. In reality the plume is heavily modified with real-time weather conditions such as ambiant temperature, humidity, wind caracteristics, etc. 
  
  <figure style="max-width:800px; margin:0 auto; text-align:center;">
  <img src="../assets/img/smoke_plume_vs_puffs.png" width="300" alt="Gaussian Plume vs puffs">
  <figcaption><em>Figure 1.</em> Gaussian Plume vs Puffs modeling of a smoke plume.</figcaption>
  </figure>
  <br>

  A solution would be to discretize the plume into individual puffs emitted at fixed intervals. Each puff’s concentration field is modeled by

  <div align="center">

  $$
  C_i(x,y,z,t)
  = \frac{Q_{p,i}}{(2\pi)^{3/2}\,\sigma_h^2\,\sigma_v}
  \exp\!\biggl(-\frac{1}{2}\frac{(x - x_{c,i}(t))^2 + (y - y_{c,i}(t))^2}{\sigma_h^2}\biggr)
  \;\times\;
  \exp\!\biggl(-\frac{1}{2}\frac{(z - z_{c,i}(t))^2 + (z + z_{c,i}(t))^2}{\sigma_v^2}\biggr)
  $$

  </div>

  where $\ x,\ y,\ z$ are the coordinates of calculated position, $\sigma_h$ and $\sigma_v$ are the time‑varying horizontal and vertical standard deviations computed from empirical dispersion formulas, and $Q_i$ is the puff’s mass. We would have to compute $\ i$ puffs in total in order to get a more accurate depiction of the smoke plume
  
  This solution, although imperfect (as you would need an excessive number of puffs to correctly model the plume in real-time), is much better than the typical Gaussian Plume model used, and much simpler and cheaper to compute than the Navier-Stokes fluid dynamic equations. 

</p>

<h2>Estimation Method : Extended Kalman Filter</h2>
<p>

  Each puff’s state vector $\ x = [x_c, y_c, z_c, σₕ, σᵥ]^T$ evolves according to discrete‑time dynamics that update its position by advection and its spread by diffusion. Measurements $\ y_k$ are concentration samples at fixed sensor locations, related by the nonlinear function g(xₖ). The EKF alternates between a prediction and a update phase :

  <div align="center">

  **Predict**  
  $$
  \hat x_{k+1\mid k} = f(\hat x_{k\mid k}), 
  \qquad
  P_{k+1\mid k} = F_{k}\,P_{k\mid k}\,F_{k}^T + Q
  $$

  **Update**  
  $$
  K_{k+1} = P_{k+1\mid k}\,G_{k+1}^T\,(G_{k+1}\,P_{k+1\mid k}\,G_{k+1}^T + R)^{-1},
  \quad
  \hat x_{k+1\mid k+1} = \hat x_{k+1\mid k} + K_{k+1}\bigl(y_{k+1} - g(\hat x_{k+1\mid k})\bigr)
  $$

  </div>

  where $\ Fₖ$ and $\ Gₖ $ are the Jacobians of $\ f$ and $\ g$, and Q/R are the process/measurement noise covariances.

</p>

<h2>Implementation</h2>
<p>
  All components—from puff simulation to EKF recursion—are scripted interactively in a single MATLAB Live Script. First, wind speed, stability class, dispersion coefficients, and puff emission rate are defined. A loop over time instantiates new puffs, computes 3D concentration grids via nested for loops, and visualizes them. The EKF’s predict and update routines are implemented using symbolic Jacobians converted to MATLAB functions. Interactive plots continuously update to show the true versus estimated puff centroids and spreads, as well as concentration fits at sensor locations.
</p>

<h2>Results</h2>
<p>
  The 3D plume evolution is clearly visible through successive slice plots (Fig. 2), illustrating how puffs spread and advect under the prescribed wind field. The EKF‑estimated horizontal spread $\sigma_h$ closely tracks the true value (Fig .3), yielding a root‑mean‑square error below 5 %. Filtered concentration curves align with noisy sensor measurements, reducing estimation bias while maintaining real‑time performance (EKF update executes in under 1 ms per step). The MATLAB Live Script completes hundreds of puff simulations and corresponding EKF updates within seconds.

<figure style="max-width:800px; margin:0 auto; text-align:center;">
  <img src="../assets/img/puffs_visualization.png" width="500" alt="AI results">
  <figcaption><em>Figure 2.</em> Gaussian Puffs evolution over time.</figcaption>
</figure>
</p>

<figure style="max-width:800px; margin:0 auto; text-align:center;">
  <img src="../assets/img/estimation_std.png" width="500" alt="AI results">
  <figcaption><em>Figure 3.</em> Horizontal standard deviation estimation (one puff)'.</figcaption>
</figure>
</p>

<h2>What’s Next?</h2>
<p>
  This work will be analyzed and further upgrades by a good friend and fellow PhD student, <a href="https://fr.linkedin.com/in/maya-pivert/en">Maya Pivert</a>. Future work will integrate multiple spatially distributed sensors to improve observability and minimize estimation uncertainty. The Gaussian‑puff model will be extended to incorporate buoyancy effects, atmospheric stratification, and basic chemical reactions for reactive plumes in order to become even more realistic. Controlled field experiments are planned to calibrate dispersion coefficients and validate the EKF in real atmospheric conditions. Finally, critical routines will be ported to C/C++ for deployment on embedded hardware, enabling on‑site, real‑time plume monitoring.
</p>
