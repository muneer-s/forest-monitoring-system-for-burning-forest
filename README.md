# forest-monitoring-system-for-burning-forest
String header = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n";
String html_1 = R"=====(
<!DOCTYPE html>
<html>

 <head>
 
  <meta name='viewport' content='width=device-width, initial-scale=1.0'/>
  <meta charset='utf-8'>
  <style>
    body {font-size:100%;} 
    #main {display: table; margin: auto;  padding: 10px 10px 10px 10px; } 
    #content { border: 5px solid green; border-radius: 15px; padding: 10px 0px 10px 0px;}
    h2 {text-align:center; margin: 10px 0px 10px 0px;} 
    p { text-align:center; margin: 5px 0px 10px 0px; font-size: 120%;}
    #time_P { margin: 10px 0px 15px 0px;}
  </style>
 
  <script> 
    function updateTime() 
    {  
       var d = new Date();
       var t = "";
       t = d.toLocaleTimeString();
       document.getElementById('P_time').innerHTML = t;
    }
 
    function updateTemp() 
    {  
       ajaxLoad('getTemp'); 
    }
 
    var ajaxRequest = null;
    if (window.XMLHttpRequest)  { ajaxRequest =new XMLHttpRequest(); }
    else                        { ajaxRequest =new ActiveXObject("Microsoft.XMLHTTP"); }
 
    function ajaxLoad(ajaxURL)
    {
      if(!ajaxRequest){ alert('AJAX is not supported.'); return; }
 
      ajaxRequest.open('GET',ajaxURL,true);
      ajaxRequest.onreadystatechange = function()
      {
        if(ajaxRequest.readyState == 4 && ajaxRequest.status==200)
        {
          var ajaxResult = ajaxRequest.responseText;
          var tmpArray = ajaxResult.split("|");
          document.getElementById('temp_C').innerHTML = tmpArray[0];
          document.getElementById('temp_F').innerHTML = tmpArray[1];
          document.getElementById('hmd').innerHTML = tmpArray[2];
          document.getElementById('air').innerHTML = tmpArray[3];
        }
      }
      ajaxRequest.send();
    }
 
    var myVar1 = setInterval(updateTemp, 5000);  
    var myVar2 = setInterval(updateTime, 1000);  
 
  </script>
 
 
  <title>Temperature & Humidy Monitor</title>
 </head>
 
 <body>
   <div id='main'>
     <h2>FOREST MONITOR</h2>
     <div id='content'> 
       <p id='P_time'>Time</p>
       <h2>Temperature</h2>
       <p> <span id='temp_C'>--.-</span> &deg;C &nbsp;-&nbsp; <span id='temp_F'>--.-</span> &deg;F </p>
       <h2>Humidity</h2>
       <p> <span id='hmd'>--</span> % </p>
       <h2>Air Polution</h2>
       <p> <span id='air'>--</span> % </p>
     </div>
   </div> 
   <a>Check out <a href="https://techfi.web.app/" target="_blank" rel="noopener noreferrer">TECH-FI</a>.</a>
 </body>
</html>
)====="; 
 
#include <ESP8266WiFi.h>
 
// change these values to match your network
char ssid[] = "munners";       //  your network SSID (name)
char pass[] = "muneermuni";          //  your network password
 
#include "DHT.h"
#define DHTPIN 5     // what digital pin we're connected to
#define DHTTYPE DHT11   // DHT 11
DHT dht(DHTPIN, DHTTYPE);
 
float tempF = 0;
float tempC = 0;
float humid = 0;

float mq;
WiFiServer server(80);
String request = "";
 
 
void setup() 
{
    Serial.begin(115200);
    Serial.println();
    Serial.println("Serial started at 115200");
    Serial.println();
 
    dht.begin();
 
    // Connect to a WiFi network
    Serial.print(F("Connecting to "));  Serial.println(ssid);
    WiFi.begin(ssid, pass);
 
    while (WiFi.status() != WL_CONNECTED) 
    {
        Serial.print(".");
        delay(500);
    }
 
    Serial.println("");
    Serial.println(F("[CONNECTED]"));
    Serial.print("[IP ");              
    Serial.print(WiFi.localIP()); 
    Serial.println("]");
 
    // start a server
    server.begin();
    Serial.println("Server started");
 
} // void setup()
 
 
void loop() 
{
    WiFiClient client = server.available();     // Check if a client has connected
    if (!client)  {  return;  }
 
    request = client.readStringUntil('\r');     // Read the first line of the request
 
    Serial.println(request);
    Serial.println("");
    
    float ijk =analogRead(A0);
    mq = (ijk/1024)*100;
    Serial.print("air Quality :");
    Serial.print(mq);
    Serial.println(" %");
    if ( request.indexOf("getTemp") > 0 )
     { 
          Serial.println("getTemp received");
 
          // Reading temperature or humidity takes about 250 milliseconds!
          // Sensor readings may also be up to 2 seconds 'old' (its a very slow sensor)
          humid = dht.readHumidity();
          tempC = dht.readTemperature();        // Read temperature as Celsius (the default)
          tempF = dht.readTemperature(true);    // Read temperature as Fahrenheit (isFahrenheit = true)
          
          Serial.print("humidity :");
          Serial.println(humid);
          Serial.print("Temp :");
          Serial.print(tempC);
          Serial.println(" C");
          Serial.print("Temp :");
          Serial.print(tempF);
          Serial.println(" F");
          if ( !isnan(humid) && !isnan(tempC) && !isnan(tempC) )
          {
              client.print( header );
              client.print( tempC );   
              client.print( "|" );  
              client.print( tempF );  
              client.print( "|" );  
              client.print( humid );
              client.print( "|" );  
              
              client.print( mq ); 
              Serial.println("data sent");
          }
          else
          {
              Serial.println("Error reading the sensor");
          }
     }
 
     else
     {
        client.flush();
        client.print( header );
        client.print( html_1 ); 
        Serial.println("New page served");
     }
 
    delay(1);
 
 

}
