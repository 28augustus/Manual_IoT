# Manual Foster Vase
This manual explains how a Foster Vase prototype can retrieve external weather information through APIs. The Foster Vase concept was developed using principles from *Designing Connected Products*, including the 3C’s and the 4 UI’s framework [8]. The weather APIs support the connected-product concept, but they do not measure the exact conditions directly next to the vase.

## what do you need:
### Hard ware
- NodeMCU 1.0 ESP8266
- USB cable
- Laptop/computer


- **Pictures in order of the list:**
<img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/nodemcu.png" alt="NodeMCU 1.0 ESP8266" width="100" height="200" />
<img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/usb_cable.jpg" width="100" height="200" />

  
### Soft ware
- Arduino IDE
- ESP8266 board package
- ArduinoJson library by Benoit Blanchon
- OpenWeatherMap account and API key
- Open-Meteo Forecast API
- Wi-Fi network with 2.4 GHz support
- Internet connection

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

**Source:** The ESP8266 board installation steps are based on the official ESP8266 Arduino Core documentation [1].

# Step 2 Install libraries
- Install the libraries you can also find them at Sketch > Include library > Manage libraries
- Install ArduinoJson by Benoit Blanchon

**Source:** ArduinoJson installation and documentation can be found in the official ArduinoJson documentation [2].

# Step 3 Api OpenWeatherMap Key
We are gonna get the first API to get the weather info for the vase
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
 - Getting the API can take up to 2 hours before it activates

**Source:** The OpenWeatherMap API key and forecast request are based on the OpenWeather documentation [3].

# Step 4 Open Arduino IDE
use this starter code:
```cpp
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
**API source:** The OpenWeatherMap forecast request in this code is based on the OpenWeatherMap API documentation [3].

**Code reference:** The OpenWeatherMap starter code is based on school code by D. de Vries. During development, the NodeMCU weather code reference [6] was consulted.

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
- Upload the code, you need to connect your NodeMCU with a cable first then
- Connect the cable directly to your computer/laptop
- Then you may upload it
- To see the data, go to: Tools > Serial Monitor
- A tab will appear with hopefully the right data after uploading (see troubleshooting if not)
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/code_works_api.png" />


# Step 5 Add Open-Meteo API
Now we are going to add another API tho show what diffrent API's can do and how it can be useful for your Foster vase
- Now if everything works and you see something like shown in this image we can move on to an extra API (Open-Meteo)
- <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/code_works_api.png" />

- We are going to use Latitude and longitude from the Amsterdam area
- [Link to Open-Meteo forecast](https://api.open-meteo.com/v1/forecast?latitude=52.3676&longitude=4.9041&current_weather=true&daily=temperature_2m_max&timezone=auto)
  If you see this, then we can go to the next step
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/amsterdam_json.png" />

- Find this code (it's near the first line of code)
```cpp
#include <ArduinoJson.h>
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
```

Add this beneath it
```cpp
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecureBearSSL.h>
```

To get something like this:
```cpp
#include <ArduinoJson.h>
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecureBearSSL.h>
```

Find
```cpp
String nameOfCity = "Amsterdam,NL";
String apiKey = "YOUR_OPENWEATHERMAP_API_KEY";
```

Add
```cpp
// Open-Meteo uses latitude and longitude instead of a city name.
const char* latitude = "52.3676";
const char* longitude = "4.9041";
```

to get:
```cpp
String nameOfCity = "Amsterdam,NL";
String apiKey = "YOUR_OPENWEATHERMAP_API_KEY";

// Open-Meteo uses latitude and longitude instead of a city name.
const char* latitude = "52.3676";
const char* longitude = "4.9041";
```

Find the complete void loop() function:
```cpp
void loop() {
  if (millis() - lastConnectionTime > postInterval) {
    lastConnectionTime = millis();
    makeHttpRequest();
  }
}
```


Replace the entire void loop() function with this code:
```cpp
void loop() {
  if (millis() - lastConnectionTime > postInterval) {
    lastConnectionTime = millis();

    // API 1: OpenWeatherMap
    makeHttpRequest();

    // API 2: Open-Meteo
    getOpenMeteoWeather();
  }
}
```
Paste this on the bottom of your code
```cpp
// ============================================================
// OPEN-METEO API
// ============================================================

void getOpenMeteoWeather() {
  Serial.println();
  Serial.println("Requesting weather data from Open-Meteo...");

  String url = "https://api.open-meteo.com/v1/forecast?latitude=";
  url += latitude;
  url += "&longitude=";
  url += longitude;
  url += "&current_weather=true";
  url += "&daily=temperature_2m_max";
  url += "&timezone=auto";

  // Open-Meteo uses HTTPS.
  BearSSL::WiFiClientSecure secureClient;

  // This is acceptable for a classroom prototype.
  // It avoids certificate problems on the ESP8266.
  secureClient.setInsecure();

  HTTPClient http;
  http.setTimeout(10000);

  if (!http.begin(secureClient, url)) {
    Serial.println("OPEN-METEO ERROR: Could not start HTTPS connection.");
    return;
  }

  int httpCode = http.GET();

  if (httpCode == HTTP_CODE_OK) {
    String response = http.getString();

    DynamicJsonDocument doc(4096);

    DeserializationError error = deserializeJson(doc, response);

    if (error) {
      Serial.println("OPEN-METEO ERROR: JSON could not be read.");
      Serial.print("JSON error: ");
      Serial.println(error.c_str());

      http.end();
      return;
    }

    if (doc["current_weather"].isNull()) {
      Serial.println("OPEN-METEO ERROR: Current weather data was not found.");

      http.end();
      return;
    }

    float currentTemperature = doc["current_weather"]["temperature"];
    float maximumTemperature = doc["daily"]["temperature_2m_max"][0];

    Serial.println("OPEN-METEO SUCCESS");

    Serial.print("Current temperature: ");
    Serial.print(currentTemperature);
    Serial.println(" C");

    Serial.print("Maximum temperature today: ");
    Serial.print(maximumTemperature);
    Serial.println(" C");

  } else {
    Serial.println("OPEN-METEO ERROR: API request failed.");

    Serial.print("HTTP status code: ");
    Serial.println(httpCode);

    Serial.print("API response: ");
    Serial.println(http.getString());
  }

  http.end();
}
```
**Source:** The Open-Meteo URL structure and weather parameters are based on the official Open-Meteo API documentation [4].
 
**HTTPS source:** The Open-Meteo HTTPS client setup is based on the ESP8266 BearSSL WiFi client documentation [7].

# Step 6 Test the code
Upload the changed code and see what happens
- Error? Look at Troubleshooting to resolve it

- You should see something like this
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/serial_monitor_both_api_work.png" />

# Step 7 Change the API update interval
- Alr if you have everything done and figuerd out we can change some code because getting an api request every 10 seconds is a lot, for testing it is very useful, but for normal use it is too frequent

Find this piece of code (line: 32):
```cpp
const unsigned long postInterval = 10000;
```

And change it to this (it will make the weather request every 10 minutes:
```cpp
const unsigned long postInterval = 600000;
```

- This way you will only get it every 10 minutes, you can change it lower or higher if you desire a diffrent setting

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
There is a possibility that the Serial Monitor does not match the Baud rate in the code
- Check the baud, put it on 9600 baud, because the code uses
```cpp
Serial.begin(9600);
```
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/baud_check.png" />

If you use a diffrent serial.begin() then change it to 9600


### If you only get dots then
- Check Wi-Fi name and Password
- Test with a hotspot for 2.4 GHz not your wifi, because ESP8266 supports 2.4 GHz Wi-Fi not a 5 GH-z only network
  


## Open-Meteo API
- You see this
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/error_mateo.png" />
- Then you probaply have this too
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/mateo_error_cause_54_millis.png" />
  The void loop has been deleted and a if statement can not be outside a void loop()
  Just paste this in the place to fix it
```cpp
void loop() {
  if (millis() - lastConnectionTime > postInterval) {
    lastConnectionTime = millis();

    // API 1: OpenWeatherMap
    makeHttpRequest();

    // API 2: Open-Meteo
    getOpenMeteoWeather();
  }
}
```


## Use of AI
HvA AI Chat was used to support troubleshooting, explain compiler errors and improve the clarity of this manual. The NodeMCU setup, code uploads, API tests, error tests and screenshots were completed and documented by the author (28augustus).
## References

[1] Arduino ESP8266 Community (n.d.) *Installing the ESP8266 Arduino Core*. Available at: https://arduino-esp8266.readthedocs.io/en/latest/installing.html (Accessed: 9 October 2026).

[2] ArduinoJson (n.d.) *ArduinoJson documentation*. Available at: https://arduinojson.org/v6/doc/ (Accessed: 9 October 2026).

[3] OpenWeather (n.d.) *5 day weather forecast API*. Available at: https://openweathermap.org/forecast5 (Accessed: 9 October 2026).

[4] Open-Meteo (n.d.) *Weather Forecast API documentation*. Available at: https://open-meteo.com/en/docs (Accessed: 9 October 2026).

[5] Hogeschool van Amsterdam (2026) *HvA AI Chat* [AI chatbot]. Available at: https://aichat.hva.nl/chat/ (Accessed: 9 October 2026).

[6] icecream4all (n.d.) *NodeMCU weather Arduino code*. GitHub Gist. Available at: https://gist.github.com/icecream4all/7e9db0333f44192a6071eb73efe23329#file-nodemcu-weather-ino (Accessed: 9 October 2026).

[7] ESP8266 Arduino Core (n.d.) *BearSSL WiFi secure client class*. Available at: https://arduino-esp8266.readthedocs.io/en/latest/esp8266wifi/bearssl-client-secure-class.html (Accessed: 9 October 2026).

[8] Rowland, C., Goodman, E., Charlier, M., Light, A. and Lui, A. (2015) *Designing Connected Products: UX for the Consumer Internet of Things*. Sebastopol, CA: O’Reilly Media.
