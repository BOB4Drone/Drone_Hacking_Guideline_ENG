# Project results <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->

- [1. Issue Commits](#1-issue-commits)
- [2. Vulnerability report](#2-vulnerability-report)
   - [2.1 PX4 firmware Vulnerabilities ](#21-px4-firmware-vulnerabilities)
      - [2.1.1 msgID #332 BOF](#211-msgid-332-bof)
      - [2.1.2 msgID #147 oob-read](#212-msgid-147-oob-read)
      - [2.1.3 dos attack through mavftp](#213-dos-attack-through-mavftp)
   - [2.2 QGround Control Vulnerabilities](#22-qground-control-vulnerabilities)
      - [2.2.1 msgID #126 oob-read](#221-msgid-126-oob-read)
      - [2.2.2 msgID #266, #267 oob-read](#222-msgid-266-267-oob-read)
      - [2.2.3 null exception in airmap login function](#223-null-pointer-dereference-in-airmap-login-function)
      - [2.2.4 oob-read in mavlink console](#224-oob-read-in-mavlink-console)
   - [2.3 MAVROS Vulnerabilities](#23-#23-mavros-vulnerabilities)
       - [2.3.1 msgID #192 oob-read](#231-msgid-192-bof)
       - [2.3.2 msgID #332 oob-read](#232-msgid-332-bof)
- [3. Discovered Vulnerability Code Patch](#3-discovered-vulnerability-code-patch)
- [4. Reproducing discovery vulnerabilities through docker](#4-reproducing-discovery-vulnerabilities-through-docker)
- [5. Submission of a paper](#5-submission-of-a-paper)

---

### 1. Issue Commits

| No. | Vulnerability  | link  |
|---|---|---|
| 1 | [PX4] Bug Found in msgID #332 mavlink protocol BOF  | https://github.com/PX4/PX4-Autopilot/issues/18369  |
| 2 | [PX4] Bug Found in msgID #147 mavlink protocol OOB | https://github.com/PX4/PX4-Autopilot/issues/18385  |
| 3 | [PX4] A Bug in MAV_CMD_LOGGING_START (msgID #76 - command 2510) |  https://github.com/PX4/PX4-Autopilot/issues/18580 |
| 4 | [QGC] Bug Found in msgID #126 mavlink protocol OOB read  | https://github.com/mavlink/qgroundcontrol/issues/10020  |
| 5 | [PX4] loop of mavftp protocol crash drone |  https://github.com/PX4/PX4-Autopilot/issues/18651 |
| 6 | [QGC] oob read occurs in the handler function handling msgid 0x10a, 0x10b  |  https://github.com/mavlink/qgroundcontrol/issues/10037 |
| 7 | [QGC] Found a bug in the QGC Airmap function |  https://github.com/mavlink/qgroundcontrol/issues/10035 |
| 8 | [MAVROS] BOF occurs in the handler function handling msgid 192 |  https://github.com/mavlink/mavros/issues/1668 |
| 9 | [MAVROS] BOF occurs in the handler function handling msgid 332 |  https://github.com/mavlink/mavros/issues/1666 |
| 10 | [QGC] OOB READ in mavlink console |  https://github.com/mavlink/qgroundcontrol/issues/10059 |
| 11 | [QGC] use after free occurs in GeoTag Images menu |  https://github.com/mavlink/qgroundcontrol/issues/10068 |
---

## 2. Vulnerability report

### 2.1 PX4 firmware Vulnerabilities 



#### 2.1.1 msgID #332 BOF 

* msgID #332 is a TRAJECTORY_REPRESENTATION_WAYPOINTS packet, which states in the protocol document that it receives up to five valid points.
![image](https://user-images.githubusercontent.com/91944211/145849438-b036099c-e19b-4b51-ae16-c5b2bb9c9aeb.png)

* Thus, even in the actual code, the waypoint array is declared to be a five-size array, as in the protocol document.
   ```
   struct vehicle_trajectory_waypoint_s {
   #endif
        uint64_t timestamp;
        uint8_t type;
        uint8_t _padding0[7]; // required for logger
        struct trajectory_waypoint_s waypoints[5];
   ```

* Also, since the waypoint field type is uint8_t, it is possible to enter up to 0xff, so if it is over 5, a syntax for exception treatment is required.

* However, it does not process any exceptions when decoding packets are being decoded.
   ```
   trajectory_representation_waypoints->valid_points=mavlink_msg_trajectory_representation_waypoints_get_valid_points(msg);
   ```

* In addition, when using parsed data values, there is no other exception processing syntax and it is used as the number of repetitions of the for statement, so memory can be polluted with true even outside the waypoints array.
   ```
   for (int i = 0; i < number_valid_points; ++i) {
		trajectory_waypoint.waypoints[i].point_valid = true;
   }
   ```
---

#### 2.1.2 msgID #147 oob-read

* A vulnerability occurs when adding voltage to the currently connected batteries in packets of msgID #147 BATTERY_STATUS.

* The code adds the voltage of the battery as follows.

	```
	while (battery_mavlink.voltages[cell_count] < UINT16_MAX && cell_count < 10) {
		battery_status.voltage_cell_v[cell_count] = (float)(battery_mavlink.voltages[cell_count]) / 1000.0f;
		voltage_sum += battery_status.voltage_cell_v[cell_count];
		cell_count++;
	}
	```

* Since the optimization option is activated when compiling in PX4, the conditional statement in the loop changes as follows.
	```
	while (battery_mavlink.voltages[cell_count] < UINT16_MAX && cell_count < 10)
	```
	
	```
	while (1 && cell_count < 10)
	```
	
	```
	while (1 < 10)
	```
	
	```
	while (1)
	```

* Therefore, when compiling, the conditions in the loop are always changed to be true.

* In fact, after checking the compiled code through ida, we were able to confirm that the internal code of the loop had been erased.
	![image](https://user-images.githubusercontent.com/91944211/145856918-5ec925f8-3e43-4a5f-a4b5-69845f82589c.png)

* When the loop is always true, the cell_count increases infinitely, resulting in oob-read.


---

#### 2.1.3 dos attack through mavftp

* We can create a directory through mavftp.
	![image](https://user-images.githubusercontent.com/91944211/145863374-99c0ec79-7fa8-436d-aaa2-9aedef56b05c.png)

* When creating a directory repeatedly, it goes beyond the ram limit (px4 uses vfs, so it is loaded in RAM when creating a directory) and the drone ends.

* Therefore, it can terminate the drone like dos attack by using this. 

---

### 2.2 QGround Control Vulnerabilities 

#### 2.2.1 msgID #126 oob-read

* A vulnerability exists in serial_control packet MSGID #126.

* In the protocol document, the data field is up to 70 bytes, but the count field representing the length of the data is uint8_t, allowing you to enter up to 0xff. Therefore, exceptions are needed when the actual length of the count field and data are different.

	![image](https://user-images.githubusercontent.com/91944211/145865023-e2e0909b-8a76-4a89-ba01-15efa1dc68bf.png)

* However, we could confirm that the count was used without exception.
	```
	case MAVLINK_MSG_ID_SERIAL_CONTROL:
    {
        mavlink_serial_control_t ser;
        mavlink_msg_serial_control_decode(&message, &ser);
        emit mavlinkSerialControl(ser.device, ser.flags, ser.timeout, ser.baudrate, QByteArray(reinterpret_cast<const char*>(ser.data), ser.count));
    }
	```

* Thus, the crafted ser.data enters the QByteArray function.
	```
	QByteArray(reinterpret_cast<const char*>(ser.data), ser.count)
	```

* The QByteArray function copies data by count size, so sending packets with count value greater than the data length results in out-of-bound read.
	```
	QByteArray::QByteArray(const char *data, qsizetype size)
	{
  		if (!data) {
  			d = DataPointer();
  		} else {
   	 	 if (size < 0)
    	 		size = qstrlen(data);
    		 if (!size) {
   	 		d = DataPointer::fromRawData(&_empty, 0);
   	 	 } else {
    	  		d = DataPointer(Data::allocate(size), size);
       			Q_CHECK_PTR(d.data());
          			memcpy(d.data(), data, size);
        			d.data()[size] = '\0';
       		 }
    	}
	}
	```
	
---

#### 2.2.2 msgID #266, #267 oob-read

* A vulnerability exists in the LOGGING_DATA, LOGGING_DATA_ACKED packets, both of which have the following structure.
	![image](https://user-images.githubusercontent.com/91944211/145867572-d61c2a20-e52d-4957-a2c8-6130186910ac.png)

* While the maximum data length is 249, such as the Serial_Control packet, the maximum value of the length field representing the data length is 256. Therefore, an exception should exist when the length of the length of the data is different from the length of the length field.

* However, exceptions do not exist and call the QByteArray function immediately.

	```
	void Vehicle::_handleMavlinkLoggingData(mavlink_message_t& message)
    {
        mavlink_logging_data_t log;
        mavlink_msg_logging_data_decode(&message, &log);
        emit mavlinkLogData(this, log.target_system, log.target_component, log.sequence, log.first_message_offset, QByteArray((const char*)log.data, log.length), false);
	```

* The QByteArray function copies data by count size, so sending packets with a length value greater than the data length results in out-of-bound read.
	```
	QByteArray::QByteArray(const char *data, qsizetype size)
	{
  		if (!data) {
  			d = DataPointer();
  		} else {
   	 	 if (size < 0)
    	 		size = qstrlen(data);
    		 if (!size) {
   	 		d = DataPointer::fromRawData(&_empty, 0);
   	 	 } else {
    	  		d = DataPointer(Data::allocate(size), size);
       			Q_CHECK_PTR(d.data());
          			memcpy(d.data(), data, size);
        			d.data()[size] = '\0';
       		 }
    	}
	}
	```

---
	
#### 2.2.3 null pointer dereference in airmap login function

* It occurs in Select Tool > Application Settings > AirMap > Show Flight List menu and when click refresh.

	![image](https://user-images.githubusercontent.com/91944211/145871047-7d4e5fd8-6d8e-4c91-b0b8-06f3f4f970de.png)

* When calling functions with factors such as username, password, etc. pressing refresh without entering username and password, a null extension occurs during this process.

	```
    Authenticator::AuthenticateWithPassword::Params params;
    params.oauth.username = _settings.userName.toStdString();
    params.oauth.password = _settings.password.toStdString();
    params.oauth.client_id = _settings.clientID.toStdString();
    params.oauth.device_id = "QGroundControl";
    qCDebug(AirMapManagerLog) << "User authentication" << _settings.userName;
    _client->authenticator().authenticate_with_password(params,
            [this](const Authenticator::AuthenticateWithPassword::Result& result) {
	```

* Therefore, the program is terminated with a null extension and a dos attack is possible.

---

#### 2.2.4 oob-read in mavlink console

* When using the Mavlink shell in QGroundControl, it generates a Serial_control packet to communicate with the shell.

* The generation of serial_control packets results in out-of-bound read in the following code as they are larger than the packet data chunk.

	```
	   while(data.size()) {
        QByteArray chunk{data.left(MAVLINK_MSG_SERIAL_CONTROL_FIELD_DATA_LEN)};
        uint8_t flags = SERIAL_CONTROL_FLAG_EXCLUSIVE |  SERIAL_CONTROL_FLAG_RESPOND | SERIAL_CONTROL_FLAG_MULTI;
        if (close) flags = 0;
        auto protocol = qgcApp()->toolbox()->mavlinkProtocol();
        auto link = _vehicle->vehicleLinkManager()->primaryLink();
        mavlink_message_t msg;
        mavlink_msg_serial_control_pack_chan(
                    protocol->getSystemId(),
                    protocol->getComponentId(),
                    sharedLink->mavlinkChannel(),
                    &msg,
	```

---

### 2.3 MAVROS Vulnerabilities 

#### 2.3.1 msgID #192 bof

* A vulnerability exists in packets that send calibration results called MAG_CAL_REPORT.

* If you look at the protocol document corresponding to the msgID, the pass_id can be set up to 0xff with uint8_t.

	![image](https://user-images.githubusercontent.com/91944211/145876102-0d4b1784-3dd7-4eb0-8eee-63a0f836524d.png)

* Compass_id does not have a separate exception, and a code exists to access the index in the calibration_show array and cover the index with false..

	```
	if (calibration_show[mr.compass_id]) {
      auto mcr = mavros_msgs::msg::MagnetometerReporter();

      mcr.header.stamp = node->now();
      mcr.header.frame_id = std::to_string(mr.compass_id);
      mcr.report = mr.cal_status;
      mcr.confidence = mr.orientation_confidence;
      mcr_pub->publish(mcr);
      calibration_show[mr.compass_id] = false;
    }
	```

* The calibration_show array is  an array with a size of 8, but the index Compass_id can be entered up to 0xff, so both out-of-bound read and write are possible.

	```
	private:
  		rclcpp::Publisher<std_msgs::msg::UInt8>::SharedPtr mcs_pub;
 		rclcpp::Publisher<mavros_msgs::msg::MagnetometerReporter>::SharedPtr mcr_pub;

  		std::array<bool, 8> calibration_show;
  		std::array<uint8_t, 8> _rg_compass_cal_progress;
	```

---

#### 2.3.2 msgID #332 bof

* Same as the [msgID #332 BOF](#211-msgid-332-bof) vulnerability found in PX4 and further rediscovered in ROS as patch has not yet progressed.

---

### 3. Discovered Vulnerability Code Patch

| No. | 내용  | 링크  |
|---|---|---|
| 1 | [2.2.2 msgID #266, #267 oob-read](#222-msgid-266-267-oob-read)  | https://github.com/mavlink/qgroundcontrol/pull/10038  |
| 2 | [2.3.1 msgID #192 oob-read](#231-msgid-192-bof)  | https://github.com/mavlink/mavros/pull/1675  |
| 3 | [2.3.2 msgID #332 oob-read](#232-msgid-332-bof)  | https://github.com/mavlink/mavros/pull/1667  |

---

### 4. Reproducing discovery vulnerabilities through docker

For those who try to analyze the vulnerabilities of drones in the future, we plan to distribute fuzzers that were developed by organizing a docker environment together.
It is distributed to below link and can be used through the following process.
[Docker Hub Repository](https://hub.docker.com/repository/docker/ashinedo/4-drone)

**How to use**

image download
```
$ docker pull ashinedo/4-drone:PX4-1.0
```

image execution
The PX4 version, where the vulnerability was detected, is installed and the fuzzer is automatically executed.

```
$ docker run -it --rm ashinedo/4-drone:PX4-1.0
```

you don't want the fuzzer to run automatically, you can enter the command directly through the bash option.
```
$ docker run -it --rm ashinedo/4-drone:PX4-1.0 bash
```

Docker images take up a lot of capacity, so remove them after use

```
docker image listing
$ docker images

Remove local images using tags.
$ docker rmi ashinedo/4-drone:PX4-1.0
```

Docker Image under distribution

**[PX4-1.0](https://hub.docker.com/layers/178199113/ashinedo/4-drone/PX4-1.0/images/sha256-c1623af96905cf568545083de81363e87fcd66dd38b717dae300b5e84184c711?context=repo)** - MAVLink fuzzer(sysid header)

---

### 5. Submission of a paper

While working on this project, one thesis was submitted. The information on the paper is as follows.

```
Subject of the thesis : Vulnerability Analysis of PX4 Autopilot's MAVLink Module
Author : Kyung Kyu-chang, Kim Dong-hyun, Choi Soo-bin, Lee Joon-oh, Kim Ji-soo, Ryu Hyung-ho, Cho Jin-sung
Conference: 2021 Winter Conference of the Korean Information Protection Society (CISC-W'21)
Link: [To Be Announced]
```

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
