# Manual Foster Vase

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
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/install_esp_library.jpeg" />

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
 * Simpel weerstation met ESP8266 en OpenWeatherMap API
 * D. de Vries
 */

#include <ArduinoJson.h>
#include <ESP8266WiFi.h>
#include <WiFiClient.h>

// === CONFIGURATIE ===
char ssid[] = "YOUR_WIFI_NAAM";         // Vul hier de naam van je WiFi in
char pass[] = "YOUR_WIFI_WACHTWOORD";  // Vul hier je WiFi-wachtwoord in

const char server[] = "api.openweathermap.org";
String nameOfCity = "STAD,LANDCODE";    // Bijvoorbeeld "Amsterdam,NL"
String apiKey = "YOUR_OPENWEATHERMAP_API_KEY";  // Vul hier je eigen API-sleutel in

WiFiClient client;

#define JSON_BUFF_DIMENSION 8192
String text;

unsigned long lastConnectionTime = 0;
const unsigned long postInterval = 10000;  // elke 10 sec

void setup() {
  Serial.begin(9600);
  while (!Serial) { ; }

  text.reserve(JSON_BUFF_DIMENSION);

  Serial.println("Verbinden met WiFi...");
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi verbonden!");

  // === Testbare API URL printen voor browser ===
  String testURL = "http://api.openweathermap.org/data/2.5/forecast?q=" + nameOfCity + "&APPID=" + apiKey + "&mode=json&units=metric&cnt=1";
  Serial.println("\nTest deze URL in je browser:");
  Serial.println(testURL);
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
    client.println("GET /data/2.5/forecast?q=" + nameOfCity + "&APPID=" + apiKey + "&mode=json&units=metric&cnt=1 HTTP/1.1");
    client.println("Host: api.openweathermap.org");
    client.println("Connection: close");
    client.println();

    // Header overslaan
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
      Serial.println("Fout: Geen JSON-data ontvangen!");
    }

    client.stop();
  } else {
    Serial.println("Fout: Verbinding met server mislukt!");
  }
}

void parseJson(const char* jsonString) {
  DynamicJsonDocument doc(JSON_BUFF_DIMENSION);
  DeserializationError error = deserializeJson(doc, jsonString);
  if (error) {
    Serial.println("Fout bij JSON: " + String(error.c_str()));
    return;
  }

  JsonArray list = doc["list"];
  if (list.isNull() || list.size() < 1) {
    Serial.println("Fout: JSON bevat geen voorspelling!");
    return;
  }

  JsonObject forecast = list[0];
  String city = doc["city"]["name"];
  String weather = forecast["weather"][0]["main"];
  String description = forecast["weather"][0]["description"];
  float temp = forecast["main"]["temp"];

  Serial.println("Voorspelling voor " + city + ": " + weather + " (" + description + ")");
  Serial.println("Temperatuur: " + String(temp) + "°C");


  weather.toUpperCase();

  if (weather == "RAIN") {
    Serial.println("Het gaat regenen! 🌧️");
    // TODO: hier iets met je hardware doen
  } else if (weather == "SNOW") {
    Serial.println("Het gaat sneeuwen! ❄️");
    // TODO: hier iets met je hardware doen
  } else if (weather == "CLEAR") {
    Serial.println("Het wordt zonnig! ☀️");
    // TODO: hier iets met je hardware doen
  } else if (description.indexOf("HAIL") != -1) {
    Serial.println("Het gaat hagelen! 🌨️");
    // TODO: hier iets met je hardware doen
  } else if (weather == "CLOUDS") {
    Serial.println("Het wordt bewolkt ☁️");
    // TODO: hier iets met je hardware doen
  } else {
    Serial.println("Ander weer: " + description);
    // TODO: hier iets met je hardware doen
  }
}
```
 
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
