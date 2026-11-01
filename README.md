# Project Journals for 6WD Self Driving Rover using ESP32-CAM and Web UI

## Introduction
This page will detail the different steps I took to develop my 6WD Robotic Rover. Each day details what issues I faced, how I connected the components and what progress I made.
---

## System Wiring & Pinout Reference

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

---

### Day 1: Building the chassis
I began developing my 6WD Robotic Rover by attaching all the DC motors to the chassis. I connected the motors in pairs to the left and right sides so they run in unison.

I struggled to connect the power to the system. The battery provides 14 volts which is great for giving the motors more torque via the L298N motor driver, however this would cause damage to the ESP32 or sensors. I resolved this issue by connecting the 14volt to the L298N motor driver. I also connected another wire to the DC-DC Buck converter and brought it down to 5 volts. Finally I also connected all components together with a common ground connection (GND) to ensure nothing would float.
* **Key milestone:** Connected the 6WD chassis together successfully. Separated all connections so the 14volt can be used for the motors and 5 volt for the logic circuitry. Used the buck converter to do this.
* **Total time taken:** 5 hours.

---

### Day 2: Connecting to microcontroller.

Next I connected the motor driver to the ESP32. I connected the following connections:
- Right Motor In1/In2 – `GPIO12` and `GPIO13`
- Left Motor In1/In2 – `GPIO25` and `GPIO26`
I then created the following functions in arduino C++. (Move Forward, Move Backward, Turn Left, Turn Right, Stop Robot). This was very exciting to test and all 6 motors moved in the direction I wanted.
* **Key Milestone:** Connected the motor driver and created basic movement code.
* **Total time taken:** 3 hours.

---

### Day 3: Camera Mount and Distance Sensor
Today I worked on adding more mobility to the camera and adding distance sensors. I attached two SG90 servos together to create a Pan/Tilt mount. I connected these to `GPIO14` (Pan) and `GPIO27` (Tilt). I struggled with the pan mount initially as I had connected it the wrong way round. When I powered it on it reset to 90 degrees but it was looking to the side. I disconnected some of the servos and adjusted the servo horn to get it pointing straight.

I then connected the HC-SR04 distance sensor to `GPIO32` (Trig) and `GPIO33` (Echo). I then created a short function to calculate the pulse width using pulseIn.
* **Key Milestone:** Sorted the pan servo positioning. Got real time data from the distance sensor.
* **Total time taken:** 6 hours.

---

### Day 4: Setting up the Access Point and Web Interface.
I wanted my rover to be completely wireless so I created a Wi-Fi Access Point on the ESP32. This gave me a unique network (`Rover_Control`) with password: `Password123`. The IP address was `192.168.4.1`.

I created a responsive web interface using HTML, CSS and JavaScript. This was all stored in PROGMEM on the ESP32. I added a live counter for distance. This used asynchronous JavaScript to continuously poll the sensor every 500ms. I also added a responsive control panel and buttons to adjust the pan and tilt of the camera.
* **Key Milestone:** Created a self hosted Wi-Fi access point. Added web interface to control with.
* **Total time taken:** 5 hours.

---

### Day 5: Optimising ESP32-CAM and final setup.
The final task was to get the ESP32-CAM to stream videos without lagging. Firstly I configured the AI-Thinker to use its external PSRAM (`CAMERA_FB_IN_PSRAM`). I added double buffering (`fb_count = 2`) to increase the frame rate. I didn’t want to use large AI models so I created a simple MJPEG server using `esp_http_server.h`. This provided crisp SVGA frames (800x600). All this was streamed via port 80.

Flashing the ESP32-CAM can sometimes be tricky. I had the issue of "No serial data received" (Access is denied on COM12) a couple of times. I solved this by connecting `GPIO0` to ground so it entered flash mode. I pressed the reset button when the upload came up with "Connecting...". I also made sure all serial monitors were closed. Once the code had been uploaded I disconnected `GPIO0` and pressed reset. Then I connected my phone to the rover’s Wi-Fi. The live video opened instantly. Everything worked!
* **Key Milestone:** Optimised the ESP32-CAM firmware for fast streaming of SVGA Video. Final setup complete.
* **Total time taken:** 6 hours.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/ba98c6bf-f57b-4ec2-b937-8950cb443f47" />
<img width="1280" height="960" alt="photo_2026-10-27_15-56-54" src="https://github.com/user-attachments/assets/847fb425-79dc-4ea6-92cd-0c29e64ce854" />
