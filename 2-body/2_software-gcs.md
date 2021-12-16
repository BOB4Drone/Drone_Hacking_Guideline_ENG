
# GCS(Ground Control System) <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->

- [1. 개요](#1-개요)
    - [1.1. GCS 개념](#11-GCS-개념)
    - [1.2. GCS 기능](#12-GCS-기능)
    - [1.3. 드론별 GCS 현황](#13-드론별-gcs-현황)
    - [1.4. ELF vs AppImage](#14-elf-vs-appimage)
- [2. 분석 환경 구축](#2-분석-환경-구축)
    - [2.1. QGC 빌드](#21-qgc-빌드)
      - [2.1.1. QT 설치](#211-qt-설치)	
      - [2.1.2. GUI를 이용한 빌드](#212-gui를-이용한-빌드)
      - [2.1.3. CLI를 이용한 빌드](#213-cli를-이용한-빌드)
    - [2.2. Sanitizer 사용](#22-sanitizer-사용)
      - [2.2.2. GUI 이용](#221-gui를-이용한-sanitizer-사용)
      - [2.2.3. CLI 이용](#222-cli를-이용한-sanitizer-사용)
- [3. 취약점 분석 방법론](#3-취약점-분석-방법론)
    - [3.1. 소스코드 오디팅을 통한 취약점 분석](#31-소스코드-오디팅을-통한-취약점-분석)
      - [3.1.1. MAVLINK Handle 함수 오디팅](#311-mavlink-handle-함수-오디팅)
      - [3.1.2. QGC GUI 기능 함수 오디팅](#312-qgc-gui-기능-함수-오디팅)
    - [3.2. Fuzzing을 통한 취약점 분석](#21-qgc-빌드)
      - [3.2.1. Mavlink Fuzzer](#321-mavlink-fuzzer)
      - [3.2.2. QGC With AFL++](#322-qgc-with-afl)

## 1. 개요

### 1.1. GCS Concepts

GCS, short for Ground Control Station, is software designed for various operations such as drone control and mission. ([QGroundControl](https://github.com/mavlink/qgroundcontrol) is recommended in PX4)


### 1.2. GCS Functions

  - Autonomous flight and real-time flight path control.
  - Map data processing, flight route analysis,
  - Geofence (If necessary, create a no-fly zone)

### 1.3. 드론별 GCS 현황
|ardupilot|px4|dji|
|------|---|---|
|mission planner|qground control|ground station|


### 1.4. ELF vs AppImage
Prior to analyzing vulnerabilities in QGC, we identified the Release file [QGroundControl.AppImage](https://docs.qgroundcontrol.com/master/en/releases/release_notes.html) using checksec.sh to check memory protection techniques.  
![Untitled](https://user-images.githubusercontent.com/91944211/145853059-f851a947-f62a-4850-8d73-2fb56cf68cea.png)

All protection techniques except NX can be seen to be turned off, due to the nature of AppImage extensions.
![Untitled (1)](https://user-images.githubusercontent.com/91944211/145853378-2a6ecd6c-15ab-4e3a-b997-85b34383104e.png)

As shown in the following figure, AppImage is stored together in the form of squashfs with the execution binaries and libraries to meet dependencies. So we're going to... `sudo mount ./qgroundcontrol.AppImage mountpoint -o offset=188392`  I checked the binaries that were actually built through the command, and again the memory protection techniques. 
![Untitled (2)](https://user-images.githubusercontent.com/91944211/145854132-68ed37d7-623e-47ce-803f-bf079a7cef12.png)
![Untitled (3)](https://user-images.githubusercontent.com/91944211/145854403-426d88d4-87e9-46fb-b1d0-ef62d9f02d1d.png)

## 2. Analytical Environment

### 2.1. Build QGC

#### 2.1.1. Installing QT

Since QGC uses the QT library as a Cross-Platform library, it downloads QT and compiles it through QT Creator.

1. Download and run the [Qt installer](https://www.qt.io/download-open-source)

2. QT version (5.15.2) and component settings
![image](https://user-images.githubusercontent.com/91944211/145797702-876c007d-83f1-481f-a01d-ef3b9fded745.png)

3. Install additional packages
    ```
    sudo apt-get install speech-dispatcher libudev-dev libsdl2-dev patchelf
    ```
4. Clone the repository, including the submodule
    ```
    git clone --recursive -j8 https://github.com/mavlink/qgroundcontrol.git
    ```
2. Submodule Update
    ```
    git submodule update --recursive
    ```
#### 2.1.2. Build using GUI

1. qgroundcontrol project open

2. Click on the hammer to proceed with the build

    ![image](https://user-images.githubusercontent.com/91944211/145805224-3b2287ba-9d1f-4baa-85c4-71be54e58e66.png)

#### 2.1.3. Build using CLI
1. Create makefile 
    ```
    cd qgroundcontrol
    mkdir build
    cd build
    ~/Qt/5.15.2/gcc_64/bin/qmake ../
    ```
2. Build 
    ```
    make
    ```
### 2.2. Sanitizer

#### 2.2.1. Using the Sanitizer with the GUI

1. Projects > Build > Build Steps > qmake > Additional arguments

2. Modify Argument as below

    ![image](https://user-images.githubusercontent.com/91944211/145806129-c2a81e84-2da1-4b68-9b99-10b8a1663711.png)


#### 2.2.2. Using Sanitizer with CLI

1. Create makefile
    ```
    cd qgroundcontrol
    mkdir build
    cd build
    ~/Qt/5.15.2/gcc_64/bin/qmake ../
    ```
2. Add sanitizer on makefile's CC, CXX, LINK

    ![image](https://user-images.githubusercontent.com/91944211/145806760-400381d7-a375-491d-b071-091c9bef09e5.png)


## 3. Vulnerability Analysis Methodology

### 3.1. Vulnerability Analysis through Source Code Auditing

#### 3.1.1. MAVLINK Handle Function Auditing
From the QGC source directory `src/Vehicle/Vehicle.cc`If you look at the [593line](https://github.com/mavlink/qgroundcontrol/blob/9718d3c71a57034fdaedc1a0aea355c4fd6c2506/src/Vehicle/Vehicle.cc#L593) of , it handles the mavlink packet in a function called Vehicle:::_mavlinkMessageRecovered(). Therefore, we audited according to the flow of the function.


#### 3.1.2. QGC GUI Functional Auditing
In the QGC program, the functions implemented with GUI were analyzed to find vulnerabilities.
- [#Issue10035](https://github.com/mavlink/qgroundcontrol/issues/10035) 
- [#Issue10059](https://github.com/mavlink/qgroundcontrol/issues/10059)
- [#Issue10068](https://github.com/mavlink/qgroundcontrol/issues/10068)

### 3.2. Vulnerability Analysis through Fuzzing

#### 3.2.1. Mavlink Fuzzer
Since QGC also uses the same mavlink protocol as PX4, it was able to find vulnerabilities in the three functions using the [fuzzer](https://github.com/BOB4Drone/4D-Fuzzer) that was previously made.

#### 3.2.2. QGC With AFL++
We used AFL++ for code coverage-based fuzzing. First, since AFL++ basically performs purging using file I/O, we changed `afl-forkserver.c` to use UDP as follows.   
```
int client_socket;
    struct sockaddr_in serverAddress;



    memset(&serverAddress, 0, sizeof(serverAddress));
    

    serverAddress.sin_family = AF_INET;
    inet_aton("127.0.0.1", (struct in_addr*) &serverAddress.sin_addr.s_addr);
    serverAddress.sin_port = htons(14540);
    serverAddresss.sin_family = AF_INET;


    // socket create
    if ((client_socket = socket(PF_INET, SOCK_DGRAM, 0)) == -1)
    {
        printf("socket create fail\n");
        exit(0);
    }


    sendto(client_socket, buf, len, 0, (struct sockaddr*)&serverAddress, sizeof(serverAddress));

    // socket close
    close(client_socket);
```   
Afterwards, he modified Makefile and changed compiler to apl-g++ to conduct purging. However, three problems arise if we proceed this way.
1. AFL sends packets first before QGC is turned on.
2. If you do not receive Heartbeat, you do not receive other packets.
3. AFL can be measured only when the program is terminated.

We used the following methods to solve problems.   

![Untitled Diagram drawio (1) (1)](https://user-images.githubusercontent.com/91944211/145868099-68bcb159-9b80-442d-838c-db0df9d6bfa5.png)   
For the first troubleshooting, the following UDP Server broadcasts were required to load the socket of QGC before sending heartbeat packets and generated packets.   

```
if (heatbeatFlag){exit(0);}
heatbeatFlag = 1
```
Next, to solve the second and third problems, `_uas->receiveMessage(message)` at src/Vehicle/Vehicle.cc; the code was added below to terminate the QGC when receiving the heartbeat and the generated packet.

![Untitled (5)](https://user-images.githubusercontent.com/91944211/145869720-87e71283-3b88-4a4b-92d4-c45491cb6555.png)   
As a result of fuzzing in the above two ways, 120 uniq hands were found as follows, but no crash came out. Because the method mutates the full packet, it cannot pass through the crc verification routine of mavlink.  

![GCS_fuzzing drawio](https://user-images.githubusercontent.com/91944211/145870675-c787b919-a846-40fa-afb0-906f3c042b93.png)   
Therefore, we modified the code of the UDP relay server and proceeded with fuzzing again with the following structure.   
The structure mutates only the 'payload' part of Mavlink in AFL++ and transmits a full packet containing CRC values from the UDP server.

![Untitled (6)](https://user-images.githubusercontent.com/91944211/145870720-2ba6f5fe-1ce6-47aa-a177-1ec67268d062.png)   
As a result, many crashes could be obtained as follows. All of these crashes were vulnerabilities found in the 4dfuzzer above, but it seems that they can be useful as future work by applying them.


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

