/**
 * Main Arduino source code to try and interface the Arduino with the
 * PIXHAWK.
 * 
 * Copyright (C) 2014  Chua Shao Wei
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation, either version 3 of the License, or
 * (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program.  If not, see <http://www.gnu.org/licenses/>.
 */
    
/**
 * Declare the array for the data
 *
 * data[0] is the origin XBee
 * data[1] is the destination XBee
 * data[2] is the integer value to be displayed on 7-segment display
 * data[3] is mode option
 *
 * For the XBee identifiers:
 * coordinator is coordinator's XBee
 * router is router's XBee
 * endDevice is end device's XBee
 *
 * Order of the signals is as such:
 *   coordinator   <-->  router  <--> end device
 * (groundstation) <--> (master) <-->  (slave)
 *
 * Modes:
 * mode_0 - Display same number on both Arduinos
 * mode_1 - Display number on router Arduino.
 *          If data[1] is set to endDevice, send it out to endDevice, delay,
 *          add 1, display it, add another 1, send it back to router, display
 *          new number, add another 1, send it back to coordinator.
 */
char data[] = {0, 0, 0, 0};
const char coordinator = '1';
const char router = '2';
const char endDevice = '3';
const char mode_0 = '0';
const char mode_1 = '1';

// configure pins for LEDs
const int greenLed = 6;
const int redLed = 5;
const int bothLed = 0; // can use any number as this is for conditions
 const int throttle=7;
 const int starter=A3;
 const int forward=4;
 const int roll=3;
// configure pins for 7 segment display
const int a = 8; // a is pin 8
const int b = 9; // b is pin 9
const int c = 10; // c is pin10
const int d = 11; // d is pin 11

unsigned int i; // integer for for loops

// function to display a number on 7 segment display
void display(char number) {
  switch (number) {
    case '0':
    { int power=0;
      int power1=0;
      analogWrite(throttle,power);
      analogWrite(forward,power1);
      analogWrite(roll,power1);
 
      digitalWrite(a,0);
      digitalWrite(b,0);
      digitalWrite(c,0);
      digitalWrite(d,0);
      break;}
      
    case '1':
      {  int power=0;
       int power1=0;
    
      analogWrite(throttle,power);
      analogWrite(forward,power1);
      analogWrite(roll,power1);
 
      delay(1000);
   
     power=255;
     analogWrite(throttle,power);
     analogWrite(forward,power1);
     analogWrite(roll,power1);
     delay(10000);
   
     power=0;
     analogWrite(throttle,power);
     analogWrite(forward,power1);
     analogWrite(roll,power1);
     delay(10);
      digitalWrite(a,1);
      digitalWrite(b,0);
      digitalWrite(c,0);
      digitalWrite(d,0);
      break;}
    case '2':
      {  int power=0;
       int power1=0;
    
      analogWrite(throttle,power);
      analogWrite(forward,power1);
      analogWrite(roll,power1);
 
      delay(1000);
   
     power=255;
     analogWrite(throttle,power);
     analogWrite(forward,power1);
     analogWrite(roll,power1);
     delay(10000);
   
     power=0;
     analogWrite(throttle,power);
     analogWrite(forward,power1);
     analogWrite(roll,power1);
     delay(10);
      digitalWrite(a,0);
      digitalWrite(b,1);
      digitalWrite(c,0);
      digitalWrite(d,0);
      break;}
    
    case '3':
      digitalWrite(a,1);
      digitalWrite(b,1);
      digitalWrite(c,0);
      digitalWrite(d,0);
      break;
    case '4':
      digitalWrite(a,0);
      digitalWrite(b,0);
      digitalWrite(c,1);
      digitalWrite(d,0);
      break;
    case '5':
      digitalWrite(a,1);
      digitalWrite(b,0);
      digitalWrite(c,1);
      digitalWrite(d,0);
      break;
    case '6':
      digitalWrite(a,0);
      digitalWrite(b,1);
      digitalWrite(c,1);
      digitalWrite(d,0);
      break;
    case '7':
      digitalWrite(a,1);
      digitalWrite(b,1);
      digitalWrite(c,1);
      digitalWrite(d,0);
      break;
    case '8':
      digitalWrite(a,0);
      digitalWrite(b,0);
      digitalWrite(c,0);
      digitalWrite(d,1);
      break;
    case '9':
      digitalWrite(a,1);
      digitalWrite(b,0);
      digitalWrite(c,0);
      digitalWrite(d,1);
      break;
    default:
      blink(bothLed, 1, 5);
      break;
  }
}

// Blink the green LED
void blinkGreen(int rate) {
  digitalWrite(greenLed,1);
  delay(rate);
  digitalWrite(greenLed,0);
  delay(rate);
}

// Blink the red LED
void blinkRed(int rate) {
  digitalWrite(redLed,1);
  delay(rate);
  digitalWrite(redLed,0);
  delay(rate);
}

// Blink both LEDs
void blinkBoth(int rate) {
  digitalWrite(greenLed,1);
  digitalWrite(redLed,1);
  delay(rate);
  digitalWrite(greenLed,0);
  digitalWrite(redLed,0);
  delay(rate);
}

// Function to blink the LEDs
void blink(int led, int times, int rate) {
  for (i = 0; i < times; i++) {
    switch (led) {
      case greenLed: // green LED
        blinkGreen(rate);
        break;
      case redLed: // red LED
        blinkRed(rate);
        break;
      case bothLed: // both LEDs
        blinkBoth(rate);
      default:
        break;
    }
  }
}

void returnStatus(bool status) {
  if (status == true)
    Serial.write("Confirmed direction.");
  else if (status == false) {
    Serial.write("Error! Please contact your system administrator."); // But I'm the system administrator... :(
    blink(redLed, 3, 10);
  }
}

char addOne(char oldNumber) {
  char newNumber;

  if(oldNumber < '9')
    newNumber = (char)((int)(oldNumber) + 1);
  else
    newNumber = '0';

  return newNumber;
}

// function meant for router Arduino
void from_coordinator_do() {
  display(data[2]);

  if (data[1] == endDevice) {
    // if data[1] returns endDevice, change data[0] to reflect new origin, and 
    // write data[] array to serial port (XBee) for transmission
    delay(5000); // delay kept long for demo purposes, feel free to change
    data[0] = router;
    for (i = 0; i < 4; i++) {
      Serial.write(data[i]);
    } 
  }
}

// function for endDevice Arduino
void from_router_do() {
  // checks if endDevice is intended destination for data[]
  if (data[1] == endDevice) {
    if (data[3] == mode_0) {
      delay(2000); // delay kept long for demo purposes, feel free to change
      display(data[2]);
      data[0] = endDevice;
      data[1] = router;
      for (i = 0; i < 4; i++) {
        Serial.write(data[i]);
      }
    }
    else if (data[3] == mode_1) {
      // adds 1 to data[2] and displays it
      data[2] = addOne(data[2]);
      display(data[2]);
      delay(2000); // delay kept long for demo purposes, feel free to change
      // adds another 1 to data[2] before sending it back
      data[2] = addOne(data[2]);
      // change data[0] to reflect new origin, change data[1] to reflect new 
      // destination, and write data[] array to serial port (XBee) for transmission
      data[0] = endDevice;
      data[1] = router;
      for (i = 0; i < 4; i++) {
        Serial.write(data[i]);
      }
    }
    else
      // error
      blink(redLed, 3, 10);
  }
}

// function also meant for router Arduino
void from_endDevice_do() {
  if (data[3] == mode_0) {
    delay(5000); // delay kept long for demo purposes, feel free to change
    blink(greenLed, 3, 10);
    returnStatus(true);
  }
  else if (data[3] == mode_1) {
    delay(5000); // delay kept long for demo purposes, feel free to change
    display(data[2]);
    // adds 1 to data[2], change data[0] to reflect new origin, change data[1]
    // to reflect new destination. and write data[] array to serial port (XBee) 
    // for transmission
    data[2] = addOne(data[2]);
    data[0] = router;
    data[1] = coordinator;
    //for (i = 0; i < 4; i++) {
    //  Serial.write(data[i]);
    //}
    Serial.write(" Received:");
    Serial.write(data[2]);
    Serial.write(" ");
    delay(500);
  }
  else
    returnStatus(false);
}

void setup() {
  // Initialise the serial port with baud rate of 57600 to match the PIXHAWK's 
  // baud rate of 57600
  // TODO: change the baud rate
  Serial.begin(9600);

  // Initialise the 6 I/O pins as output pins
  // LED pins
   pinMode(greenLed,OUTPUT);
  pinMode(redLed,OUTPUT);
  pinMode (throttle, OUTPUT);
  pinMode (forward, OUTPUT);
  pinMode (roll, OUTPUT);

  pinMode(greenLed,OUTPUT);
  pinMode(redLed,OUTPUT);

  // 7 segment decoder pins
  pinMode(a,OUTPUT);
  pinMode(b,OUTPUT);
  pinMode(c,OUTPUT);
  pinMode(d,OUTPUT);

  /**
   * From XBee-Arduino's RemoteAtCommand.pde example:
   *
   * When powered on, XBee radios require a few seconds to start up
   * and join the network.
   * During this time, any packets sent to the radio are ignored.
   * Series 2 radios send a modem status packet on startup.
   *
   * it took about 4 seconds for mine to return modem status.
   * In my experience, series 1 radios take a bit longer to associate.
   * Of course if the radio has been powered on for some time before the sketch runs,
   * you can safely remove this delay.
   * Or if you both commands are not successful, try increasing the delay.
   */
  delay(4000);

  // ready indicator
  blink(bothLed, 3, 10);
}

void loop() {
  for (i = 0; i < 4; i++) {
    while (Serial.available() == false);
    char inByte = Serial.read();
    data[i] = inByte;
  }

  // indicate received status using LEDs
  blink(greenLed, 1, 5);

  // switch function to check for data's origin location and respond respectively
  switch (data[0]) {
    case coordinator: // meant for router Arduino
      // if data received is from coordinator
      from_coordinator_do();
      break;
    case router: // meant for endDevice Arduino
      // if data received is from router
      from_router_do();
      break;
    case endDevice: // also meant for router Arduino
      from_endDevice_do();
      break;
    deafult:
      // error
      blink(redLed, 3, 10);
      break;
  }
}
