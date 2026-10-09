# Manual Foster Vase
We are gonna work with API's to show how the Vase can give accurate data via de API
Change the things i mention with:

## what do you need:
### Hard ware
- NodeMCU 1.0 ESP8266
- USB cable


- **Pictures in order of the list:**
<img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/nodemcu.png" alt="NodeMCU 1.0 ESP8266" width="100" height="200" />
<img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/usb_cable.jpg" width="100" height="200" />

  
### Soft ware
- Arduino IDE
- ESP8266 board package
- Adafruit NeoPixel library
- ArduinoJson library
- Adafruit IO
- OpenWeatherMap account and API key
- Wi-Fi network

# Step 1 Install ESP8266
- Open Arduino IDE
- Go to file > Prefferences
- Paste this link in the section **Additional boards manager URLs:** http://arduino.esp8266.com/stable/package_esp8266com_index.json
- <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/1_preferences.png" />
- Press OK
- Go to Tool > Board > Boards Manager
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/install_esp_library.jpeg" />
- Install esp8266 by ESP8266 Community
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/install_esp_library_community.png" />
- Then select NodeMCU 1.0 (ESP-12E Module)
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/right_nodemcu.png" />

# Step 2 Install libraries
- Install the libraries you can also find them at Sketch > Include library > Manage libraries
- Install Adafruit NeoPixel
- Install ArduinoJson by Benoit Blanchon

# Step 3 Api OpenWeatherMap Key
- [Go to OpenWeatherMap](https://openweathermap.org/)
- Create an account or log in
- get_api_key
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/get_api_key.png" />
- Create account or sign in if you already have an account
- If you can't sign in or make a new account (like me) Then it's probaply becasue you already have an account you can do the following:
- - Choose forgot password and make a new password
- Once you made an account click on API Key
- If you dont see it you might need to click on get api again
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/api_code.png" />
- Then make sure you copy or safe your code safely
- **DO NOT SHARE YOUR CODE ONLINE** (that's why mine is hidden)
- Keep private
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/api_code_copy.png" />
 
# Step 4 Open Arduino IDE
use this starter code:
```cpp
/*
/*
 * Simple weather station with ESP8266 and OpenWeatherMap API
 * Based on school code by D. de Vries
 */

#include <ArduinoJson.h>
#include <ESP8266WiFi.h>
#include <WiFiClient.h>

// === CONFIGURATION ===

char ssid[] = "YOUR_WIFI_NAME";
char pass[] = "YOUR_WIFI_PASSWORD";

const char server[] = "api.openweathermap.org";

String nameOfCity = "Amsterdam,NL";
String apiKey = "YOUR_OPENWEATHERMAP_API_KEY";

WiFiClient client;

#define JSON_BUFF_DIMENSION 8192
String text;

unsigned long lastConnectionTime = 0;
const unsigned long postInterval = 10000;  // Request weather every 10 seconds


void setup() {
  Serial.begin(9600);
  while (!Serial) { ; }

  text.reserve(JSON_BUFF_DIMENSION);

  Serial.println("Connecting to Wi-Fi...");

  WiFi.begin(ssid, pass);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWi-Fi connected!");
}


void loop() {
  if (millis() - lastConnectionTime > postInterval) {
    lastConnectionTime = millis();
    makeHttpRequest();
  }
}


void makeHttpRequest() {
  client.stop();

  if (client.connect(server, 80)) {

    client.println(
      "GET /data/2.5/forecast?q=" + nameOfCity +
      "&APPID=" + apiKey +
      "&mode=json&units=metric&cnt=1 HTTP/1.1"
    );

    client.println("Host: api.openweathermap.org");
    client.println("Connection: close");
    client.println();

    // Skip the HTTP header and save the JSON response.
    bool headerSkipped = false;
    text = "";

    unsigned long timeout = millis();

    while (client.connected() && millis() - timeout < 10000) {
      String line = client.readStringUntil('\n');

      if (!headerSkipped && (line == "\r" || line == "")) {
        headerSkipped = true;
        text = client.readString();
        break;
      }
    }

    if (text.length() > 0) {
      parseJson(text.c_str());
    } else {
      Serial.println("ERROR: No JSON data received.");
    }

    client.stop();

  } else {
    Serial.println("ERROR: Could not connect to the server.");
  }
}


void parseJson(const char* jsonString) {
  DynamicJsonDocument doc(JSON_BUFF_DIMENSION);

  DeserializationError error = deserializeJson(doc, jsonString);

  if (error) {
    Serial.println("ERROR: JSON could not be read.");
    Serial.println(error.c_str());
    return;
  }

  JsonArray list = doc["list"];

  if (list.isNull() || list.size() < 1) {
    Serial.println("ERROR: JSON contains no forecast data.");
    return;
  }

  JsonObject forecast = list[0];

  String city = doc["city"]["name"];
  String weather = forecast["weather"][0]["main"];
  String description = forecast["weather"][0]["description"];
  float temp = forecast["main"]["temp"];

  Serial.println();
  Serial.println("Weather forecast for " + city + ":");
  Serial.println("Weather: " + weather);
  Serial.println("Description: " + description);
  Serial.println("Temperature: " + String(temp) + " C");
}
```

Change thsese parts to your own data
```cpp
// ******************* Change these *******************

char ssid[] = "YOUR_WIFI_NAAM";         // Put your own wifi name
char pass[] = "YOUR_WIFI_WACHTWOORD";  // Put your own password

String nameOfCity = "STAD,LANDCODE";    // city and landcode example: "Amsterdam,NL"
String apiKey = "YOUR_OPENWEATHERMAP_API_KEY";  // Put here your own API key

// **********************************************************************
```
- **From now on if there is an issue look at the troubleshooting area to resolve your problems**

- Change all the specific area's in your code

# Troubleshooting
## API OpenWeatherMap
- If you see this:
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/wrong_port.png" />
- Then you connected to the wrong port do the following:
- Make sure your NodeMCU is plugged in with a cable
- Open in Arduino tools > port and hover over it while the cable is plugged in
- Make a picture or look at all the numers you are seeing
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/cable_port_in.png" />
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/cable_port_out.jpeg" />
- Remove the cable from your device and select Tools > port again
- Look for the missing number or port, that is the port you need to select
- Put the cable back in and select your port

  - OTHER
  - Use a diffrent USB cable
  - Use a diffrent port on device

- Make sure you selected the right ModeMCU
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/right_nodemcu.png" />

### Random symbols in Serial monitor
- Check the baud, put it on 115200
  <img src="https://github.com/user-attachments/assets/f45dae0e-e7bb-4d53-be80-3601b99a312a" />


### If you only get dots then
- Check Wi-Fi name and Password
- Test with a hotspot for 2.4 GHz not your wifi, because ESP8266 supports 2.4 GHz Wi-Fi not a 5 GH-z only network
  

## LED
- Change the serial monitor

## Other API
- Has Api already activated?
 


## List of sources
- Designing Connected Products : UX for the Consumer Internet of Things van Claire Rowland". Bekijk via O'Reilly
- OpenWeahterMap: https://openweathermap.org/
- Troubleshooting with and making the Arduino Code: [aichat.hva.nl](https://aichat.hva.nl/chat/)
- Api code: https://gist.github.com/icecream4all/7e9db0333f44192a6071eb73efe23329#file-nodemcu-weather-ino
