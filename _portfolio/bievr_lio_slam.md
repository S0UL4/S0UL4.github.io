---
title: "BIEVR-LIO-SLAM - LiDAR-Inertial SLAM & Localization"
excerpt: "An extension of the BIEVR-LIO odometry framework into a full SLAM and localization stack: Scan Context loop closure, GTSAM pose-graph optimization, map saving and global relocalization — all running downstream of the odometry, which stays untouched.<br/><img src='https://img.youtube.com/vi/d-zujtjVo9A/hqdefault.jpg' width='500' >"
collection: portfolio
---

## Overview
**BIEVR-LIO-SLAM** turns the **BIEVR-LIO** LiDAR–Inertial Odometry framework into a complete **SLAM and localization system**.

The base odometry uses a high-resolution, voxel-wise **oriented height image map** to exploit subtle geometric variations in challenging, degenerate environments. On top of it, this project adds **loop closure**, **pose-graph optimization**, **persistent mapping** and **global relocalization**.

<p align="center">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/d-zujtjVo9A" title="SLAM and Localization | BIEVR-LIO-SLAM" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</p>

## Architecture
The guiding design principle is that **all SLAM modules live downstream of the odometry**, connected through a single observer hook.

- The observer receives only `(timestamp, body pose, undistorted point cloud)` per frame
- Corrections are **published alongside** the odometry and **never fed back into it**
- The odometry therefore behaves *identically* whether the SLAM modules run or not

Because the coupling is that thin, the same stack ports to any LOAM-like LIO system (FAST-LIO, LIO-SAM, …) that can expose those three outputs.

## Method
- **Place recognition** — Scan Context descriptors (rotation-invariant polar ring/sector image)
- **Back-end** — GTSAM pose-graph optimization with incremental **iSAM2**
- **Keyframing** — gating by translation and rotation thresholds
- **Robustness** — Cauchy noise model on loop constraints to reject false positives
- **Localization** — ICP tracking against a prior map in the map frame, with automatic Scan Context relocalization or manual seeding from RViz *2D Pose Estimate*
- **Scalability** — tile-based map caching, so maps larger than available RAM stay usable

## Key Features
- Loop closure with pose-graph optimization to correct trajectory drift
- Dual-mode operation: **mapping** (build & save) and **localization** (track against a prior map)
- Saved *map bundles* (`cloud.pcd`, `scan_context.bin`, `poses_tum.txt`, `meta.yaml`)
- Works with bare `.pcd` maps produced by other SLAM systems

## Implementation
- **Language** — C++ (Eigen, Ceres, PCL, GTSAM 4.2, yaml-cpp)
- **Middleware** — ROS 2 Jazzy (full SLAM support); ROS 1 Noetic (odometry only)
- **Sensors** — standard industrial LiDARs plus Livox gen1/gen2
- **Datasets** — ready-made sensor configs for ENWIDE, Newer College, GEODE, MARS-LVIG and GrandTour

## Use Cases
- Large-scale mapping and map reuse
- Long-term localization in GPS-denied environments
- Field and industrial robotics, autonomous navigation

## 🔗 Project Access
 👉 <a href="https://github.com/S0UL4/BIEVR-LIO-SLAM" class="btn btn--primary">View on GitHub</a>
 👉 <a href="https://www.youtube.com/watch?v=d-zujtjVo9A" class="btn btn--primary">Watch the Demo</a>
