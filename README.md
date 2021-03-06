# Drone-Hacking-Guideline


**KITRI BoB 10th Vulnerability Analysis Track: Project 'Vulnerability Study on Drone'**

> Study period: September 2021 to December 2021 (4 months)

This project was conducted by the 10th KITRI BoB (Best of the Best), and recorded the vulnerability study of drones for 4 months.
We shared the results of our team's analysis and disclosed the records to lower entry barriers to those who are newly entering drone hacking.

Vulnerability analysis experiments were conducted on drone software with high market share, and significant vulnerabilities were identified through this.
In addition, the patching work on the issue was completed by reporting so that the relevant content could be taken action quickly.

※ 4DFUZZER [(PublicRepository)](https://github.com/BOB4Drone/4D-Fuzzer)

※ 한국어 문서 [here](https://github.com/BOB4Drone/Drone_Hacking_Guideline)

---

### Author

> KITRI BoB 10th | vulnerability analysis track | 4-Drone

- Mentor: Park Chun Sung, Kang Inwook.
- PL(Project Leader): Won Yohan.
- Mentee: PM Lee Junoh, [Kyeong Kyuchang](https://github.com/l3u9), [Kim Donghyun](https://github.com/jjanguu), [Ryu Hyeongho](https://github.com/Ryuuuuu), [Choi Subin](https://github.com/dubini0), Kim Jisoo.

## Table of Contents <!-- omit in toc -->

### Introduction <!-- omit in toc -->
   1. [Research background and goal](/1-intro/about-drone-research.md)
   2. [Previous studies](/1-intro/related-work.md)

### Methodology <!-- omit in toc -->
   1. [UAV software](/2-body/1_software-uav.md)
      1. [Flight controller software](/2-body/1_software-uav.md/#1-fcsflight-controller-software)
      2. [MAVROS](/2-body/1_software-uav.md/#2-nuttx-rtos)
   2. [Ground Control System software](/2-body/2_software-gcs.md/)
      1. [Introduction](/2-body/2_software-gcs.md/#1-introduction)
      2. [Analyze Environment](/2-body/2_software-gcs.md#2-analyze-environment)
      3. [Vulneralility Analysis Methodology](/2-body/2_software-gcs.md#3-vulnerability-analysis-methodology)
   4. [Hardware](/2-body/3_hardware.md)
       1. [GPS Module](/2-body/3_hardware.md/#1-gps-module)
       2. [PX4 Optical Flow](/2-body/3_hardware.md/#2-px4-optical-flow)
       3. [PX4 Telemetry Radio](/2-body/3_hardware.md/#3-px4-telemetry-radio)
       4. [Wifi Module](/2-body/3_hardware.md/#4-wifi-module)

### Result <!-- omit in toc -->
   1. [Project results](/3-conclusion/result.md)

