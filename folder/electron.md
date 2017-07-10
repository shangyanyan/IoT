```
//This project is named "smart fridge". 
//We try to solve the problems that people put food in the fridge and then forget about it. 
//So food goes bad without people notice it. The food box can remind people about their food.
//The concepts includes two parts: a physical food box connected with electron and a Blynk App on mobile.
//The physicak food box can sense whether there is food in the box or not.
//There are also three different color of leds to tell users the freshness of the food.
//Green means the food is fresh. Yellow means food is okay, but need to be consumed soon. Red means the food is already bad.
//There is also photocells which can sense whether the fridge is open or not. There is also a buzzer to attract users' attentions.
//There is a temperature sensor senses the temperature which will be displayed on Blynk App.
//Mobile App is for users to look at the food conditions away from the fridge.
//There are a lcd display in the App to tell users how much time before the food goes bad.
//There are also three different colors of leds which serve the similar function with physical ones.
//We will put some labels (meat, milk, eggs, vegetables) on the food box to tell people which type of food should go in that box. 
//There are two preset benchmark time, one is "food is going to bad, eat it soon" time to remind people to consume the food as soon as possible.
//The another one is "food is bad" time to tell users the food is already bad, you need to throw them away.
//The lcd on Blynk displays "Box is Empty" to tell users the food box is available to use when there is no food in it.

//When people put the food in that food box, the buttons are pressed, so the timer in electron begins to count. 
//Users will get a message on their mobile phone or e-mail tell them the food type, when will it expire, the machine begins to work.
//The physical green led will turn on to tell users the food is fresh. 
//The available light on blynk will turn on to show the box is occupied.
//The green led in Blynk will also turn on and the lcd tells users:"Food is Fresh. Time Left: XXX".

//When the time goes to "food is going to bad", The physical yellow led will turn on to remind users to consume the food as soon as possible.
//Users will get a message on their mobile phone or e-mail tell them the food type, when will it expire, and please consume it soon.
//The yellow led in Blynk will also turn on and the lcd tells users:"Hurry up. Time Left: XXX ".
//When users open the fridge, the buzzer will make sound to attract users' attention to remind them to grab the food as soon as possible.

//When the time goes to "food is bad", The physical red led will turn on to tell users that food is bad and please throw it away.
//Users will get a message on their mobile phone or e-mail tell them the food type, and it's already expired, please throw it away.
//The red led in Blynk will also turn on and the lcd tells users:"Food is Bad, please throw it away!".
//When users open the fridge, the buzzer will make sound to attract users' attention to remind them to throw the food away as soon as possible.

//When users take the food outside of the box, the timer resets and available light on Blynk turn off to tell people the box is available now.
//Users will get a message on their mobile phone or e-mail tell them the food box is cleared up and ready to work again.
//The lcd on Blynk will also display: "Box is Empty". 

#define BLYNK_PRINT Serial
#include <SPI.h>
#include <Ethernet.h>
#include <BlynkSimpleEthernet.h>

char auth[] = "8cdd6bbe56bd43eebcb3518d8f67e24d";

//Physical LEDs
const int greenLedPin = D7;
const int yellowLedPin = D3;
const int redLedPin = C5;

//Time benchmarks
long freshTime = 20000;
long badTime = 40000;

unsigned long currentTime = 0;

//Sense door is open or not
int photocellValue;

int tempValue;

const int piezoPin = A1;

const int switchPin1 = A5;
const int switchPin2 = A4;
const int switchPin3 = A3;
const int switchPin4 = A2;

//Digital dispaly in Blynk
WidgetLCD b_lcd(V1);
WidgetLED b_led1(V2);
WidgetLED b_led2(V3);
WidgetLED b_led3(V4);
WidgetLED b_led4(V5);

//Chech whether there is food in the box
bool checkButtons()
{

    bool buttonPressed = false; 
    
    if(digitalRead(switchPin1) == HIGH)
    {
        buttonPressed = true;
    }
    else if(digitalRead(switchPin2) == HIGH)
    {
        buttonPressed = true;
    } 
    else if(digitalRead(switchPin3) == HIGH)
    {
        buttonPressed = true;
    }
    else if(digitalRead(switchPin4) == HIGH)
    {
        buttonPressed = true;
    }
    return buttonPressed;
}

void display_clear(){
    b_led1.off();
    b_led2.off();
    b_led3.off();
    b_led4.off();
}
void display_somethingInBox()
{
    b_led1.off();
    b_led2.off();
    b_led3.off();
    b_led4.on();
}

void setup()
{
    //start communication lines
    Serial.begin(9600);
    delay(5000);
    Blynk.begin(auth, IPAddress(45,55,130,102), 8442);
    
    //clear display
    display_clear();

    //set app display
    lcd.print(0, 0, "Box is Empty");
}



void loop() { 
    
//check buttons

//if a button is pressed, start timer 

if(checkButtons()){
currentTime = millis(); 
photocellValue = analogRead(B0);
tempValue = analogRead(A4);


//display something is in box
display_somethingInBox();
  
Serial.print("Sensor Value:");
Serial.println(photocellValue);
//delay(200);
Serial.print("Time: ");
Serial.println(currentTime);
delay(1000);

//Check food is fresh
if(currentTime > 0 && currentTime <= freshTime) {
  
  //if food is fresh, turn physical green light on
  digitalWrite(greenLedPin, HIGH);
  digitalWrite(yellowLedPin, LOW);
  digitalWrite(redLedPin, LOW);
  
  //if food is fresh, LCD displays
  Blynk.run();
  lcd.print(0, 0, "Food is Fresh. Time Left:";
  lcd.print(0, 1, (40000-currentTime)/1000);
  delay(100);
  
  //Send email notification to users; Info: food name, left time, timer on
  
  //Turn digital green light on
  b_led1.on();
  b_led2.off();
  b_led3.off();
  b_led4.on();
 }

//Check food is going to be bad
if(currentTime > freshTime && currentTime <= badTime) {
 
  //if food is going to be bad, turn physical LED yellow on
  digitalWrite(greenLedPin, LOW);
  digitalWrite(yellowLedPin, HIGH);
  digitalWrite(redLedPin, LOW);
  
  //if food is going to be bad, LCD displays
  Blynk.run();
  lcd.print(0, 0, "Hurry up. Time Left:";
  lcd.print(0, 1, (40000-currentTime)/1000);
  delay(100);
  
  //Send email notification to users; Info: food name, left time, Hurry up
  
  //if food is going to be bad, turn digital yellow light on
  b_led1.off();
  b_led2.on();
  b_led3.off();
  b_led4.on();
  
  //if fridge is open, buzzer makes sound. if fridge is close, no sound
  if(photocellValue <= 200) {noTone(piezoPin);}
  else {tone(piezoPin, 330);}
  }

//Check food is bad
if(currentTime > badTime) {
 
  //if food is bad, turn physical LED red on 
  digitalWrite(greenLedPin, LOW);
  digitalWrite(yellowLedPin, LOW);
  digitalWrite(redLedPin, HIGH);
  
  //if food is bad, turn LCD display
  Blynk.run();
  lcd.print(0, 0, "Food is Bad,");
  lcd.print(0, 1, "Please throw it away!";
  delay(100);
  
  //Send email notification to users; Info: food name, food is bad, throw it away
  
  //if food is bad, turn digital red light on
  b_led1.off();
  b_led2.off();
  b_led3.on();
  b_led4.on();
  
  //if fridge is open, buzzer makes sound. if fridge is close, no sound
  if(photocellValue <= 200) {noTone(piezoPin);}
  else {tone(piezoPin, 660);}
  }
  }
  else{delay(100);}
}

```
