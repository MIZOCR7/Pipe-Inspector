# Autonomous 6WD Rover with ESP32 and Live Video Streaming

A six-wheel-drive robotic rover controlled through a web dashboard, with live camera streaming, adjustable pan and tilt, and ultrasonic obstacle detection. The goal of this project was to build a rover that could operate independently without depending on an existing Wi-Fi router.

The system uses two ESP32 boards: one for movement, sensor readings, and the control interface, and an ESP32-CAM for the live video feed. Once powered on, the rover creates its own Wi-Fi network, allowing a phone or computer to connect and control it directly.

## Table of Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [Hardware Requirements](#hardware-requirements)
- [Pin Configuration](#pin-configuration)
- [How It Works](#how-it-works)
- [Development Process](#development-process)
- [Web Control Interface](#web-control-interface)
- [Power Distribution](#power-distribution)
- [Setup and Usage](#setup-and-usage)
- [Challenges and Fixes](#challenges-and-fixes)
- [Possible Improvements](#possible-improvements)

## Project Overview

I wanted to build a rover that I could drive remotely while seeing what the camera sees in real time. I also wanted the controls to work without a home router, so I configured the main ESP32 to act as a standalone Wi-Fi access point.

The rover uses a 6WD chassis, with the motors on each side controlled together for forward movement, reversing, and turning. An ultrasonic sensor provides distance readings, while two SG90 servos let me adjust the camera's viewing direction.

The control dashboard runs in a web browser, so there is no need to install a dedicated mobile application.

## Features

- Six-wheel-drive movement with forward, reverse, left, right, and stop controls.
- Standalone Wi-Fi access point hosted by the ESP32.
- Browser-based control dashboard.
- Live MJPEG video streaming from the ESP32-CAM.
- Pan and tilt camera positioning using two SG90 servos.
- HC-SR04 ultrasonic distance measurements.
- Automatic dashboard updates for distance readings.
- Responsive interface designed for phone and computer screens.
- Compact implementation using Arduino C++ and embedded HTML, CSS, and JavaScript.

## Hardware Requirements

| Component | Purpose |
|---|---|
| ESP32 development board | Main controller, Wi-Fi access point, web server, and sensor handling |
| ESP32-CAM AI-Thinker | Camera capture and video streaming |
| OV2640 camera | Live video feed |
| 6WD robotic chassis | Rover frame and six-wheel movement |
| DC geared motors | Drive the wheels |
| L298N motor driver | Controls motor direction |
| 14V 18650 battery pack | Main power source |
| DC-DC buck converter | Steps down the battery voltage for the electronics |
| 2 × SG90 micro servos | Pan and tilt camera movement |
| HC-SR04 ultrasonic sensor | Measures distance to nearby obstacles |
| Jumper wires and breadboard | Prototyping and connections |
| Push buttons | Manual input or additional control functions |

## Pin Configuration

The following table shows the GPIO assignments used for the main ESP32.

### Motor Driver

| L298N Input | ESP32 GPIO | Function |
|---|---:|---|
| IN1 | GPIO 12 | Right-side motor direction |
| IN2 | GPIO 13 | Right-side motor direction |
| IN3 | GPIO 25 | Left-side motor direction |
| IN4 | GPIO 26 | Left-side motor direction |

The motors on each side are grouped together so the rover can move like a differential-drive vehicle.

### Camera Pan and Tilt

| Component | ESP32 GPIO |
|---|---:|
| Pan servo signal | GPIO 14 |
| Tilt servo signal | GPIO 27 |

Both servos are used to adjust the camera's horizontal and vertical viewing angles.

### Ultrasonic Sensor

| HC-SR04 Pin | ESP32 GPIO |
|---|---:|
| TRIG | GPIO 32 |
| ECHO | GPIO 33 |

**Important:** The HC-SR04 echo output is typically 5V, while ESP32 GPIO pins are not 5V tolerant. Use an appropriate voltage divider or level shifter on the ECHO connection.

Check the pin assignments against your actual wiring before uploading the firmware.

## How It Works

The rover is split into two main systems.

**1. Main ESP32 controller**

The main ESP32 handles motor commands, distance measurements, servo positioning, and the web dashboard. It creates a Wi-Fi network named `Rover_Control` and serves the control page at `192.168.4.1`.

The dashboard sends commands to the ESP32, which updates the motor driver inputs or servo positions. Distance readings are requested periodically using JavaScript `fetch()` calls.

**2. ESP32-CAM**

The AI-Thinker ESP32-CAM handles image capture and serves an MJPEG video stream. The OV2640 camera supplies the frames, and the HTTP server sends them to a connected browser.

Keeping the camera and main control functions on separate boards helps divide the workload between the two systems.

## Development Process

### Day 1: Chassis Assembly and Power

I started by assembling the chassis and mounting the six motors. The motors on the left side were connected together, as were the motors on the right, allowing each side to be controlled as a group.

Power distribution needed some attention. The battery pack supplied the motor power, while a buck converter provided a lower voltage for the electronics. I also connected the grounds together so the ESP32 and motor driver would share a common electrical reference.

### Day 2: Motor Control

Once the chassis and wiring were ready, I moved on to the movement functions in Arduino C++.

I implemented the basic commands:

- `moveForward()`
- `moveBackward()`
- `turnLeft()`
- `turnRight()`
- `stopRobot()`

Testing each movement separately made it easier to check the motor directions and correct any wiring mistakes. With the left and right motor groups responding as expected, the basic driving system was ready for integration.

### Day 3: Camera Mount and Obstacle Sensing

Next, I assembled the pan-and-tilt mount using two SG90 servos. The first alignment was off, so the camera did not face straight ahead when the servo returned to its default position. I adjusted the servo horn and mount until the camera was centered.

I then connected the HC-SR04 sensor and added a distance-measurement function using `pulseIn()`. The echo duration is converted into an approximate distance in centimeters, giving the dashboard a simple way to display how close an obstacle is.

At this stage, the sensor provides distance information; autonomous obstacle avoidance would require additional movement logic.

### Day 4: Wi-Fi Access Point and Web Dashboard

I configured the main ESP32 to create its own wireless network rather than connecting through a router.

The network settings were:

- **SSID:** `Rover_Control`
- **Control address:** `192.168.4.1`

I built the dashboard with HTML, CSS, and JavaScript and stored the page in the ESP32 firmware using `PROGMEM`.

The interface includes movement controls, pan and tilt adjustments, and a distance display. JavaScript periodically requests updated sensor readings, allowing the displayed distance to refresh without reloading the page.

### Day 5: ESP32-CAM Video Streaming

The final stage was setting up the camera stream. I configured the AI-Thinker ESP32-CAM to use its available PSRAM and tested an MJPEG stream through `esp_http_server.h`.

The camera configuration used SVGA resolution (800 × 600) and two frame buffers to help maintain a responsive stream. Actual frame rate and latency depend on the camera settings, wireless conditions, and available memory.

I also ran into the familiar `No serial data received` upload error while flashing the camera board. Checking the serial connection, closing the Serial Monitor, entering download mode, and resetting the board helped resolve the issue.

After flashing, I returned GPIO 0 to its normal operating state and restarted the board to test the camera.

With the control interface and camera server running, I could connect to the rover's network from my phone and access the system through a browser.

## Web Control Interface

The dashboard is designed to keep the main controls in one place.

It provides:

- **Movement controls:** Drive forward, reverse, turn, or stop.
- **Camera controls:** Adjust pan and tilt independently.
- **Distance display:** View the latest ultrasonic measurement.
- **Live video:** View the camera feed through the ESP32-CAM stream endpoint.

The control page is embedded in the main ESP32 firmware, which avoids the need to host the interface on an external server.

The main control page and camera stream may use different endpoints or ports depending on the firmware configuration. If the two boards host separate servers, the dashboard must use the ESP32-CAM's actual stream address.

## Power Distribution

The rover uses a battery pack rated at approximately 14V, a motor driver, and a DC-DC buck converter for the lower-voltage electronics.

The general power arrangement is:

- Battery pack to the motor driver's motor supply.
- Battery pack to the buck converter input.
- Buck converter output to the electronics, using the voltage and current ratings required by each board and peripheral.
- Common ground between the main ESP32, motor driver, sensor electronics, and other connected components.

**Power considerations**

- Verify the battery pack's fully charged voltage, not just its nominal voltage.
- Confirm that the motor driver can handle the motors' current requirements.
- Do not assume the L298N can supply the battery voltage directly to the motors without accounting for its voltage drop and thermal limits.
- Check the required input voltage for each ESP32 board before connecting power.
- Power the servos from a suitable regulated supply that can handle their current peaks; do not rely on the ESP32's GPIO pins to supply servo power.
- Connect the HC-SR04 ECHO pin through a suitable level-shifting circuit.
- Use appropriate battery protection, insulation, and secure wiring.

Motor and servo current spikes can cause resets or unstable sensor readings if the supply is undersized.

## Setup and Usage

### 1. Assemble the Hardware

Mount the motors, wheels, motor driver, controller boards, sensor, battery, and camera bracket on the chassis. Check all power and signal connections before applying power.

### 2. Configure the Main ESP32

Install the ESP32 board support package in the Arduino IDE, select the correct board and port, and upload the main controller firmware.

The firmware should configure the motor pins, servo outputs, ultrasonic sensor, Wi-Fi access point, and web server.

### 3. Configure the ESP32-CAM

Select the appropriate AI-Thinker ESP32-CAM board configuration and upload the camera firmware. Make sure the selected camera pin definitions match the AI-Thinker module.

If your programming adapter requires it, use the board's download-mode procedure to flash the firmware. Restore normal boot mode before running the camera.

### 4. Connect to the Rover

1. Power on the rover.
2. Open the Wi-Fi settings on your phone or computer.
3. Connect to `Rover_Control`.
4. Open `http://192.168.4.1` in a browser.
5. Use the dashboard to test movement, camera positioning, and distance readings.
6. Open the configured ESP32-CAM stream URL if the video is served separately.

The actual URLs depend on how the main ESP32 and ESP32-CAM firmware are configured.

## Challenges and Fixes

| Challenge | What I changed or checked |
|---|---|
| Camera mount was not centered | Repositioned the servo horn and adjusted the bracket |
| ESP32-CAM upload failed | Checked the serial connection, download mode, reset timing, and open serial monitors |
| Unstable electronics during motor operation | Checked the power supply and common-ground connections |
| Distance readings were unreliable | Checked the sensor wiring and measurement timing |
| Camera stream was slow | Adjusted the frame size and buffer configuration |

These checks made the project easier to troubleshoot by separating hardware problems from firmware and network issues.

## Possible Improvements

There are several ways I would like to develop the rover further:

- Add automatic obstacle avoidance instead of only displaying distance readings.
- Implement a command timeout that stops the motors if the control connection is lost.
- Add adjustable camera streaming quality and frame rate.
- Improve motor speed control with PWM and suitable motor-driver hardware.
- Add battery-voltage monitoring to the dashboard.
- Improve sensor filtering to reduce noisy distance readings.
- Add a clearer connection-status indicator for the video stream and control interface.
- Organize the firmware into separate modules for motor control, sensing, networking, and camera handling.

## Conclusion

Building this rover brought together mechanical assembly, motor control, embedded networking, sensor integration, and video streaming in one project.

The most useful part was getting the different systems to work together: the main ESP32 handles driving and the dashboard, while the ESP32-CAM provides the live camera feed. The standalone access point makes it possible to control the rover without relying on an existing router.

There is still room to improve reliability and add more autonomous behavior, but the project provides a solid starting point for a remotely operated 6WD rover.

## Technologies Used

- Arduino C++
- ESP32 and ESP32-CAM
- HTML, CSS, and JavaScript
- Wi-Fi SoftAP
- HTTP server
- MJPEG video streaming
- HC-SR04 ultrasonic sensing
- SG90 servo control
- L298N motor driver

---

**Note:** This README describes the hardware and configuration used in the project. To reproduce the build, the firmware source files, actual stream URL, and any board-specific configuration should be included in the repository.

<img width="1280" height="960" alt="photo_2026-10-27_15-56-54" src="https://github.com/user-attachments/assets/aec83fa5-1703-4788-b75a-feaa7ca37541" />
