# Greenoffice---edge-esp32
Luigi Borghi RM:563096
Levi de Jesus RM:563279
Matheus Brasil RM:561456

  Descrição do Problema
Ambientes corporativos têm consumo elevado de energia e pouca automação. O objetivo é criar um sistema inteligente que otimize o uso de luzes e monitore conforto ambiental.
  Descrição da Solução
O GreenOffice utiliza ESP32 e sensores no Wokwi para monitorar:
temperatura
umidade
luminosidade
presença
E controlar automaticamente a iluminação.

main.ino - codigo completo ESP32
#include <WiFi.h>
#include <HTTPClient.h>
#include "DHTesp.h"

DHTesp dht;
const int DHT_PIN = 15;
const int LDR_PIN = 34;
const int PIR_PIN = 14;
const int LED_PIN = 2;

const char* ssid = "Wokwi-GUEST";
const char* password = "";

void setup() {
  Serial.begin(115200);
  
  dht.setup(DHT_PIN, DHTesp::DHT22);
  
  pinMode(LED_PIN, OUTPUT);
  pinMode(PIR_PIN, INPUT);
  
  WiFi.begin(ssid, password);
  Serial.print("Conectando ao WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConectado!");
}

void loop() {
  TempAndUmidade data = dht.getTempAndUmidade();
  int luz = analogRead(LDR_PIN);
  int presenca = digitalRead(PIR_PIN);

  Serial.println("--- Leitura Sensores ---");
  Serial.println("Temp: " + String(data.temperature));
  Serial.println("Umidade: " + String(data.humidity));
  Serial.println("Luz: " + String(luz));
  Serial.println("Presença: " + String(presenca));

  // Automação da luz
  if (presenca == 1 && luz < 2000) {
    digitalWrite(LED_PIN, HIGH);
  } else {
    digitalWrite(LED_PIN, LOW);
  }

  // Envio via HTTP
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    http.begin("https://jsonplaceholder.typicode.com/posts");
    http.addHeader("Content-Type", "application/json");

    String json = "{\"temp\":" + String(data.temperature) +
                  ",\"umidade\":" + String(data.humidity) +
                  ",\"luz\":" + String(luz) +
                  ",\"presenca\":" + String(presenca) + "}";

    int response = http.POST(json);
    Serial.println("HTTP Response: " + String(response));
    http.end();
  }

  delay(2000);
}

