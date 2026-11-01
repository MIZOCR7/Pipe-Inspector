# ESP32 and ESP32-CAM-Based 6WD Autonomous Robotic Rover with Pan/Tilt Vision and Web Dashboard

## Abstract
This project outlines the development of an advanced, wireless-controlled 6-Wheel Drive (6WD) robotic rover powered by an ESP32 microcontroller and an ESP32-CAM module. The system integrates a standalone Wi-Fi Access Point web server for remote real-time piloting, a dual-servo Pan/Tilt camera vision system for high-definition MJPEG video streaming, and an HC-SR04 ultrasonic sensor for proximity telemetry. High-current motor actuation is managed via an L298N dual H-bridge driver, governed by a multi-tiered power architecture that isolates high-voltage motor supplies from sensitive logic components.

## 1. System Architecture
The system integrates digital and analog components to execute real-time wireless communication, video streaming, motor locomotion, and sensor telemetry simultaneously without execution suspension.

### Subsystems Overview
1. **Locomotion and Motor Control:** A 6-wheel drive chassis driven in parallel pairs by DC motors via an L298N motor driver, enabling precise bidirectional movement (Forward, Backward, Left, Right, Stop).
2. **Vision and Live Streaming:** An ESP32-CAM module utilizing external PSRAM (`CAMERA_FB_IN_PSRAM`) and double-buffering (`fb_count = 2`) to stream high-quality MJPEG video over an embedded HTTP server.
3. **Pan/Tilt Camera Actuation:** Dual SG90 micro-servos govern the horizontal (Pan) and vertical (Tilt) orientation of the camera platform for dynamic field-of-view adjustments.
4. **Proximity Telemetry:** An HC-SR04 ultrasonic sensor measures obstacle clearance using precise pulse-width calculations (`pulseIn`), feeding real-time metric updates to the control interface.
5. **Wireless Command Interface:** A self-hosted Wi-Fi Access Point (AP) serves a responsive HTML/CSS/JavaScript dashboard (`PROGMEM`) for asynchronous control and sensor monitoring.

## 2. Hardware Specifications

### Microcontrollers
- ESP32 Development Board (Main System Logic and Web Server)
- ESP32-CAM (AI-Thinker Module with OV2640 Camera and PSRAM)

### Chassis & Drive
- 6WD Robotic Chassis with High-Torque DC Motors
- L298N Dual H-Bridge Motor Driver

### Power Architecture
- 14V Battery Pack (18650 Li-ion cells)
- DC-DC Buck Converter (Step-down module delivering a regulated 5V fixed output)

### Sensors & Actuators
- HC-SR04 Ultrasonic Distance Sensor (Digital)
- 2x SG90 Servo Motors (Pan and Tilt mechanism)
- Breadboard, Jumper Wires, and Prototyping Hardware

## 3. Crucial Safety and Hardware Protocols

### High-Voltage Power Warning (14V)
This project incorporates a 14V battery pack to drive the motors through the L298N driver. Direct exposure of microcontroller logic pins to this voltage level will cause catastrophic hardware failure. A DC-DC Buck Converter must be implemented to step down the supply voltage to a stable 5V logic rail, with a strict **Common Ground (GND)** topology maintained across all components.

### ESP32-CAM Bootstrapping and Flashing Constraints
To achieve successful code deployment on the ESP32-CAM, precise bootstrapping constraints must be observed:
- **GPIO 0 State:** The `GPIO 0` pin must be physically grounded (`GND`) during the firmware upload sequence to force the module into flashing mode.
- **Serial Conflict Management:** The serial communication port must remain unallocated by external monitors (`Access is denied` errors) during compilation, and a manual hardware reset via the `RST` pin may be required upon the `Connecting...` prompt.

## 4. Pin Mapping and Configuration

| Component | ESP32 / CAM GPIO | Type | Power (VCC) | Ground (GND) |
| :--- | :--- | :--- | :--- | :--- |
| Right Motor In1 | GPIO 12 | Digital Output | L298N Logic | Common GND |
| Right Motor In2 | GPIO 13 | Digital Output | L298N Logic | Common GND |
| Left Motor In1 | GPIO 25 | Digital Output | L298N Logic | Common GND |
| Left Motor In2 | GPIO 26 | Digital Output | L298N Logic | Common GND |
| Pan Servo | GPIO 14 | PWM / Digital Output | 5V (Buck Converter) | Common GND |
| Tilt Servo | GPIO 27 | PWM / Digital Output | 5V (Buck Converter) | Common GND |
| Ultrasonic Trig | GPIO 32 | Digital Output | 5V (Buck Converter) | Common GND |
| Ultrasonic Echo | GPIO 33 | Digital Input | 5V (Buck Converter) | Common GND |
| ESP32-CAM Flash | GPIO 0 | Digital Input (Boot) | 3.3V | GND (During Upload) |

## 5. Software Implementation (C++)
```cpp
// Modular firmware configuration managing Wi-Fi AP, Web Server, L298N Motor Driver, SG90 Servos, and HC-SR04 Sensor.
