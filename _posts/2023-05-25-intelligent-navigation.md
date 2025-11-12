---
layout: post
title: "TurtleBot3 Pathfinder"
date: 2023-05-25
subtitle: "Intelligent Navigation using a TurtleBot3, all built basically from scratch"
---

<div style="text-align: justify; text-justify: inter-word;">

<p>
In this project, my colleague <a href="https://fr.linkedin.com/in/gio-so-7785a8341">Gio So</a> and I designed, implemented, and evaluated an autonomous navigation system on a TurtleBot 3 (<em>Fig. 1</em>) using ROS 2. Starting from a detailed study of ROS 2, mobile-robot concepts, and differential-drive navigation, we developed three successive prototypes—ranging from basic obstacle detection to full SLAM-and-Nav2-based navigation. Each prototype refined the communication between the Raspberry Pi 4 (RPi) and the OpenCR microcontroller, introduced an Artificial Potential Field (APF) path-planning algorithm, and culminated in leveraging Nav2 for industrial-grade autonomous navigation.
</p>

<figure style="max-width:800px; margin:0 auto; text-align:center;">
  <img src="../assets/img/turtlebot3_burger_components.png" width="400" alt="TurtleBot3 Burger components">
  <figcaption><em>Figure 1.</em> TurtleBot 3 Burger components.</figcaption>
</figure>

<h2>Objectives</h2>

<ul>
  <li>Master ROS 2 concepts (nodes, topics, QoS, actions) and both C++ and Python for robotics software.</li>
  <li>Implement a dynamic trajectory-planning algorithm (APF) on an AMR with modular, real-time execution.</li>
  <li>Develop three incremental prototypes, each adding complexity (from simple LiDAR-based stopping to SLAM + Nav2).</li>
  <li>Deliver a fully programmed AMR with clear instructions and ROS 2 packages (C++ & Python) for autonomous control.</li>
</ul>

<h2>Communication Method</h2>

<p>
At first, we established a simple serial protocol: LiDAR distances were sent as ASCII frames from the Pi to the OpenCR, which parsed each line to drive the robot’s wheels. As the project evolved, we expanded the protocol to include full LiDAR scans and encoder odometry, then replaced it entirely by running micro-ROS on the OpenCR. This allowed seamless publication and subscription of native ROS 2 messages (<code>/scan</code>, <code>/odom</code>, <code>/cmd_vel</code>), eliminating custom parsing and improving latency.
</p>

<h2>Navigation Method</h2>

<p>
The core of our custom planner was an Artificial Potential Field (APF): goal attraction and obstacle repulsion vectors were computed in real time to generate velocity commands. Although this approach handled dynamic avoidance without a map, we observed the classic APF drawbacks of local minima and oscillatory motions (<em>Fig. 2</em>).
</p>

<figure style="max-width:800px; margin:0 auto; text-align:center;">
  <img src="../assets/img/apf_fail.png" width="600" alt="APF Failure visualization">
  <figcaption><em>Figure 2.</em> APF failure (goal behind the box).</figcaption>
</figure>

<p>
To overcome these issues, we later adopted the Nav2 stack—using SLAM-generated maps, layered costmaps, and global/local planners—to produce smooth, collision-free trajectories in complex environments.
</p>

<h2>Practical Implementation</h2>

<p>
Prototype 1 filtered a 60° LiDAR sector on the Pi, sent distances over serial to the OpenCR, and halted the robot before obstacles.
</p>

<figure style="max-width:800px; margin:0 auto; text-align:center;">
  <img src="../assets/img/proto1.png" width="200" alt="First prototype visualization">
  <figcaption><em>Figure 3.</em> First prototype visualization.</figcaption>
</figure>

<p>
Prototype 2 moved APF calculations onto the Pi under ROS 2, reading complete scans and odometry and publishing velocity commands back to the OpenCR. The workflow is described in the following figure.
</p>

<figure style="max-width:800px; margin:0 auto; text-align:center;">
  <img src="../assets/img/turtlebot3_nav.png" width="300" alt="Second prototype workflow">
  <figcaption><em>Figure 4.</em> Second prototype workflow.</figcaption>
</figure>

<p>
In Prototype 3, we ran Cartographer SLAM on the TurtleBot, launched Nav2 with tuned costmap parameters, and flashed micro-ROS firmware to the OpenCR so that <code>/cmd_vel</code> and sensor topics flowed natively between both controllers.
</p>

<h2>Results</h2>

<p>
I won’t elaborate much on this prototype because we used several libraries for the implementation of each navigation component—so it wasn’t truly from scratch like the earlier prototypes. However, it taught us a valuable lesson: implementing components from scratch (as in Prototypes 1 and 2) is invaluable for learning, but for industrial use cases, where time efficiency and robustness are critical, leveraging established frameworks remains the best approach.
</p>

<p>
In early tests, the robot reliably stopped for frontal obstacles but exhibited jerky starts and stops due to serial delays. The APF version reached goals in obstacle-free runs but oscillated when avoiding obstacles. Finally, with Nav2, the TurtleBot navigated smoothly around both static and moving obstacles, demonstrating industrial-grade performance and validating the effectiveness of SLAM-and-Nav2 over a pure APF approach.
</p>

<h2>What's Next</h2>

<p>
To advance this work, we planned to explore AI-enhanced planners—such as reinforcement-learning or deep-network-based potential fields—to overcome APF’s local minima issues. It is also possible to integrate additional sensors (camera, IMU) for 3D mapping and semantic perception, and to investigate multi-robot coordination using ROS 2’s DDS capabilities. Ultimately, this was an invaluable project that helped my colleague and me grow as both engineers and individuals.
</p>

</div>
