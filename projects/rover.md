---
layout: single
title: "Autonomous Mapping Rover (AMR)"
permalink: /projects/rover/
author_profile: true
header:
  teaser: /assets/images/AMR.png
---

![Autonomous Mapping Rover](/assets/images/AMR.png)

<div class="project-meta-strip">
  <div class="meta-item"><span class="meta-label">Domain:</span> <span class="meta-val">Mobile Robotics & Autonomy</span></div>
  <div class="meta-item"><span class="meta-label">Drivetrain:</span> <span class="meta-val">4WD Omnidirectional Mecanum</span></div>
  <div class="meta-item"><span class="meta-label">Stack:</span> <span class="meta-val">ROS 2 Humble, Nav2, slam_toolbox, micro-ROS</span></div>
  <div class="meta-item"><span class="meta-label">Sensors:</span> <span class="meta-val">RPLiDAR A1, 6-DOF IMU, Optical Encoders</span></div>
</div>

## Overview

The **Autonomous Mapping Rover (AMR)** is a ground robotics platform engineered for GPS-denied indoor localization, 2D SLAM, and point-to-point obstacle-aware navigation. Utilizing a **4-wheel independent Mecanum drive**, the robot achieves holonomic motion (simultaneous translation in $X/Y$ and yaw rotation $\theta$), allowing it to navigate tight corridors and zero-radius turns with high agility.

<div class="tech-pill-container">
  <span class="tech-pill">ROS 2 Humble</span>
  <span class="tech-pill">Nav2 Stack</span>
  <span class="tech-pill">slam_toolbox</span>
  <span class="tech-pill">micro-ROS</span>
  <span class="tech-pill">EKF Fusion</span>
  <span class="tech-pill">Mecanum Kinematics</span>
</div>

---

## Demonstration & Telemetry

<div class="video-showcase-container">
  <div class="video-showcase-header">
    <span>Autonomous SLAM & Dynamic Navigation</span>
    <span style="font-weight: 400; color: var(--text-muted); font-size: 0.8rem;">Field Run &bull; 180 m² Test Environment</span>
  </div>

  <div class="video-responsive-wrapper">
    <div class="video-placeholder-card">
      <div class="play-icon-box">
        <i class="fas fa-play"></i>
      </div>
      <h4>Autonomous Navigation & Obstacle Avoidance</h4>
      <p>
        Real-time telemetry showing live 2D occupancy grid generation, DWB local costmap evaluation, and holonomic crab-walk maneuvers around sudden obstacles.
      </p>
    </div>
  </div>

  <div class="milestones-list">
    <div class="milestone-item">
      <span class="time-badge">0:06</span>
      <strong>Sensor Initialization:</strong> RPLiDAR spins up to 5.5 Hz; stationary IMU gyroscope bias calibrated.
    </div>
    <div class="milestone-item">
      <span class="time-badge">0:28</span>
      <strong>Loop Closure:</strong> <code>slam_toolbox</code> registers scan matches across multiple rooms without drift.
    </div>
    <div class="milestone-item">
      <span class="time-badge">1:05</span>
      <strong>Dynamic Obstacle Avoidance:</strong> Local costmap detects obstacle; DWB controller executes lateral crab maneuver.
    </div>
    <div class="milestone-item">
      <span class="time-badge">1:48</span>
      <strong>Goal Docking:</strong> Waypoint reached within 2 cm position accuracy and under 1.5° yaw offset.
    </div>
  </div>
</div>

---

## System Architecture

The autonomy stack separates real-time motor commutation and high-level mapping to avoid CPU starvation during intensive SLAM updates.

```mermaid
flowchart TD
    subgraph PERCEPTION["Sensors & Odometry"]
        LIDAR["RPLiDAR 2D Scanner\n360° Planar @ 5.5 Hz"]
        IMU["6-DOF IMU (MPU-6050)\n3-Axis Gyro + Accel @ 100 Hz"]
        ENC["4x Optical Encoders\nQuadrature Timer Ticks"]
    end

    subgraph EMBEDDED["Embedded Layer (micro-ROS / MCU)"]
        FWD["Forward Kinematics\nTicks to Robot dx, dy, dtheta"]
        PID["4x Discrete Velocity PID\n50 Hz Closed-Loop Rate"]
    end

    subgraph AUTONOMY["Autonomy Compute (Linux SBC / ROS 2)"]
        EKF["robot_localization EKF\nWheel Odom + IMU Fusion"]
        SLAM["slam_toolbox\n2D Occupancy Grid @ 5 cm"]
        NAV["Nav2 Costmaps & DWB\nHolonomic Path Generation"]
    end

    subgraph HARDWARE["Drivetrain & Power"]
        DRV["Dual MOSFET H-Bridges\nOptocoupled Inputs"]
        MOT["4x 12V Metal Gearmotors\nMecanum Drive"]
        PWR["3S 11.1V LiPo Battery\n5V/4A & 3.3V Regulators"]
    end

    LIDAR --> SLAM
    IMU --> FWD
    ENC --> FWD
    FWD <==>|micro-ROS Serial| EKF
    EKF --> SLAM --> NAV
    NAV -->|/cmd_vel Twist| PID
    PID --> DRV --> MOT
    PWR -.-> AUTONOMY
    PWR -.-> DRV
```

---

## Hardware Architecture & Electrical Interface

The chassis utilizes a dual-deck design providing galvanic and physical isolation between high-current inductive motor switching and sensitive compute electronics.

### Interconnect & Pinout

| MCU Pin | Function | Peripheral | Signal Type | Voltage | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| <span class="pin-code">PA8 / PA9</span> | TIM1_CH1 / CH2 | Front-Left Motor Driver | PWM Out | 3.3V Logic | 20 kHz PWM, optocoupler isolated |
| <span class="pin-code">PA10 / PA11</span> | TIM1_CH3 / CH4 | Front-Right Motor Driver | PWM Out | 3.3V Logic | 20 kHz PWM, optocoupler isolated |
| <span class="pin-code">PB0 / PB1</span> | TIM3_CH1 / CH2 | Front-Left Encoder | Digital In | 3.3V TTL | Hardware quadrature decoder mode |
| <span class="pin-code">PB6 / PB7</span> | I2C1_SCL / SDA | MPU-6050 6-DOF IMU | I2C Bus | 3.3V | 400 kHz Fast Mode, 2.2 kΩ pull-ups |
| <span class="pin-code">PA2 / PA3</span> | USART2_TX / RX | Linux SBC Bridge | UART | 3.3V TTL | micro-ROS transport @ 921600 baud |

---

## Kinematics & Mathematical Formulation

For a 4-wheel Mecanum drive with wheel radius $r$, chassis half-width $l_x$, and chassis half-length $l_y$, the inverse kinematics mapping chassis velocities $(v_x, v_y, \omega_z)$ to individual wheel velocities $\omega_i$ is evaluated onboard:

$$\begin{bmatrix} \omega_1 \\ \omega_2 \\ \omega_3 \\ \omega_4 \end{bmatrix} = \frac{1}{r} \begin{bmatrix} 1 & -1 & -(l_x + l_y) \\ 1 & 1 & (l_x + l_y) \\ 1 & 1 & -(l_x + l_y) \\ 1 & -1 & (l_x + l_y) \end{bmatrix} \begin{bmatrix} v_x \\ v_y \\ \omega_z \end{bmatrix}$$

Mecanum rollers are subject to slip on smooth surfaces. An **Extended Kalman Filter (EKF)** fuses raw wheel odometry with high-rate IMU angular rate data, reducing cumulative rotational drift by **88%**.

---

## Engineering Challenges & Verification

| Challenge | Root Cause | Engineering Solution | Verification Result |
| :--- | :--- | :--- | :--- |
| **Roller Slip Drift** | 45° Mecanum roller micro-slippage during acceleration | Fused 6-DOF IMU gyro Z-axis rates with wheel odometry using EKF | Rotational drift reduced by 88% |
| **Vibration Noise** | Contact roller vibrations transmitting to LiDAR scan plane | 3D-printed TPU dampening isolators under LiDAR riser deck | Eliminated false-positive obstacle rays |
| **Power Brownouts** | Simultaneous 4-motor stall current dips on 5V SBC rail | Isolated buck regulation with separate LC filtering and Schottky diode | Zero SBC reboots over 25 stress runs |
| **Confined Maneuvers** | Turn-in-place maneuvers bumping into tight spaces | Configured DWB local controller with holonomic lateral samples ($v_y \neq 0$) | 30% faster transit through narrow passages |

---

## Bill of Materials (BOM)

| Subsystem | Component | Part # / Spec | Qty | Function |
| :--- | :--- | :--- | :--- | :--- |
| **Compute** | Linux SBC | Raspberry Pi 4B (4GB), Ubuntu 22.04 LTS | 1 | Runs ROS 2 Humble, Nav2, and slam_toolbox |
| **Compute** | Microcontroller | STM32 Nucleo-F401RE / ESP32 | 1 | Runs micro-ROS agent and 4x PID motor loops |
| **Perception** | 2D LiDAR | Slamtec RPLiDAR A1 (360° Planar, 12m) | 1 | 2D occupancy grid mapping & laser scans |
| **Perception** | 6-DOF IMU | InvenSense MPU-6050 (I2C 400 kHz) | 1 | Angular rate tracking for EKF fusion |
| **Drivetrain** | DC Gearmotors | Pololu 37D mm, 19:1 ratio, 64 CPR encoders | 4 | Independent 4-wheel actuation |
| **Drivetrain** | Mecanum Wheels | 80 mm diameter with 45° rubber rollers | 4 | Holonomic omnidirectional mobility |
| **Power** | 3S LiPo Battery | Turnigy 11.1V 2200 mAh 30C | 1 | Main system energy source |
| **Power** | Buck Regulator | LM2596 / XL4015 (5.1V @ 4A output) | 1 | Clean isolated power for SBC |

---

<div class="hero-actions" style="margin-top: 2rem;">
  <a href="https://github.com/eugeniopasos" target="_blank" rel="noopener noreferrer" class="btn--primary">
    <i class="fab fa-github"></i> View GitHub Repository
  </a>
  <a href="/#projects" class="btn--inverse">
    &larr; Back to Projects
  </a>
</div>
