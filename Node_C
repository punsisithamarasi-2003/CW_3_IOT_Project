#include <SPI.h>
#include <LoRa.h>

/* -------- PIN DEFINITIONS -------- */
#define MOISTURE_PIN A0
#define RELAY_PIN    7
#define FLOW_PIN     2

#define LORA_SS   10
#define LORA_RST  9
#define LORA_DIO0 3

/* -------- PARAMETERS -------- */
int moistureThreshold = 600;

/* -------- FLOW VARIABLES -------- */
volatile unsigned long flowPulseCount = 0;
float flowRate = 0.0;
float totalLiters = 0.0;

/* -------- FLOW SENSOR ISR -------- */
void flowPulse() {
  flowPulseCount++;
}

void setup() {
  Serial.begin(9600);

  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, HIGH);   // Relay OFF (active LOW)

  pinMode(FLOW_PIN, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(FLOW_PIN), flowPulse, RISING);

  SPI.begin();

  /* -------- LoRa Setup -------- */
  LoRa.setPins(LORA_SS, LORA_RST, LORA_DIO0);
  if (!LoRa.begin(433E6)) {
    Serial.println("LoRa failed");
    while (1);
  }
  Serial.println("Node C LoRa OK");
}

void loop() {

  /* -------- SOIL MOISTURE -------- */
  int soilValue = analogRead(MOISTURE_PIN);

  /* -------- IRRIGATION LOGIC -------- */
  bool pumpState;

  if (soilValue <= moistureThreshold) {
    digitalWrite(RELAY_PIN, HIGH);   // Pump OFF
    pumpState = false;
  } else {
    digitalWrite(RELAY_PIN, LOW);    // Pump ON
    pumpState = true;
  }

  /* -------- FLOW SENSOR -------- */
  flowPulseCount = 0;
  delay(1000);   // 1 second window

  // YF-S201: ~450 pulses per liter
  flowRate = (flowPulseCount / 450.0) * 60.0; // L/min
  totalLiters += (flowPulseCount / 450.0);

  /* -------- LORA PAYLOAD (Node A style) -------- */
  String payload = "ID=C Soil=" + String(soilValue) +
                   " Pump=" + String(pumpState ? "ON" : "OFF") +
                   " Flow=" + String(flowRate, 1) +
                   " Total=" + String(totalLiters, 2);

  LoRa.beginPacket();
  LoRa.print(payload);
  LoRa.endPacket();

  Serial.print("Sent: ");
  Serial.println(payload);
  Serial.println("-----------------------");

  delay(3000);
}
