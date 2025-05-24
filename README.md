#include <WiFi.h>
#include <PubSubClient.h>

// WiFi credentials
const char* ssid = "Bruh";
const char* password = "7a4ee42005";

// MQTT broker
const char* mqtt_server = "102.46.132.3";
const int mqtt_port = 1883;
const char* mqtt_topic_ldr = "esp32/bruh/ldr";
const char* mqtt_topic_status = "esp32/bruh/status";

WiFiClient espClient;
PubSubClient mqttClient(espClient);

// Pins
const int ldrPin = 34;        // Analog LDR pin
const int ledPin = 2; // LED PWM pin
const int ledPin2=5;
const int ledPin3=4;
const int bulbLdrPin = 27;    // Digital LDR for bulb check
const int bulbLdrPin2 = 14;
const int bulbLdrPin3 = 13;

unsigned long lastStatusTime = 0;
const unsigned long statusInterval = 5000;

void connectToWiFi() {
  Serial.printf("Connecting to: %s ", ssid);
  WiFi.begin(ssid, password);
  unsigned long start = millis();
  while (WiFi.status() != WL_CONNECTED && millis() - start < 20000) {
    delay(500);
    Serial.print(".");
  }
  Serial.println(WiFi.status() == WL_CONNECTED ? "\nConnected!" : "\nFailed to connect!");
}

void connectToMQTT() {
  while (!mqttClient.connected()) {
    Serial.print("Connecting to MQTT...");
    String clientId = "ESP32Client-" + String(random(0xffff), HEX);
    if (mqttClient.connect(clientId.c_str())) {
      Serial.println("connected!");
      mqttClient.publish(mqtt_topic_ldr, "ESP32 ready (simple LDR + bulb check)");
    } else {
      Serial.print("Failed, rc=");
      Serial.print(mqttClient.state());
      Serial.println(". Retrying in 5s...");
      delay(5000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);
  pinMode(ledPin3, OUTPUT);
  pinMode(ledPin2, OUTPUT);
  pinMode(bulbLdrPin, INPUT);
  pinMode(bulbLdrPin2, INPUT);
  pinMode(bulbLdrPin3, INPUT);
  analogReadResolution(12); // 12-bit ADC for 0-4095 range

  connectToWiFi();
  mqttClient.setServer(mqtt_server, mqtt_port);
}

void loop() {
  if (WiFi.status() != WL_CONNECTED) {
    connectToWiFi();
  }

  if (!mqttClient.connected()) {
    connectToMQTT();
  }

  mqttClient.loop();

  // LDR brightness control
  int ldrValue = analogRead(ldrPin);
  int brightness = map(ldrValue, 100, 4000, 0, 255); // Inverse
  brightness = constrain(brightness, 0, 255);
  analogWrite(ledPin, brightness);
  analogWrite(ledPin3, brightness);
  analogWrite(ledPin2, brightness);

  // Publish brightness to MQTT
  char msg[64];
  snprintf(msg, sizeof(msg), "LDR Value: %d -> Brightness: %d", ldrValue, brightness);
  //mqttClient.publish(mqtt_topic_ldr, msg);

  // Bulb status check every 5 seconds
  if (millis() - lastStatusTime >= statusInterval) {
    lastStatusTime = millis();
    int bulbState = digitalRead(bulbLdrPin);
    int bulbState2 = digitalRead(bulbLdrPin2);
    int bulbState3 = digitalRead(bulbLdrPin3);
    if (brightness >= 100 && bulbState == HIGH) {
      mqttClient.publish(mqtt_topic_status, "Lamp 1 broken");
      Serial.println("Bulb is broken");
    } else {
      mqttClient.publish(mqtt_topic_status, "Bulb is working");
      Serial.println("Bulb is working");
    }
    if (brightness >= 100 && bulbState2 == HIGH) {
      mqttClient.publish(mqtt_topic_status, "Lamp 2 broken");
      Serial.println("Bulb raqam 2 is broken");
    } else {
      mqttClient.publish(mqtt_topic_status, "Bulb 2 is working");
      Serial.println("Bulb raqam 2 is working");
    }
    if (brightness >= 100 && bulbState3 == HIGH) {
      mqttClient.publish(mqtt_topic_status, "Lamp 3 broken");
      Serial.println("Bulb raqam 3 is broken");
    } else {
      mqttClient.publish(mqtt_topic_status, "Bulb 3 is working");
      Serial.println("Bulb raqam 3 is working");
    }
  }

  delay(50);
}
