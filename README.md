
#include <TridentTD_LineNotify.h>
#define BLYNK_PRINT Serial
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#define LINE_TOKEN  "khRIanrIv6WrooazZXd7igfjazBGBlcWyFbV6IE8rea"

#include "DHT.h"
#define DHTPIN D7
#define DHTTYPE DHT21

DHT dht(DHTPIN, DHTTYPE);

const int analogInPin = A0;
const int relay = D6;

int sensorValue = 0;        // ตัวแปรค่า Analog
int outputValue = 0;        // ตัวแปรสำหรับ Map เพื่อคิด %

int statusAM = 0;   //เก็บตัวแปรเก็บค่า auto
int statusManul = 0;  //เก็บตัวแปรเก็บค่า เมนวล

int N = 50;
int runXTimes = 0;

boolean flag = true;

BlynkTimer timer;
BlynkTimer timer2;




char auth[] = "G4I_gvzhRhE2zrmdLssADOjG0Qp_d8BO";
char ssid[] = "Bua_2.4G";
char pass[] = "024248821";




void setup() {
  Serial.begin(9600);
  
  
  pinMode(relay, OUTPUT);
  
  Blynk.begin(auth, ssid, pass, "103.233.194.42", 8080);    // ใช้เป็น Server Blynk ฟรี

  timer.setInterval(10, runner);
  timer2.setInterval(100, soil);

  LINE.setToken(LINE_TOKEN);

   dht.begin();
}

void loop() {
  Blynk.run();
  timer.run();
  timer2.run();


  
    
  

}

BLYNK_WRITE(V0) {
  statusAM = param.asInt();
}
BLYNK_WRITE(V2) {
  statusManul = param.asInt();
}

void runner() {   // เลือก Mode
  if (statusAM == 1) {
    auto1();
  }
  else {
    manul();
  }
}
void temp() {
  

  
    
  

}


void auto1() {    // Mode ทำงาน Auto
  if (outputValue >= 50) {  //ตั้งค่า % ที่ต้องการจะรดน้ำต้นไม้
    
    motorOn();
  }
  else {
    motorOff();
  }
  delay(1000);
}

void manul() {    // Mode ทำงาน Manul
  if (statusManul == 1) {
    motorOn();
  }
  else {
    motorOff();
  }
}


void soil() {
  sensorValue = analogRead(analogInPin);
  outputValue = map(sensorValue, 0, 1024, 100, 0);
  
  Serial.print(" Soil Moisture = ");
  Serial.print(outputValue);
  Serial.println(" %");
  Blynk.virtualWrite(V1, outputValue);
  delay(500);

  float h = dht.readHumidity();
  float t = dht.readTemperature();
  
  Blynk.virtualWrite(V4, t);
  Blynk.virtualWrite(V5, h);

  
  
  

  Serial.print(F("Humidity: "));
  Serial.print(h);
  Serial.print(F("%  Temperature: "));
  Serial.print(t);
  
  
  

  if (outputValue < 50)  {
    while(flag==true){  
    Serial.println("Send Line");
    LINE.notify("รดน้ำได้แล้วจ้า !!");
    delay(100);
    flag = false;
  }
  }

  if(outputValue >=50){
    while(flag==false){
    Serial.println("Send Line ");
    LINE.notify("ต้นไม้อิ่มน้ำแล้ว !!");
    delay(100);
    flag = true;
    }
  }
  
  
    
  

  

 
  
}


WidgetLED led1(V3);


void motorOn() {      //มอเตอร์ปั้มน้ำทำงาน
  digitalWrite(relay, LOW);
  led1.off();
}

void motorOff() {     //มอเตอร์ปั้มน้ำหยุดทำงาน
  digitalWrite(relay, HIGH);
  led1.on();
}
