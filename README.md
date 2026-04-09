# ESP8266 IoT Sensor Dashboard 🌡️

Real-time temperature and humidity monitoring using **ESP8266 (NodeMCU ESP-12E)**, **DHT11 sensor**, **Azure IoT Hub**, **Flask**, and **Grafana**.

---

## 📸 Dashboard Preview

<p align="center">
  <img src="images/dashboard.png" width="700"/>
</p>

---

## 🏗️ Architecture

```
DHT11 Sensor
     ↓
ESP8266 (NodeMCU ESP-12E)
     ↓              ↓
  MQTT            HTTP POST
     ↓              ↓
Azure IoT Hub   Flask Server (PC)
                     ↓
               Grafana Dashboard
               ├── Temperature Chart
               ├── Humidity Chart
               ├── Current Temp Card
               └── Current Humidity Card
```

---

## 🧰 Components Used

| Component | Details |
|---|---|
| Microcontroller | ESP8266 NodeMCU ESP-12E |
| Sensor | DHT11 Temperature & Humidity |
| Cloud | Microsoft Azure IoT Hub (Free F1 tier) |
| Visualization | Grafana (Local) |
| Backend Server | Python Flask |
| Protocol | MQTT (Azure) + HTTP (Flask) |

---

## 📁 Project Structure

```
your-repo/
├── sketch/
│   ├── sketch.ino              # Main ESP8266 Arduino sketch
│   ├── secrets.h               # ❌ Not pushed (gitignored)
│   └── secrets.h.example       # ✅ Template for credentials
├── server/
│   └── sensor_server.py        # Flask server for Grafana
├── images/
│   ├── dashboard.png           # Grafana dashboard screenshot
│   ├── wiring.jpg              # Circuit wiring photo
│   └── architecture.png        # Architecture diagram
├── .gitignore
└── README.md
```

---

## ⚙️ Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/esp8266-iot-dashboard.git
cd esp8266-iot-dashboard
```

### 2. Configure Credentials

```bash
cp sketch/secrets.h.example sketch/secrets.h
```

Open `secrets.h` and fill in your values:

```cpp
#define WIFI_SSID       "your_wifi_name"
#define WIFI_PASSWORD   "your_wifi_password"
#define SAS_TOKEN       "SharedAccessSignature sr=..."
#define FLASK_URL       "http://YOUR_PC_IP:5000/data"
```

### 3. Azure IoT Hub Setup

1. Go to [portal.azure.com](https://portal.azure.com)
2. Create an **IoT Hub** (Free F1 tier)
3. Register a device: `esp32-sensor-01`
4. Copy the **Primary Connection String**
5. Generate SAS token:

```bash
az iot hub generate-sas-token \
  --hub-name your-hub-name \
  --device-id esp32-sensor-01 \
  --duration 86400
```

6. Paste the token into `secrets.h`

### 4. Install Arduino Libraries

Open Arduino IDE → **Sketch → Include Library → Manage Libraries** and install:

| Library | Author |
|---|---|
| ESP8266WiFi | ESP8266 Community |
| PubSubClient | Nick O'Leary |
| ArduinoJson | Benoit Blanchon |
| DHT sensor library | Adafruit |
| Adafruit Unified Sensor | Adafruit |

### 5. Upload Sketch to ESP8266

```
Arduino IDE
→ Tools → Board → NodeMCU 1.0 (ESP-12E Module)
→ Tools → Port → your COM port
→ Upload
```

Open Serial Monitor at **115200 baud** — you should see:
```
Connecting to WiFi..... Connected!
Connecting to IoT Hub... Connected!
Flask POST → HTTP 200
Temp: 34.4C  Hum: 32.7%
```

### 6. Run Flask Server

```bash
cd server
pip install flask flask-cors
python sensor_server.py
```

Server runs at `http://localhost:5000`

Test in browser:
```
http://localhost:5000/data    → all readings (JSON array)
http://localhost:5000/latest  → latest reading only
```

### 7. Setup Grafana Dashboard

1. Download and install [Grafana](https://grafana.com/grafana/download)
2. Open `http://localhost:3000` → login: `admin/admin`
3. Install **Infinity** plugin:
   ```
   Administration → Plugins → Search "Infinity" → Install
   ```
4. Add Infinity as a Data Source
5. Create a new Dashboard with 4 panels:

| Panel | Type | URL | Field |
|---|---|---|---|
| Temperature Chart | Time series | `/data` | temperature |
| Humidity Chart | Time series | `/data` | humidity |
| Current Temp | Stat | `/latest` | temperature |
| Current Humidity | Stat | `/latest` | humidity |

6. Set auto-refresh to **5s**

---

## 🔌 Wiring Diagram

```
DHT11 Pin 1 (VCC)  → NodeMCU 3.3V
DHT11 Pin 2 (DATA) → NodeMCU D2 (GPIO4) + 10kΩ resistor to 3.3V
DHT11 Pin 3        → Not connected
DHT11 Pin 4 (GND)  → NodeMCU GND
```

<p align="center">
  <img src="images/wiring.jpg" width="500"/>
</p>

---

## 📊 Dashboard Screenshots

<p align="center">
  <img src="images/temp_panel.png" width="45%"/>
  <img src="images/humidity_panel.png" width="45%"/>
</p>

---

## 🔒 Security Notes

- Never push `secrets.h` to GitHub
- Regenerate your Azure SAS token regularly (expires every 24h by default)
- Use X.509 certificates instead of SAS tokens for production deployments
- The `.gitignore` file already excludes all sensitive files

---

## 🛠️ Troubleshooting

| Problem | Fix |
|---|---|
| DHT11 read failed | Check wiring, add 10kΩ pull-up resistor between DATA and 3.3V |
| MQTT won't connect | Regenerate SAS token with `az iot hub generate-sas-token` |
| Flask POST → HTTP 0 | Make sure Flask server is running and PC IP is correct |
| Grafana shows no data | Check Flask URL in Infinity datasource, verify `/data` returns JSON |
| WiFi not connecting | Double check SSID and password in `secrets.h` |

---

## 📦 Dependencies

### Arduino
- ESP8266 Board Package `3.x`
- PubSubClient `2.8`
- ArduinoJson `6.x`
- DHT sensor library `1.4.x`

### Python
- Flask `3.x`
- flask-cors `6.x`

---

## 🚀 Future Improvements

- [ ] Add OLED display for local readings
- [ ] Azure Stream Analytics for data processing
- [ ] Email/SMS alerts when temperature exceeds threshold
- [ ] Historical data storage in Azure Blob Storage
- [ ] OTA (Over The Air) firmware updates
- [ ] Add more sensors (soil moisture, LDR light sensor)

---

## 👩‍💻 Author

**Likhitha**  


## 📄 License

This project is open source and available under the [MIT License](LICENSE).
