#include "thingProperties.h"
#include <U8g2lib.h>
#include <DHT.h>
#include <TinyGPS++.h>
#include <SoftwareSerial.h>

// -------------------- PIN DEFINITIONS --------------------
#define DHTPIN D5
#define DHTTYPE DHT11
#define BUZZER_PIN D8
#define RELAY_PIN D3
#define PULSE_PIN D4
#define GAS_PIN A0

#define GPS_RX D7
#define GPS_TX D6

// -------------------- OBJECTS --------------------
U8G2_SSD1306_128X64_NONAME_1_HW_I2C u8g2(U8G2_R0, U8X8_PIN_NONE);
DHT dht(DHTPIN, DHTTYPE);

TinyGPSPlus gps;
SoftwareSerial gpsSerial(GPS_RX, GPS_TX);

// -------------------- VARIABLES --------------------
float latitude = 0.0;
float longitude = 0.0;
int satellites = 0;

// DHT variables
float temp = 25.0;
float lastTemp = 25.0;
unsigned long lastDHTRead = 0;

// -------------------- SETUP --------------------
void setup() {
  Serial.begin(115200);
  delay(1500);

  initProperties();
  ArduinoCloud.begin(ArduinoIoTPreferredConnection);

  dht.begin();
  u8g2.begin();
  gpsSerial.begin(9600);

  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(RELAY_PIN, OUTPUT);
  pinMode(PULSE_PIN, INPUT);

  digitalWrite(BUZZER_PIN, LOW);
  digitalWrite(RELAY_PIN, LOW);
}

// -------------------- LOOP --------------------
void loop() {
  ArduinoCloud.update();

  // -------- DHT READ (EVERY 2 SEC) --------
  if (millis() - lastDHTRead > 2000) {
    lastDHTRead = millis();
    float newTemp = dht.readTemperature();
    if (!isnan(newTemp)) {
      // Smooth variation (avoid jump/freeze)
      temp = (temp * 0.7) + (newTemp * 0.3);
      lastTemp = temp;
      Serial.print("Temp Updated: ");
      Serial.println(temp);
    } else {
      Serial.println("DHT Error ❌");
      temp = lastTemp;
    }
  }

  // -------- OTHER SENSORS --------
  int gasValue = analogRead(GAS_PIN);
  bool pulseState = digitalRead(PULSE_PIN);

  // -------- GPS --------
  while (gpsSerial.available()) {
    gps.encode(gpsSerial.read());
  }

  satellites = gps.satellites.value();

  if (gps.location.isValid() && satellites >= 3) {
    latitude = gps.location.lat();
    longitude = gps.location.lng();
  }

  // -------- CLOUD --------
  bodytemp = (int)temp;
  aQI = gasValue;
  bPM = pulseState ? 72 : 0;

  // -------- COOLING --------
  bool cooling = false;
  if (temp > 35) {
    digitalWrite(RELAY_PIN, HIGH);
    cooling = true;
  } else {
    digitalWrite(RELAY_PIN, LOW);
  }

  // -------- BUZZER --------
  if (temp > 38 || gasValue > 500 || lED) {
    digitalWrite(BUZZER_PIN, HIGH);
    lED = true;
  } else {
    digitalWrite(BUZZER_PIN, LOW);
  }

  // -------- OLED --------
  u8g2.firstPage();
  do {
    u8g2.setFont(u8g2_font_6x10_tf);
    u8g2.drawStr(0, 10, "GUARDIAN ACTIVE");
    u8g2.drawHLine(0, 13, 128);
    u8g2.setCursor(0, 25);
    u8g2.print("Temp:");
    u8g2.print(temp,1);
    u8g2.setCursor(0, 38);
    u8g2.print("AQI:");
    u8g2.print(gasValue);
    u8g2.setCursor(0, 51);
    u8g2.print("BPM:");
    u8g2.print(bPM);
    u8g2.setCursor(70, 51);
    u8g2.print(cooling ? "CL:ON" : "CL:OFF");
    u8g2.setCursor(0, 63);
    if (satellites >= 3) {
      u8g2.print(latitude,2);
      u8g2.print(",");
      u8g2.print(longitude,2);
    } else {
      u8g2.print("GPS WAIT...");
    }

  } while (u8g2.nextPage());
}
