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

# Step 1 Install Libraries (make a short tutorial)
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
