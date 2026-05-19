# portable-env-monitor
📌 Problem Statement
> \*\*To design and implement a portable, battery-powered environmental monitoring device that measures temperature, humidity, and dew point in real-time — displayed on an OLED screen and accessible via a live Wi-Fi web dashboard.\*\*
---
📖 Introduction
This project is a self-contained, portable climate monitoring system built around the NodeMCU ESP8266 microcontroller. It combines environmental sensing, real-time clock, OLED display, and an onboard web server — all powered by a rechargeable 18650 lithium-ion battery pack with Type-C charging.
The device is designed for field use, labs, server rooms, or any environment where real-time humidity and temperature tracking matters. The dew point calculation adds a critical safety metric — especially useful for condensation monitoring in electronics and HVAC systems.
What makes it special?
📡 Wi-Fi Web Dashboard — View live readings from any browser on the same network
🔋 Truly Portable — 2S 18650 battery with Type-C charging; no wall socket needed
⏰ Real-Time Clock — DS3231 RTC maintains accurate time even after power-off
🖥️ OLED Display — SH1106 128×64 shows all data at a glance
⚠️ Smart Alerts — Web dashboard shows sensor error banners and over-temperature warnings
---
🧩 Components & Hardware
Main Controller
Component	Model	Description
Microcontroller	NodeMCU ESP8266 (ESP-12E)	Wi-Fi enabled, runs Arduino firmware
Sensors
Component	Model	Description
Temp & Humidity	DHT11	Measures °C and %RH
Real Time Clock	DS3231	Maintains date/time via I2C, battery-backed
Display
Component	Model	Description
OLED	SH1106 128×64	4-wire SPI, ultra-low power
Power System
Component	Model	Description
Battery	2 × 18650 Li-ion (3.7V)	7.4V in series (2S pack)
Charging Module	2S Type-C Boost Module	Charges 2S pack, 8.4V output
Buck Converter	LM2596S	Steps down 7.4V → 5V for ESP8266
Miscellaneous
Custom PCB (designed in KiCad/EasyEDA)
Connecting wires
Pull-up resistors (as needed)
---
🔌 Complete Wiring Guide
1. DHT11 → ESP8266
DHT11 Pin	ESP8266 Pin
VCC	3.3V
GND	GND
DATA	D5
2. RTC DS3231 → ESP8266 (I2C)
RTC Pin	ESP8266 Pin
SDA	D2
SCL	D1
VCC	3.3V
GND	GND
3. OLED SH1106 → ESP8266 (SPI)
OLED Pin	ESP8266 Pin
CLK	D4
MOSI	D3
CS	D8
DC	D0
RST	D7
VCC	3.3V
GND	GND
4. Battery → Charging Module (2S)
Battery	Charging Module
Pack +	B+
Pack Mid	BM
Pack –	B–
5. Charging Module → LM2596 Buck Converter
Charging Module	LM2596
B+ (OUT)	IN+
B– (OUT)	IN–
6. LM2596 → ESP8266
LM2596	ESP8266
OUT+	VIN
OUT–	GND
> ⚠️ \*\*Calibrate LM2596 output to exactly 5V\*\* before connecting to ESP8266!
7. Charging Input
Type-C cable → 2S Charging Module
---
🔁 Power Flow
```
Type-C Charger (5V)
        │
        ▼
2S Charging Module
(Charges 2 × 18650 in series → 8.4V)
        │
        ▼
Battery Pack (7.4V nominal)
        │
        ▼
LM2596 Buck Converter
(Steps down to 5V regulated)
        │
        ▼
ESP8266 NodeMCU (VIN)
        │
        ├──► 3.3V (onboard regulator) ──► DHT11, RTC, OLED
        └──► GND
```
---
🧠 How the System Works
Firmware Architecture
```
setup()
  │
  ├── Initialize DHT11, RTC, OLED, Wire (I2C)
  ├── Display "WELCOME" on OLED (3 sec)
  ├── Attempt Wi-Fi connection (60 sec timeout)
  │     ├── SUCCESS → Start web server, show IP on OLED
  │     └── TIMEOUT → Run in offline/standalone mode
  │
loop()
  │
  ├── server.handleClient()        ← Serve live web dashboard
  ├── Read DHT11 every 2000ms
  │     ├── Valid → Update h, t, dew
  │     └── NaN → Set sensorError = true
  ├── Calculate Dew Point → t - ((100 - h) / 5.0)
  ├── Read RTC → Format 12-hour clock string
  └── Update OLED display
```
Dew Point Formula Used
```
Dew Point (°C) = Temperature - ((100 - Humidity) / 5)
```
This is the simplified Magnus approximation, accurate for typical ambient conditions.
---
🖥️ OLED Display Layout
```
┌──────────────────────────────┐
│ T:25.5C  H:60.0%             │
│ Dew point: 17.3C             │
│ Time: 02:30:15 PM            │
│ Date: 19/5/2026              │
└──────────────────────────────┘
```
On sensor error:
```
┌──────────────────────────────┐
│ !!! SENSOR ERROR !!!         │
│ Check DHT11 Wiring           │
│ Time: 02:30:15 PM            │
│ Date: 19/5/2026              │
└──────────────────────────────┘
```
---
🌐 Web Dashboard
When connected to Wi-Fi, navigate to the ESP8266's IP address in any browser.
Features
Auto-refreshes every 5 seconds
Shows Temperature (°C), Humidity (%), and Dew Point (°C) as live cards
Red warning banner if temperature exceeds 40°C
Critical error banner if DHT11 is disconnected
Dark-themed, responsive UI — works on mobile and desktop
Dashboard Preview (Dark Theme)
```
┌─────────────────────────────────────────────┐
│         🌡 Dew Point Monitor                │
│         Live Sensor Dashboard               │
│                                             │
│  ┌──────────┐ ┌──────────┐ ┌────────────┐  │
│  │  Temp    │ │  Hum     │ │  Dew Point │  │
│  │  25.5°C  │ │  60.0%   │ │  17.3°C    │  │
│  └──────────┘ └──────────┘ └────────────┘  │
└─────────────────────────────────────────────┘
```
---
📐 PCB Design
Custom PCB designed with all components footprinted and routed:
Layer	Colour	Purpose
Top copper	Red	Signal + Power traces
Bottom copper	Blue	Ground plane + return paths
Silkscreen	White	Component labels
Components on PCB:
ESP8266 NodeMCU socket headers
DHT11 3-pin header
OLED 7-pin SPI header
DS3231 RTC 6-pin I2C header
LM2596 Buck Converter footprint
18650 battery holder pads (BT)
Type-C charging module connector
Power switch (J1, J2, J3 breakouts)
---
📚 Libraries Used
Library	Purpose
`Wire.h`	I2C communication (RTC)
`DHT.h`	DHT11 sensor reading
`RTClib.h`	DS3231 RTC interface
`U8g2lib.h`	SH1106 OLED display driver
`ESP8266WiFi.h`	Wi-Fi connection
`ESP8266WebServer.h`	HTTP web server
Install via Arduino IDE → Library Manager or `platformio.ini`.
---
⚙️ Setup & Flash Instructions
Prerequisites
Arduino IDE 1.8+ or PlatformIO
ESP8266 board package installed
Board Manager URL: `http://arduino.esp8266.com/stable/package\_esp8266com\_index.json`
All libraries listed above installed
Steps
Clone the repository
```bash
   git clone https://github.com/YOUR\_USERNAME/portable-env-monitor.git
   cd portable-env-monitor
   ```
Open `main.ino` in Arduino IDE
Configure Wi-Fi credentials in `main.ino`:
```cpp
   const char\* ssid = "YOUR\_WIFI\_SSID";
   const char\* password = "YOUR\_WIFI\_PASSWORD";
   ```
Set RTC time (first flash only):
```cpp
   bool setTime = true;   // Change to true
   // Modify this line:
   rtc.adjust(DateTime(2026, 5, 19, 10, 25, 0)); // YYYY, MM, DD, HH, MM, SS
   ```
After first flash, set `setTime = false` and reflash.
Select board: `Tools → Board → NodeMCU 1.0 (ESP-12E Module)`
Select port and click Upload
Open Serial Monitor at 115200 baud to see the IP address
Navigate to the IP in a browser on the same Wi-Fi network
---
🧪 Testing & Results
Test	Expected	Result
DHT11 reads temperature	Value in °C	✅
DHT11 reads humidity	Value in %RH	✅
Dew point calculation	Temp - ((100-Hum)/5)	✅
RTC keeps time	12-hr format with AM/PM	✅
OLED shows all 4 lines	T, Dew, Time, Date	✅
Wi-Fi connects	IP shown on OLED	✅
Web dashboard loads	Live cards refresh every 5s	✅
Sensor error detection	Banner shown if DHT disconnected	✅
Temp warning alert	Red banner if T > 40°C	✅
Offline mode fallback	Works with no Wi-Fi	✅
---
🏭 Applications
Domain	Use Case
🌾 Agriculture	Crop storage humidity & temp monitoring
🖥️ Server Rooms	Condensation & overheating alerts
🏭 Industrial	HVAC system dew point monitoring
🔬 Labs	Controlled environment tracking
🏠 Smart Home	DIY weather station
🏕️ Field Use	Portable battery-powered data logging
---
⚠️ Important Design Notes
2S battery (7.4V) used for longer backup time vs single cell
LM2596 provides stable 5V even as battery voltage drops
DS3231 RTC holds time via its onboard coin cell — no drift after power-off
OLED SH1106 chosen over LCD for low power draw (~20mA)
DHT11 sampling limited to every 2 seconds (hardware minimum)
Web server and sensor loop run concurrently via `server.handleClient()` in the main loop
---
📁 Repository Structure
```
portable-env-monitor/
│
├── README.md                        ← You are here
├── src/
│   └── main.ino                     ← Main Arduino firmware
├── hardware/
│   ├── schematic.jpg                ← KiCad/EasyEDA schematic
│   ├── pcb\_layout\_copper.jpg        ← PCB copper layer view
│   ├── pcb\_layout\_3d.jpg            ← PCB 3D render
│   └── wiring\_oled\_table.jpg        ← OLED pin connection table
├── docs/
│   ├── wiring\_guide.md              ← Full wiring reference
│   └── power\_flow.md                ← Power system explanation
├── demo/
│   └── demo\_video.mp4               ← Working demo
└── LICENSE
```
---
🛠️ Tools Used
Arduino IDE — Firmware development & flashing
KiCad / EasyEDA — PCB schematic & layout design
Serial Monitor — Debugging & IP address readout
Browser — Web dashboard access
---
📄 License
This project is submitted for academic purposes under Bharati Vidyapeeth (Deemed to Be University), College of Engineering, Pune. All rights reserved by the authors.
---
<div align="center">
  Built with 🔋 + 📡 for <strong>Portable Environmental Sensing</strong>
  <br>
  <em>Group 1 | Div 2 | E\&TC | BVUCOE Pune | 2024–25</em>
</div>📌 Problem Statement
> \*\*To design and implement a portable, battery-powered environmental monitoring device that measures temperature, humidity, and dew point in real-time — displayed on an OLED screen and accessible via a live Wi-Fi web dashboard.\*\*
---
📖 Introduction
This project is a self-contained, portable climate monitoring system built around the NodeMCU ESP8266 microcontroller. It combines environmental sensing, real-time clock, OLED display, and an onboard web server — all powered by a rechargeable 18650 lithium-ion battery pack with Type-C charging.
The device is designed for field use, labs, server rooms, or any environment where real-time humidity and temperature tracking matters. The dew point calculation adds a critical safety metric — especially useful for condensation monitoring in electronics and HVAC systems.
What makes it special?
📡 Wi-Fi Web Dashboard — View live readings from any browser on the same network
🔋 Truly Portable — 2S 18650 battery with Type-C charging; no wall socket needed
⏰ Real-Time Clock — DS3231 RTC maintains accurate time even after power-off
🖥️ OLED Display — SH1106 128×64 shows all data at a glance
⚠️ Smart Alerts — Web dashboard shows sensor error banners and over-temperature warnings
---
🧩 Components & Hardware
Main Controller
Component	Model	Description
Microcontroller	NodeMCU ESP8266 (ESP-12E)	Wi-Fi enabled, runs Arduino firmware
Sensors
Component	Model	Description
Temp & Humidity	DHT11	Measures °C and %RH
Real Time Clock	DS3231	Maintains date/time via I2C, battery-backed
Display
Component	Model	Description
OLED	SH1106 128×64	4-wire SPI, ultra-low power
Power System
Component	Model	Description
Battery	2 × 18650 Li-ion (3.7V)	7.4V in series (2S pack)
Charging Module	2S Type-C Boost Module	Charges 2S pack, 8.4V output
Buck Converter	LM2596S	Steps down 7.4V → 5V for ESP8266
Miscellaneous
Custom PCB (designed in KiCad/EasyEDA)
Connecting wires
Pull-up resistors (as needed)
---
🔌 Complete Wiring Guide
1. DHT11 → ESP8266
DHT11 Pin	ESP8266 Pin
VCC	3.3V
GND	GND
DATA	D5
2. RTC DS3231 → ESP8266 (I2C)
RTC Pin	ESP8266 Pin
SDA	D2
SCL	D1
VCC	3.3V
GND	GND
3. OLED SH1106 → ESP8266 (SPI)
OLED Pin	ESP8266 Pin
CLK	D4
MOSI	D3
CS	D8
DC	D0
RST	D7
VCC	3.3V
GND	GND
4. Battery → Charging Module (2S)
Battery	Charging Module
Pack +	B+
Pack Mid	BM
Pack –	B–
5. Charging Module → LM2596 Buck Converter
Charging Module	LM2596
B+ (OUT)	IN+
B– (OUT)	IN–
6. LM2596 → ESP8266
LM2596	ESP8266
OUT+	VIN
OUT–	GND
> ⚠️ \*\*Calibrate LM2596 output to exactly 5V\*\* before connecting to ESP8266!
7. Charging Input
Type-C cable → 2S Charging Module
---
🔁 Power Flow
```
Type-C Charger (5V)
        │
        ▼
2S Charging Module
(Charges 2 × 18650 in series → 8.4V)
        │
        ▼
Battery Pack (7.4V nominal)
        │
        ▼
LM2596 Buck Converter
(Steps down to 5V regulated)
        │
        ▼
ESP8266 NodeMCU (VIN)
        │
        ├──► 3.3V (onboard regulator) ──► DHT11, RTC, OLED
        └──► GND
```
---
🧠 How the System Works
Firmware Architecture
```
setup()
  │
  ├── Initialize DHT11, RTC, OLED, Wire (I2C)
  ├── Display "WELCOME" on OLED (3 sec)
  ├── Attempt Wi-Fi connection (60 sec timeout)
  │     ├── SUCCESS → Start web server, show IP on OLED
  │     └── TIMEOUT → Run in offline/standalone mode
  │
loop()
  │
  ├── server.handleClient()        ← Serve live web dashboard
  ├── Read DHT11 every 2000ms
  │     ├── Valid → Update h, t, dew
  │     └── NaN → Set sensorError = true
  ├── Calculate Dew Point → t - ((100 - h) / 5.0)
  ├── Read RTC → Format 12-hour clock string
  └── Update OLED display
```
Dew Point Formula Used
```
Dew Point (°C) = Temperature - ((100 - Humidity) / 5)
```
This is the simplified Magnus approximation, accurate for typical ambient conditions.
---
🖥️ OLED Display Layout
```
┌──────────────────────────────┐
│ T:25.5C  H:60.0%             │
│ Dew point: 17.3C             │
│ Time: 02:30:15 PM            │
│ Date: 19/5/2026              │
└──────────────────────────────┘
```
On sensor error:
```
┌──────────────────────────────┐
│ !!! SENSOR ERROR !!!         │
│ Check DHT11 Wiring           │
│ Time: 02:30:15 PM            │
│ Date: 19/5/2026              │
└──────────────────────────────┘
```
---
🌐 Web Dashboard
When connected to Wi-Fi, navigate to the ESP8266's IP address in any browser.
Features
Auto-refreshes every 5 seconds
Shows Temperature (°C), Humidity (%), and Dew Point (°C) as live cards
Red warning banner if temperature exceeds 40°C
Critical error banner if DHT11 is disconnected
Dark-themed, responsive UI — works on mobile and desktop
Dashboard Preview (Dark Theme)
```
┌─────────────────────────────────────────────┐
│         🌡 Dew Point Monitor                │
│         Live Sensor Dashboard               │
│                                             │
│  ┌──────────┐ ┌──────────┐ ┌────────────┐  │
│  │  Temp    │ │  Hum     │ │  Dew Point │  │
│  │  25.5°C  │ │  60.0%   │ │  17.3°C    │  │
│  └──────────┘ └──────────┘ └────────────┘  │
└─────────────────────────────────────────────┘
```
---
📐 PCB Design
Custom PCB designed with all components footprinted and routed:
Layer	Colour	Purpose
Top copper	Red	Signal + Power traces
Bottom copper	Blue	Ground plane + return paths
Silkscreen	White	Component labels
Components on PCB:
ESP8266 NodeMCU socket headers
DHT11 3-pin header
OLED 7-pin SPI header
DS3231 RTC 6-pin I2C header
LM2596 Buck Converter footprint
18650 battery holder pads (BT)
Type-C charging module connector
Power switch (J1, J2, J3 breakouts)
---
📚 Libraries Used
Library	Purpose
`Wire.h`	I2C communication (RTC)
`DHT.h`	DHT11 sensor reading
`RTClib.h`	DS3231 RTC interface
`U8g2lib.h`	SH1106 OLED display driver
`ESP8266WiFi.h`	Wi-Fi connection
`ESP8266WebServer.h`	HTTP web server
Install via Arduino IDE → Library Manager or `platformio.ini`.
---
⚙️ Setup & Flash Instructions
Prerequisites
Arduino IDE 1.8+ or PlatformIO
ESP8266 board package installed
Board Manager URL: `http://arduino.esp8266.com/stable/package\_esp8266com\_index.json`
All libraries listed above installed
Steps
Clone the repository
```bash
   git clone https://github.com/YOUR\_USERNAME/portable-env-monitor.git
   cd portable-env-monitor
   ```
Open `main.ino` in Arduino IDE
Configure Wi-Fi credentials in `main.ino`:
```cpp
   const char\* ssid = "YOUR\_WIFI\_SSID";
   const char\* password = "YOUR\_WIFI\_PASSWORD";
   ```
Set RTC time (first flash only):
```cpp
   bool setTime = true;   // Change to true
   // Modify this line:
   rtc.adjust(DateTime(2026, 5, 19, 10, 25, 0)); // YYYY, MM, DD, HH, MM, SS
   ```
After first flash, set `setTime = false` and reflash.
Select board: `Tools → Board → NodeMCU 1.0 (ESP-12E Module)`
Select port and click Upload
Open Serial Monitor at 115200 baud to see the IP address
Navigate to the IP in a browser on the same Wi-Fi network
---
🧪 Testing & Results
Test	Expected	Result
DHT11 reads temperature	Value in °C	✅
DHT11 reads humidity	Value in %RH	✅
Dew point calculation	Temp - ((100-Hum)/5)	✅
RTC keeps time	12-hr format with AM/PM	✅
OLED shows all 4 lines	T, Dew, Time, Date	✅
Wi-Fi connects	IP shown on OLED	✅
Web dashboard loads	Live cards refresh every 5s	✅
Sensor error detection	Banner shown if DHT disconnected	✅
Temp warning alert	Red banner if T > 40°C	✅
Offline mode fallback	Works with no Wi-Fi	✅
---
🏭 Applications
Domain	Use Case
🌾 Agriculture	Crop storage humidity & temp monitoring
🖥️ Server Rooms	Condensation & overheating alerts
🏭 Industrial	HVAC system dew point monitoring
🔬 Labs	Controlled environment tracking
🏠 Smart Home	DIY weather station
🏕️ Field Use	Portable battery-powered data logging
---
⚠️ Important Design Notes
2S battery (7.4V) used for longer backup time vs single cell
LM2596 provides stable 5V even as battery voltage drops
DS3231 RTC holds time via its onboard coin cell — no drift after power-off
OLED SH1106 chosen over LCD for low power draw (~20mA)
DHT11 sampling limited to every 2 seconds (hardware minimum)
Web server and sensor loop run concurrently via `server.handleClient()` in the main loop
---
📁 Repository Structure
```
portable-env-monitor/
│
├── README.md                        ← You are here
├── src/
│   └── main.ino                     ← Main Arduino firmware
├── hardware/
│   ├── schematic.jpg                ← KiCad/EasyEDA schematic
│   ├── pcb\_layout\_copper.jpg        ← PCB copper layer view
│   ├── pcb\_layout\_3d.jpg            ← PCB 3D render
│   └── wiring\_oled\_table.jpg        ← OLED pin connection table
├── docs/
│   ├── wiring\_guide.md              ← Full wiring reference
│   └── power\_flow.md                ← Power system explanation
├── demo/
│   └── demo\_video.mp4               ← Working demo
└── LICENSE
```
---
🛠️ Tools Used
Arduino IDE — Firmware development & flashing
KiCad / EasyEDA — PCB schematic & layout design
Serial Monitor — Debugging & IP address readout
Browser — Web dashboard access

code:
#include <Wire.h>
#include <DHT.h>
#include <RTClib.h>
#include <U8g2lib.h>
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>

// --- WIFI CREDENTIALS ---
const char* ssid = "OnePlus 7 Pro";      
const char* password = "123456789";  

ESP8266WebServer server(80);

#define DHTPIN D5
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);
RTC_DS3231 rtc;

// OLED
U8G2_SH1106_128X64_NONAME_F_4W_SW_SPI u8g2(
  U8G2_R0, D4, D3, D8, D0, D7
);

// 🔥 SET TIME ONLY ONCE
bool setTime = false;

// --- DISPLAY SHIFT ---
const int Y_OFFSET = 12; 

// Global variables
float h = 0.0;
float t = 0.0;
float dew = 0.0;
String timeString = "";
bool sensorError = false; 
bool wifiConnected = false; 

// ==========================================
// --- STUNNING WEB DASHBOARD HTML & CSS ---
// ==========================================
const char index_html[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="refresh" content="5">
    <title>Environmental Monitor</title>
    <style>
        :root {
            --bg: #0b1120;
            --card-bg: #1e293b;
            --text-main: #f8fafc;
            --text-muted: #94a3b8;
            --accent: #06b6d4;
            --danger: #ef4444;
            --danger-bg: #450a0a;
        }
        body {
            background-color: var(--bg);
            color: var(--text-main);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0;
            padding: 30px 15px;
        }
        h1 {
            font-size: 2.5rem;
            margin: 0 0 5px 0;
            color: var(--accent);
            text-shadow: 0 0 15px rgba(6, 182, 212, 0.4);
            text-align: center;
        }
        .subtitle {
            color: var(--text-muted);
            margin-bottom: 40px;
            font-size: 1.1rem;
            letter-spacing: 1px;
            text-transform: uppercase;
        }
        .grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 25px;
            width: 100%;
            max-width: 1000px;
        }
        .card {
            background-color: var(--card-bg);
            border-radius: 20px;
            padding: 30px 20px;
            text-align: center;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            border: 1px solid rgba(255,255,255,0.05);
        }
        .card h2 {
            margin: 0;
            font-size: 1.2rem;
            color: var(--text-muted);
            text-transform: uppercase;
            letter-spacing: 2px;
        }
        .value {
            font-size: 4rem;
            font-weight: 700;
            margin: 15px 0 0 0;
            color: var(--text-main);
        }
        .unit {
            font-size: 1.5rem;
            color: var(--accent);
            margin-left: 5px;
        }
        .warning {
            background-color: var(--danger-bg);
            color: #fca5a5;
            border: 2px solid var(--danger);
            padding: 15px 30px;
            border-radius: 12px;
            margin-bottom: 30px;
            font-weight: bold;
            display: %WARNING_DISPLAY%; 
            text-align: center;
            width: 100%;
            max-width: 935px;
            box-sizing: border-box;
        }
        .error-banner {
            background-color: #7f1d1d;
            color: #fecaca;
            border: 2px solid #ef4444;
            padding: 15px 30px;
            border-radius: 12px;
            margin-bottom: 30px;
            font-weight: bold;
            display: %ERROR_DISPLAY%; 
            text-align: center;
            width: 100%;
            max-width: 935px;
            box-sizing: border-box;
        }
    </style>
</head>
<body>
    <h1>Dew Point Monitor</h1>
    <div class="subtitle">Live Sensor Dashboard</div>
    <div class="error-banner">CRITICAL SENSOR ERROR: DHT11 Disconnected!</div>
    <div class="warning">WARNING: Temperature has exceeded 40&deg;C!</div>
    <div class="grid">
        <div class="card" style="%TEMP_STYLE%">
            <h2>&#127777; Temp</h2> 
            <div class="value">%TEMP%<span class="unit">&deg;C</span></div>
        </div>
        <div class="card">
            <h2>Hum</h2> 
            <div class="value">%HUM%<span class="unit">%</span></div>
        </div>
        <div class="card">
            <h2>Dew Point</h2>
            <div class="value">%DEW%<span class="unit">&deg;C</span></div>
        </div>
    </div>
</body>
</html>
)rawliteral";

void handleRoot() {
  String html = FPSTR(index_html); 
  if (sensorError) {
    html.replace("%ERROR_DISPLAY%", "block");
    html.replace("%WARNING_DISPLAY%", "none"); 
    html.replace("%TEMP_STYLE%", "border-color: #ef4444; opacity: 0.5;");
    html.replace("%TEMP%", "--");
    html.replace("%HUM%", "--");
    html.replace("%DEW%", "--");
  } else {
    html.replace("%ERROR_DISPLAY%", "none");
    html.replace("%TEMP%", String(t, 1));
    html.replace("%HUM%", String(h, 1));
    html.replace("%DEW%", String(dew, 1));
    if (t > 40.0) {
      html.replace("%WARNING_DISPLAY%", "block");
      html.replace("%TEMP_STYLE%", "border-color: #ef4444; box-shadow: 0 0 25px rgba(239,68,68,0.3);");
    } else {
      html.replace("%WARNING_DISPLAY%", "none");
      html.replace("%TEMP_STYLE%", "");
    }
  }
  server.send(200, "text/html", html);
}

void setup() {
  Serial.begin(115200);
  Wire.begin(D2, D1);
  dht.begin();
  u8g2.begin();

  if (!rtc.begin()) {
    while (1);
  }

  if (setTime) {
    rtc.adjust(DateTime(2026, 3, 20, 10, 25, 0));
  }

  u8g2.clearBuffer();
  u8g2.setFont(u8g2_font_6x12_tf);
  u8g2.drawStr(35, 25 + Y_OFFSET, "WELCOME");
  u8g2.sendBuffer();
  delay(3000); 

  WiFi.begin(ssid, password);
  unsigned long startAttemptTime = millis();
  while (WiFi.status() != WL_CONNECTED && millis() - startAttemptTime < 60000) {
    int secondsLeft = 60 - ((millis() - startAttemptTime) / 1000);
    u8g2.clearBuffer();
    u8g2.drawStr(5, 10 + Y_OFFSET, "Connecting Wi-Fi...");
    u8g2.setCursor(5, 30 + Y_OFFSET);
    u8g2.print("Timeout in: ");
    u8g2.print(secondsLeft);
    u8g2.print("s");
    u8g2.sendBuffer();
    delay(200);
  }

  if (WiFi.status() == WL_CONNECTED) {
    wifiConnected = true; 
    server.on("/", handleRoot);
    server.begin();
    u8g2.clearBuffer();
    u8g2.drawStr(10, 15 + Y_OFFSET, "Connected!");
    u8g2.drawStr(10, 30 + Y_OFFSET, "Browser IP:");
    u8g2.setCursor(10, 45 + Y_OFFSET);
    u8g2.print(WiFi.localIP().toString());
    u8g2.sendBuffer();
    delay(10000); 
  } else {
    wifiConnected = false; 
    u8g2.clearBuffer();
    u8g2.drawStr(10, 20 + Y_OFFSET, "Wi-Fi Timeout!");
    u8g2.drawStr(10, 35 + Y_OFFSET, "Running Offline...");
    u8g2.sendBuffer();
    delay(3000); 
  }
}

void loop() {
  if (wifiConnected) {
    server.handleClient(); 
  }

  static unsigned long lastDHT = 0;
  if (millis() - lastDHT > 2000) {
    lastDHT = millis();
    float newH = dht.readHumidity();
    float newT = dht.readTemperature();
    if (isnan(newH) || isnan(newT)) {
      sensorError = true; 
    } else {
      sensorError = false; 
      h = newH;
      t = newT;
      dew = t - ((100 - h) / 5.0); 
    }
  }

  // --- 12-HOUR CLOCK LOGIC ---
  DateTime now = rtc.now();
  int hour12 = now.hour();
  String period = "AM";

  if (hour12 >= 12) {
    period = "PM";
  }
  if (hour12 == 0) {
    hour12 = 12;
  } else if (hour12 > 12) {
    hour12 -= 12;
  }

  char timeBuf[20];
  sprintf(timeBuf, "%02d:%02d:%02d %s", hour12, now.minute(), now.second(), period.c_str());
  timeString = String(timeBuf);

  // --- OLED DISPLAY ---
  u8g2.clearBuffer();
  u8g2.setFont(u8g2_font_6x12_tf);

  if (sensorError) {
    u8g2.setCursor(0, 15 + Y_OFFSET);
    u8g2.print("!!! SENSOR ERROR !!!");
    u8g2.setCursor(0, 30 + Y_OFFSET);
    u8g2.print("Check DHT11 Wiring");
  } else {
    u8g2.setCursor(0, 10 + Y_OFFSET);
    u8g2.print("T:"); u8g2.print(t, 1); u8g2.print("C  H:"); u8g2.print(h, 1); u8g2.print("%");
    u8g2.setCursor(0, 22 + Y_OFFSET);
    u8g2.print("Dew point: "); u8g2.print(dew, 1); u8g2.print("C");
  }

  u8g2.setCursor(0, 34 + Y_OFFSET);
  u8g2.print("Time: "); u8g2.print(timeString);

  u8g2.setCursor(0, 46 + Y_OFFSET);
  u8g2.print("Date: ");
  u8g2.print(now.day()); u8g2.print("/"); u8g2.print(now.month()); u8g2.print("/"); u8g2.print(now.year());

  u8g2.sendBuffer();
  delay(10); 
}

---
📄 License
This project is submitted for academic purposes under Bharati Vidyapeeth (Deemed to Be University), College of Engineering, Pune. All rights reserved by the authors.
---
<div align="center">
  Built with 🔋 + 📡 for <strong>Portable Environmental Sensing</strong>
  <br>
  <em>Group 1 | Div 2 | E\&TC | BVUCOE Pune | 2024–25</em>
</div>
