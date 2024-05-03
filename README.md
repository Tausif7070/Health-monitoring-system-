# Health-monitoring-system-
#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>
#include <DHT.h>
#include <MAX30001.h>
#include <Wire.h>

// WiFi credentials
char ssid[] = "your_wifi_ssid";
char pass[] = "your_wifi_password";

// Blynk authentication token
char auth[] = "your_blynk_auth_token";

// DHT11 sensor setup
#define DHTPIN 4          // Digital pin connected to the DHT sensor
#define DHTTYPE DHT11     // DHT 11

DHT dht(DHTPIN, DHTTYPE);

// MAX30001 sensor setup
MAX30001 max30001;

// PulseSensor setup
#define PULSE_SENSOR_PIN A0 // Analog pin connected to PulseSensor

// Variables to store sensor readings
float temperature;
float humidity;
int heartRate;

void setup()
{
  Serial.begin(9600);

  // Initialize DHT sensor
  dht.begin();

  // Initialize MAX30001 sensor
  if (!max30001.begin(Wire))
  {
    Serial.println("MAX30001 not detected");
    while (1);
  }

  // Initialize PulseSensor
  pinMode(PULSE_SENSOR_PIN, INPUT);

  // Connect to WiFi
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }

  // Connect to Blynk server
  Blynk.begin(auth, ssid, pass);
}

void loop()
{
  Blynk.run();

  // Read DHT11 sensor data
  humidity = dht.readHumidity();
  temperature = dht.readTemperature();

  // Read MAX30001 sensor data
  if (max30001.available())
  {
    heartRate = max30001.getHeartRate();
  }

  // Read PulseSensor data
  int pulseSensorValue = analogRead(PULSE_SENSOR_PIN);
  // Convert pulseSensorValue to BPM
  // Adjust the calculation according to your PulseSensor output
  heartRate = map(pulseSensorValue, 0, 1023, 30, 220);

  // Send sensor data to Blynk
  Blynk.virtualWrite(V0, temperature); // Virtual pin V0 for temperature
  Blynk.virtualWrite(V1, humidity);    // Virtual pin V1 for humidity
  Blynk.virtualWrite(V2, heartRate);   // Virtual pin V2 for heart rate

  delay(1000); // Delay before taking next sensor readings
}
