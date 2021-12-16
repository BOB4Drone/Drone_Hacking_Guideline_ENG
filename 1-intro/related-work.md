# Previous studies <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->

- [1. RVFuzzer: Finding Input Validation Bugs in Robotic Vehicles through Control-Guided Testing](#1-rvfuzzer-finding-input-validation-bugs-in-robotic-vehicles-through-control-guided-testing)
  - [1.1. 요약](#11-요약)
- [2. Security Analysis of the Drone Communication Protocol: Fuzzing the MAVLink protocol](#2-security-analysis-of-the-drone-communication-protocol-fuzzing-the-mavlink-protocol)
  - [2.1. 요약](#21-요약)
- [3. How to Analyze the Cyber Threat from Drones](#3-how-to-analyze-the-cyber-threat-from-drones)
  - [3.1. 요약](#31-요약)

- - -

## 1. RVFuzzer: Finding Input Validation Bugs in Robotic Vehicles through Control-Guided Testing

### 1.1. Summary

As the RV (robot vehicle) models become more diverse, the parameters that need to be tested have become very numerous and increasingly difficult to proceed with fuzzing. Over time, fuzzers satisfying these parameters appeared, but it was difficult to apply them directly to the RV system because it was difficult to determine whether the input via the fuzzer caused a malfunction. Thus, in this paper, the RVF detector, which detects the controllable status of the robot, was implemented so that it could easily be determined for malfunction and easily combined with the existing RV simulation framework, thus eliciting vulnerabilities.

## 2. Security Analysis of the Drone Communication Protocol: Fuzzing the MAVLink protocol

### 2.1. Summary

It is a paper that has built a direct fuzzer on Mavlink. It is a paper that explains what you have experienced while making fuzzers, starting with structural analysis, fuzzing methodology, mutation method, and CRC calculation. Through this fuzzer, the author discovered a vulnerability and left it as future works to improve the fuzzer.

## 3. How to Analyze the Cyber Threat from Drones

### 3.1. Summary
This report investigates the impact of UAS cybersecurity threats from the perspective of DHS (Department of Homeland Security). It focuses primarily on classifying UAS threats in relation to cybersecurity and presents approaches that can help analyze these threats to direct efforts in mitigation and defense strategies.

In this report, a total of four scenarios are presented by dividing the view of UAS as an attack target and the view of UAS as an attack means.

Furthermore, it is concluded by suggesting that senior DHS officials should continue to work harder to establish a safer UAS system.

---

## 카테고리 <!-- omit in toc -->

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

