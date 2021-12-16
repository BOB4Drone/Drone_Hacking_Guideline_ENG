# 하드웨어 <!-- omit in toc -->

## 목차 <!-- omit in toc -->

- [1. GPS Module](#1-gps-모듈)
  - [1.1. Overview](#11-개요)
  - [1.2. Vulnerability Analysis Methodology](#12-취약점-분석-방법론)
    - [1.2.1. GPS Spoofing (Jamming)](#121-gps-spoofing-jamming)
    - [1.2.2. GPS Spoofing (Location Change)](#122-gps-spoofing-location-change)
- [2. PX4 Optical Flow](#2-px4-optical-flow)
  - [2.1. Overview](#21-개요)
  - [2.2. Vulnerability Analysis Methodology](#22-취약점-분석-방법론)
    - [2.2.1. IMU Sensor & Ultrasonic Sensor](#221-imu-sensor--ultrasonic-sensor)
    - [2.2.2. Image Sensor](#222-image-sensor)
- [3. PX4 Telemetry Radio](#3-px4-telemetry-radio)
  - [3.1. Overview](#31-개요)
  - [3.2. Vulnerability Analysis Methodology](#32-취약점-분석-방법론)
- [4. Wifi Module](#4-wifi-모듈)
  - [4.1. Overview](#41-개요)
  - [4.2. Vulnerability Analysis Methodology](#42-취약점-분석-방법론)


## outline

In this project, vulnerability analysis was conducted on four sensors as follows.

![image](https://user-images.githubusercontent.com/91944211/145882093-94ad9ea0-4c00-44b1-b647-e2631babce0f.png)

In this project, six possible attacks were selected for four sensors. The attack was screened through the analysis of the existing paper/conference presentation/technical report, and four of the total eight attacks were successful.

![image](https://user-images.githubusercontent.com/91944211/145883340-f0a455fb-bf4e-4716-ac81-4e713d2bc0eb.png)


## 1. GPS Module

### 1.1. Overview

GPS module is a module that sends and receives GPS information from PX4-Autopilot.

A total of two way of GPS spoofing attempts were made through the GPS module, and although GPS spoofing failed, Jamming succeeded.


### 1.2. Vulnerability Analysis Methodology

#### 1.2.1. GPS Spoofing (Jamming)

We applied the GPS Spoofing technique, the most representative attack, to carry out Jamming attacks that continued to send GPS information.

I succeeded in Jamming through GPS Spoofing without much difficulty.

#### 1.2.2. GPS Spoofing (Location Change)

Note Link :  https://gpspatron.com/spoofing-a-multi-band-rtk-gnss-receiver-with-hackrf-one-and-gnss-jammer/

The attack was attempted in the same way as the above link, but failed due to interference from other satellite signals (GNSS).


## 2. PX4 Optical Flow

### 2.1. Overview

Optical Flow can be easily said to be a camera. Optical Flow consists of an Initial Measurement Unit (IMU) sensor, an Ultasonic sensor, and an Image Sensor.

For IMU Sensors and Ultrasonic Sensors, resonant frequency attacks were carried out but failed, and for Image Sensors, sensors using strong light were successfully neutralized.

### 2.2. Vulnerability Analysis Methodology

The following papers were referenced for vulnerability analysis : https://ieeexplore.ieee.org/abstract/document/6630805/

#### 2.2.1. IMU Sensor & Ultrasonic Sensor

For IMU Sensors and Ultrasonic Sensors, resonant frequency attacks were carried out.

First, in the case of IMU Sensor, we tried an internal spring high frequency attack. But it failed.

Note : https://www.usenix.org/sites/default/files/conference/protected-files/enigma17_slides_kim.pdf

Next, for the Ultrasonic Sensor, a resonance frequency attack was carried out using Arduino.

But it was not successful because Frequency was not right. (40KHz → 42KHz)

#### 2.2.2. Image Sensor

For Image Sensor, a sensor neutralization attack was carried out using strong light.

He was able to succeed in the attack without much difficulty.

Note : https://www.usenix.org/sites/default/files/conference/protected-files/woot_slides_park.pdf


## 3. PX4 Telemetry Radio

### 3.1. Overview

Telemetry Radio is a sensor that transmits/receives RF signals at PX4-Autopilot. This sensor allows you to send MAVLink information or Remote Controller information.

Therefore, in this project, RF Replay Attack was performed by dividing it into MAVLink and Remote Controller information respectively, and in conclusion, only RF Replay Attack through Remote Controller information was successful.


### 3.2. Vulnerability Analysis Methodology

RF Replay Attack was performed for Telemetry Radio.

The RF Replay Attack was performed separately because information could be sent by MAVLink or Remote Controller.

In conclusion, only the Remote Controller was successful. This is because MAVLink uses the FHSS technique and RC uses the DSSS technique.

## 4. Wifi Module

### 4.1. Overview

Wi-Fi module is literally a sensor that sends/receives information through Wi-Fi. In this project, the ESP8266 Wi-Fi module was used. Deauth Attack was conducted for ESP8266 Wi-Fi modules.

### 4.2. Vulnerability Analysis Methodology

Through MAVLink Bridge Wifi, we proceeded Deauth Attack, and we were able to succeed without much difficulty.

---

## Category <!-- omit in toc -->

### Introduction <!-- omit in toc -->
   1. [Research background and goal](/1-intro/about-drone-research.md)
   2. [Previous studies](/1-intro/related-work.md)

### Methodology <!-- omit in toc -->
   1. [UAV software](/2-body/1_software-uav.md)
      1. [Flight controller software](/2-body/1_software-uav.md/#1-fcsflight-controller-software)
      2. [MAVROS](/2-body/1_software-uav.md/#2-nuttx-rtos)
   2. [Ground Control System software](/2-body/2_software-gcs.md/)
   3. [Hardware](/2-body/3_hardware.md)
       1. [GPS Module](/2-body/3_hardware.md/#1-gps-모듈)
       2. [PX4 Optical Flow](/2-body/3_hardware.md/#2-px4-optical-flow)
       3. [PX4 Telemetry Radio](/2-body/3_hardware.md/#3-px4-telemetry-radio)
       4. [Wifi Module](/2-body/3_hardware.md/#4-wifi-모듈)

### Result <!-- omit in toc -->
   1. [Project results](/3-conclusion/result.md)
