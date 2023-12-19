#include <Servo.h>
#include <NewPing.h>

#define TRIGGER_PIN 4
#define ECHO_PIN 3
#define MAX_DISTANCE 200

#define IN1 8  // Motor1 L298 Pin in1
#define IN2 9  // Motor1 L298 Pin in2
#define IN3 10 // Motor2 L298 Pin in3
#define IN4 11 // Motor2 L298 Pin in4

#define SERVO_PIN 7
#define ENA 5  // PWM pin for Motor 1 speed control
#define ENB 6  // PWM pin for Motor 2 speed control

NewPing sonar(TRIGGER_PIN, ECHO_PIN, MAX_DISTANCE);
Servo servo1;

boolean goesForward = false;
int distance = 0;

void setup() {
  servo1.attach(SERVO_PIN);
  servo1.write(90); // Set the servo to the center position

  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  
  // Set ENA and ENB pins as outputs for motor speed control
  pinMode(ENA, OUTPUT);
  pinMode(ENB, OUTPUT);

  // Perform initial readings to avoid glitches
  delay(100);
  readDistance();
  delay(100);
  readDistance();
  delay(100);
  readDistance();
  delay(100);
  readDistance();
}

void loop() {
  int distanceR = 0;
  int distanceL = 0;

  delay(40);

  // Read the distance
  distance = readDistance();

  if (distance <= 12) { // Obstacle detected
    moveStop();
    delay(100);
    moveBackward();
    delay(300);
    moveStop();
    delay(200);

    // Look right
    distanceR = lookRight();
    delay(500);

    // Look left
    distanceL = lookLeft();
    delay(500);

    if (distanceR > distanceL) {
      turnRight();
      moveStop();
    } else if (distanceR < distanceL) {
      turnLeft();
      moveStop();
    }
  } else {
    moveForward();
  }
}

int readDistance() {
  delay(70);
  int cm = sonar.ping_cm();
  if (cm == 0) {
    cm = 250; // Handle out-of-range readings
  }
  return cm;
}

void moveStop() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
}

void moveForward() {
  goesForward = true;
  analogWrite(ENA, 100); // Set motor 1 speed to 150
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  analogWrite(ENB, 100); // Set motor 2 speed to 150
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void moveBackward() {
  goesForward = false;
  analogWrite(ENA, 100); // Set motor 1 speed to 150
  digitalWrite(IN1, LOW );
  digitalWrite(IN2, HIGH);
  analogWrite(ENB, 100); // Set motor 2 speed to 150
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}

void turnRight() {
  analogWrite(ENA, 100); // Set motor 1 speed to 150
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  analogWrite(ENB, 100); // Set motor 2 speed to 150
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, HIGH);
  delay(600);
}

void turnLeft() {
  analogWrite(ENA, 100); // Set motor 1 speed to 150
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, HIGH);
  analogWrite(ENB, 100); // Set motor 2 speed to 150
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
  delay(600);
}

int lookRight() {
  servo1.write(45);  // Turn servo right
  delay(500);
  int distance = readDistance();
  delay(100);
  servo1.write(90); // Reset servo to center
  return distance;
}

int lookLeft() {
  servo1.write(45); // Turn servo left
  delay(500);
  int distance = readDistance();
  delay(100);
  servo1.write(90); // Reset servo to center
  return distance;
}
