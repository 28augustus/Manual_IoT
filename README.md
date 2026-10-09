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
- [Go to] (https://openweathermap.org/)
- Create an account or log in
- Go to My API Keys (Picture needed)
- <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/get_api_key.png" />
- get_api_key
- Create account or sign in if you already have an account
- If you can't sign in or make a new account (like me) Then it's probaply becasue you already have an account you can do the following:
- - Choose forgot password and make a new password
- Once you made an account click on API Key
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/api_code.png" />
- Then make sure you copy or safe your code safely
- **DO NOT SHARE YOUR CODE ONLINE** (that's why mine is hidden)
- Keep private
  <img src="https://github.com/28augustus/Manual_IoT/blob/main/afbeeldingen/api_code_copy.png" />
 

 
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
