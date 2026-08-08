# 🚦 Smart Traffic Management System

An ESP32-based Smart Traffic Management System that combines **IR
vehicle detection**, **RFID-based vehicle identification**, **adaptive
signal timing**, **74HC595 shift registers**, and **Blynk IoT
monitoring**.

The system controls traffic lights for four directions:

-   North
-   South
-   East
-   West

It also recognizes configured **Emergency** and **VIP RFID tags** and
provides an alert indication.

> **Important:** This README documents the behavior of the Arduino/ESP32
> code provided for this project. It does not claim features that are
> not implemented in the current source code.

------------------------------------------------------------------------

## 📌 Table of Contents

-   [Project Overview](#-project-overview)
-   [Features](#-features)
-   [System Architecture](#-system-architecture)
-   [Hardware Requirements](#-hardware-requirements)
-   [Pin Configuration](#-pin-configuration)
-   [Traffic Light Control](#-traffic-light-control)
-   [IR Sensor Operation](#-ir-sensor-operation)
-   [RFID Operation](#-rfid-operation)
-   [Blynk IoT](#-blynk-iot)
-   [Software Requirements](#-software-requirements)
-   [Libraries](#-libraries)
-   [Configuration](#-configuration)
-   [How the Program Works](#-how-the-program-works)
-   [Traffic Timing](#-traffic-timing)
-   [RFID IDs](#-rfid-ids)
-   [74HC595 Shift Register](#-74hc595-shift-register)
-   [Installation and Setup](#-installation-and-setup)
-   [Testing](#-testing)
-   [Expected Serial Monitor Output](#-expected-serial-monitor-output)
-   [Troubleshooting](#-troubleshooting)
-   [Current Implementation
    Limitations](#-current-implementation-limitations)
-   [Project Flow](#-project-flow)
-   [Future Improvements](#-future-improvements)
-   [Security Note](#-security-note)
-   [License](#-license)

------------------------------------------------------------------------

## 🔎 Project Overview

The Smart Traffic Management System is implemented using an **ESP32** as
the main controller.

The ESP32:

1.  Reads vehicle-presence information from four IR sensors.
2.  Displays traffic status on a Blynk dashboard.
3.  Controls 12 traffic LEDs through two 74HC595 shift registers.
4.  Reads RFID cards using an MFRC522 RFID reader.
5.  Recognizes configured Emergency and VIP RFID tags.
6.  Activates a physical alert LED for Emergency/VIP events.
7.  Temporarily pauses the normal traffic sequence when an Emergency
    RFID tag is detected.

The system uses Wi-Fi to communicate with the **Blynk Cloud**.

------------------------------------------------------------------------

## ✨ Features

### 1. Four-Direction Traffic Lights

Traffic signals are represented for:

-   North
-   South
-   East
-   West

Each direction has:

-   🔴 Red
-   🟡 Yellow
-   🟢 Green

A total of **12 LEDs** are controlled.

### 2. Vehicle Presence Detection

Four IR sensors monitor the four directions.

A sensor is considered active when its digital output is **LOW**.

``` text
LOW  → Vehicle detected
HIGH → No vehicle detected
```

The sensor information is also sent to Blynk.

### 3. Adaptive Green-Time Selection

The code selects between:

-   **3 seconds** minimum green-time value
-   **7 seconds** maximum green-time value

When traffic is detected by the corresponding IR sensor, the maximum
value is selected.

When traffic is not detected, the minimum value is selected.

> The current program uses a single digital IR sensor per direction, so
> it detects vehicle presence rather than calculating an exact number of
> vehicles or a numerical queue length.

### 4. RFID Vehicle Identification

An MFRC522 RFID reader is used to identify configured RFID tags.

Two tag IDs are currently configured:

-   Emergency vehicle
-   VIP vehicle

### 5. Emergency Mode

When the configured Emergency RFID tag is detected:

-   Emergency mode is enabled.
-   The Emergency Blynk indicator is activated.
-   The physical alert LED is turned ON for 10 seconds.
-   The normal traffic sequence is temporarily prevented while emergency
    mode is active.

**The current code does not automatically select a specific emergency
lane or switch that lane to green.**

### 6. VIP Detection

When the configured VIP RFID tag is detected:

-   A VIP message is printed to Serial Monitor.
-   The Blynk emergency/VIP indicator is set to an intermediate value.
-   The physical alert LED is turned ON for 2 seconds.

The current code does **not** change the traffic-light state for the VIP
vehicle.

### 7. Blynk Monitoring

The system sends lane vehicle-presence information to Blynk using
virtual pins.

The Blynk dashboard can therefore indicate whether traffic is currently
detected on each monitored direction.

------------------------------------------------------------------------

# 🏗 System Architecture

``` text
                     ┌─────────────────────┐
                     │       Blynk         │
                     │   IoT Dashboard     │
                     └──────────┬──────────┘
                                │ Wi-Fi
                                │
                       ┌────────▼────────┐
                       │      ESP32      │
                       │ Main Controller │
                       └───────┬─────────┘
                               │
          ┌────────────────────┼─────────────────────┐
          │                    │                     │
          ▼                    ▼                     ▼
   ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
   │  IR Sensors │      │ MFRC522 RFID│      │  Alert LED  │
   │ N/S/E/W     │      │   Reader    │      │   GPIO 27   │
   └─────────────┘      └─────────────┘      └─────────────┘

                       ┌────────────────┐
                       │ 74HC595 #1/#2  │
                       │ Shift Registers│
                       └───────┬────────┘
                               │
                       ┌───────▼────────┐
                       │ 12 Traffic LEDs│
                       │ N/S/E/W × RYG  │
                       └────────────────┘
```

------------------------------------------------------------------------

# 🔩 Hardware Requirements

  Component                      Quantity Purpose
  ------------------------- ------------- ------------------------------
  ESP32 development board               1 Main controller
  MFRC522 RFID reader                   1 RFID tag detection
  RFID tags/cards               2 or more Emergency/VIP identification
  IR sensors                            4 Vehicle-presence detection
  74HC595 shift registers               2 Traffic LED output expansion
  Red LEDs                              4 Traffic signals
  Yellow LEDs                           4 Traffic signals
  Green LEDs                            4 Traffic signals
  Alert LED                             1 Emergency/VIP indication
  Resistors                   As required LED current limiting
  Breadboard/PCB                        1 Circuit assembly
  Jumper wires                As required Connections
  5V power supply                       1 ESP32/system power

------------------------------------------------------------------------

# 📌 Pin Configuration

## ESP32 → 74HC595

  Function     ESP32 GPIO
  ---------- ------------
  Data            GPIO 14
  Latch           GPIO 12
  Clock           GPIO 13

The program controls the shift registers using Arduino's `shiftOut()`
function.

## ESP32 → IR Sensors

  Direction     ESP32 GPIO Active State
  ----------- ------------ --------------
  North            GPIO 32 LOW
  South            GPIO 33 LOW
  East             GPIO 25 LOW
  West             GPIO 26 LOW

## ESP32 → MFRC522 RFID

The code uses the ESP32's default SPI interface together with the
configured RFID pins.

  MFRC522 Pin     ESP32 GPIO
  ------------- ------------
  SDA/SS              GPIO 5
  RST                GPIO 22
  SCK                GPIO 18
  MOSI               GPIO 23
  MISO               GPIO 19
  VCC                   3.3V
  GND                    GND

## Alert LED

  Component     ESP32 GPIO
  ----------- ------------
  Alert LED        GPIO 27

> Use an appropriate current-limiting resistor with LEDs.

------------------------------------------------------------------------

# 🚦 Traffic Light Control

The program stores traffic-light patterns in two bytes.

``` cpp
const byte states[][2] = {
  {NORTH}, {NORTH_YELLOW},
  {SOUTH}, {SOUTH_YELLOW},
  {EAST},  {EAST_YELLOW},
  {WEST},  {WEST_YELLOW}
};
```

The available states are:

``` text
NORTH
NORTH_YELLOW

SOUTH
SOUTH_YELLOW

EAST
EAST_YELLOW

WEST
WEST_YELLOW
```

An `ALL_RED` pattern is also defined and is used during startup.

### Traffic sequence

The programmed sequence is:

``` text
North
  ↓
North Yellow
  ↓
South
  ↓
South Yellow
  ↓
East
  ↓
East Yellow
  ↓
West
  ↓
West Yellow
  ↓
Repeat
```

The two 74HC595 registers are updated by:

``` cpp
updateLights(byte shiftReg1, byte shiftReg2);
```

------------------------------------------------------------------------

# 🚘 IR Sensor Operation

The program reads each IR sensor using `digitalRead()`.

For example:

``` cpp
bool northTraffic = digitalRead(IR_NORTH) == LOW;
```

Therefore:

``` text
IR sensor = LOW
      ↓
Vehicle detected
      ↓
Blynk status = 255
```

and:

``` text
IR sensor = HIGH
      ↓
No vehicle detected
      ↓
Blynk status = 0
```

### Important

The current implementation uses **one digital IR sensor per direction**.

Therefore, it provides **vehicle presence detection**, not an exact
vehicle count.

------------------------------------------------------------------------

# 🪪 RFID Operation

The MFRC522 library is used to communicate with the RFID reader.

The program checks for a new card:

``` cpp
if (!rfid.PICC_IsNewCardPresent() ||
    !rfid.PICC_ReadCardSerial())
    return;
```

The RFID UID is converted into a hexadecimal string.

Example:

``` text
RFID Tag Scanned: afac611f
```

The program then compares the UID against the configured Emergency and
VIP IDs.

------------------------------------------------------------------------

# 🚨 Emergency RFID Behavior

Emergency RFID UID:

``` text
afac611f
```

When detected:

``` text
RFID detected
     ↓
Emergency Vehicle Detected
     ↓
emergencyMode = true
     ↓
Blynk V6 = 255
     ↓
Alert LED ON
     ↓
10-second alert period
     ↓
Normal sequence resumes
```

### Important implementation detail

The current code **does not identify which of the four directions the
RFID vehicle is approaching from**.

It also does not explicitly change the traffic lights to:

``` text
Emergency direction → GREEN
Other directions → RED
```

Therefore this README does not claim that behavior.

------------------------------------------------------------------------

# ⭐ VIP RFID Behavior

VIP RFID UID:

``` text
241451a7
```

When detected:

``` text
RFID detected
     ↓
VIP Vehicle Detected
     ↓
Blynk V6 = 128
     ↓
Alert LED ON
     ↓
2 seconds
     ↓
Alert LED OFF
```

The current code does not modify the traffic-light state for VIP
vehicles.

------------------------------------------------------------------------

# 📱 Blynk IoT

The project uses Blynk for monitoring.

## Blynk Template

``` text
Template ID:
TMPL3c4Ogzfwl

Template Name:
TRAFFIC MANAGEMENT SYSTEM
```

> Do not publish your actual Blynk authentication token in this README
> or in a public repository.

## Virtual Pins

  Virtual Pin   Purpose                   Value
  ------------- ------------------------- ----------------
  V2            North traffic status      0 or 255
  V3            South traffic status      0 or 255
  V4            East traffic status       0 or 255
  V5            West traffic status       0 or 255
  V6            Emergency/VIP indicator   0, 128, or 255

### V2--V5

``` text
0   → No vehicle detected
255 → Vehicle detected
```

### V6

``` text
0   → Normal
128 → VIP detected
255 → Emergency detected
```

------------------------------------------------------------------------

# 💻 Software Requirements

## Arduino IDE

Install the Arduino IDE and configure it for ESP32 development.

## ESP32 Board Package

Install the ESP32 board package through the Arduino IDE Board Manager.

## Required Libraries

The source code includes:

``` cpp
#include <SPI.h>
#include <MFRC522.h>
#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
```

### Libraries used

  Library            Purpose
  ------------------ ---------------------------
  SPI                SPI communication
  MFRC522            RFID reader communication
  WiFi               ESP32 Wi-Fi connectivity
  BlynkSimpleEsp32   Blynk IoT communication

------------------------------------------------------------------------

# ⚙️ Configuration

Before uploading the program, configure your Wi-Fi and Blynk
credentials.

The source code contains:

``` cpp
#define BLYNK_TEMPLATE_ID "YOUR_TEMPLATE_ID"
#define BLYNK_TEMPLATE_NAME "TRAFFIC MANAGEMENT SYSTEM"
#define BLYNK_AUTH_TOKEN "YOUR_BLYNK_AUTH_TOKEN"

char auth[] = "YOUR_BLYNK_AUTH_TOKEN";
char ssid[] = "YOUR_WIFI_SSID";
char pass[] = "YOUR_WIFI_PASSWORD";
```

Replace the placeholders with your own credentials.

### Example

``` cpp
#define BLYNK_TEMPLATE_ID "YOUR_TEMPLATE_ID"
#define BLYNK_TEMPLATE_NAME "TRAFFIC MANAGEMENT SYSTEM"
#define BLYNK_AUTH_TOKEN "YOUR_BLYNK_AUTH_TOKEN"

char auth[] = "YOUR_BLYNK_AUTH_TOKEN";
char ssid[] = "YOUR_WIFI_SSID";
char pass[] = "YOUR_WIFI_PASSWORD";
```

------------------------------------------------------------------------

# ⏱ Traffic Timing

The program defines:

``` cpp
const unsigned long DEFAULT_GREEN_TIME = 5000;
const unsigned long MIN_GREEN_TIME = 3000;
const unsigned long MAX_GREEN_TIME = 7000;
const unsigned long YELLOW_TIME = 1000;
const unsigned long EMERGENCY_TIME = 10000;
```

  Parameter               Value Purpose
  -------------------- -------- -------------------------
  DEFAULT_GREEN_TIME      5 sec Defined default value
  MIN_GREEN_TIME          3 sec Minimum adaptive value
  MAX_GREEN_TIME          7 sec Maximum adaptive value
  YELLOW_TIME             1 sec Yellow-state interval
  EMERGENCY_TIME         10 sec Emergency mode duration

### Note about `DEFAULT_GREEN_TIME`

`DEFAULT_GREEN_TIME` is defined in the source code but is **not
currently used** by the control logic.

------------------------------------------------------------------------

# 🆔 RFID IDs

The current source code contains the following configured UIDs:

  RFID UID        Type        Action
  --------------- ----------- ----------------------------------
  `afac611f`      Emergency   Emergency mode + 10-second alert
  `241451a7`      VIP         VIP indication + 2-second alert
  Any other UID   Unknown     Serial Monitor message only

Example:

``` text
RFID Tag Scanned: afac611f
🚨 Emergency Vehicle Detected!
```

Unknown tag:

``` text
RFID Tag Scanned: xxxxxxxx
🔹 Unknown RFID Tag.
```

------------------------------------------------------------------------

# 🔌 74HC595 Shift Register

Two 74HC595 shift registers are used to control the traffic LEDs while
reducing the number of ESP32 GPIO pins required.

The ESP32 sends two bytes:

``` cpp
shiftOut(dataPin, clockPin, MSBFIRST, shiftReg2);
shiftOut(dataPin, clockPin, MSBFIRST, shiftReg1);
```

The latch sequence is:

``` cpp
digitalWrite(latchPin, LOW);

shiftOut(...);
shiftOut(...);

digitalWrite(latchPin, HIGH);
```

This allows the system to update the traffic-light output pattern using
only three ESP32 control pins:

``` text
GPIO 14 → DATA
GPIO 12 → LATCH
GPIO 13 → CLOCK
```

------------------------------------------------------------------------

# 🚀 Installation and Setup

## Step 1 --- Install Arduino IDE

Install Arduino IDE on your computer.

## Step 2 --- Configure ESP32

Install the ESP32 board package.

Select the appropriate ESP32 board from:

``` text
Tools → Board
```

## Step 3 --- Install Libraries

Install:

``` text
MFRC522
Blynk
```

`SPI.h` and `WiFi.h` are provided with the ESP32/Arduino environment.

## Step 4 --- Connect Hardware

Connect:

-   Four IR sensors
-   MFRC522 RFID reader
-   Two 74HC595 shift registers
-   12 traffic LEDs
-   Alert LED

according to the pin configuration in this README.

## Step 5 --- Configure Credentials

Enter your:

-   Wi-Fi SSID
-   Wi-Fi password
-   Blynk authentication token

## Step 6 --- Configure Blynk

Create/configure the Blynk template and widgets corresponding to:

``` text
V2 → North
V3 → South
V4 → East
V5 → West
V6 → Emergency/VIP
```

## Step 7 --- Upload

Compile and upload the program to the ESP32.

## Step 8 --- Open Serial Monitor

Set:

``` text
Baud Rate: 115200
```

You can then observe RFID messages and system activity.

------------------------------------------------------------------------

# 🔄 How the Program Works

## Startup

When the ESP32 starts:

``` text
Start Serial communication
        ↓
Connect to Wi-Fi/Blynk
        ↓
Initialize SPI
        ↓
Initialize RFID reader
        ↓
Configure GPIO pins
        ↓
Set traffic lights to ALL RED
```

## Main Loop

The main loop repeatedly performs:

``` text
Blynk.run()
     ↓
Check traffic sensors
     ↓
Update Blynk lane indicators
     ↓
Check RFID
     ↓
Check emergency state
     ↓
Update traffic-light sequence
```

The main loop uses `millis()` for the traffic-state timing.

------------------------------------------------------------------------

# 🧪 Testing

## IR Sensor Tests

  Test              Input               Expected Result
  ----------------- ------------------- -------------------------------------
  North detection   Activate North IR   V2 becomes 255
  South detection   Activate South IR   V3 becomes 255
  East detection    Activate East IR    V4 becomes 255
  West detection    Activate West IR    V5 becomes 255
  No vehicle        Sensor HIGH         Corresponding Blynk value becomes 0

## RFID Tests

  Test             Input           Expected Result
  ---------------- --------------- ----------------------------------
  Emergency RFID   `afac611f`      Emergency message + 10-sec alert
  VIP RFID         `241451a7`      VIP message + 2-sec alert
  Unknown RFID     Any other UID   Unknown RFID message

## Traffic Light Tests

  Test               Expected Result
  ------------------ --------------------------------------------------
  Startup            All-red state is displayed
  Normal operation   Program cycles through configured states
  Traffic detected   Corresponding adaptive timing uses maximum value
  No traffic         Corresponding adaptive timing uses minimum value
  Emergency RFID     Normal sequence is temporarily paused
  VIP RFID           Alert indication is activated

## Blynk Tests

  Test                     Expected Result
  ------------------------ --------------------------------
  Wi-Fi/Blynk connected    Device communicates with Blynk
  North vehicle detected   V2 = 255
  South vehicle detected   V3 = 255
  East vehicle detected    V4 = 255
  West vehicle detected    V5 = 255
  Emergency RFID           V6 = 255
  VIP RFID                 V6 = 128

------------------------------------------------------------------------

# 🖥 Expected Serial Monitor Output

Example normal RFID detection:

``` text
RFID Tag Scanned: afac611f
🚨 Emergency Vehicle Detected!
```

VIP:

``` text
RFID Tag Scanned: 241451a7
⭐ VIP Vehicle Detected!
```

Unknown:

``` text
RFID Tag Scanned: xxxxxxxx
🔹 Unknown RFID Tag.
```

------------------------------------------------------------------------

# 🛠 Troubleshooting

## ESP32 does not connect to Wi-Fi

Check:

-   SSID
-   Wi-Fi password
-   Wi-Fi availability
-   Blynk authentication token
-   ESP32 power supply

## Blynk does not update

Check:

-   Internet connection
-   Blynk credentials
-   Template configuration
-   Correct virtual pins
-   Blynk library installation

## RFID reader does not detect cards

Check:

-   3.3V power
-   GND
-   SDA/SS → GPIO 5
-   RST → GPIO 22
-   SCK → GPIO 18
-   MOSI → GPIO 23
-   MISO → GPIO 19

Also verify that the RFID tag is compatible with the MFRC522 reader.

## IR sensor always reports traffic

Check:

-   Sensor power
-   GND
-   OUT pin
-   Sensor sensitivity adjustment
-   Whether your sensor uses active-LOW output

The current code assumes:

``` text
LOW = detected
HIGH = not detected
```

## Traffic LEDs behave incorrectly

Check:

-   DATA → GPIO 14
-   LATCH → GPIO 12
-   CLOCK → GPIO 13
-   Shift-register power
-   Common ground
-   LED wiring
-   LED resistor connections
-   Order of the two 74HC595 registers

------------------------------------------------------------------------

# ⚠️ Current Implementation Limitations

The following points describe the current source code accurately.

### 1. Emergency lane is not identified

There is only one MFRC522 reader, and the code does not associate an
RFID tag with North, South, East, or West.

Therefore the program cannot determine which direction the emergency
vehicle is approaching from.

### 2. Emergency mode does not change traffic lights

Emergency detection activates the alert mechanism and temporarily
interrupts the normal sequence, but the code does not explicitly
execute:

``` text
Emergency direction → GREEN
Other directions → RED
```

### 3. VIP detection does not change traffic lights

The VIP RFID event only activates the alert mechanism and Blynk
indication.

### 4. IR sensors detect presence, not vehicle count

The system has one digital IR sensor per direction.

It does not calculate:

-   exact vehicle count
-   queue length
-   vehicle speed
-   percentage occupancy

### 5. Blynk is currently used for monitoring

The provided source code sends values to Blynk using `virtualWrite()`.

There are no `BLYNK_WRITE()` handlers in the current code, so remote
Blynk commands are not implemented.

### 6. Alert LED uses blocking delay

The function:

``` cpp
delay(duration);
```

is used for the Emergency/VIP alert.

A 10-second emergency alert therefore blocks execution for that period.

### 7. `DEFAULT_GREEN_TIME` is unused

The constant:

``` cpp
DEFAULT_GREEN_TIME = 5000;
```

is defined but not used by the current timing logic.

------------------------------------------------------------------------

# 🔁 Project Flow

``` text
                    START
                      │
                      ▼
              Initialize ESP32
                      │
                      ▼
             Connect to Blynk/Wi-Fi
                      │
                      ▼
            Initialize RFID + SPI
                      │
                      ▼
              Initialize GPIOs
                      │
                      ▼
                ALL RED
                      │
                      ▼
              ┌───────────────┐
              │    Main Loop  │
              └───────┬───────┘
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
   Read IR Sensors          Read RFID
          │                       │
          ▼                       ▼
   Update V2–V5           Emergency / VIP?
                                  │
                           ┌──────┴──────┐
                           │             │
                          Yes            No
                           │             │
                           ▼             │
                    Alert / Blynk        │
                           │             │
                           └──────┬──────┘
                                  ▼
                         Traffic Sequence
                                  │
                                  ▼
                           Repeat Main Loop
```

------------------------------------------------------------------------

# 🔮 Future Improvements

The project can be extended with:

-   🚑 Direction-specific emergency vehicle detection
-   🚦 Automatic emergency green-light priority
-   ⭐ VIP traffic priority
-   📊 Actual vehicle counting
-   📈 Queue-length estimation
-   📱 Blynk-based manual traffic-light control
-   🔔 Non-blocking emergency alerts
-   🧠 More advanced traffic-density algorithms
-   📷 Camera-based vehicle detection
-   🚗 Multiple RFID readers for direction identification
-   📡 Better network failure handling
-   💾 Historical traffic data storage
-   🤖 Machine-learning-based traffic prediction
-   🚶 Pedestrian crossing detection

------------------------------------------------------------------------

# 🔐 Security Note

**Never commit real credentials to GitHub.**

Do not upload:

``` text
Blynk authentication token
Wi-Fi password
Private API keys
```

Use placeholders in the public source code:

``` cpp
#define BLYNK_AUTH_TOKEN "YOUR_BLYNK_AUTH_TOKEN"

char ssid[] = "YOUR_WIFI_SSID";
char pass[] = "YOUR_WIFI_PASSWORD";
```

If credentials have already been exposed publicly, regenerate them
before publishing the repository.

------------------------------------------------------------------------

# 📁 Suggested Repository Structure

``` text
Smart-Traffic-Management-System/
│
├── README.md
│
├── Traffic_Management_System.ino
│
├── images/
│   ├── circuit-diagram.png
│   ├── hardware-setup.png
│   └── blynk-dashboard.png
│
└── docs/
    └── project-report.pdf
```

------------------------------------------------------------------------

# 📜 License

You may add your preferred open-source license here.

For example:

``` text
MIT License
```

or specify your institution/project-specific licensing terms.

------------------------------------------------------------------------

## 👨‍💻 Project Summary

This project demonstrates an IoT-enabled traffic management prototype
using an **ESP32**, **IR sensors**, **MFRC522 RFID**, **74HC595 shift
registers**, **traffic LEDs**, and **Blynk IoT**.

The current implementation focuses on:

``` text
Vehicle Presence Detection
        +
Adaptive Signal Timing
        +
RFID Identification
        +
Emergency/VIP Alerting
        +
Blynk Monitoring
        +
ESP32-Based Traffic Control
```

The design provides a foundation that can be extended into a more
advanced smart-intersection system with direction-aware emergency
priority, vehicle counting, remote control, and intelligent traffic
prediction.
