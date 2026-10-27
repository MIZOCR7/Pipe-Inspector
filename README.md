#  Autonomous 6WD Rover with ESP32-CAM, Pan/Tilt Vision, and Web Dashboard

## About The Project
I built this project to create an advanced, wireless-controlled **6-Wheel Drive (6WD) robotic rover** from scratch. I wanted a platform that could stream live video, navigate using an integrated distance sensor, and be fully controlled remotely through a custom-built web dashboard hosted on a standalone Wi-Fi network. 

---

## Components & Hardware Used
* **Microcontrollers:** 
  * ESP32 Development Board (Main system logic and web server)
  * ESP32-CAM (AI-Thinker module with OV2640 camera and PSRAM)
* **Chassis & Drive:** 
  * 6WD Robotic Chassis with DC Motors
  * L298N Dual H-Bridge Motor Driver
* **Power Architecture:** 
  * 14V Battery Pack (18650 Li-ion cells)
  * DC-DC Buck Converter (Step-down module delivering stable 5V output)
* **Sensors & Actuators:** 
  * HC-SR04 Ultrasonic Distance Sensor
  * 2x SG90 Servo Motors (Pan and Tilt mechanism)
* **Prototyping:** Breadboard, jumper wires, and push buttons.

---

## 📌 System Wiring & Pinout Reference
* **Motor Drivers (L298N):**
  * Right Motors In1/In2: `GPIO 12`, `GPIO 13`
  * Left Motors In1/In2: `GPIO 25`, `GPIO 26`
* **Pan/Tilt Servos:**
  * Pan Servo: `GPIO 14`
  * Tilt Servo: `GPIO 27`
* **Ultrasonic Sensor (HC-SR04):**
  * Trig: `GPIO 32`
  * Echo: `GPIO 33`

---

##  Day-by-Day Development Log

### Day 1: Chassis Assembly & Power Distribution Architecture
I started by assembling the physical 6WD robotic chassis and mounting the DC motors in parallel pairs per side. For power management, I wired my 14V 18650 battery pack to feed the high-voltage input of the L298N motor driver while routing the power through a DC-DC Buck Converter to step it down to a stable 5V. I established a common ground (`Common GND`) across the entire system to protect the delicate logic components from high-current motor spikes.

### Day 2: Microcontroller Integration & Locomotion Logic
Next, I connected the L298N logic control inputs to my ESP32 development board. I wrote the foundational movement functions in Arduino C++ (`moveForward`, `moveBackward`, `turnLeft`, `turnRight`, and `stopRobot`) to precisely govern the 6-wheel drive layout and tested the basic movement mechanics.

### Day 3: Pan/Tilt Camera Platform & Distance Sensing
I integrated the dual SG90 servos to form a responsive Pan/Tilt mechanism for the camera. I calibrated the bottom Pan servo horn mechanically so that a software-defined neutral position of 90 degrees pointed directly forward. Then, I wired the HC-SR04 ultrasonic distance sensor to measure obstacles ahead, implementing precise pulse-width (`pulseIn`) calculations to convert echo timings into real-time distance metrics in centimeters.

### Day 4: Wireless Access Point & Custom Web Dashboard UI
To make the rover completely untethered, I programmed the ESP32 to launch its own standalone Wi-Fi Access Point (`Rover_Control`) at IP address `192.168.4.1`. I designed an embedded, responsive HTML/CSS/JavaScript dashboard (`PROGMEM`) featuring:
* A real-time distance display that automatically fetches and updates sensor metrics every 500ms.
* A touch/mouse-friendly directional control grid.
* Dedicated buttons to adjust the camera's pan and tilt angles on the fly.

### Day 5: ESP32-CAM Video Streaming Optimization & Final Deployment
For the vision system, I configured an AI-Thinker ESP32-CAM module utilizing external PSRAM (`CAMERA_FB_IN_PSRAM`) and double-buffering (`fb_count = 2`) to maximize frame rates. I set up an embedded HTTP server routine (`esp_http_server.h`) to stream high-quality `FRAMESIZE_SVGA` (800x600) MJPEG video over port 80. After resolving flash boot configurations (grounding `GPIO 0` during upload) and mapping serial ports, I successfully deployed the firmware.
<img width="1280" height="960" alt="photo_2026-10-27_15-56-54" src="https://github.com/user-attachments/assets/46a2da49-2b73-47c0-a657-8ef005f47a2b" />
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/0629d854-8ba9-4ae5-9f70-cc57fa3b6c38" />
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/062bb9cf-9d96-4bda-978a-352ed8663f98" />

