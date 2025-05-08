#include <Arduino.h>
#if defined(ESP32)
#include <WiFi.h>
#elif defined(ESP8266)
#include <ESP8266WiFi.h>
#endif
#include <Firebase_ESP_Client.h>

#include "addons/TokenHelper.h"
#include "addons/RTDBHelper.h"

#define WIFI_SSID "A4447"
#define WIFI_PASSWORD "12345678"

#define API_KEY "AIzaSyDMatHtw6O7KLnO93g-JEvCoFOENiC5_cY"

#define DATABASE_URL "https://machine-c5407-default-rtdb.firebaseio.com/"

FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;


unsigned long sendDataPrevMillis = 0;
bool signupOK = false;

bool Value1Flag = false;
bool Value2Flag = false;
bool Value3Flag = false;
bool Value4Flag = false;
bool Value5Flag = false;
bool Value6Flag = false;

String Value1 = "", Value2 = "", Value3 = "", Value4 = "", Value5 = "", Value6 = "";

void setup() {
  Serial.begin(9600);
  Serial.print("Connecting to Wi-Fi");
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);

  unsigned long startTime = millis();
  while (WiFi.status() != WL_CONNECTED) {
    if (millis() - startTime > 15000) {
      Serial.println("Failed to connect to Wi-Fi.");
      return;
    }
    Serial.print(".");
    delay(500);
  }
  Serial.println();
  Serial.print("Connected with IP: ");
  Serial.println(WiFi.localIP());

  config.api_key = API_KEY;
  config.database_url = DATABASE_URL;

  if (Firebase.signUp(&config, &auth, "", "")) {
    Serial.println("Firebase signup successful");
    signupOK = true;
  } else {
    Serial.printf("Firebase signup error: %s\n", config.signer.signupError.message.c_str());
  }

  config.token_status_callback = tokenStatusCallback;
  Firebase.begin(&config, &auth);
  Firebase.reconnectWiFi(true);
}

void loop() {
    readSerialData();
  if (Firebase.ready() && signupOK ) {
    updateFirebaseData();
    getFirebaseData();
  }
  delay(4000);
}


void readSerialData() {
  while (Serial.available() > 0) {
    char inchar = Serial.read();
    Serial.println("Inchar:" + String(inchar));

    if (inchar == '*') {
      Value1 = readUntil('#');//vignesh
      Value1Flag = true;
    }
    else if (inchar == '@') {
      Value2 = readUntil('#');
      Value2Flag = true;
    }
    else if (inchar == '$') {
      Value3 = readUntil('#');
      Value3Flag = true;
    }
    else if (inchar == '%') {
      Value4 = readUntil('#');
      Value4Flag = true;
    }
    else if (inchar == '^') {
      Value5 = readUntil('#');
      Value5Flag = true;
    }
    else if (inchar == '&') {
      Value6 = readUntil('#');
      Value6Flag = true;
    }
  }
}

String readUntil(char terminator) {
  String result = "";
  while (Serial.available() > 0) {
    char c = Serial.read();
    if (c == terminator) {
      break;
    }
    result += c;
  }
  return result;
}


void updateFirebaseData() {
  if (Value1Flag) {
    Value1Flag = false;
    updateFirebase("Monitoring/status", Value1);
  }

  if (Value2Flag) {
    Value2Flag = false;
    updateFirebase("Monitoring/flow1", Value2);
  }

  if (Value3Flag) {
    Value3Flag = false;
    updateFirebase("Monitoring/flow2", Value3);
  }

  if (Value4Flag) {
    Value4Flag = false;
    updateFirebase("Monitoring/Value4", Value4);
  }

  if (Value5Flag) {
    Value5Flag = false;
    updateFirebase("Monitoring/Value5", Value5);
  }
  if (Value6Flag) {
    Value6Flag = false;
    updateFirebase("Monitoring/Value6", Value6);
  }
}

// Function to send data to a specified Firebase path
void updateFirebase(String path, String value) {
  if (Firebase.RTDB.setString(&fbdo, path, value)) {
    Serial.println("PASSED: " + path);
    Serial.println("Value: " + value);
  } else {
    Serial.println("FAILED to update " + path);
    Serial.println("Reason: " + fbdo.errorReason());
  }
}


void getFirebaseData() {
  if (Firebase.RTDB.getString(&fbdo, "/Monitoring/Data")) {
    if (fbdo.dataType() == "string") {
      String intValue = fbdo.stringData();
      Serial.println("Firebase data: " + intValue);
      if (intValue == "A") {
        Serial.println("Detected 'A'. Performing action...");
      }
      if (intValue == "B") {
        Serial.println("Detected 'B'. Performing action...");
      }
    }
  } else {
    Serial.printf("Firebase read error: %s\n", fbdo.errorReason().c_str());
  }
}
