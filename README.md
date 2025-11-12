# Arduino-IDE
아두이노 IDE
#include <Wire.h>
#include <Adafruit_BMP085.h>
#include <EEPROM.h>

Adafruit_BMP085 bmp;

float seaLevelPressure = 102237; // 기준 기압 (Pa)
float baseAltitude = 0;
int addr = 0;                     // EEPROM 저장 위치
int buzzerPin = 8;                // 부저 핀
float thresholdPressure = 1012.0;  // 임계 기압 (hPa)
bool buzzerState = false;

void setup() {
  Serial.begin(9600);
  Serial.println("EEPROM Save Pressure & Altitude with Buzzer Test");

  pinMode(buzzerPin, OUTPUT);
  digitalWrite(buzzerPin, HIGH); // 반전된 부저, HIGH = OFF

  if (!bmp.begin()) {
    Serial.println("BMP180 Sensor not found!");
    while (1);
  }

  delay(2000);
  baseAltitude = bmp.readAltitude(seaLevelPressure);
  Serial.print("Base altitude set to ");
  Serial.print(baseAltitude, 2);
  Serial.println(" m (Zero point)");
}

void loop() {
  if (addr + 8 <= EEPROM.length()) {  // float 2개 = 8byte
    float pressure = bmp.readPressure() / 100.0; // hPa
    float altitude = bmp.readAltitude(seaLevelPressure) - baseAltitude; // 보정 고도

    // EEPROM 저장
    EEPROM.put(addr, pressure);
    EEPROM.put(addr + 4, altitude);

    Serial.print("Saved -> Pressure: ");
    Serial.print(pressure);
    Serial.print(" hPa  |  EEPROM Addr: ");
    Serial.println(addr);

    addr += 8;

    // 임계 기압 경보
    if (pressure < thresholdPressure) {
      digitalWrite(buzzerPin, LOW); // 반전 부저, LOW = 울림
      buzzerState = true;
      Serial.println("⚠️ Pressure too low! Buzzer ON");
    } else {
      digitalWrite(buzzerPin, HIGH); // 정상 시 OFF
      buzzerState = false;
    }

    // 부저 상태 표시
    Serial.print("Buzzer state: ");
    Serial.println(buzzerState ? "ON" : "OFF");
  } else {
    Serial.println("EEPROM full! Stopping logging.");
    while (1);
  }

  delay(3000); // 3초마다 저장
}
