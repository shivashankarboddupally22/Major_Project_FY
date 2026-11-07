# Major_Project_FY
An Arduino Uno-based embedded system that detects alcohol consumption and vehicle accidents, obtains GPS location, and sends emergency SMS alerts through a GSM module.
# Automatic Accident, Alcohol Detection and Alert System

 An Arduino-Based Embedded System for Vehicle Safety and Emergency Alerting

# Project Overview

The Automatic Accident, Alcohol Detection and Alert System is an embedded system designed to improve vehicle safety by detecting alcohol consumption and accident conditions.

The system uses an Arduino Uno as its main controller. An MQ-3 alcohol sensor detects alcohol, while a limit switch is used to identify an accident condition.

When an accident or alcohol detection event occurs, the system uses GPS to obtain the vehicle's geographical location and GSM to send an emergency SMS alert to predefined mobile numbers.

An I2C LCD displays the system status and GPS information.

# Objectives

- Detect alcohol consumption using the MQ-3 sensor.
- Identify accident conditions using a limit switch.
- Obtain real-time geographical coordinates using GPS.
- Send emergency SMS alerts using GSM.
- Display system status and GPS information on an LCD.
- Improve vehicle safety through automated alert generation.

# Hardware Components

| Component | Purpose |
|---|---|
| Arduino Uno | Main controller |
| MQ-3 Sensor | Alcohol detection |
| Limit Switch | Accident detection input |
| GPS Module | Location tracking |
| GSM Module | Emergency SMS communication |
| I2C LCD 16x2 | Status and location display |
| Buzzer (if implemented) | Audio alert |
| Jumper Wires | Circuit connections |
| Power Supply | System power |

# Software Requirements

- Arduino IDE
- Arduino AVR Boards
- LiquidCrystal_I2C Library
- TinyGPS++ Library
- SoftwareSerial Library

# Pin Connections

| Component | Arduino Pin |
|---|---|
| Limit Switch | D2 |
| MQ-3 Digital Output | D3 |
| GPS TX | D4 |
| GPS RX | D5 |
| GSM TX | D7 |
| GSM RX | D6 |
| I2C LCD SDA | A4 |
| I2C LCD SCL | A5 |

**Note:** GPS and GSM RX/TX connections are based on the SoftwareSerial constructor configuration in the Arduino code. Use suitable voltage-level compatibility for the modules.

# Working Principle

# 1. System Initialization
The Arduino initializes the GPS, GSM, and LCD modules. The LCD displays the system-ready message.

# 2. Alcohol Detection
The MQ-3 sensor monitors alcohol concentration. When its digital output reaches the configured detection condition, the Arduino identifies an alcohol detection event.

# 3. Accident Detection
The limit switch acts as an accident detection input. When the switch is activated, the Arduino identifies the accident condition.

# 4. GPS Location Tracking
The GPS module provides geographical coordinates, including latitude and longitude.

# 5. GSM Alert Communication
The GSM module sends an emergency SMS containing the event information and Google Maps location link to predefined mobile numbers.

# 6. LCD Display
The LCD displays the GPS coordinates and system status.

# System Flow

       SYSTEM START
             |
             v
      Initialize Arduino
             |
             v
       Initialize GPS,
       GSM and LCD
             |
             v
       Read GPS Data
             |
       +-----+------+
       |            |
       v            v
  Alcohol       Accident
  Detection?    Detection?
       |            |
       v            v
  Generate      Generate
  Alcohol       Accident
  Alert         Alert
       |            |
       +-----+------+
             |
             v
       Get GPS Location
             |
             v
        Send SMS
        Through GSM
             |
             v
        Update LCD
             |
             v
        Repeat Loop
```

# Project Structure

Automatic-Accident-Alcohol-Detection-System/
│
├── Arduino_Code/
│   └── Accident_Alcohol_Detection.ino
│
├── Circuit_Diagram/
├── Block_Diagram/
├── Project_Images/
├── Documentation/
└── README.md
```

# Applications

- Vehicle safety systems
- Alcohol detection systems
- Emergency accident alert systems
- Vehicle location tracking
- Automotive embedded applications
- Reduces the Responce time

# Limitations

- GPS accuracy depends on satellite availability.
- GSM communication requires network coverage.
- MQ-3 readings depend on calibration and environmental conditions.
- A limit switch alone cannot reliably identify every real-world vehicle accident.

# Future Enhancements

- Integration of ESP32 for advanced IoT functionality.
- Integration of an accelerometer or gyroscope for improved accident detection.
- Real-time vehicle tracking through a mobile application.
- Cloud-based vehicle monitoring.
- Automatic emergency contact notification.
- Advanced alcohol measurement and calibration.

# Developed By

**Shiva Shankar Boddupally**

Electronics and Communication Engineering

Interested in Embedded Systems | Arduino | ESP32 | C Programming | VLSI

# License

This project is intended for educational and learning purposes.



# Source Code
/*
  Accident & Alcohol Detection with GPS + GSM SMS Alert
  Board   : Arduino Uno
  Modules : NEO-6M GPS, SIM800L GSM, MQ-3 (digital out), limit switch, 16x2 I2C LCD
  Libraries: LiquidCrystal_I2C, TinyGPSPlus
*/
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <SoftwareSerial.h>
#include <TinyGPS++.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);

#define LIMIT_SWITCH 2   // to GND when pressed (impact)
#define MQ3_PIN      3   // MQ-3 DO pin, LOW = alcohol detected

// ---------- GPS & GSM ----------
SoftwareSerial gpsSerial(4, 5);  // RX, TX
SoftwareSerial gsmSerial(7, 6);  // RX, TX
TinyGPSPlus gps;

// ---------- Mobile numbers (replace with real numbers) ----------
const String number1 = "+91XXXXXXXXXX";
const String number2 = "+91XXXXXXXXXX";

bool accidentSent = false;
bool alcoholSent  = false;

// Keep feeding the GPS parser while waiting
void smartDelay(unsigned long ms) {
  unsigned long start = millis();
  while (millis() - start < ms) {
    while (gpsSerial.available()) gps.encode(gpsSerial.read());
  }
}

void sendSMS(const String &number, const String &message) {
  gsmSerial.println("AT+CMGF=1");          // text mode
  delay(1000);
  gsmSerial.print("AT+CMGS=\"");
  gsmSerial.print(number);
  gsmSerial.println("\"");
  delay(1000);
  gsmSerial.print(message);
  delay(500);
  gsmSerial.write(26);                     // CTRL+Z sends
  delay(3000);
}

String buildMessage(const char *title) {
  String msg = String(title) + "\nLocation:\n";
  if (gps.location.isValid()) {
    msg += "http://maps.google.com/?q=";
    msg += String(gps.location.lat(), 6) + "," + String(gps.location.lng(), 6);
  } else {
    msg += "GPS fix not available";
  }
  return msg;
}

void alertBoth(const char *title) {
  String msg = buildMessage(title);
  sendSMS(number1, msg);
  sendSMS(number2, msg);
  gpsSerial.listen();                      // make sure GPS is the active listener
}

void setup() {
  Serial.begin(9600);
  gpsSerial.begin(9600);
  gsmSerial.begin(9600);
  gpsSerial.listen();                      // only one SoftwareSerial can receive at a time

  pinMode(LIMIT_SWITCH, INPUT_PULLUP);
  pinMode(MQ3_PIN, INPUT);

  lcd.init();
  lcd.backlight();
  lcd.print("System Ready");
  delay(2000);
  lcd.clear();
}

void loop() {
  while (gpsSerial.available()) gps.encode(gpsSerial.read());

  // ---- Accident detection ----
  
  bool impact = (digitalRead(LIMIT_SWITCH) == LOW);
  if (impact && !accidentSent) {
    lcd.clear();
    lcd.print("Accident!");
    alertBoth("Accident Alert!");
    accidentSent = true;
  }
  
  if (!impact) accidentSent = false;      // re-arm after switch released

  // ---- Alcohol detection ----
  bool alcohol = (digitalRead(MQ3_PIN) == LOW);
  if (alcohol && !alcoholSent) {
    lcd.clear();
    lcd.print("Alcohol Detected");
    alertBoth("Alcohol Detected!");
    alcoholSent = true;
  }
  if (!alcohol) alcoholSent = false; // re-arm when air is clear

  // ---- Normal display ----
  lcd.clear();
  lcd.setCursor(0, 0);
  if (gps.location.isValid()) {
    lcd.print("Lat:");
    lcd.print(gps.location.lat(), 2);
    lcd.setCursor(0, 1);
    lcd.print("Lng:");
    lcd.print(gps.location.lng(), 2);
  } else {
    lcd.print("Waiting for GPS");
  }

  smartDelay(1000);
}
