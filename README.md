# PipeCrawler: Pipe & Sewer 3D Mapping System

[![ROS 2](https://img.shields.io/badge/ROS_2-Jazzy-blue.svg)](https://docs.ros.org/en/jazzy/)
[![Platform](https://img.shields.io/badge/Platform-Raspberry_Pi_5_%7C_Arduino-green.svg)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**PipeCrawler** is a robotic platform designed to traverse, inspect, and generate 3D spatial maps of municipal pipeline networks and underground sewer infrastructure. By combining multi-sensor odometry with RGB-D depth sensing, the system reconstructs internal pipe geometry to aid in structural inspection, blockage detection, and preventative maintenance.

---

## 🎯 Project Roadmap & Phases

- [x] **Phase 1: Benchtop Sensor Fusion & Odometry** — Integrating IMU and wheel encoders for drift-compensated pose estimation.
- [x] **Phase 2: Depth Sensing & Point Cloud Generation** — Real-time 3D spatial reconstruction in dry, controlled pipe environments.
- [ ] **Phase 3: City Water & Storm Drain Deployment** — Ruggedizing chassis traction for wet, low-friction municipal pipes.
- [ ] **Phase 4: Sewer Infrastructure Adaptation** — Environmental sealing (IP67+), corrosion resistance, and tether/wireless telemetry optimization for subterranean sewer networks.

---

## 🛠️ Hardware Architecture

The platform separates high-level spatial processing from real-time motor control:

* **Compute:** Raspberry Pi 5 (running Ubuntu 24.04 & ROS 2 Jazzy)
* **Microcontroller:** Arduino (real-time PWM motor control and quadrature encoder polling)
* **3D Vision / Depth:** Orbbec Astra Mini Pro RGB-D Camera
* **Inertial Measurement:** Bosch BNO055 9-DOF Absolute Orientation IMU
* **Odometry:** Hall-effect / optical rotary wheel encoders
* **Chassis & Drivetrain:** Modified frame with 5000 series Builda motors
---

## 💻 Software Stack

* **Middleware:** ROS 2 Jazzy Jalisco
* **Visualization & Debugging:** RViz2 and Foxglove Studio
* **Languages:** C++, Python, Arduino C


