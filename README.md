# LFR-Project code

#include <SD.h>
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 32 // OLED display height, in pixels



Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT);

File myFile;
// Motor control pins
const int IN1 = 9;
const int IN2 = 8;
const int IN3 = 7;
const int IN4 = 6;
const int ENA = 10;
const int ENB = 5;

// IR sensor pins
const int rightIR = A0;
const int leftIR = A1;

float vOUT = 0.0;
float vIN = 0.0;
float R1 = 30000.0;
float R2 = 7500.0;

void setup() {
  // put your setup code here, to run once:
 display.begin(SSD1306_SWITCHCAPVCC, 0X3C); 
 Serial.begin(9600);
 while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

  if (!SD.begin(4)) {
    display.print("initialization failed!");
    
  }
  myFile = SD.open("test.txt", FILE_WRITE);

  // if the file opened okay, write to it:
  if (myFile) {
    display.print(" Line Follower");
    myFile.print("TURNS");
    // close the file:
    myFile.close();
  } else {
    // if the file didn't open, print an error:
    display.print("error opening test.txt");
  }


  display.print("Initializing SD card...");

delay(500);
 
 pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  pinMode(ENA, OUTPUT);
  pinMode(ENB, OUTPUT);
  pinMode(rightIR, INPUT);
  pinMode(leftIR, INPUT);

  // Set initial motor direction
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);

  // Set initial motor speed
  analogWrite(ENA, 100);
  analogWrite(ENB, 100);
}
void loop(){
   myFile = SD.open("test.txt", FILE_WRITE);
   linefollower();
   voltagesensor();
  
  }
void voltagesensor() {

  int sensorValue = analogRead(A0);
  
  vOUT = (sensorValue / 1023.0) * 5.0;
  vIN = vOUT / (R2/(R1+R2));
  vOUT=vIN*(R2/(R1+R2));
display.clearDisplay();
  display.setTextSize(2);      // Normal 1:1 pixel scale
  display.setTextColor(SSD1306_WHITE); // Draw white text
  display.setCursor(0, 0); 
  display.print("VOLTS :");
  display.println(vIN);
   delay(200);
  display.display();

  
}


void linefollower() {
  int rightSensor = digitalRead(rightIR);
  int leftSensor = digitalRead(leftIR);

  if (rightSensor == HIGH && leftSensor == HIGH) {
    // Both sensors are on the black line, move backward
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, HIGH);
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, HIGH);
  } else if (rightSensor == HIGH && leftSensor == LOW) {
    // Right sensor is on the black line, turn right
    digitalWrite(IN1, HIGH);
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, LOW);
  } else if (rightSensor == LOW && leftSensor == HIGH) {
    // Left sensor is on the black line, turn left
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, HIGH);
    digitalWrite(IN4, LOW);
  } else {
    // Neither sensor is on the black line, move forward
    digitalWrite(IN1, HIGH);
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, HIGH);
    digitalWrite(IN4, LOW);
  }
}
