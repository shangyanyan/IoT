```
#include <LiquidCrystal.h>
LiquidCrystal lcd(12,11,5,4,3,2);

int switchPin = 13;
unsigned long currentTime = 0;

boolean buttonHasEverBeenPressed = false;

const int greenLedPin = 8;
const int yellowLedPin = 9;
const int redLedPin = 10;

long freshTime = 20000;
long badTime = 40000;

int photocellValue;

const int piezoPin = 7;

void setup() {
  
Serial.begin(9600);

lcd.begin(16,2);
lcd.print("Container for");
lcd.setCursor(0,1);
lcd.print("Vegetables");
  
pinMode(greenLedPin, OUTPUT);
pinMode(yellowLedPin, OUTPUT);
pinMode(redLedPin, OUTPUT);
pinMode(piezoPin, OUTPUT);
pinMode(switchPin, INPUT);

photocellValue = analogRead(A0);
 
testTimer();
}

// the loop routine runs over and over again forever:
void loop() {

if (isButtonPressed() == true)
  {
    Serial.println("yes");
    resetTimer();
  }

if (buttonHasEverBeenPressed == true)
  {
    Serial.println(getTimerTime());
  }
 
  illuminateLED();

  delay(1);  

}
```
```
boolean isButtonPressed()
{
  boolean retval = false;
  if (digitalRead(switchPin) == 1)
  { retval = true;
    buttonHasEverBeenPressed = true;
  }
  return retval;
}
//timerstuff

void testTimer()
{
  Serial.print("millis = ");
  Serial.println(millis());
  resetTimer();
  delay(1000);
  Serial.print("getTimerTime = ");
  Serial.println(getTimerTime());
}
void resetTimer()
{
  //remember what time it was when this function was called, store it in timerTime
  currentTime = millis();
}

unsigned long getTimerTime()
{
  //whenever this function is called, retun how much time has passed since timer was reset.
  return (millis() - currentTime);
}

void illuminateLED()
{
  
  if (buttonHasEverBeenPressed == true)
  {
    
    if (buttonHasEverBeenPressed == true){

Serial.print("Senor Value:");
Serial.println(photocellValue);
delay(200);
Serial.print("Time: ");
Serial.println(currentTime);
delay(1000);

if(currentTime > 0 && currentTime <= freshTime) {
  digitalWrite(greenLedPin, HIGH);
  digitalWrite(yellowLedPin, LOW);
  digitalWrite(redLedPin, LOW);
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Eat it in:");
  lcd.setCursor(0,1);
  lcd.print((40000-currentTime)/1000);
  lcd.setCursor(2,1);
  lcd.print("s");}

if(currentTime > freshTime && currentTime <= badTime) {
  digitalWrite(greenLedPin, LOW);
  digitalWrite(yellowLedPin, HIGH);
  digitalWrite(redLedPin, LOW);
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Eat it in:");
  lcd.setCursor(0,1);
  lcd.print((40000-currentTime)/1000);
  lcd.setCursor(2,1);
  lcd.print("s");
  
  if(photocellValue <= 600) {noTone(piezoPin);}
  else {tone(piezoPin, 330);}
  }

if(currentTime > badTime) {
  digitalWrite(greenLedPin, LOW);
  digitalWrite(yellowLedPin, LOW);
  digitalWrite(redLedPin, HIGH);
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Please Throw");
  lcd.setCursor(0,1);
  lcd.print("this shit");
  
  if(photocellValue <= 600) {noTone(piezoPin);}
  else {tone(piezoPin, 660);}
  }
  }
}
}
```
