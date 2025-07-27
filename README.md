# esp32-chat
#include <WiFi.h>
#include <ESP32Servo.h>

// Wi-Fi credentials
const char* ssid = "TITO";
const char* password = "khushi2006";

// Servo objects
Servo leftLeg, rightLeg, leftWheel, rightWheel;

// Ultrasonic pins
const int trigPin = 18;
const int echoPin = 19;

// Servo pins
const int leftLegPin = 33;
const int rightLegPin = 32;
const int leftWheelPin = 25;
const int rightWheelPin = 26;

WiFiServer server(80);

void setup() {
  Serial.begin(115200);
  Serial.println("Connecting to WiFi...");
  WiFi.begin(ssid, password);

  int attempts = 0;
  while (WiFi.status() != WL_CONNECTED && attempts < 20) {
    delay(500);
    Serial.print(".");
    attempts++;
  }

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\n✅ Connected to WiFi.");
    Serial.print("IP Address: ");
    Serial.println(WiFi.localIP());

    server.begin();

    leftLeg.attach(leftLegPin);
    rightLeg.attach(rightLegPin);
    leftWheel.attach(leftWheelPin);
    rightWheel.attach(rightWheelPin);

    pinMode(trigPin, OUTPUT);
    pinMode(echoPin, INPUT);
  } else {
    Serial.println("\n❌ WiFi connection failed.");
  }
}

void loop() {
  WiFiClient client = server.available();

  if (client) {
    String request = client.readStringUntil('\r');
    Serial.println("Received request: " + request);
    client.flush();

    if (request.indexOf("/walk") != -1) walk();
    else if (request.indexOf("/wheels") != -1) wheels();
    else if (request.indexOf("/stop") != -1) stopAll();
    else if (request.indexOf("/forward") != -1) moveForward();
    else if (request.indexOf("/backward") != -1) moveBackward();
    else if (request.indexOf("/left") != -1) turnLeft();
    else if (request.indexOf("/right") != -1) turnRight();
    else if (request.indexOf("/automatic") != -1) automaticMode();

    client.println("HTTP/1.1 200 OK");
    client.println("Content-Type: text/html");
    client.println();
    client.println("<html><body><h1>Command Executed</h1></body></html>");
    delay(1);
  }
}

// ====== Movement Functions ======

void walk() {
  Serial.println("Walking mode");
  leftLeg.write(80);
  delay(100);
  rightLeg.write(100);
  delay(100);
  leftLeg.write(60);
  delay(100);
  rightLeg.write(120);
  delay(100);
  leftLeg.write(90);
  rightLeg.write(90);
}

void stopAll() {
  Serial.println("Stopping all");
  leftWheel.write(90);
  rightWheel.write(90);
  leftLeg.write(90);
  rightLeg.write(90);
}

void wheels() {
  Serial.println("Wheels mode");
  delay(50);
  leftWheel.write(0);
  rightWheel.write(180);
}

void moveForward() {
  Serial.println("Switching to Forward");
  delay(50);
  Serial.println("Now moving forward");
  leftWheel.write(0);
  rightWheel.write(180);
}

void moveBackward() {
  Serial.println("Switching to Backward");
  delay(50);
  Serial.println("Now moving backward");
  leftWheel.write(180);
  rightWheel.write(0);
}

void turnLeft() {
  Serial.println("Turning Left");
  delay(50);
  leftWheel.write(90);
  rightWheel.write(0);
}

void turnRight() {
  Serial.println("Turning Right");
  delay(50);
  leftWheel.write(180);
  rightWheel.write(90);
}

void automaticMode() {
  long duration, distance;

  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH);
  distance = duration * 0.034 / 2;

  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  if (distance > 0 && distance < 20) {
    stopAll();
    delay(500);

    moveBackward();
    delay(500);

    stopAll();
    delay(500); 

    turnLeft();
    delay(700);

    stopAll();
    delay(500);    
  } else {
    moveForward();
  }
  delay(50);
}
