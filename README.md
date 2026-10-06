# Manual Foster Vase

## what do you need:
### Hard ware
- NodeMCU ESP8266
- water-level sensor
- temperature sensor
- LEDs / NeoPixel LED strip
- Physical button

### Soft ware
- Arduino IDE
- ESP8266 board package
- Library
- LED library, (Adafruit NeoPixel)
- Adafruit IO
- Weather API
- Wi-Fi network

# Step 1 Install Libraries
Open Arduino IDE and install the following libraries:
- Adafruit NeoPixel
- ArduinoJson by Benoit Blanchon

Go to tools -> board -> NodeMCU 1.0 (ESP-12E Module)


# Step 2 Connecting Api
**Put the code in Arduino IDE**
[API Code] (https://gist.github.com/icecream4all/7e9db0333f44192a6071eb73efe23329#file-nodemcu-weather-ino)

Go to Openweather to get an API key:
[Visit Open Weather] (https://openweathermap.org/api)

- Press get API key and fill in the form
- Go to My API Keys and make a Api key. Keep acces to the code, you will need it later to put it in the code (DO NOT SHARE YOUR KEY ON A PUBLIC SPACE)
- Go to Arduino IDE and paste the given code (API Code)
- Fill in the wifi name and password with **your** wifi/password name
- Paste with String apiKey your api code that is not supposed to be shared in public



# Step 3 Connecting LED strip
