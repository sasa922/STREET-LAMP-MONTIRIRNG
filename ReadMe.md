Smart Street Lamp Monitoring System

This project implements a complete IoT-based street lamp monitoring system using ESP32 microcontrollers, MQTT communication (via Mosquitto), a Flask-based web server with WAN accessibility, and a responsive HTML/JavaScript interface. The system provides real-time status updates of multiple street lamps, user authentication, reset functionalities, and is secured with basic encryption and web authentication mechanisms.

Table of Contents

1. Overview

2. System Components

3. Architecture

4. Hardware Setup

5. MQTT Configuration

6. Flask Web Server

7. Web Interface

8. Security and Authentication

9. WAN Deployment and Port Forwarding

10. ESP32 Firmware Overview

11. Future Improvements

12. References

13. Overview

The system provides centralized control and monitoring of three simulated street lamps. The ESP32 devices detect lamp status and publish messages to an MQTT broker. A Flask server subscribes to these messages and updates a live dashboard accessible via web interface. Basic security is enforced via HTTP authentication and hashed passwords.

2. System Components

| Layer           | Technology                   |
| --------------- | ---------------------------- |
| Microcontroller | ESP32 (with Wi-Fi)           |
| Communication   | MQTT (Mosquitto Broker)      |
| Server Backend  | Flask (Python)               |
| Frontend        | HTML, CSS, JavaScript        |
| Authentication  | HTTP Basic Auth with Hashing |
| Hosting         | Localhost with WAN exposure  |

3. Architecture

```
[ESP32 Devices] ──MQTT──> [Mosquitto Broker] ──MQTT──> [Flask Server] ──HTTP──> [Web Browser]
                                                             ↑
                                                   Port Forwarding
                                                             ↑
                                                  Public Internet Access
```

4. Hardware Setup

* ESP32 Development Board
* Three simulated street lamps (LEDs or small bulbs)
* Light sensors (e.g., LDRs) or simulated triggers
* Optional push buttons for simulating faults
* Power supply and breadboard wiring

5. MQTT Configuration

Installation (Linux):

```bash
sudo apt install mosquitto mosquitto-clients
```

Configuration File Example:

```
listener 1883
allow_anonymous true
```

Broker Startup:

```bash
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
```

MQTT Topics Used:

* `esp32/bruh/status`
* `esp32/bruh/control`

6. Flask Web Server

Requirements:

```bash
pip install flask flask-httpauth paho-mqtt
```

Features:

* Subscribes to MQTT topic
* Updates state and dashboard
* Auto-refreshing web interface
* Manual reset capabilities

Execution:

```bash
python3 app.py
```

7. Web Interface

* Displays lamp status with color indicators
* Includes reset buttons
* Shows timestamped log messages
* Refreshes every 5 seconds

JavaScript snippet:

```javascript
setInterval(() => {
    window.location.reload();
}, 5000);
```

8. Security and Authentication

* HTTP Basic Auth with hashed passwords
* Flask session key generated with `secrets.token_hex`
* Recommend switching to MQTT over TLS (port 8883) for production

9. WAN Deployment and Port Forwarding

Steps:

1. Enable port forwarding in your router
2. Forward port 5000 to internal IP
3. Access via `http://<public-ip>:5000`

Security Notes:

* Use strong credentials
* Deploy HTTPS via reverse proxy
* Restrict IP access if possible

10. ESP32 Firmware Overview

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "YOUR_SSID";
const char* password = "YOUR_PASSWORD";
const char* mqtt_server = "BROKER_IP";

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  client.setServer(mqtt_server, 1883);
}

void loop() {
  if (!client.connected()) {
    // Reconnect logic
  }
  client.loop();

  client.publish("esp32/bruh/status", "Lamp 1 working");
  delay(5000);
}
```

11. Future Improvements

* Store logs in database (SQLite, PostgreSQL)
* Deploy HTTPS and TLS for MQTT
* Implement role-based access control
* Add analytics and dashboards
* Containerize with Docker

12. References

* Flask: [https://flask.palletsprojects.com/](https://flask.palletsprojects.com/)
* MQTT: [https://mqtt.org/](https://mqtt.org/)
* Mosquitto: [https://mosquitto.org/](https://mosquitto.org/)
* ESP32: [https://randomnerdtutorials.com/](https://randomnerdtutorials.com/)
* Flask-HTTPAuth: [https://flask-httpauth.readthedocs.io/](https://flask-httpauth.readthedocs.io/)
