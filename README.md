#include <NewPing.h> 
#include <Servo.h> 
#include <Adafruit_MPU6050.h> 
#include <Adafruit_Sensor.h> 
#include <Wire.h> 
#include <SoftwareSerial.h> 
#include <TinyGPS++.h> 
Adafruit_MPU6050 mpu;                                   
const int trigPin = 10; 
const int echoPin = 9;
const int buzzerPin = 6;
const int servoPin = 3; 
const int vibrationPin = A3; 
const int maxDistance = 20; 
NewPing sonar(trigPin, echoPin); // Create an instance of the NewPing class for ultrasonic sensing
Servo servo; // Create an instance of the Servo class for servo motor control
int RXPin = 5; 
int TXPin = A0;
TinyGPSPlus gps; // Create an instance of the TinyGPSPlus class for GPS parsing
int GPSBaud = 9600; 
SoftwareSerial gpsSerial(RXPin, TXPin); 
SoftwareSerial bluetoothSerial(A2, A1); 
float angleX, angleY, angleZ; 
const int cancelButtonPin = 2;
const int sendLocationButtonPin = 7;
bool mpuEnabled = true; 
int angle = 0; 
int step = 1; 
int delayTime = 10; 
unsigned long previousMillis = 0; 
void setup() {
  Serial.begin(9600); 
  gpsSerial.begin(GPSBaud); 
pinMode(buzzerPin, OUTPUT);
  servo.attach(servoPin);                           
pinMode(vibrationPin, OUTPUT); 
  if (!mpu.begin()) { 
    Serial.println("Failed to find MPU6050 chip");
    while (1) {
      delay(10);
    }
  }
  Serial.println("MPU6050 Found!");
  mpu.setAccelerometerRange(MPU6050_RANGE_8_G); 
  mpu.setGyroRange(MPU6050_RANGE_500_DEG); 
  mpu.setFilterBandwidth(MPU6050_BAND_21_HZ); 
  delay(100);
  bluetoothSerial.begin(9600); 
  Serial.println("HC-05 Bluetooth Module Example");
  Serial.println("Enter '1' to turn on the LED, '0' to turn it off");
pinMode(cancelButtonPin, INPUT_PULLUP); 
  pinMode(sendLocationButtonPin, INPUT_PULLUP);
}
void loop() {
  unsigned int distance = sonar.ping_cm(); 
  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");
if (distance < maxDistance) {
    analogWrite(buzzerPin, 50);
    bluetoothSerial.println("OBSTACLE !");
  } else {
    digitalWrite(buzzerPin, LOW);
    delay(100);                      
sensors_event_t a, g, temp;
    mpu.getEvent(&a, &g, &temp); 
     Serial.print("Acceleration X: ");
    Serial.print(a.acceleration.x);
    Serial.print(", Y: ");
    Serial.print(a.acceleration.y);
    Serial.print(", Z: ");
    Serial.print(a.acceleration.z);
    Serial.println(" m/s^2");
    Serial.print("Rotation X: ");
    Serial.print(g.gyro.x);
    Serial.print(", Y: ");
    Serial.print(g.gyro.y);
    Serial.print(", Z: ");
    Serial.print(g.gyro.z);
    Serial.println(" rad/s");
    Serial.print("Temperature: ");
    Serial.print(temp.temperature);
    Serial.println(" degC");
    Serial.println("");
    delay(100);
while (gpsSerial.available() > 0) { // Displays information when new GPS sentence is available
      Serial.write(gpsSerial.read());
    }
  if (bluetoothSerial.available()) {
      char data = bluetoothSerial.read();
      Serial.print("Received data: ");
      Serial.println(data);
      if (data == '1') {
        digitalWrite(LED_BUILTIN, HIGH);                                
      } else if (data == '0') {
        digitalWrite(LED_BUILTIN, LOW);
      }
    }
if (angleX >= 0 && angleX <= 10 && angleY >= 0 && angleY <= 10 && angleZ >= 0 && angleZ <= 10) {
      bluetoothSerial.println("user fallen!"); 
      if (angleX == 0 && angleY == 0 && angleZ == 0) { 
        if (gps.location.isValid()) {
          String latitude = String(gps.location.lat(), 6);
          String longitude = String(gps.location.lng(), 6);
          String message = "Location: " + latitude + ", " + longitude;
          bluetoothSerial.println(message);
          bluetoothSerial.println("user fallen!");
        }
      }
    }

    if (digitalRead(cancelButtonPin) == LOW) { 
      mpuEnabled = false; 
      Serial.println("MPU6050 readings canceled");
      delay(100); 
    }
if (digitalRead(sendLocationButtonPin) == LOW) {
      if (gps.location.isValid()) {
        String latitude = String(gps.location.lat(), 6);
        String longitude = String(gps.location.lng(), 6);
        String message = "Location: " + latitude + ", " + longitude;
        bluetoothSerial.println(message); 
        Serial.println("Location sent via Bluetooth");                                                
        bluetoothSerial.println("user fallen!");
      }
      delay(100); 
    }
  }
}

