# Manual Foster Vase
We are gonna work with API's to show how the Vase can give accurate data via de API
Change the things i mention with:
```cpp
// ******************* YOUR DATA: CHANGE HERE *******************
```
## what do you need:
### Hard ware
- NodeMCU 1.0 ESP8266
- NeoPixel LED strip
- Physical push button
- USB cable
- Jumper wires

- Useful later:
  - water-level sensor
  - temperature sensor

- **Pictures in order of the list:**
<img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/nodemcu.png" alt="NodeMCU 1.0 ESP8266" width="100" height="200" />
<img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/ledstrip.png" width="100" height="200" />
<img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/push_button.jpg" width="100" height="200" />
<img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/usb_cable.jpg" width="100" height="200" />
<img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/jumper_wires.jpg" width="100" height="200" />
<img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/watersensor.jpg" width="100" height="200" />
<img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/temphu.png" width="100" height="200" />

  
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
  Foster Vase — Two Weather APIs, Serial Monitor version
  -------------------------------------------------------
  API 1: OpenWeatherMap Current Weather API
  API 2: Open-Meteo Forecast API

  This version has NO LEDs yet.
  It is made to test Wi-Fi, APIs, JSON and error messages first.
*/

#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecureBearSSL.h>
#include <ArduinoJson.h>

// ******************* YOUR GEGEVENS: HIER AANPASSEN *******************

const char* WIFI_NAME = "YOUR_WIFI_NAAM";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";

const char* CITY_NAME = "Amsterdam";
const char* COUNTRY_CODE = "NL";

const char* OPENWEATHER_API_KEY = "YOUR_OPENWEATHERMAP_API_KEY";

// Amsterdam coordinates for Open-Meteo
const char* LATITUDE = "52.3676";
const char* LONGITUDE = "4.9041";

// **********************************************************************


// ******************* TIJDINSTELLINGEN ********************************

// New API request every 10 minutes
const unsigned long WEATHER_INTERVAL = 600000;

unsigned long lastWeatherUpdate = 0;

// **********************************************************************


// ******************* STATUSVARIABELEN ********************************

bool wifiConnected = false;
bool openWeatherWorks = false;
bool openMeteoWorks = false;

float openWeatherTemperature = 0;
float openMeteoCurrentTemperature = 0;
float openMeteoMaximumTemperature = 0;

// **********************************************************************


void setup() {
  Serial.begin(115200);
  delay(500);

  Serial.println();
  Serial.println("================================================");
  Serial.println("FOSTER VASE - TWO WEATHER APIs");
  Serial.println("No LEDs connected yet - Serial Monitor test");
  Serial.println("================================================");

  printHelp();
  connectToWifi();

  // Do an API request immediately after Wi-Fi connects
  if (WiFi.status() == WL_CONNECTED) {
    getAllWeatherData();
    lastWeatherUpdate = millis();
  }
}


void loop() {
  // Listen for commands typed in Serial Monitor
  handleSerialCommands();

  // Reconnect when Wi-Fi is lost
  if (WiFi.status() != WL_CONNECTED) {
    wifiConnected = false;

    Serial.println();
    Serial.println("================================");
    Serial.println("WIFI ERROR: Connection lost.");
    Serial.println("Trying to reconnect...");
    Serial.println("================================");

    connectToWifi();
  }

  // Request new weather data automatically
  if (WiFi.status() == WL_CONNECTED &&
      millis() - lastWeatherUpdate >= WEATHER_INTERVAL) {

    getAllWeatherData();
    lastWeatherUpdate = millis();
  }
}


// ============================================================
// WIFI CONNECTION
// ============================================================

void connectToWifi() {
  Serial.println();
  Serial.println("WIFI: Starting connection...");
  Serial.print("WIFI: Network name: ");
  Serial.println(WIFI_NAME);
  Serial.println("WIFI: ESP8266 only supports 2.4 GHz Wi-Fi.");

  WiFi.mode(WIFI_STA);
  WiFi.disconnect();
  delay(500);

  WiFi.begin(WIFI_NAME, WIFI_PASSWORD);

  unsigned long startTime = millis();
  const unsigned long WIFI_TIMEOUT = 20000;

  while (WiFi.status() != WL_CONNECTED &&
         millis() - startTime < WIFI_TIMEOUT) {

    Serial.print(".");
    delay(500);
  }

  Serial.println();

  if (WiFi.status() == WL_CONNECTED) {
    wifiConnected = true;

    Serial.println("WIFI CONNECTED");
    Serial.print("WIFI: IP address: ");
    Serial.println(WiFi.localIP());

    Serial.print("WIFI: Signal strength: ");
    Serial.print(WiFi.RSSI());
    Serial.println(" dBm");

  } else {
    wifiConnected = false;

    Serial.println("WIFI ERROR: Connection failed.");
    printWifiError(WiFi.status());
  }
}


void printWifiError(wl_status_t status) {
  Serial.print("WIFI: Status code: ");
  Serial.println(status);

  switch (status) {
    case WL_NO_SSID_AVAIL:
      Serial.println("WIFI ERROR: Wi-Fi network was not found.");
      Serial.println("Check the Wi-Fi name and make sure 2.4 GHz is available.");
      break;

    case WL_CONNECT_FAILED:
      Serial.println("WIFI ERROR: Connection failed.");
      Serial.println("Possible cause: incorrect Wi-Fi password.");
      break;

    case WL_CONNECTION_LOST:
      Serial.println("WIFI ERROR: Wi-Fi connection was lost.");
      break;

    case WL_DISCONNECTED:
      Serial.println("WIFI ERROR: ESP8266 is disconnected.");
      break;

    default:
      Serial.println("WIFI ERROR: Unknown Wi-Fi status.");
      break;
  }
}


// ============================================================
// GET DATA FROM BOTH APIs
// ============================================================

void getAllWeatherData() {
  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("API ERROR: No Wi-Fi connection.");
    return;
  }

  Serial.println();
  Serial.println("================================================");
  Serial.println("FOSTER VASE: Starting API update");
  Serial.println("================================================");

  openWeatherWorks = false;
  openMeteoWorks = false;

  getOpenWeatherMapData();
  getOpenMeteoData();

  printWeatherSummary();

  Serial.println("================================================");
  Serial.println("FOSTER VASE: API update finished");
  Serial.println("================================================");
}


// ============================================================
// API 1 — OPENWEATHERMAP
// ============================================================

void getOpenWeatherMapData() {
  Serial.println();
  Serial.println("OPENWEATHERMAP: Starting request...");

  String url = "https://api.openweathermap.org/data/2.5/weather?q=";
  url += CITY_NAME;
  url += ",";
  url += COUNTRY_CODE;
  url += "&appid=";
  url += OPENWEATHER_API_KEY;
  url += "&units=metric&lang=en";

  // HTTPS connection for ESP8266
  BearSSL::WiFiClientSecure client;
  client.setInsecure();

  HTTPClient http;
  http.setTimeout(10000);

  if (!http.begin(client, url)) {
    Serial.println("OPENWEATHERMAP ERROR: Could not start HTTPS connection.");
    return;
  }

  int httpCode = http.GET();

  if (httpCode == HTTP_CODE_OK) {
    String response = http.getString();

    DynamicJsonDocument doc(4096);
    DeserializationError jsonError = deserializeJson(doc, response);

    if (jsonError) {
      Serial.println("OPENWEATHERMAP ERROR: JSON could not be read.");
      Serial.print("JSON error: ");
      Serial.println(jsonError.c_str());

      http.end();
      return;
    }

    String weatherMain = doc["weather"][0]["main"].as<String>();
    String description = doc["weather"][0]["description"].as<String>();
    openWeatherTemperature = doc["main"]["temp"];

    if (weatherMain.length() == 0) {
      Serial.println("OPENWEATHERMAP ERROR: No weather condition found.");
      http.end();
      return;
    }

    openWeatherWorks = true;

    Serial.println("OPENWEATHERMAP SUCCESS");
    Serial.print("Main weather: ");
    Serial.println(weatherMain);

    Serial.print("Description: ");
    Serial.println(description);

    Serial.print("Current temperature: ");
    Serial.print(openWeatherTemperature);
    Serial.println(" C");

  } else {
    Serial.println("OPENWEATHERMAP ERROR: API request failed.");
    printOpenWeatherError(httpCode, http.getString());
  }

  http.end();
}


void printOpenWeatherError(int httpCode, String response) {
  Serial.print("OPENWEATHERMAP HTTP status code: ");
  Serial.println(httpCode);

  switch (httpCode) {
    case 401:
      Serial.println("ERROR 401: API key is invalid, missing or not active yet.");
      Serial.println("Check the API key in your code.");
      break;

    case 404:
      Serial.println("ERROR 404: City not found.");
      Serial.println("Try: Amsterdam,NL");
      break;

    case 429:
      Serial.println("ERROR 429: Too many API requests.");
      Serial.println("Wait longer before requesting new weather data.");
      break;

    case 500:
    case 502:
    case 503:
      Serial.println("SERVER ERROR: OpenWeatherMap is temporarily unavailable.");
      break;

    default:
      Serial.println("Unknown OpenWeatherMap error.");
      break;
  }

  Serial.print("API response: ");
  Serial.println(response);
}


// ============================================================
// API 2 — OPEN-METEO
// ============================================================

void getOpenMeteoData() {
  Serial.println();
  Serial.println("OPEN-METEO: Starting request...");

  String url = "https://api.open-meteo.com/v1/forecast?latitude=";
  url += LATITUDE;
  url += "&longitude=";
  url += LONGITUDE;
  url += "&current_weather=true";
  url += "&daily=temperature_2m_max";
  url += "&timezone=auto";

  BearSSL::WiFiClientSecure client;
  client.setInsecure();

  HTTPClient http;
  http.setTimeout(10000);

  if (!http.begin(client, url)) {
    Serial.println("OPEN-METEO ERROR: Could not start HTTPS connection.");
    return;
  }

  int httpCode = http.GET();

  if (httpCode == HTTP_CODE_OK) {
    String response = http.getString();

    DynamicJsonDocument doc(8192);
    DeserializationError jsonError = deserializeJson(doc, response);

    if (jsonError) {
      Serial.println("OPEN-METEO ERROR: JSON could not be read.");
      Serial.print("JSON error: ");
      Serial.println(jsonError.c_str());

      http.end();
      return;
    }

    if (doc["current_weather"].isNull()) {
      Serial.println("OPEN-METEO ERROR: No current weather data found.");
      http.end();
      return;
    }

    if (doc["daily"].isNull()) {
      Serial.println("OPEN-METEO ERROR: No daily forecast data found.");
      http.end();
      return;
    }

    openMeteoCurrentTemperature = doc["current_weather"]["temperature"];
    openMeteoMaximumTemperature = doc["daily"]["temperature_2m_max"][0];

    openMeteoWorks = true;

    Serial.println("OPEN-METEO SUCCESS");

    Serial.print("Current temperature: ");
    Serial.print(openMeteoCurrentTemperature);
    Serial.println(" C");

    Serial.print("Expected maximum temperature today: ");
    Serial.print(openMeteoMaximumTemperature);
    Serial.println(" C");

  } else {
    Serial.println("OPEN-METEO ERROR: API request failed.");

    Serial.print("OPEN-METEO HTTP status code: ");
    Serial.println(httpCode);

    Serial.print("API response: ");
    Serial.println(http.getString());
  }

  http.end();
}


// ============================================================
// SHOW RESULTS FROM BOTH APIs
// ============================================================

void printWeatherSummary() {
  Serial.println();
  Serial.println("---------------- WEATHER SUMMARY ----------------");

  if (openWeatherWorks) {
    Serial.print("OpenWeatherMap current temperature: ");
    Serial.print(openWeatherTemperature);
    Serial.println(" C");
  } else {
    Serial.println("OpenWeatherMap: FAILED");
  }

  if (openMeteoWorks) {
    Serial.print("Open-Meteo current temperature: ");
    Serial.print(openMeteoCurrentTemperature);
    Serial.println(" C");

    Serial.print("Open-Meteo max temperature today: ");
    Serial.print(openMeteoMaximumTemperature);
    Serial.println(" C");
  } else {
    Serial.println("Open-Meteo: FAILED");
  }

  if (openWeatherWorks && openMeteoWorks) {
    float difference = abs(openWeatherTemperature - openMeteoCurrentTemperature);

    Serial.print("Difference between both current temperatures: ");
    Serial.print(difference);
    Serial.println(" C");

    if (difference >= 4) {
      Serial.println("NOTE: The APIs differ by 4 C or more.");
      Serial.println("This can happen because APIs use different weather stations and update moments.");
    }
  }

  Serial.println("-------------------------------------------------");
}


// ============================================================
// SERIAL MONITOR COMMANDS
// ============================================================

void handleSerialCommands() {
  if (Serial.available() > 0) {
    String command = Serial.readStringUntil('\n');

    command.trim();
    command.toLowerCase();

    if (command == "") {
      return;
    }

    Serial.println();
    Serial.print("SERIAL COMMAND: ");
    Serial.println(command);

    if (command == "weather" || command == "weer" || command == "api") {
      Serial.println("Manual API update requested.");
      getAllWeatherData();
      lastWeatherUpdate = millis();

    } else if (command == "status") {
      printStatus();

    } else if (command == "help") {
      printHelp();

    } else {
      Serial.println("SERIAL ERROR: Unknown command.");
      Serial.println("Type 'help' for all commands.");
    }
  }
}


void printStatus() {
  Serial.println();
  Serial.println("---------------- STATUS ----------------");

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("Wi-Fi: CONNECTED");

    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());

    Serial.print("Signal strength: ");
    Serial.print(WiFi.RSSI());
    Serial.println(" dBm");

  } else {
    Serial.println("Wi-Fi: NOT CONNECTED");
  }

  if (openWeatherWorks) {
    Serial.println("OpenWeatherMap: LAST REQUEST SUCCESSFUL");
  } else {
    Serial.println("OpenWeatherMap: LAST REQUEST FAILED OR NOT DONE");
  }

  if (openMeteoWorks) {
    Serial.println("Open-Meteo: LAST REQUEST SUCCESSFUL");
  } else {
    Serial.println("Open-Meteo: LAST REQUEST FAILED OR NOT DONE");
  }

  Serial.println("----------------------------------------");
}


void printHelp() {
  Serial.println();
  Serial.println("--------------- COMMANDS ----------------");
  Serial.println("weather / weer / api -> Request both APIs now");
  Serial.println("status               -> Show Wi-Fi and API status");
  Serial.println("help                 -> Show this command list");
  Serial.println("-----------------------------------------");
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
 
STEP 5
- Change the API code to make it work (WIFI, SSID)
- Serial Monitor
- Speed to 9600 baud
- Look at display
- Later comes temperature

STEP 6
- Connect the weather API to LED
- Show code that needs adding

TROUBLE SHOOTING

Open Arduino IDE and install the following things:
- ESP8266 board package
- esp8266 By ESP8266 Community
- **Libraries:**
  - Adafruit NeoPixel
  - ArduinoJson by Benoit Blanchon

Go to tools -> board -> NodeMCU 1.0 (ESP-12E Module)


# Step 3 Connecting LED strip
- Get an LED strip
- Look closely there are some indicators on the strip, like: +5v, Din and G
- Get some wires and place them in those points, best is to put a red one on +5v, yellow on Din and Black on G, but thats your own choice
- Put Din (yellow) on D1, +5v (red) on a 3v and G (black) on a G
- If you already installed the Adafruit NeoPixel then move on otherwise go back and install it

## Open Simple (Fix later)
- Put your NodeMCU on
- Navigate to File>Examples>Adafruit Neopixel open simple (might need to scroll all the way down)
- Look if your board connects well
- - Go to Tools -> Board -> esp8266 -> NodeMCU 1.0 (ESP-12E Module)
  - Select the right port: Tools -> Port -> Port where you on
  - Put speed on 921600 to make sure it works
  - Make the pin number correct
  - Fix LED amount
  - Upload code and see if you need to trouble shoot

## List of sources
- Designing Connected Products : UX for the Consumer Internet of Things van Claire Rowland". Bekijk via O'Reilly
- OpenWeahterMap: https://openweathermap.org/
- Troubleshooting with: [aichat.hva.nl](https://aichat.hva.nl/chat/)
- Api code: https://gist.github.com/icecream4all/7e9db0333f44192a6071eb73efe23329#file-nodemcu-weather-ino
