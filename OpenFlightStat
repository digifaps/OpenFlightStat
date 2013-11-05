#include <math.h>
#include "U8glib.h"
#include <SPI.h>
#include <SD.h>

const int chipSelect_RTC = 11; //chip select for the RTC
const int chipSelect_SD = 12;//chip select for the SD card

int ledr1 = 5 ;
int ledg1 = 6 ;
int ledb1 = 7 ;

int ledr2 = 8 ;
int ledg2 = 9 ;
int ledb2 = 10 ;

int InteruptPin = 18;
int lightPin = 2 ;
int Writeled = 22 ;
int RTCled = 23 ;

String timedate;

int RTC_init(){ 
	  //pinMode(chipSelect_RTC,OUTPUT); // chip select
	  // start the SPI library:
	  SPI.begin();
	  SPI.setBitOrder(LSBFIRST); 
	  SPI.setDataMode(SPI_MODE3); // both mode 1 & 3 should work 
	  //set control register
          digitalWrite(chipSelect_SD, HIGH);
	  digitalWrite(chipSelect_RTC, LOW);  
	  SPI.transfer(0x8E);
	  SPI.transfer(0x67); //60= disable Osciallator and Battery SQ wave @1hz, temp compensation, Alarms enabled interupt enabled
	  digitalWrite(chipSelect_RTC, HIGH);
          digitalWrite(chipSelect_SD, HIGH);
}

String ReadTimeDate(){
	String temp;
        RTC_init();
	int TimeDate [7]; //second,minute,hour,null,day,month,year		
	for(int i=0; i<=6;i++){
		if(i==3)
			i++;
		digitalWrite(chipSelect_SD, HIGH);
                digitalWrite(chipSelect_RTC, LOW);
		digitalWrite(RTCled, HIGH);
                SPI.setBitOrder(MSBFIRST);
                SPI.transfer(i+0x00); 
		unsigned int n = SPI.transfer(0x00);        
		digitalWrite(chipSelect_RTC, HIGH);
		digitalWrite(RTCled, LOW);
                int a=n & B00001111;    
		if(i==2){	
			int b=(n & B00110000)>>4; //24 hour mode
			if(b==B00000010)
				b=20;        
			else if(b==B00000001)
				b=10;
			TimeDate[i]=a+b;
		}
		else if(i==4){
			int b=(n & B00110000)>>4;
			TimeDate[i]=a+b*10;
		}
		else if(i==5){
			int b=(n & B00010000)>>4;
			TimeDate[i]=a+b*10;
		}
		else if(i==6){
			int b=(n & B11110000)>>4;
			TimeDate[i]=a+b*10;
		}
		else{	
			int b=(n & B01110000)>>4;
			TimeDate[i]=a+b*10;	
			}
	}
	//temp.concat(TimeDate[4]);
	//temp.concat("/") ;
	//temp.concat(TimeDate[5]);
	//temp.concat("/") ;
	//temp.concat(TimeDate[6]);
	//temp.concat(" ") ;
        if( TimeDate[2] < 10 )
         temp.concat("0");
	temp.concat(TimeDate[2]);
	temp.concat(":") ;
        if( TimeDate[1] < 10 )
          temp.concat("0");
	temp.concat(TimeDate[1]);
	temp.concat(":") ;
        if( TimeDate[0] < 10 )
          temp.concat("0");
	temp.concat(TimeDate[0]);
  digitalWrite(chipSelect_SD, LOW);
  digitalWrite(chipSelect_RTC, HIGH);  
  return(temp);
 }

// setup u8g object
U8GLIB_ST7920_128X64 u8g(2, 3, 4, U8G_PIN_NONE); 

double Thermister(int RawADC) {
  double Temp;
  Temp = log(((10240000/RawADC) - 10000));
  Temp = 1 / (0.001129148 + (0.000234125 * Temp) + (0.0000000876741 * Temp * Temp * Temp));
  Temp = Temp - 273.15;   // Convert Kelvin to Celcius
 
  return Temp;
}

void draw(void) {
  timedate = ReadTimeDate();
  // graphic commands to redraw the complete screen should be placed here  
  digitalWrite(4,LOW);
  u8g.setFont(u8g_font_fixed_v0);
  u8g.setPrintPos(0, 7);
  u8g.print("Engine");
  u8g.drawLine(0, 10, 63, 10 );
  u8g.setPrintPos(0, 20); 
  u8g.print("Temp 1:");
  u8g.print(int(Thermister(analogRead(0))));
  u8g.print("C");
  u8g.setPrintPos(0, 29); 
  u8g.print("Temp 2:");
  u8g.print(int(Thermister(analogRead(1))));
  u8g.print("C");
  u8g.drawLine(0, 32, 63, 32 );
  u8g.setPrintPos(0, 42);
  u8g.print("Fuel");
  u8g.drawLine(0, 45, 63, 45 );
  u8g.setPrintPos(0, 55 ); 
  u8g.print("Left  :");
  u8g.setPrintPos(0, 64 ); 
  u8g.print("Use    :");
  u8g.print(analogRead(lightPin)/100.0);
  u8g.setPrintPos(0, 73 ); 
  u8g.print("Avg/h :");
  u8g.setPrintPos(0, 82 ); 
  u8g.setPrintPos(8, 128 ); 
  u8g.print(timedate);
  digitalWrite(4,HIGH);
}

void EngineLed() {
  if (int(Thermister(analogRead(0)))<25) {
    // Blue LED 
    digitalWrite(ledb1, HIGH);  
    digitalWrite(ledg1, LOW);
    digitalWrite(ledr1, LOW);
    } 
  else if (int(Thermister(analogRead(0)))>30) {
    // Red LED
    digitalWrite(ledr1, HIGH);
    digitalWrite(ledb1, LOW);
    digitalWrite(ledg1, LOW);
    } 
  else {  
    // Green LED
    digitalWrite(ledr1, LOW);
    digitalWrite(ledb1, LOW);
    digitalWrite(ledg1, HIGH);
  }
  if (int(Thermister(analogRead(1)))<25) {
    // Blue LED 
    digitalWrite(ledb2, HIGH); 
    digitalWrite(ledg2, LOW);
    digitalWrite(ledr2, LOW);
    } 
  else if (int(Thermister(analogRead(1)))>30) {
    // Red LED
    digitalWrite(ledr2, HIGH);
    digitalWrite(ledb2, LOW);
    digitalWrite(ledg2, LOW);
    } 
  else {  
    // Green LED
    digitalWrite(ledr2, LOW);
    digitalWrite(ledb2, LOW);
    digitalWrite(ledg2, HIGH);
  }
}

void LogWrite() {
  digitalWrite(chipSelect_SD, LOW);
  // open the file. note that only one file can be open at a time,
  // so you have to close this one before opening another.
  File dataFile = SD.open("datalog.txt", FILE_WRITE);
  // if the file is available, write to it:
  if (dataFile) {
    digitalWrite(Writeled, HIGH);
    dataFile.print(timedate);
    dataFile.print(" ");
    dataFile.print("Temp 1 : ");
    dataFile.print(int(Thermister(analogRead(0))));
    dataFile.print("C ");
    dataFile.print("Temp 2 : ");
    dataFile.print(int(Thermister(analogRead(1))));
    dataFile.print("USAGE : ");
    dataFile.print(analogRead(lightPin)/100.0);
    dataFile.println(" L");
    dataFile.close();
    digitalWrite(chipSelect_SD, HIGH);
    digitalWrite(Writeled, LOW);
    // print to the serial port too:
    Serial.print(int(Thermister(analogRead(1))));
    Serial.println("C");
  }  
  // if the file isn't open, pop up an error:
  else {
    Serial.println("error opening datalog.txt");
  }
}

void setup(void) {
  // flip screen
  u8g.setRot90();
  Serial.begin(9600);
  attachInterrupt(5, Trigger, RISING);
  pinMode(chipSelect_SD, OUTPUT);
  pinMode(chipSelect_RTC, OUTPUT);
  pinMode(Writeled, OUTPUT);
  pinMode(RTCled, OUTPUT);
  pinMode(ledr1, OUTPUT);
  pinMode(ledg1, OUTPUT);
  pinMode(ledb1, OUTPUT);
  pinMode(ledr2, OUTPUT);
  pinMode(ledg2, OUTPUT);
  pinMode(ledb2, OUTPUT);
  pinMode(53, OUTPUT);
  pinMode(4, OUTPUT);
  digitalWrite(4, HIGH);
  digitalWrite(chipSelect_SD, HIGH);
  digitalWrite(chipSelect_RTC, HIGH);
  Serial.print("Initializing RTC ...");
  RTC_init();
  Serial.print("Initializing SD card ...");
  if (!SD.begin(chipSelect_SD)) {
    Serial.println("Card failed, or not present");
    return;
  }
  Serial.println("card initialized.");
}

void Trigger(void) {
  // picture loop
  u8g.firstPage();  
  do {
    draw();
  } while( u8g.nextPage() );
  LogWrite();  
  EngineLed();
}
  
  

void loop(void) {
  // Nothing to see here
 }
