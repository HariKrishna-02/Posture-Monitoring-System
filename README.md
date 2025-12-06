# Intelligent Posture Monitoring Using ConvLSTM via Edge AI

Intelligent wearable posture monitoring system using ESP32, MPU6050 IMU, and three FSR sensors to detect neck and back slouch in real time. The device gives haptic feedback via a vibration motor and streams labeled 9‑D sensor data to a Raspberry Pi over WiFi for logging and future ConvLSTM edge‑AI model training.

---

## Materials Required

- ESP32 development board  
- MPU6050 6‑axis IMU sensor  
- Force Sensitive Resistors (FSR) × 3  
- Vibration motor (3–5 V)  
- BC547 NPN transistor  
- Resistors: 10 kΩ × 3, 1 kΩ × 1  
- Diodes 1N4007 × 4 (3 for FSRs, 1 flyback for motor)  
- Raspberry Pi with WiFi, SD card, power supply  
- Breadboard and jumper wires  
- USB cable for ESP32  
- PC with Arduino IDE and Python 3 installed  

---

## Step‑Wise Procedure

### 1. Hardware Wiring

- Connect MPU6050 to ESP32 (3.3 V, GND, SDA → GPIO 21, SCL → GPIO 22).  
- Build three FSR voltage dividers:
  - Back FSR → GPIO 34 + 10 kΩ to GND + diode from 3.3 V.  
  - Left FSR → GPIO 35 + 10 kΩ to GND + diode from 3.3 V.  
  - Right FSR → GPIO 32 + 10 kΩ to GND + diode from 3.3 V.  
- Vibration motor driver:
  - Motor + → 3.3 V.  
  - Motor − → BC547 collector.  
  - BC547 emitter → GND.  
  - ESP32 GPIO 25 → 1 kΩ → BC547 base.  
  - Flyback diode across motor (cathode to +, anode to −).  

### 2. ESP32 Firmware Setup

- Install Arduino IDE and ESP32 board package.  
- Install libraries: **Adafruit_MPU6050** and **Adafruit Unified Sensor**.  
- Open the ESP32 posture monitoring sketch from this repo.  
- Update WiFi credentials and Raspberry Pi URL in the code:
  - `const char* ssid = "YOUR_WIFI_SSID";`  
  - `const char* password = "YOUR_WIFI_PASSWORD";`  
  - `const char* serverURL = "http://<PI_IP>:5000/sensor_data";`  
- Select the correct ESP32 board and port, then upload the sketch.  
- Open Serial Monitor at 115200 baud and confirm WiFi connection and sensor readings.

### 3. Raspberry Pi Configuration

- Connect Pi to the same WiFi network as the ESP32.  
- Install required packages:
  - `sudo apt update`  
  - `sudo apt install python3-flask python3-pandas -y`  
- Create `sensor_receiver.py` from this repository and save it on the Pi.  
- Run the Flask server:
  - `python3 sensor_receiver.py`  
- Note the Pi IP address from `hostname -I` and ensure it matches `serverURL` in ESP32 code.

### 4. System Operation

- Wear or mount the prototype on the upper back with:
  - Back FSR on the chair backrest contact point.  
  - Left and Right FSRs under the seat region.  
- Power the Raspberry Pi and start the Flask server.  
- Power the ESP32; wait for “WiFi connected” in Serial Monitor.  
- Sit in different postures:
  - Good posture (upright).  
  - Neck slouch (bending forward).  
  - Back slouch (leaning too far forward/back/side).  

### 5. Processing Pipeline (ESP32)

1. Read 9‑D raw values (3 FSR + 3‑axis accel + 3‑axis gyro) at ~100 Hz.  
2. Compute pitch and roll from accelerometer:  
   - `pitch = atan2(-Accel_X, sqrt(Accel_Y^2 + Accel_Z^2))`  
   - `roll  = atan2(Accel_Y, Accel_Z)`  
3. Apply posture thresholds:  
   - Neck slouch if `pitch < -20°`.  
   - Back slouch if `roll < -25°` or `roll > 25°`.  
4. If slouch detected, set motor pin HIGH to trigger vibration; otherwise LOW.  
5. Every 1 second, send a JSON payload over WiFi to the Raspberry Pi with:
   - FSR values, 6‑axis IMU data, pitch, roll, posture label, and motor status.

### 6. Data Logging and ML Preparation

- The Raspberry Pi Flask server receives each JSON packet at `/sensor_data`.  
- Data is appended into `convlstm_posture_data.csv` with:
  - Timestamp  
  - `fsr_back`, `fsr_left`, `fsr_right`  
  - `ax`, `ay`, `az`, `gx`, `gy`, `gz`  
  - `pitch`, `roll`, `posture`, and optional quality flags  
- These CSV files form the dataset for training ConvLSTM‑based posture classification models that can later run on the Pi as an edge‑AI module.

---

## Expected Output

- **On ESP32 Serial Monitor**
  - Continuous logs of:
    - FSR sensor values  
    - Pitch and roll angles  
    - Posture status: `Good`, `Neck Slouch`, `Back Slouch`, or `Neck & Back Slouch`  
    - Motor status: `ON` when slouching, `OFF` during good posture  

- **Physical Feedback**
  - Vibration motor activates whenever the user’s neck or back angles cross the slouch thresholds and stops when posture returns to the safe range.

- **On Raspberry Pi**
  - Terminal prints each received sample with timestamp and posture label.  
  - `convlstm_posture_data.csv` grows over time with labeled 9‑D sequences ready for deep‑learning experiments.

---

## Credits

Created by **[JP HariKrishna Raj]**  
Project: **Intelligent Posture Monitoring Using ConvLSTM via Edge AI**
