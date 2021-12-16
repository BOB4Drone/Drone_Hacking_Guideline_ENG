# Background and Objectives of Research <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->

- [1. Research Background](#1-연구-배경)
  - [1.1. Drone Components](#11-드론-구성요소)
    - [1.1.1. Hardware](#111-하드웨어)
    - [1.1.2. Software](#112-소프트웨어)
  - [1.2. Possible damage](#12-발생-가능한-피해)
- [2. Scope of Research](#2-연구-범위)
  - [2.1. Hardware](#21-하드웨어)
  - [2.2. Software](#22-소프트웨어)
- [3. Research Objectives](#3-연구-목표)
  - [3.1. Non-Compromised](#31-non-compromised)
  - [3.2. Compromised](#32-compromised)
  - [2.3. Expected performance](#23-기대성과)

- - -

## 1. Research Background

A Study on the Methodology of Finding Vulnerabilities to Reduce the Damage of Drone Security Accidents

### 1.1. Drone Components

#### 1.1.1. Hardware

**Basic Configuration**
```
- Drone Board: Fixhawk is widely used for large/special purpose drones, so Fixhawk 4 boards are used for research.
- Electronic speed control: Electronic speed control is used to control motor speed.
- Motor/Propeller: Motor has a fixed rotation direction and is divided according to the presence or absence of a brush.
- Battery: The output of the motor is determined by the number of cells.
- Charger: The motor does not work without a battery.
```

**Additional Configuration for Flight**
```
- GPS & Compass: Provide location information.
- Transmitter, Receiver: Generally, 2.4GHz band are used.
- Telemetry: The location and state of the drone are provided to a PC on the ground, using a 915 MHz or 433 MHz wireless channel.
- Frame: Fixing the boards and sensors.
```

#### 1.1.2. Software
```
- Operating System(OS)
  - Managing the drone's system hardware and applications.
- Ground Control Station(GCS)
  - Controlling drones on the ground.
  - e.g.) QGroundControl, Misson Planner, etc.
- Micro Air Vehicke Link(MAVLink)
  -  Protocol for Communicating with small unmanned vehicle.
```

### 1.2. Possible damage
  - Deodorization
  - Eavesdropping and Filming
  - Construction: Casualty
  - Agriculture: Crop or Economic Damage
  - Military: Crime, Terrorism, etc.


## 2. Scope of Research

It targets both open and closed sources used in drones.

Hardware has selected devices that are compatible with the software.

### 2.1. Hardware
- Sensors for Instrumentation
- RC Communications

### 2.2. Software
- Flight controller software(PX4, Ardupilot)
- OS(Nuttx)
- GCS(QGroundControl, Mission Planner)


## 3. Research Objectives

We aimed to explore new vulnerabilities and verify existing threat scenarios to improve the security of drone platforms.

Existing threat scenarios have verified vulnerabilities by attempting attacks on hardware such as major sensors and modules.
  - The method was classified as Non-Compromised because it could attack from outside without specific conditions.

The new vulnerability search verifies the vulnerability by attempting an attack with the drone/GCS in control.
  - When attacking drones, it is assumed that the GCS is in control.
  - When attacking the GCS, it is assumed that the drone is in control.

### 3.1. Non-Compromised

*When an attack is possible from outside without any set conditions.*

![img_01](../img/non-compromised.png)

### 3.2. Compromised

*When the drone's network is accessible.*

![img_02](../img/compromised.png)




### 2.3. Expected performance
- Making a hacking scenario.
- Report on vulnerabilities, issue CVE.
- Extracting Guideline.
- contribution to a paper.

---

## Table of Contents <!-- omit in toc -->

### 소개 <!-- omit in toc -->
   1. [연구배경 및 목표](/1-intro/about-drone-research.md)
   2. [선행 연구](/1-intro/related-work.md)

### 접근 방법론 <!-- omit in toc -->
   1. [무인항공기(UAV) 소프트웨어](/2-body/1_software-uav.md)
      1. [Flight controller software](/2-body/1_software-uav.md/#1-fcsflight-controller-software)
      2. [Nuttx RTOS](/2-body/1_software-uav.md/#2-nuttx-rtos)
   2. [지상 관제(GCS) 소프트웨어](/2-body/2_software-gcs.md/)
   3. [하드웨어](/2-body/3_hardware.md)
       1. [GPS 모듈](/2-body/3_hardware.md/#1-gps-모듈)
       2. [PX4 Optical Flow](/2-body/3_hardware.md/#2-px4-optical-flow)
       3. [PX4 Telemetry Radio](/2-body/3_hardware.md/#3-px4-telemetry-radio)
       4. [Wifi 모듈](/2-body/3_hardware.md/#4-wifi-모듈)

### 결과 <!-- omit in toc -->
   1. [프로젝트 성과](/3-conclusion/result.md)
   2. [프로젝트 후기](/3-conclusion/conclusion.md)
