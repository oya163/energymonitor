#include <SPI.h>
#include <GSM.h>

#define PINNUMBER ""

// initialize the library instance
GSM gsmAccess;
GSMVoiceCall vcs;
GSM_SMS sms;

void setup()
{
  delay(10000);
  // to start GSM shield 10 seconds after Ethernet shield start-up time  
  Serial.begin(9600);
  boolean notConnected = true;
  sendDataToDisplay("Initializing GSM");
  // to automatially turn on the GSM shield
  pinMode(9, OUTPUT);
  digitalWrite(9, HIGH);
  // Start GSM shield
  // If your SIM has PIN, pass it as a parameter of begin() in quotes
  while(notConnected)
  {
    if((gsmAccess.begin(PINNUMBER)==GSM_READY))
    {
      notConnected = false;

    }
  }
  sendDataToDisplay("GSM Connected");    
}

void loop()
{
  if(Serial.available())
  {
    String s = recieveSerialValue();
    sendSMS(s); 
  }
  else
  {
    recieveSMS();
  }
  delay(2000);
}

String recieveSerialValue()
{
  String m=Serial.readStringUntil('\n');
  return m;
}

void recieveSMS()
{
  char senderNumber[20];
  String message="";
  char c;
  // If there are any SMSs available()  
  if (sms.available())
  { 
    sms.remoteNumber(senderNumber, 20);
    // Read message bytes 
    while(c=sms.read())
    {
      message+=c;
    } 
    sms.flush();
    message=String("1")+senderNumber+String("@")+message; 
    Serial.println(message);
  }
}

void sendSMS(String msg1)
{
  char remoteNum[11];
  if(msg1.substring(0,4)=="1299")
  {
    char txtMsg[msg1.length()-13];
    msg1.substring(4,14).toCharArray(remoteNum,11);
    msg1.substring(14).toCharArray(txtMsg,msg1.length()-13);

    String ctMsg = txtMsg;
    String current = ctMsg.substring(0,ctMsg.indexOf('$'));
    String voltage = ctMsg.substring(ctMsg.lastIndexOf('$')+1);
    String ctData = "Energy Reading is\nCurrent : " + current + " A\nVoltage : " + voltage + " V\n";
    sms.beginSMS(remoteNum);  
    sms.println(ctData);
    //sendDataToDisplay(ctData);
  }
  else
  { 
    char txtMsg[msg1.length()-12];
    msg1.substring(3,13).toCharArray(remoteNum,11);
    msg1.substring(13).toCharArray(txtMsg,msg1.length()-12);
    if (msg1.substring(0,3) == "911")
    {
      if(vcs.voiceCall(remoteNum))
      {
        //sendDataToDisplay("Call Established. Enter line to end");
        // Wait for some input from the line
        if(vcs.getvoiceCallStatus()==TALKING)          
        {// And hang up
          vcs.hangCall();
        }
      }
    }
    else
    {
//      char txtMsg[msg1.length()-12];
//      msg1.substring(3,13).toCharArray(remoteNum,11);
//      msg1.substring(13).toCharArray(txtMsg,msg1.length()-12);
      sms.beginSMS(remoteNum);
      sms.println(txtMsg);
      //sendDataToDisplay(txtMsg);  
    }
  }
  sms.endSMS();

  //  if (sms.endSMS() == 1)
  //  {
  //    String ack=String("2")+remoteNum+String("@101"); 
  //    Serial.println(ack);
  //  }

}

void sendDataToDisplay(String data)
{
  data='0'+data;
  Serial.println(data);
}
