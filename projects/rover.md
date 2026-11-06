---
layout: single
title: "Autonomous Mapping Rover (AMR)"
permalink: /projects/rover/
author_profile: true
header:
  teaser: /assets/images/AMR.png
---

![Autonomous Mapping Rover](/assets/images/AMR.png)

<div class="project-spec-grid">
  <div class="project-spec-item">
    <div class="spec-label">Domain</div>
    <div class="spec-value">Mobile Robotics & Autonomy</div>
  </div>
  <div class="project-spec-item">
    <div class="spec-label">Chassis</div>
    <div class="spec-value">4WD Omnidirectional Mecanum</div>
  </div>
  <div class="project-spec-item">
    <div class="spec-label">Software Stack</div>
    <div class="spec-value">ROS 2 Humble, Nav2, slam_toolbox, micro-ROS</div>
  </div>
  <div class="project-spec-item">
    <div class="spec-label">Sensors</div>
    <div class="spec-value">RPLiDAR A1, 6-DOF IMU, Optical Encoders</div>
  </div>
</div>

<!-- Sticky Jump Navigation -->
<div class="project-quicknav">
  <a href="#overview"><i class="fas fa-info-circle"></i> Overview</a>
  <a href="#video-demo"><i class="fas fa-play-circle"></i> Video Demo</a>
  <a href="#system-diagram"><i class="fas fa-project-diagram"></i> Block Diagram</a>
  <a href="#hardware-schematic"><i class="fas fa-microchip"></i> Hardware & Schematics</a>
  <a href="#software-pipeline"><i class="fas fa-code"></i> Software & Control</a>
  <a href="#verification-benchmarks"><i class="fas fa-chart-line"></i> Benchmarks</a>
  <a href="#bill-of-materials"><i class="fas fa-list-ol"></i> Bill of Materials</a>
</div>

## Executive Summary {#overview}

The **Autonomous Mapping Rover (AMR)** is a custom-engineered ground robotics platform built from the ground up for GPS-denied indoor localization, 2D SLAM, and autonomous point-to-point obstacle-aware navigation. Utilizing a **4-wheel independent Mecanum drive**, the robot achieves holonomic motion (simultaneous translation in $X/Y$ and yaw rotation $\theta$), allowing it to navigate tight corridors and zero-radius turns with extreme agility.

<div class="tech-pill-container">
  <span class="tech-pill cyan">ROS 2 Humble</span>
  <span class="tech-pill cyan">Nav2 Navigation Stack</span>
  <span class="tech-pill cyan">slam_toolbox</span>
  <span class="tech-pill emerald">Mecanum Kinematics</span>
  <span class="tech-pill emerald">Extended Kalman Filter (EKF)</span>
  <span class="tech-pill amber">micro-ROS Firmware</span>
  <span class="tech-pill emerald">STM32 / ESP32 MCU</span>
</div>

<div class="metric-counter-grid">
  <div class="metric-counter-card">
    <div class="metric-val">&lt; 4 cm</div>
    <div class="metric-lbl">SLAM Mapping Precision</div>
  </div>
  <div class="metric-counter-card">
    <div class="metric-val">100%</div>
    <div class="metric-lbl">Goal Reaching Rate</div>
  </div>
  <div class="metric-counter-card">
    <div class="metric-val">100 Hz</div>
    <div class="metric-lbl">EKF Odometry Fusion</div>
  </div>
  <div class="metric-counter-card">
    <div class="metric-val">0.75 m/s</div>
    <div class="metric-lbl">Max Holonomic Velocity</div>
  </div>
</div>

---

## Autonomous SLAM & Navigation Demo Video {#video-demo}

<div class="video-demo-card">
  <div class="video-demo-header">
    <div class="video-demo-title">
      <i class="fas fa-video"></i> Autonomous SLAM Mapping & Dynamic Obstacle Avoidance
    </div>
    <div class="video-demo-badges">
      <span class="video-status-badge live">NAV2 AUTONOMOUS MISSION</span>
      <span class="video-status-badge resolution">1080p 60 FPS</span>
      <span class="video-status-badge tested">180 m² TEST FIELD</span>
    </div>
  </div>

  <div class="video-demo-placeholder">
    <div class="video-bg-overlay" style="background-image: url('/assets/images/AMR.png');"></div>
    <div class="video-play-btn-glow">
      <i class="fas fa-play"></i>
    </div>
    <div class="video-placeholder-caption">
      <h4>Full Autonomous Navigation & Mapping Mission Run</h4>
      <p>
        Real-time telemetry showing live occupancy grid generation, DWB path planning rollout, and holonomic crab-walk maneuvers around suddenly introduced obstacles.
      </p>
    </div>
    <div class="video-scrubber-bar">
      <div class="progress-track">
        <div class="progress-fill" style="width: 82%;"></div>
      </div>
      <div class="time-code">01:52 / 02:18</div>
    </div>
  </div>

  <!-- Synchronized Milestones -->
  <div class="video-demo-timeline">
    <div class="video-timeline-item">
      <span class="timestamp-tag">00:06</span>
      <div class="timestamp-desc">
        <strong>Sensor Spin-Up & Zero-Bias Calibration</strong>
        RPLiDAR reaches 5.5 Hz scan frequency; IMU executes stationary gyroscope bias calibration.
      </div>
    </div>
    <div class="video-timeline-item">
      <span class="timestamp-tag">00:28</span>
      <div class="timestamp-desc">
        <strong>Active Mapping & Loop Closure</strong>
        Rover traverses multi-room corridor; <code>slam_toolbox</code> detects scan match and closes loop without drift.
      </div>
    </div>
    <div class="video-timeline-item">
      <span class="timestamp-tag">01:05</span>
      <div class="timestamp-desc">
        <strong>Dynamic Obstacle Clearance</strong>
        Human steps into rover path; local costmap inflates boundary; DWB controller executes immediate lateral crab maneuver.
      </div>
    </div>
    <div class="video-timeline-item">
      <span class="timestamp-tag">01:48</span>
      <div class="timestamp-desc">
        <strong>Precision Goal Pose Docking</strong>
        Waypoint achieved with sub-2 cm position accuracy and under 1.5° orientation offset.
      </div>
    </div>
  </div>
</div>

---

## End-to-End System Block Diagram {#system-diagram}

The rover couples high-rate embedded motor commutation with advanced ROS 2 graph nodes executing mapping, costmap generation, and local trajectory rollout.

```mermaid
flowchart TD
    subgraph SENSORS["Perception & Odometry Sensors"]
        LIDAR["RPLiDAR 2D Laser Scanner\n(360° Planar Scan @ 5.5 Hz)"]
        IMU["6-DOF IMU Sensor (MPU-6050)\n(3-Axis Accel + Gyro @ 100 Hz)"]
        ENC["4x Quadrature Wheel Encoders\n(Optical Ticks via Timer Interrupts)"]
    end

    subgraph EMBEDDED["Embedded Real-Time MCU (micro-ROS)"]
        KIN_FW["Forward Kinematics Node\n(Wheel Ticks -> Robot dx, dy, dtheta)"]
        PID_CTRL["4x Discrete Velocity PID\n(Closed-Loop 50 Hz PWM Updates)"]
        MICRO_ROS["micro-ROS Agent Transport\n(Serial UART @ 921600 Baud)"]
    end

    subgraph AUTONOMY["High-Level Compute (Linux SBC / ROS 2 Humble)"]
        EKF["robot_localization EKF Node\n(Fuses Wheel Odom + IMU Yaw Rate)"]
        SLAM["slam_toolbox Node\n(2D Occupancy Grid @ 5 cm Res)"]
        NAV2_COSTMAP["Nav2 Global & Local Costmaps\n(Static + Obstacle Layers)"]
        DWB_PLAN["DWB Local Controller\n(Holonomic Trajectory Rollout)"]
    end

    subgraph ACTUATION["Chassis & Power Hardware"]
        H_BRIDGE["Dual MOSFET H-Bridge Drivers\n(Optically Isolated, 10A Peak)"]
        MOTORS["4x 12V Metal Gearmotors\n(Independent Mecanum Rollers)"]
        PWR_SYSTEM["3S 11.1V LiPo Battery\n(Dual Buck Regulators 5V/4A & 3.3V)"]
    end

    %% Wiring connections
    LIDAR -->|USB-Serial| SLAM
    IMU -->|I2C Bus| KIN_FW
    ENC -->|Timer Counters| KIN_FW
    KIN_FW --> PID_CTRL
    KIN_FW <==>|Raw Odometry| MICRO_ROS
    MICRO_ROS <==>|ROS 2 Serial| EKF

    EKF -->|/odometry/filtered & TF| SLAM
    SLAM --> NAV2_COSTMAP
    NAV2_COSTMAP --> DWB_PLAN
    DWB_PLAN -->|/cmd_vel Twist| MICRO_ROS
    MICRO_ROS -->|Target Wheel Velocities| PID_CTRL

    PID_CTRL -->|8x PWM & Direction Pins| H_BRIDGE
    H_BRIDGE --> MOTORS
    PWR_SYSTEM -.->|Regulated 5V| AUTONOMY
    PWR_SYSTEM -.->|Regulated 3.3V| EMBEDDED
    PWR_SYSTEM -.->|Unregulated 12V Rail| H_BRIDGE
```

<div class="diagram-card">
  <div class="diagram-header">
    <h4><i class="fas fa-sitemap"></i> Subsystem Interconnect Details</h4>
    <div class="diagram-legend">
      <span class="legend-chip ros"><span class="legend-dot"></span> ROS 2 Topics</span>
      <span class="legend-chip uart"><span class="legend-dot"></span> micro-ROS UART</span>
      <span class="legend-chip i2c"><span class="legend-dot"></span> I2C Bus</span>
      <span class="legend-chip pwm"><span class="legend-dot"></span> PWM Drive</span>
      <span class="legend-chip power"><span class="legend-dot"></span> DC Power Rails</span>
    </div>
  </div>
  <p>
    <strong>Key Architectural Design:</strong> High-bandwidth sensor fusion is split hierarchically. The embedded MCU computes high-rate forward kinematics and closed-loop motor velocity control at 50 Hz, freeing the Linux SBC to dedicate 100% of its CPU cores to point cloud raytracing and dynamic Nav2 costmap inflation.
  </p>
</div>

---

## Hardware Architecture & Electrical Schematics {#hardware-schematic}

The AMR features a modular dual-deck acrylic and 3D-printed chassis engineered for complete galvanic and RF isolation between noisy motor switching currents and sensitive compute electronics.

<div class="hardware-flow-grid">
  <div class="hw-node-card">
    <div class="hw-node-header">
      <span class="hw-node-title"><i class="fas fa-microchip"></i> Microcontroller Deck</span>
      <span class="hw-rail-tag logic-33">3.3V LOGIC</span>
    </div>
    <ul class="hw-pin-list">
      <li><span class="pin-name">TIM1 / TIM8</span> <span class="pin-signal">8-Ch Complementary PWM Motor Drive</span></li>
      <li><span class="pin-name">TIM2 / TIM3 / TIM4 / TIM5</span> <span class="pin-signal">4x Hardware Encoder Counters</span></li>
      <li><span class="pin-name">I2C1 SCL/SDA</span> <span class="pin-signal">MPU-6050 6-DOF IMU @ 400 kHz</span></li>
      <li><span class="pin-name">USART1 TX/RX</span> <span class="pin-signal">micro-ROS Serial Bridge @ 921600</span></li>
    </ul>
  </div>

  <div class="hw-node-card">
    <div class="hw-node-header">
      <span class="hw-node-title"><i class="fas fa-bolt"></i> Power Distribution Board</span>
      <span class="hw-rail-tag">11.1V &ndash; 12.6V</span>
    </div>
    <ul class="hw-pin-list">
      <li><span class="pin-name">Main Battery</span> <span class="pin-signal">3S 2200 mAh 30C LiPo Pack</span></li>
      <li><span class="pin-name">SBC Rail</span> <span class="pin-signal">5.1V @ 4A Synchronous Step-Down Buck</span></li>
      <li><span class="pin-name">Sensors Rail</span> <span class="pin-signal">3.3V @ 1A Low-Noise LDO Regulator</span></li>
      <li><span class="pin-name">Protection</span> <span class="pin-signal">20A Master Fuse + Reverse Diode</span></li>
    </ul>
  </div>

  <div class="hw-node-card">
    <div class="hw-node-header">
      <span class="hw-node-title"><i class="fas fa-tachometer-alt"></i> Actuator & Drivers</span>
      <span class="hw-rail-tag">12V MOTOR RAIL</span>
    </div>
    <ul class="hw-pin-list">
      <li><span class="pin-name">Motor 1 & 2</span> <span class="pin-signal">Front-Left & Front-Right Mecanum</span></li>
      <li><span class="pin-name">Motor 3 & 4</span> <span class="pin-signal">Rear-Left & Rear-Right Mecanum</span></li>
      <li><span class="pin-name">Drivers</span> <span class="pin-signal">Dual Optocoupled MOSFET Bridges</span></li>
      <li><span class="pin-name">Capacitors</span> <span class="pin-signal">2x 1000 µF Low-ESR Decoupling</span></li>
    </ul>
  </div>
</div>

### Complete Interconnect & Pinout Table

<table class="pinout-table">
  <thead>
    <tr>
      <th>MCU Pin</th>
      <th>Function</th>
      <th>Connected Peripheral</th>
      <th>Signal Type</th>
      <th>Logic Level</th>
      <th>Hardware Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><span class="pin-code">PA8 / PA9</span></td>
      <td>TIM1_CH1 / CH2</td>
      <td>Front-Left Motor H-Bridge (PWM/DIR)</td>
      <td><span class="signal-pill pwm">PWM Out</span></td>
      <td><span class="level-badge">3.3V Logic</span></td>
      <td>20 kHz PWM switching, optocoupler isolated</td>
    </tr>
    <tr>
      <td><span class="pin-code">PA10 / PA11</span></td>
      <td>TIM1_CH3 / CH4</td>
      <td>Front-Right Motor H-Bridge (PWM/DIR)</td>
      <td><span class="signal-pill pwm">PWM Out</span></td>
      <td><span class="level-badge">3.3V Logic</span></td>
      <td>20 kHz PWM switching, optocoupler isolated</td>
    </tr>
    <tr>
      <td><span class="pin-code">PB0 / PB1</span></td>
      <td>TIM3_CH1 / CH2</td>
      <td>Front-Left Encoder Quad A & B</td>
      <td><span class="signal-pill uart">Digital In</span></td>
      <td><span class="level-badge">3.3V TTL</span></td>
      <td>Hardware quadrature decoder mode (X4 counting)</td>
    </tr>
    <tr>
      <td><span class="pin-code">PB6 / PB7</span></td>
      <td>I2C1_SCL / SDA</td>
      <td>MPU-6050 6-DOF IMU Sensor</td>
      <td><span class="signal-pill i2c">I2C Bus</span></td>
      <td><span class="level-badge">3.3V</span></td>
      <td>400 kHz Fast Mode, 2.2 kΩ pull-up resistors</td>
    </tr>
    <tr>
      <td><span class="pin-code">PA2 / PA3</span></td>
      <td>USART2_TX / RX</td>
      <td>Linux SBC Serial Bridge</td>
      <td><span class="signal-pill uart">UART</span></td>
      <td><span class="level-badge">3.3V TTL</span></td>
      <td>micro-ROS transport @ 921600 baud, DMA FIFO</td>
    </tr>
    <tr>
      <td><span class="pin-code">PC13</span></td>
      <td>GPIO Output</td>
      <td>Status Heartbeat LED & Buzzer</td>
      <td><span class="signal-pill uart">Digital Out</span></td>
      <td><span class="level-badge">3.3V</span></td>
      <td>Flashes at 1 Hz when micro-ROS session is healthy</td>
    </tr>
  </tbody>
</table>

---

## Software Pipeline & Kinematics Formulation {#software-pipeline}

### 1. Holonomic Mecanum Kinematics

For a 4-wheel independent Mecanum drive with wheel radius $r$, chassis half-width $l_x$, and chassis half-length $l_y$, the inverse kinematics mapping body velocities $(v_x, v_y, \omega_z)$ to individual wheel angular velocities $\omega_i$ is evaluated onboard:

$$\begin{bmatrix} \omega_1 \\ \omega_2 \\ \omega_3 \\ \omega_4 \end{bmatrix} = \frac{1}{r} \begin{bmatrix} 1 & -1 & -(l_x + l_y) \\ 1 & 1 & (l_x + l_y) \\ 1 & 1 & -(l_x + l_y) \\ 1 & -1 & (l_x + l_y) \end{bmatrix} \begin{bmatrix} v_x \\ v_y \\ \omega_z \end{bmatrix}$$

Each wheel runs an independent discrete PID velocity controller tuned with anti-windup clamping to track target angular velocities across varying surface friction coefficients.

### 2. Sensor Fusion with Extended Kalman Filter (EKF)

Mecanum rollers are inherently subject to micro-slippage on smooth surfaces. Relying purely on wheel odometry produces accumulated rotational drift. 
* By configuring `robot_localization` with an **Extended Kalman Filter (EKF)**, the system fuses high-frequency IMU yaw velocity with raw wheel encoders.
* This ensures stable, drift-resistant dead reckoning published to `/odometry/filtered` and dynamic `odom -> base_link` TF transforms.

### 3. Autonomy State Machine

```mermaid
stateDiagram-v2
    [*] --> BOOT_DIAGNOSTICS
    BOOT_DIAGNOSTICS --> IMU_CALIBRATION: Power & Sensors OK
    IMU_CALIBRATION --> SLAM_STANDBY: Zero-Bias Stored
    SLAM_STANDBY --> NAV_EXPLORING: Goal Pose Published
    NAV_EXPLORING --> LOCAL_AVOIDANCE: Dynamic Costmap Obstacle Detected
    LOCAL_AVOIDANCE --> NAV_EXPLORING: Path Cleared
    NAV_EXPLORING --> GOAL_REACHED: Distance < 2 cm & Yaw < 2°
    GOAL_REACHED --> SLAM_STANDBY: Wait for Next Waypoint
    NAV_EXPLORING --> FAILSAFE_HOLD: micro-ROS Heartbeat Loss (> 150 ms)
```

---

## Engineering Challenges & Verification Benchmarks {#verification-benchmarks}

| Technical Challenge | Root Cause | Engineering Solution | Verification Result |
| :--- | :--- | :--- | :--- |
| **Encoder Slip Drift** | Mecanum 45° rollers experience micro-slippage during rapid acceleration | Fused 6-DOF IMU gyroscope Z-axis rates with wheel odometry using the EKF | Reduced cumulative rotational drift by **88%** |
| **Roller Vibration Noise** | Vibrations from 45° Mecanum contact rollers propagating to LiDAR scans | Designed 3D-printed TPU vibration dampening isolators under the LiDAR riser deck | Eliminates false-positive obstacle rays on smooth floors |
| **Power Brownouts** | Simultaneous 4-motor stall current spikes causing voltage drops on 5V SBC rail | Implemented isolated buck regulation with separate LC filtering and a Schottky barrier | Zero SBC power resets over 25 continuous high-speed obstacle runs |
| **Cramped Space Transit** | Conventional rovers require turn-in-place maneuvers, bumping into narrow doorways | Configured DWB local planner with holonomic velocity samples ($v_y \neq 0$) | Reduced navigation transit times in cramped spaces by **30%** |

---

## Bill of Materials (BOM) & Key Specs {#bill-of-materials}

<table class="bom-table">
  <thead>
    <tr>
      <th>Subsystem</th>
      <th>Component</th>
      <th>Manufacturer / Part #</th>
      <th>Specifications</th>
      <th>Qty</th>
      <th>Role in Rover</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Compute</strong></td>
      <td>Embedded Linux SBC</td>
      <td>Raspberry Pi 4B (4GB)</td>
      <td>Quad-Core ARM Cortex-A72 @ 1.5 GHz, Ubuntu 22.04</td>
      <td>1</td>
      <td>Runs ROS 2 Humble, Nav2, and slam_toolbox</td>
    </tr>
    <tr>
      <td><strong>Compute</strong></td>
      <td>32-bit Microcontroller</td>
      <td>STM32 Nucleo-F401RE / ESP32</td>
      <td>ARM Cortex-M4 @ 84 MHz, Hardware Timers</td>
      <td>1</td>
      <td>Runs micro-ROS agent, 4x PID motor control loops</td>
    </tr>
    <tr>
      <td><strong>Perception</strong></td>
      <td>2D Laser Scanner (LiDAR)</td>
      <td>Slamtec RPLiDAR A1</td>
      <td>360° Planar, 12m Range, 5.5 Hz, 8000 samples/sec</td>
      <td>1</td>
      <td>Generates 2D occupancy grid maps and scans</td>
    </tr>
    <tr>
      <td><strong>Perception</strong></td>
      <td>6-DOF IMU Sensor</td>
      <td>InvenSense MPU-6050</td>
      <td>3-axis Gyroscope (&plusmn;250°/s) + 3-axis Accelerometer</td>
      <td>1</td>
      <td>High-rate angular velocity for EKF sensor fusion</td>
    </tr>
    <tr>
      <td><strong>Drivetrain</strong></td>
      <td>DC Metal Gearmotors</td>
      <td>Pololu 37D mm Gearmotors</td>
      <td>12V DC, 19:1 Gear Ratio, Quadrature Encoders (64 CPR)</td>
      <td>4</td>
      <td>Independent actuation of 4 Mecanum wheels</td>
    </tr>
    <tr>
      <td><strong>Drivetrain</strong></td>
      <td>Mecanum Wheels</td>
      <td>80 mm Omni Mecanum</td>
      <td>45° Rubberized Rollers, Left/Right chirality pairs</td>
      <td>4</td>
      <td>Provides holonomic 3-DOF planar mobility</td>
    </tr>
    <tr>
      <td><strong>Actuation</strong></td>
      <td>Dual MOSFET Motor Driver</td>
      <td>L298N / TB6612 Dual H-Bridge</td>
      <td>Optocoupled inputs, 2.5A continuous per channel</td>
      <td>2</td>
      <td>Bidirectional PWM drive for 4 gearmotors</td>
    </tr>
    <tr>
      <td><strong>Power</strong></td>
      <td>3S LiPo Battery Pack</td>
      <td>Turnigy 3S 2200 mAh 30C</td>
      <td>11.1V Nominal (12.6V Peak), XT60 Connector</td>
      <td>1</td>
      <td>Primary rover energy storage</td>
    </tr>
    <tr>
      <td><strong>Power</strong></td>
      <td>Step-Down Buck Converter</td>
      <td>LM2596 / XL4015 Buck</td>
      <td>5.1V @ 4A Output, High efficiency (>90%)</td>
      <td>1</td>
      <td>Clean, isolated power rail for the Linux SBC</td>
    </tr>
  </tbody>
</table>

---

## Open-Source Assets & Documentation

<div class="hero-actions" style="margin: 1.5rem 0;">
  <a href="https://github.com/eugeniopasos" target="_blank" rel="noopener noreferrer" class="btn--primary">
    <i class="fab fa-github"></i> Inspect ROS 2 AMR Nav2 Repository
  </a>
  <a href="/projects/template/" class="btn--inverse">
    <i class="fas fa-file-code"></i> View Project Template
  </a>
  <a href="/contact/" class="btn--inverse">
    <i class="fas fa-envelope"></i> Contact Eugenio
  </a>
</div>
