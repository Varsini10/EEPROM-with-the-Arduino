# EEPROM-with-the-Arduino
// Include the 12C Wire Library
#include<Wire.h>
// Include the Servo Library
#include<Servo.h>
// EEPROM 12C Address
#define EEPROM_I2C_ADDRESS 0x50
// Analog pin for potentiometer
int analogPin = 0;
// Integer to hold potentiometer value
int val = 0;
// Byte to hold data read from EEPROM
int readVal = 0;
// Integer to hold the number of addresses to fill
int maxaddress = 1500;
//Create a Servo object
Servo myservo;
// Function to write to EEPROM
void writeEEPROM(int address,byte val, int i2c_address)
{
// Begin transmission to 12C EEPROM
Wire.beginTransmission(i2c_address);
// Send memory address as two 8-bit bytes
Wire.write((int)(address >> 8)); //MSB
Wire.write((int)(address & 0xFF)); //LSB
// Send data to be stored
Wire.write(val);
// End the transmission
Wire.endTransmission();
// Add 5ms delay for EEPROM
delay (5);
}
//function to read from EEPROM
byte readEEPROM(int address,int i2c_address)
{
// Define byte for received data
byte rcvData = 0xFF;
// Begin transmission to 12C EEPROM 
Wire.beginTransmission(i2c_address);
// Send memory address as two 8-bit bytes 
Wire.write((int)(address>> 8)); // MSB
Wire.write((int) (address & 0xFF)); // LSB
// End the transmission
Wire.endTransmission();
// Request one byte of data at the current memory address 
Wire.requestFrom(i2c_address, 1);
// Read the data and assign it to variable 
rcvData = Wire.read();
// Return the data as function output 
return rcvData;
}
void setup()
{
// Connect to 12C bus as master
Wire.begin();
// Setup Serial Monitor 
Serial.begin(9600);
// Attach servo on pin 9 to the servo object
myservo.attach(9);
// Print to Serial Monitor 
Serial.println("Start Recording...");
// Run until the maximum address is reached
for (int address = 0; address <= maxaddress; address++) {
// Read pot and map to range of 0.180 for servo 
val = map(analogRead(analogPin), 0, 1023, 0, 180);
// Write to the servo
// Delay to allow servo to settle in position
myservo.write(val);
delay(15);
// Record the position in the external EEPROM 
writeEEPROM(address, val, EEPROM_I2C_ADDRESS);
// Print to Serial Monitor
Serial.print("ADDR = ");
Serial.print(address);
Serial.print("\t"); 
Serial.println(val);
}
// Print to Serial Monitor
Serial.println("Recording Finished!");
// Delay 5 Second
delay(5000)
// Print to Serial Monitor
Serial.println("Begin Playback...")
// Run until maximum address is reached
for (int address = 0; address <= maxaddress; address++) {
// Read value from EEPROM
readVal = readEEPROM(address, EEPROM_I2C_ADDRESS);
// Write to the servo
// Delay to allow servo to settle in position
// Convert value to integer for servo
myservo.write(readVal);
delay(15);
// Print to Serial Monitor
Serial.print("ADDR = ");
Serial.print(address); 
Serial.print("\t");
Serial.println(readVal);
}
// Print to Serial Monitor
Serial.println("Playback Finished!");
}
void loop() {
 // Nothing in loop
}
