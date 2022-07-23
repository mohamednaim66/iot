 1 - install a library called as Talkie
 2-  including all the required libraries :
 #include <WiFi.h>
#include <WiFiClient.h>
#include <WebServer.h>
#include <ESPmDNS.h>
#include <Talkie.h>
3- Next define an object voice for Talkie.
Talkie voice;
4-  defining the numbers.  also add more words/phonemes by recording the sound for each one and converting them into hex code.
const uint8_t spZERO[]     PROGMEM = {0x69, 0xFB, 0x59, 0xDD, 0x51, 0xD5, 0xD7, 0xB5, 0x6F, 0x0A, 0x78, 0xC0, 0x52, 0x01, 0x0F, 0x50, 0xAC, 0xF6, 0xA8, 0x16, 0x15, 0xF2, 0x7B, 0xEA, 0x19, 0x47, 0xD0, 0x64, 0xEB, 0xAD, 0x76, 0xB5, 0xEB, 0xD1, 0x96, 0x24, 0x6E, 0x62, 0x6D, 0x5B, 0x1F, 0x0A, 0xA7, 0xB9, 0xC5, 0xAB, 0xFD, 0x1A, 0x62, 0xF0, 0xF0, 0xE2, 0x6C, 0x73, 0x1C, 0x73, 0x52, 0x1D, 0x19, 0x94, 0x6F, 0xCE, 0x7D, 0xED, 0x6B, 0xD9, 0x82, 0xDC, 0x48, 0xC7, 0x2E, 0x71, 0x8B, 0xBB, 0xDF, 0xFF, 0x1F};
5-Then it checks if the number is greater than or equal to thousand so that it can attach “thousands” before saying it. Then it checks remainder by dividing the number by 1000 to tell the digits after thousand. The same logic is used for hundreds or three digit number.
void sayNumber(long n) {
  if (n<0) {
    voice.say(spMINUS);
    sayNumber(-n);
  } else if (n==0) {
    voice.say(spZERO);
  } else {
    if (n>=1000) {
      int thousands = n / 1000;
      sayNumber(thousands);
      voice.say(spTHOUSAND);
      n %= 1000;
      if ((n > 0) && (n<100)) voice.say(spAND);
    }
    if (n>=100) {
      int hundreds = n / 100;
      sayNumber(hundreds);
      voice.say(spHUNDRED);
      n %= 100;
      if (n > 0) voice.say(spAND);
    }
    6-  set up the Wi-Fi
    const char* ssid     = "CircuitLoop";
    const char* password = "xxxx";
    WebServer server ( 80 );
    7-define an array htmlResponse to get the input from webpage:
    char htmlResponse[3000];
7- write the JavaScript code:
void handleRoot() {
  snprintf ( htmlResponse, 3000,
"<!DOCTYPE html>\
<html lang=\"en\">\
  <head>\
    <meta charset=\"utf-8\">\
    <meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">\
  </head>\
  <body>\
          <h1>Text To Speech</h1>\
          <input type='text' name='msg' id='msg' size=7 autofocus> Number \
          <div>\
          <br><button id=\"speak_button\">Speak</button>\
          </div>\
    <script src=\"https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js\"></script>\    
    <script>\
      var Number;\
      $('#speak_button').click(function(e){\
        e.preventDefault();\
        Number = $('#msg').val();\       
        $.get('/save?Number=' + Number , function(data){\
          console.log(data);\
        });\
      });\      
    </script>\
  </body>\
</html>"); 
   server.send ( 200, "text/html", htmlResponse );  
}
8- define a function handleSave(). Here we convert the string into integer, because the input we are getting is a string and to speak it out we have to use it as an integer.
void handleSave() {
  if (server.arg("Number")!= "")
  {
    Serial.println("Message: " + server.arg("Number"));
    String serverData = String(server.arg("Number"));
    int finalServer = serverData.toInt();
    sayNumber(finalServer);
    delay(2000);
  }
}


void setup() {
  pinMode(25, OUTPUT);
  digitalWrite(25, HIGH);
  delay(10);
  // Start serial
  Serial.begin(115200);
  delay(10);

  // Connecting to a WiFi network
  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");  
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  server.on ( "/", handleRoot );
  server.on ("/save", handleSave);

  server.begin();
  Serial.println ( "HTTP server started" );
}
