# ESP_SRW_WEB


/*Libraries for WIFI and WEBSERVER*/

#include <WiFi.h>
#include <WebServer.h>

/* Put your SSID & Password */

const char* ssid = "ESP32";  // Enter SSID here
const char* password = "12345678";  //Enter Password here

/* Put IP Address details */

IPAddress local_ip(192,168,1,1);
IPAddress gateway(192,168,1,1);
IPAddress subnet(255,255,255,0);

/*WEB SERVER PORT*/

WebServer server(80);

/*ESP SIGNAL GPIO5*/

uint8_t motorSignal = 5;
bool motorStatus = LOW;

/*COMMUNICATION*/

void setup() {
  Serial.begin(115200);
  pinMode(motorSignal, OUTPUT);

/*AP settings*/

  WiFi.softAP(ssid, password);
  WiFi.softAPConfig(local_ip, gateway, subnet);
  delay(100);

  /*SERVER START*/
  
  server.on("/", handle_OnConnect);

  server.on("/motorOn", handle_motorOn);
  server.on("/motorOff", handle_motorOff);
  server.onNotFound(handle_NotFound);
  
  server.begin();
  Serial.println("HTTP server started");
}
void loop() {
  server.handleClient();
  
  if(motorStatus)
  {digitalWrite(motorSignal, HIGH);}
  else
  {digitalWrite(motorSignal, LOW);}
}

/*BEGINNING STATE*/

void handle_OnConnect() {
  motorStatus = LOW;
  Serial.println("GPIO5 Status: OFF");
  server.send(200, "text/html", SendHTML(motorStatus)); 
}

/*ON SIGNAL*/

void handle_motorOn() {
  motorStatus = HIGH;
  Serial.println("GPIO5 Status: ON");
  server.send(200, "text/html", SendHTML(true)); 
}

/*OFF SIGNAL*/

void handle_motorOff() {
  motorStatus = LOW;
  Serial.println("GPIO5 Status: OFF");
  server.send(200, "text/html", SendHTML(false)); 
}
/*Prevent error*/

void handle_NotFound(){
  server.send(404, "text/plain", "Not found");
}

/*HTML code render*/

String SendHTML(uint8_t motorStat){
  String ptr = "<!DOCTYPE html> <html>\n";
  ptr +="<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0, user-scalable=no\">\n";
  ptr +="<title>LED Control</title>\n";
  ptr +="<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}\n";
  ptr +="body{margin-top: 50px;} h1 {color: #444444;margin: 50px auto 30px;} h3 {color: #444444;margin-bottom: 50px;}\n";
  ptr +=".button {display: block;width: 80px;background-color: #3498db;border: none;color: white;padding: 13px 30px;text-decoration: none;font-size: 25px;margin: 0px auto 35px;cursor: pointer;border-radius: 4px;}\n";
  ptr +=".button-on {background-color: #3498db;}\n";
  ptr +=".button-on:active {background-color: #2980b9;}\n";
  ptr +=".button-off {background-color: #34495e;}\n";
  ptr +=".button-off:active {background-color: #2c3e50;}\n";
  ptr +="p {font-size: 14px;color: #888;margin-bottom: 10px;}\n";
  ptr +="</style>\n";
  ptr +="</head>\n";
  ptr +="<body>\n";
  ptr +="<h1>ESP32 Web Server</h1>\n";
  ptr +="<h3>Using Access Point(AP) Mode</h3>\n";
  
  /*CONDITIONAL SWITCH RENDER*/
  
  if(motorStat)
  {ptr +="<p>Motor Status: ON</p><a class=\"button button-off\" href=\"/motorOff\">OFF</a>\n";}
  else
  {ptr +="<p>Motor Status: OFF</p><a class=\"button button-on\" href=\"/motorOn\">ON</a>\n";}

  ptr +="</body>\n";
  ptr +="</html>\n";
  return ptr;
}
