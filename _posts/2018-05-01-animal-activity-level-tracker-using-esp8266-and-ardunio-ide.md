---
title: Animal Activity Level Tracker Using ESP8266 (Wifi-Arduino)
categories:
- blog
excerpt: How I created an animal activity tracker using ESP8266 micro-controller and
  ADXL345 accelerometer
tags:
- ESP8266
- Firebase
- Ardunio
- C++
- Animal
- Activity
- Tracker
- DH11
- adxl345
- Accelerometer
date: 2018-05-01T04:00:00.000+00:00
header:
  overlay_image: "/assets/images/animal-activity-level-tracker-system-diagram.png"
  overlay_filter: "0.7"
  show_overlay_title: true

---
![](/assets/images/animal-activity-level-tracker-system-diagram.png)

The device is intended to be distributed to thousands of animal owners in the future. We intend to collect data from all these devices and enable researchers to use this "Big Data". Thus, we call this device the Bio Animal Big Data (BioABD). For details, please check the documentation below.

### Project Scope

The device is intended to be distributed to thousands of animal owners in the future. We intend to collect data from all these devices and enable researchers to use this "Big Data". Thus, we call this device the Bio Animal Big Data (BioABD).

The BioABD device measures acceleration in X, Y, and Z axis using the ADXL-345 accelerometer. It also measures temperature using the DS18B20 Temperature Sensor. ESP8266 microcontroller takes the readings from these sensors and sends it to Firebase (cloud database). These data are then viewable from an Android smartphone using our app. Extensive testing was done and these testing results are well documented on [our documentation](/files/ENG4000-Final-Report-BioABD.pdf).

![](/assets/images/animal-activity-tracker-system-diagram-state.png)

### Critical Design Review

{% include youtube.html id="TJRIo1g7_dk" %}

### Test Readiness Review

{% include youtube.html id="QcT-Il6SQbo" %}

### Test Review

{% include youtube.html id="UIK-N0HM-rw" %}

![Dog with activity level tracker beside myself](/assets/images/animal-activity-level-tracker-dog-mj.jpg)

![Dog wearing activity level tracker beside its owner](/assets/images/animal-activity-level-tracker-dog-trainer.jpg)

### Animal Activity Tracker Documentation (BioABD)

[![](/assets/images/ENG4000-Final-Report-BioABD-thumbnail.png)](/files/ENG4000-Final-Report-BioABD.pdf)

### Code for ESP8266

[https://github.com/kimmminjae/Animal-Activity-Tracker-ESP8266-Firebase](https://github.com/kimmminjae/Animal-Activity-Tracker-ESP8266-Firebase "https://github.com/kimmminjae/Animal-Activity-Tracker-ESP8266-Firebase")

    // Import Libraries
    #include <ESP8266WiFi.h>
    #include <FirebaseArduino.h>
    #include <Wire.h>
    #include <Adafruit_Sensor.h>
    #include <Adafruit_ADXL345_U.h>
    #include <OneWire.h> 
    #include <DallasTemperature.h>
    
    // Define Constants
    #define FIREBASE_HOST "bioabd-662cf.firebaseio.com"
    #define FIREBASE_AUTH "AUTH_STR_HERE"
    #define WIFI_SSID "MJK"
    #define WIFI_PASSWORD "WIFI_PASS"
    
    // Assign ID to the accelerometer
    Adafruit_ADXL345_Unified accel = Adafruit_ADXL345_Unified(12345);
    
    // Assign pin for the temperature sensor
    #define ONE_WIRE_BUS 2
    // Setup for the temperature sensor
    OneWire oneWire(ONE_WIRE_BUS); 
    DallasTemperature sensors(&oneWire);
    
    void setup() {
      Serial.begin(9600);
      // Attempt connection with given SSID and Password
      WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
      Serial.print("connecting");
      
      // Connect to Wi-Fi
      while (WiFi.status() != WL_CONNECTED) {
        Serial.print(".");
        delay(500);
      }
      
      Serial.println();
      Serial.print("connected: ");
      Serial.println(WiFi.localIP());
      
      // Connect to Firebase server with the given host name and authentication code
      Firebase.begin(FIREBASE_HOST, FIREBASE_AUTH);
      #ifndef ESP8266
      while (!Serial);
    #endif
      
      // Sensor initialization
      if(!accel.begin())
      {
        // Sensor detection error for the accelerometer
        Serial.println("No ADXL345 detected. Check the wiring.");
        while(1);
      }
      // Maximum measured force @ 16g
      accel.setRange(ADXL345_RANGE_16_G);
      
      // Start the temperature sensor
      sensors.begin();
    }
    
    void loop() {
      // Request the temperature
      sensors.requestTemperatures();
      sensors_event_t event; 
      
      // Request the acceleration
      accel.getEvent(&event);
      StaticJsonBuffer<200> jsonBuffer;
      JsonObject& root = jsonBuffer.createObject();
      
      // Measure the time in ms and send it to the server
      root["unixTime"] = millis();
      double xval = event.acceleration.x;
      // Measure the x y z acceleration values and send it to the server
      root["xValue"] = xval;
      root["yValue"] = event.acceleration.y;
      root["zValue"] = event.acceleration.z;
      // Measure the temperature and send it to the server
      root["tempValue"] = sensors.getTempCByIndex(0);
      // Push the measured data to the server
      String name = Firebase.push("/adxl345", root);
      // Error handler
      Serial.println(xval);
      if (Firebase.failed()) {
          Serial.print("pushing /logs failed:");
          Serial.println(Firebase.error());
          return;
      }
      // Delay for 1 sec and repeat
      delay(1000);
    }
