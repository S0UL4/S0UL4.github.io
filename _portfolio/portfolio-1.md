---
title: "IncLIO - Incremental LiDAR-Inertial Odometry"
excerpt: "A real-time LiDAR-Inertial Odometry system built on an Iterated Error-State Kalman Filter (IESKF) with Normal Distribution Transform (NDT) scan-to-map registration. Designed for high-rate, low-latency pose estimation on robotic platforms.<br/><img src='/images/demo.gif' width='500' >"
collection: portfolio
---

## Overview
**IncLIO** is a real-time LiDAR–Inertial Odometry system designed for high-frequency and low-latency state estimation in robotic platforms.

It combines **IMU propagation** with **LiDAR scan-to-map registration** using a tightly-coupled filtering approach.

<p align="center">
  <img src="/images/demo.gif" width="500">
</p>

## Method
The system is built on:

- **Iterated Error-State Kalman Filter (IESKF)** for state estimation  
- **NDT (Normal Distribution Transform)** for scan-to-map alignment  
- **Incremental map update** for real-time performance  

This follows the family of modern LIO systems where LiDAR and IMU are fused to achieve accurate pose estimation in real-time robotic applications.

## Key Features
- Real-time pose estimation  
- Tight LiDAR–IMU fusion  
- Low-latency pipeline  
- Designed for embedded robotic systems  

## Use Cases
- Autonomous navigation  
- Field robotics  
- GPS-denied environments  

## 🔗 Project Access
 👉 <a href="https://github.com/S0UL4/IncLIO" class="btn btn--primary">View on GitHub</a>