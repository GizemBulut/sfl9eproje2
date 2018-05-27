# sfl9eproje2
meteoroloji istasyonu
#include <DHT.h> 
#include <MQ2.h>
#include <SFE_BMP180.h>.
#include <Wire.h> 
#include <LCD5110_Graph.h> 

#define DHTPIN 2 
#define DHTTYPE DHT22 
#define ALTITUDE 1285.0 

DHT dht(DHTPIN, DHTTYPE); 
LCD5110 myGLCD(8,9,10,11,12); 
SFE_BMP180 bmp180;
extern uint8_t SmallFont[];
extern uint8_t MediumNumbers[];
extern uint8_t BigNumbers[];
extern uint8_t TinyFont[];
extern uint8_t nem_bitmap[];
extern uint8_t sicaklik_bitmap[];
extern uint8_t basinc_bitmap[];
extern uint8_t yagmurlu[];
extern uint8_t gunesli[];
extern uint8_t karli[];
extern uint8_t uparrow[];
extern uint8_t downarrow[];
extern uint8_t equal[];

int Analog_Input = A0;
int lpg, co, smoke;
int gecenzaman=0;
int basinckarsilastirma[24] ={0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0}; //
 int firsttime =0;  
const int switchpin =6; 
int toplam=0;
int ortalama=0;
int fark=0;
int tahmin;
int yonelim;
MQ2 mq2(Analog_Input);

void setup(){
  Serial.begin(9600);
  pinMode(switchpin, INPUT);
  pinMode(A3, INPUT);
  pinMode(7, OUTPUT);
  pinMode(13,OUTPUT);
  pinMode(3, OUTPUT);
  digitalWrite(13,LOW);  
  digitalWrite(7,HIGH);
  dht.begin(); 
  bmp180.begin(); 
  mq2.begin();
  myGLCD.InitLCD(); 
  myGLCD.setContrast(70); 
}

void loop() {
  
int nem = dht.readHumidity();
int sicaklik = dht.readTemperature(); 
int basinc = readPressure();
  
float sicaklikf = dht.readTemperature(); 
float nemf = dht.readHumidity();  
float basincf = readPressure(); 


  float* values= mq2.read(true); 
  //lpg = values[0];
  lpg = mq2.readLPG();
  //co = values[1];
  co = mq2.readCO();
  //smoke = values[2];
  smoke = mq2.readSmoke();

toplam = basinckarsilastirma[0] + basinckarsilastirma[1] + basinckarsilastirma[2] + basinckarsilastirma[3] + basinckarsilastirma[4] + basinckarsilastirma[5] + basinckarsilastirma[6] + basinckarsilastirma[7] + basinckarsilastirma[8] + basinckarsilastirma[9] + basinckarsilastirma[10] + basinckarsilastirma[11] + basinckarsilastirma[12] + basinckarsilastirma[13] + basinckarsilastirma[14] + basinckarsilastirma[15] + basinckarsilastirma[16] + basinckarsilastirma[17] + basinckarsilastirma[18] + basinckarsilastirma[19] + basinckarsilastirma[20] + basinckarsilastirma[21] + basinckarsilastirma[22] + basinckarsilastirma[23]; 
ortalama = toplam / 24; 
fark = basinc - ortalama; 



    
  if (firsttime == 0) { 
    myGLCD.clrScr(); 
    {  
    myGLCD.setFont(TinyFont);
     for (int i=84; i>=7; i--)
    myGLCD.print("METEROLOJI ISTASYONU", i, 37);
    delay(20);
    myGLCD.update();
    }
    
    for (int i=24; i>=19; i--) {
    myGLCD.clrScr();
    myGLCD.setFont(TinyFont);
    myGLCD.print("SENSORLER OKUNUYOR", 8, 37);
    myGLCD.print("METEROLOJI ISTASYONU", 5, i);
    delay(50);
    myGLCD.update();
    }
    
    delay(7500);
    firsttime= 1; 
  
  }
  else {
  }

  //SICAKLIK GÖSTERİLİR...
      myGLCD.clrScr();
      myGLCD.drawBitmap(0, 0, sicaklik_bitmap, 84, 48);
      myGLCD.setFont(BigNumbers);
      myGLCD.printNumF(sicaklikf,1,12 ,18);
      myGLCD.setFont(SmallFont);
      myGLCD.print("C",72,27);
      myGLCD.setFont(TinyFont);
      myGLCD.print("o",70,22);
      myGLCD.update();
      delay(2500);
 
      
  // NEM GÖSTERİLİR...
      myGLCD.clrScr();
      myGLCD.drawBitmap(0, 0, nem_bitmap, 84, 48);
      myGLCD.setFont(BigNumbers);
      myGLCD.printNumF(nemf,1,20, 20);
      myGLCD.setFont(SmallFont);
      myGLCD.print("%",9,28);
      myGLCD.update();
      delay(2500);
   //GAZ GÖSTERİLİR
      myGLCD.clrScr();
      myGLCD.setFont(SmallFont);
      myGLCD.print("GAZ",35,0);
      myGLCD.setFont(SmallFont);
      myGLCD.print("LPG:",0,10);
      myGLCD.printNumI(lpg,25,10);
      myGLCD.print("ppm",65,10);
      myGLCD.print("CO:",0,25);
      myGLCD.printNumI(co,25,25);
      myGLCD.print("ppm",65,25);
      myGLCD.print("SMOKE:",0,40);
      myGLCD.printNumI(smoke,40,40);
      myGLCD.print("ppm",65,40);
      myGLCD.update();
      delay(5000);
      
   // BASINÇ GÖSTERİLİR...   
  
      myGLCD.clrScr();
      myGLCD.drawBitmap(0, 0, basinc_bitmap, 84, 48);
      myGLCD.setFont(BigNumbers);
      myGLCD.printNumF(basincf,1,0,18);
      myGLCD.setFont(TinyFont);
      myGLCD.print("hPa",40,43);
      myGLCD.update();
      delay(2500);
      
       // BASINÇ ÇİZELGESİ 
  
      myGLCD.clrScr();
      myGLCD.drawRect(35, 10, 61, 47);
      myGLCD.setFont(TinyFont);
      myGLCD.print("BASINC (SON 24 SAAT)",6,0);
      myGLCD.print("1000 hPa",0,42);
      myGLCD.print("1050 hPa",0,11);
   
    for (int i=36; i<61; i++)
    {
     int koordinat;
     switch (i) {  
     case 36:
     koordinat= map(basinckarsilastirma[0],990,1050,46,11);
     break;   
     case 37:
     koordinat= map(basinckarsilastirma[1],990,1050,46,11);
     break;
     case 38:
     koordinat= map(basinckarsilastirma[2],990,1050,46,11);
     break;
     case 39:
     koordinat= map(basinckarsilastirma[3],990,1050,46,11);
     break;
     case 40:
     koordinat= map(basinckarsilastirma[4],990,1050,46,11);
     break;
     case 41:
     koordinat= map(basinckarsilastirma[5],990,1050,46,11);
     break;
     case 42:
     koordinat= map(basinckarsilastirma[6],990,1050,46,11);
     break;
     case 43:
     koordinat= map(basinckarsilastirma[7],990,1050,46,11);
     break;
     case 44:
     koordinat= map(basinckarsilastirma[8],990,1050,46,11);
     break;
     case 45:
     koordinat= map(basinckarsilastirma[9],990,1050,46,11);
     break;
     case 46:
     koordinat= map(basinckarsilastirma[10],990,1050,46,11);
     break;
     case 47:
     koordinat= map(basinckarsilastirma[11],990,1050,46,11);
     break;
     case 48:
     koordinat= map(basinckarsilastirma[12],990,1050,46,11);
     break;
     case 49:
     koordinat= map(basinckarsilastirma[13],990,1050,46,11);
     break;
     case 50:
     koordinat= map(basinckarsilastirma[14],990,1050,46,11);
     break;
     case 51:
     koordinat= map(basinckarsilastirma[15],990,1050,46,11);
     break;
     case 52:
     koordinat= map(basinckarsilastirma[16],990,1050,46,11);
     break;
     case 53:
     koordinat= map(basinckarsilastirma[17],990,1050,46,11);
     break;
     case 54:
     koordinat= map(basinckarsilastirma[18],990,1050,46,11);
     break;
     case 55:
     koordinat= map(basinckarsilastirma[19],990,1050,46,11);
     break;
     case 56:
     koordinat= map(basinckarsilastirma[20],990,1050,46,11);
     break;
     case 57:
     koordinat= map(basinckarsilastirma[21],990,1050,46,11);
     break;
     case 58:
     koordinat= map(basinckarsilastirma[22],990,1050,46,11);
     break;
     case 59:
     koordinat= map(basinckarsilastirma[23],990,1050,46,11);
     break;
     }
   
      myGLCD.invPixel(i,koordinat);
      myGLCD.update();
      delay(20);
    }
    
  if (yonelim == 1) { 
    
  for (int c=0; c<4; c++) { 
    
     for (int i=20; i>10; i--){  
     myGLCD.drawBitmap(64, i, uparrow, 24, 24);
     myGLCD.update();
     delay(75);
     }
     for (int y=10; y<20; y++) {  
     myGLCD.drawBitmap(64, y, uparrow, 24, 24);
     myGLCD.update();
     delay(75);
     }
  }
  }
  
   if (yonelim == 0) {  
    
  for (int c=0; c<4; c++) { 
    
     for (int i=62; i<65; i++){ 
     myGLCD.drawBitmap(i, 20, equal, 22, 17);
     myGLCD.update();
     delay(125);
     }
     for (int y=65; y>62; y--) { 
     myGLCD.drawBitmap(y, 20, equal, 22, 17);
     myGLCD.update();
     delay(125);
     }
  }
  }
  
  if (yonelim == -1) { 
    
  for (int c=0; c<4; c++) { 
      
     for (int i=10; i<20; i++) {   
     myGLCD.drawBitmap(64, i, downarrow, 24, 24);
     myGLCD.update();
     delay(75);
      }
     for (int y=20; y>10; y--) { 
     myGLCD.drawBitmap(64, y, downarrow, 24, 24);
     myGLCD.update();
     delay(75);
    }
  }
  }

  delay(2000);


//BASINÇ ORTALAMALARI
      myGLCD.clrScr();
      myGLCD.setFont(TinyFont);
      myGLCD.print("BASINC (SON 24 SAAT)",6,0);
      myGLCD.setFont(SmallFont);
      myGLCD.print("Ortalama: ",0,10);
      myGLCD.printNumI(ortalama,55,10);
      myGLCD.print("Guncel: ",0,25);
      myGLCD.printNumI(basinc,55,25);
      myGLCD.print("Fark: ",5,40);
      myGLCD.printNumI(fark,55,40);
      myGLCD.update();
      delay(5000);  

      ++gecenzaman; 
   }

 switch (gecenzaman){  
   case 180:
   basinckarsilastirma[0] = basinc;
   break;
   
   case 360:
   basinckarsilastirma[1] = basinc;
   if ((basinckarsilastirma[0]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[0]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[0]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 540:
   basinckarsilastirma[2] = basinc;
   if ((basinckarsilastirma[1]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[1]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[1]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 720:
   basinckarsilastirma[3] = basinc;
   if ((basinckarsilastirma[2]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[2]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[2]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 900:
   basinckarsilastirma[4] = basinc;
   if ((basinckarsilastirma[3]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[3]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[3]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 1080:
   basinckarsilastirma[5] = basinc;
   if ((basinckarsilastirma[4]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[4]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[4]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 1260:
   basinckarsilastirma[6] = basinc;
   if ((basinckarsilastirma[5]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[5]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[5]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 1440:
   basinckarsilastirma[7] = basinc;
   if ((basinckarsilastirma[6]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[6]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[6]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 1620:
   basinckarsilastirma[8] = basinc;
   if ((basinckarsilastirma[7]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[7]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[7]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 1800:
   basinckarsilastirma[9] = basinc;
   if ((basinckarsilastirma[8]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[8]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[8]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 1980:
   basinckarsilastirma[10] = basinc;
   if ((basinckarsilastirma[9]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[9]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[9]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 2160:
   basinckarsilastirma[11] = basinc;
   if ((basinckarsilastirma[10]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[10]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[10]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 2340:
   basinckarsilastirma[12] = basinc;
   if ((basinckarsilastirma[11]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[11]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[11]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 2520:
   basinckarsilastirma[13] = basinc;
   if ((basinckarsilastirma[12]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[12]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[12]) == (basinc)) { 
   yonelim=0;
   }
   break;

 case 2700:
   basinckarsilastirma[14] = basinc;
   if ((basinckarsilastirma[13]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[13]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[13]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 2880:
   basinckarsilastirma[15] = basinc;
   if ((basinckarsilastirma[14]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[14]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[14]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 3060:
   basinckarsilastirma[16] = basinc;
   if ((basinckarsilastirma[15]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[15]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[15]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 3240:
   basinckarsilastirma[17] = basinc;
   if ((basinckarsilastirma[16]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[16]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[16]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 3420:
   basinckarsilastirma[18] = basinc;
   if ((basinckarsilastirma[17]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[17]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[17]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 3600:
   basinckarsilastirma[19] = basinc;
   if ((basinckarsilastirma[18]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[18]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[18]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 3780:
   basinckarsilastirma[20] = basinc;
   if ((basinckarsilastirma[19]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[19]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[19]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 3960:
   basinckarsilastirma[21] = basinc;
   if ((basinckarsilastirma[20]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[20]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[20]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
   case 4140:
   basinckarsilastirma[22] = basinc;
   if ((basinckarsilastirma[21]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[21]) < (basinc)) { 
   yonelim=1; 

   }
   if ((basinckarsilastirma[21]) == (basinc)) { 
   yonelim=0;
   }
   break;
   
      case 4320:
   basinckarsilastirma[23] = basinc;
   if ((basinckarsilastirma[22]) > (basinc)) { 
   yonelim=-1;
   }
   if ((basinckarsilastirma[22]) < (basinc)) { 
   yonelim=1;
   }
   if ((basinckarsilastirma[22]) == (basinc)) { 
   yonelim=0;
   }
   break;
 } 

}

float readPressure()
{
  char status;
  double T,P,p0,a;

  status = bmp180.startTemperature();
  if (status != 0)
  {
    delay(status);
    status = bmp180.getTemperature(T);
    if (status != 0)
    { 
      status = bmp180.startPressure(3);
      if (status != 0)
      {
        delay(status);
        status = bmp180.getPressure(P,T);
        if (status != 0)
        {
          p0 = bmp180.sealevel(P,ALTITUDE);       
          return p0;
        }
      }
    }
  }
}

