# Task-3-Smart-Garage-Door-using-lot
Project Summary: Smart garage doors powered by IoT offer a swift solution to the inconvenience of manual operation.

Source Code

#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "YourWiFiSSID";
const char* password = "YourWiFiPassword";
const char* mqtt_server = "YourMQTTBrokerIPAddress";

WiFiClient espClient;
PubSubClient client(espClient);

#define DOOR_PIN 12
#define LASER_PIN 14

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  
  String message = "";
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  Serial.println(message);

  if (strcmp(topic, "garage/door") == 0) {
    if (message.equals("open")) {
      digitalWrite(DOOR_PIN, HIGH); // Open the garage door
      delay(1000); // Adjust the delay according to the door opening time
      digitalWrite(DOOR_PIN, LOW);
    }
  }
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ESP32Client")) {
      Serial.println("connected");
      client.subscribe("garage/door");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(DOOR_PIN, OUTPUT);
  pinMode(LASER_PIN, INPUT);

  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Check laser sensor for obstruction
  if (digitalRead(LASER_PIN) == HIGH) {
    Serial.println("Obstruction detected! Closing the door.");
    digitalWrite(DOOR_PIN, HIGH); // Close the garage door
    delay(1000); // Adjust the delay according to the door closing time
    digitalWrite(DOOR_PIN, LOW);
  }
}


This code sets up an ESP32-based microcontroller to connect to your Wi-Fi network, subscribe to an MQTT topic ('garage/door'), and control a garage door motor connected to pin 12.
