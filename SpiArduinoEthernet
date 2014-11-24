#include <Ethernet.h>
#include <SPI.h>
#include <LiquidCrystal.h>
#include "EmonLib.h" 
#include "floatToString.h" 

EnergyMonitor emon1;

byte mac[] = {
  0x90, 0xA2, 0xDA, 0x0E, 0xD0, 0x93 };
IPAddress ip(192,168,0,216);
// the gateway
IPAddress gateway(192, 168,0, 5);
// the dns server ip
IPAddress dnServer(192, 168, 0, 27);
// the subnet
IPAddress subnet(255, 255, 255, 0);


char accessCode[] = "";
char apiServer[] = "192.168.0.65";
unsigned int apiPort = 80;

EthernetServer server(3300);
EthernetClient client1;
EthernetClient client2;


String msg = "";
LiquidCrystal lcd( 2,3, 14, 15, 16, 17);

String ctData = "";
char i[5];
char v[7];
String current = "";
String voltage = "";


unsigned int smsCounter = 2;
unsigned int callCounter = 0;


void(*resetFunc) (void)=0;

void setup()
{
  lcd.begin(16,2);
  lcd.print("Arduino Ethernet");
  Serial.begin(9600);
  //Ethernet.begin(mac, ip, dnServer, gateway, subnet);
  
  if(Ethernet.begin(mac)==0)
   {
   //Serial.println(F("1999849389720@Failed to configure Ethernet using DHCP"));   
   displayToLCD("0Failed to configure Ethernet using DHCP",0);
   resetFunc();
   //while(true);
   
   }   
  server.begin();
  
  lcd.setCursor(0, 1);
  lcd.print(Ethernet.localIP());

  emon1.voltage(5, 246, 1.7);  // measurement according to UPS
  //emon1.voltage(5, 249, 1.7);  // Voltage: input pin, calibration, phase_shift
  //emon1.voltage(5, 241, 1.7);  // Exact measurement from multimeter
  emon1.current(4, 60.6);  // around 28 A at full day
  //emon1.current(4, 111.1);     //// around 55 A

  delay(10000);
  lcd.clear();
}


void loop()
{
  EthernetClient client = server.available();
  if (client.available())
  {
    receivePacket(&client);
    executePacket(); 
    client.stop();
    delay(5000);
  }
  else
  {
    energyMonitor();
    delay(5000);
    if (Serial.available())
    {
      String s=recieveSerialValue();
      if(s.startsWith("1"))
      {
        displayToLCD("0Uploading SMS",0);
        delay(500);
        uploadData(apiServer,apiPort, s,"SMS");
      }  
      else if(s.startsWith("2"))
      {
        displayToLCD("0Sending ACK",0);
        if(client2.connect(apiServer,apiPort)==1)
        {
          client2.println(s);
          client2.println();
          client2.stop();
        }
      }
      else
      {
        displayToLCD(s,0); 
      }
      delay(5000);
    }
  }
}

void energyMonitor()
{
  ctData = "";
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Energy Monitor");
  emon1.calcVI(20,2000);          // Calculate all. No.of crossings, time-out  
  lcd.setCursor(0,1);
  lcd.print(emon1.Vrms);
  lcd.print(" V ");
  lcd.print(emon1.Irms);
  lcd.print(" A     ");
  floatToString(i,emon1.Irms,2);
  current = i;
  floatToString(v,emon1.Vrms,2);
  voltage = v;
  ctData = current + "$" + voltage;
  //Serial.println(ctData);
  uploadData(apiServer,apiPort,ctData,"CT");
  if (emon1.Vrms < 215)
  {
    //displayToLCD("0Low voltage drop : " + voltage,0);
    while (smsCounter != 0)
    {
      String warning = "1999843406028Voltage drop : " + String(voltage);
      Serial.println(warning);
      smsCounter -= 1;
      break;
    }
    if (smsCounter == 0 && callCounter == 50)
    {
      Serial.println("9119843406028Voice call");
    }
    callCounter += 1;
  }
  else
  {
    smsCounter = 2;
    callCounter = 0;
  }

}

String recieveSerialValue()
{
  String message="";
  if (Serial.available() > 0) 
  {
    message=Serial.readString();
  }
  return message;
}

void receivePacket(EthernetClient* client)
{
  msg = "";
  char isLineFinished = 0; 
  // Loop while the client is connected.
  while (client->connected())
  {
    // Read available bytes.
    while (client->available())
    {
      // Read a byte.
      char c = client->read();

      // Exit loop if end of line.
      if ( !client->available())
      {
        isLineFinished=1;
        msg +=c;
        break;        
      }
      msg += c;  
    }
    if(isLineFinished==1)
    {
      break;
    }
  }
}

void executePacket()
{
  displayToLCD("0Sending SMS",0);
  Serial.println(msg);
}

void uploadData(char* domain, int port, String data, String head)
{
  unsigned int counter=0;
  boolean notConnected = true;
  //while(counter < 3)
  //{
  if(client1.connect(domain,port) == 1 )
  {
    if (head == "SMS")
    {
      displayToLCD("0Web Api connected",0);
      client1.println("POST /spiArduino/api/uploadSMS HTTP/1.1");
    } 
    else
    {
      client1.println("POST /spiArduino/api/uploadCT HTTP/1.1");
    }
    client1.println("Host: 192.168.0.65");
    client1.println("User-Agent: Arduino/1.0");
    client1.println("AccessCode: 71EEB515-1411-4843-81AE-657D24107674");
    client1.println("Connection: close");
    client1.println("Content-Type: application/json");
    client1.print("Content-Length: ");
    client1.println(data.length());
    client1.println();
    client1.println(data);
    client1.println();
    client1.stop();
    notConnected=false;
    //break;
  }
  //}

  if(notConnected)
  {
    displayToLCD("0Web API not reachable from arduino",0);
    //Serial.println(F("1999849389720@Web API not reachable from arduino"));    
    char mobileNumber[20];
    data.substring(5,15).toCharArray(mobileNumber,11);
    if (data.substring(16,19) == "299")
    {
      String data="1299"+String(mobileNumber)+ctData;
      Serial.println(data);
    }
    else if (data.substring(16,19) == "199")
    {
      String data="199"+String(mobileNumber)+" Web API service is not reachable. Please try later";
      Serial.println(data);
    }
  }
}

void displayToLCD(String data,int row)
{
  lcd.clear();
  if(data.length()>17)
  {
    lcd.setCursor(0, 1);
    lcd.print(data.substring(17));
  }
  lcd.setCursor(0, 0);
  lcd.print(data.substring(1,17));  
}

