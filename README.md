#include <Arduino.h>
#if defined(ESP32)
#include <WiFi.h>
#elif defined(ESP8266)
#include <ESP8266WiFi.h>
#endif
#include <ESP_Mail_Client.h>

const char *ssid = "CTU Legacy Devices 2.4GHz_forti";  // Replace with your WIFI SSID
const char *pass = "ctu@#2023";         // Replace with your WIFI PASSWORD

void send_event(const char *event);
const char *host = "maker.ifttt.com";
const char *privateKey = "nSXG7TQ73Qnca6sV81_fGXVNSE84uE9YoizAP4RjWkN";

const int smoke_analogPin = A0;
const int buzzer_pin = 13;
const int Greenled_pin = 4; 
const int Redled_pin = 5;
int smokeval;
void setup() {
  Serial.begin(115200);
  // pinMode(sensor,INPUT);
  pinMode(Greenled_pin, OUTPUT);
  pinMode(Redled_pin, OUTPUT);
  pinMode(buzzer_pin, OUTPUT);     
  
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
}

void loop() {
  smokeval = analogRead(smoke_analogPin);
  Serial.println(smokeval);
  //int output = talRead(sensor);
  if (smokeval > 80) {
    Serial.print("FIRE DETECTED!");
    tone(buzzer_pin, 2000); 
    digitalWrite(Greenled_pin, LOW);
    digitalWrite(Redled_pin, HIGH);
    send_event("alert");
    delay(1000);
  }
  noTone(buzzer_pin);
    digitalWrite(Greenled_pin, HIGH);
    digitalWrite(Redled_pin, LOW);
}

void send_event(const char *event) {
  Serial.print("Connecting to ");
  Serial.println(host);
  WiFiClient client;
  const int httpPort = 80;
  if (!client.connect(host, httpPort)) {
    Serial.println("Connection failed");
    return;
  }
  
  String url = "https://maker.ifttt.com/trigger/Security Alert/json/with/key/nSXG7TQ73Qnca6sV81_fGXVNSE84uE9YoizAP4RjWkN";

  digitalWrite(Redled_pin,HIGH);
  Serial.print("Requesting URL:");
  Serial.println(url);
  client.print(String("GET ") + url + " HTTP/1.1\r\n" + "Host: " + host + "\r\n" + "Connection: close\r\n\r\n");
  while (client.connected()) {
    if (client.available()) {
      String line = client.readStringUntil('\r');
      Serial.print(line);
    } else {
      delay(500);
    };
  }
  Serial.println();
  Serial.println("closing connection");
  client.stop();
}
