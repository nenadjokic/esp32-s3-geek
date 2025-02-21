# 🚀 ESP32-S3 GEEK - Bluetooth Proxy with Display for Home Assistant  

Transform your **ESP32-S3 GEEK** into a powerful **Bluetooth Proxy** with a real-time display for **Home Assistant**!  
This ESPHome configuration allows you to extend Bluetooth coverage while displaying **WiFi status, HA connection status, and system uptime** on the built-in **1.14" ST7789V display**.

---

## 📌 Features  

✅ **Bluetooth Proxy for Home Assistant** – Extends BLE device range and forwards BLE signals.  
✅ **WiFi & HA Status Monitoring** – Displays **Connected/Disconnected** states with color coding.  
✅ **Uptime Display** – Shows how long the device has been running in a **Xd Xh Xm Xs** format.  
✅ **WiFi Signal Strength Display** – Allows you to monitor connection stability.  
✅ **Screen Switching via Button** – Toggle between different screens with a button press.  
✅ **Optimized Display Updates** – Reduces unnecessary updates for smoother performance.  

---

## 🛠️ Hardware Compatibility  

This configuration is designed for **ESP32-S3 GEEK** with a **1.14" ST7789V (240x135) display**, commonly available from **Waveshare** and other vendors.

---

## 📖 Installation Guide  

### 🔹 1. Prerequisites  

- **Home Assistant** with **ESPHome integration** installed.  
- **ESP32-S3 GEEK** board.  
- **USB cable** for flashing.  

### 🔹 2. Flash ESPHome to Your ESP32-S3 GEEK  

1. **Install ESPHome** in **Home Assistant** if not already installed.  
2. Copy the provided **`esp32-s3-geek.yaml`** configuration.  
3. Create a **new device** in **ESPHome**, paste the YAML configuration, and replace **WiFi credentials**:
  
```yaml
   wifi:
     ssid: "YourWiFiSSID"
     password: "YourWiFiPassword"
```

or use secrets file
```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

---

# 🎛️ How to Use  

## 🖥️ On-Screen Information  
- **WiFi:** Displays connection status (**Green = Connected, Red = Disconnected**).  
- **HA:** Shows if Home Assistant API is connected.
- **Uptime Display** – Shows how long the device has been running in a **Xd Xh Xm Xs** format.  
- **WiFi Signal Strength:** Available on the second screen.  

## 🔘 Button Functions  
- **Pressing the button toggles screens** (between status and signal strength).  

## 📡 Bluetooth Proxy  
- Once flashed, the device will automatically start **forwarding BLE signals** to Home Assistant.  

---

# 📝 Notes & Troubleshooting  

## **WiFi Connection Issues**  
🔹 Ensure your **WiFi SSID and password** are correct.  
🔹 Try enabling **fast_connect: true** in the WiFi section.  
🔹 Ensure your router supports **2.4GHz WiFi** (ESP32 does not support 5GHz).  

## **BLE Device Detection Issues**  
🔹 Make sure **Bluetooth is enabled** in Home Assistant.  
🔹 Place the **ESP32-S3 GEEK** near your BLE devices.  
🔹 Restart the ESP device if BLE devices are not appearing.  

## **Display Issues**  
🔹 If the screen is not updating, check if the **SPI pins are correctly defined**.  
🔹 Increase `update_interval` in the display configuration to **1.5s or 2s** for better performance.  
🔹 If text appears cut off, adjust the **X/Y coordinates** in the lambda function.  

# 🔑 API Encryption & OTA Password Explained  

## **🔹 Home Assistant API Encryption**  

The `api` section enables communication between **ESPHome** and **Home Assistant**.  
This ensures that your ESP32-S3 GEEK can send and receive data securely.  
```yaml
api:
  encryption:
    key: "YOUR ENCRYPTION KEY"

```
## **🔹 OTA (Over-the-Air) Update Password**

The ota section allows you to update your ESP device wirelessly, without connecting it via USB.
```ota:
  platform: esphome
  password: "YOUR PASSWORD"
```

# 📥 Download & Contribute  

This project is **open-source**! Feel free to modify and improve it.  
If you encounter issues or have suggestions, open a **GitHub Issue** or **Pull Request**.  

📌 **Download the latest ESPHome configuration from this repository and enhance your smart home today!** 🚀  

---

# Configuration
```yaml
esphome:
  name: esp32-s3-geek
  friendly_name: ESP32-S3-GEEK
  platformio_options:
    board_build.flash_mode: dio
    board_build.f_flash: 80000000L
    board_build.arduino.memory_type: qio_opi
    build_flags: 
      - "-D FORECAST_DAYS=5"

esp32:
  board: esp32-s3-devkitc-1
  framework:
    type: esp-idf
    version: recommended
  flash_size: 16MB
  variant: ESP32S3

# Enable logging
logger:
  level: DEBUG

# Enable Home Assistant API
api:
  encryption:
    key: "lwaqwLSj0NrA7PePqL4szFDwosCnrXN8oqvRqRIV8vQ="
  on_client_connected:
    - lambda: |-
        id(ha_status).publish_state("Connected");
  on_client_disconnected:
    - lambda: |-
        id(ha_status).publish_state("Disconnected");

ota:
  platform: esphome
  password: "4f73a9a0023c1c907f5734d114453985"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true
  on_connect:
    - lambda: |-
        id(wifi_status).publish_state("Connected");
  on_disconnect:
    - lambda: |-
        id(wifi_status).publish_state("Disconnected");

# Bluetooth Proxy
bluetooth_proxy:
  active: true

# Global variables for display
globals:
  - id: screen_index
    type: int
    restore_value: no
    initial_value: '0'

text_sensor:
  - platform: template
    name: "WiFi Status"
    id: wifi_status
    update_interval: never
  - platform: template
    name: "HA Status"
    id: ha_status
    update_interval: never
  - platform: template
    name: "Uptime"
    id: uptime_tracker_sensor
    update_interval: 10s
    lambda: |-
      int seconds = (int)id(uptime_sensor).state;
      int days = seconds / 86400;
      seconds %= 86400;
      int hours = seconds / 3600;
      seconds %= 3600;
      int minutes = seconds / 60;
      seconds %= 60;
      return { (std::to_string(days) + "d " + std::to_string(hours) + "h " + std::to_string(minutes) + "m " + std::to_string(seconds) + "s").c_str() };

sensor:
  - platform: uptime
    name: "Uptime Sensor"
    id: uptime_sensor
    update_interval: 10s
  - platform: wifi_signal
    name: "WiFi Signal"
    id: wifi_strength
    update_interval: 30s

# Bluetooth Tracker
esp32_ble_tracker:

binary_sensor:
  - platform: gpio
    pin: 
      number: GPIO1
      mode: INPUT_PULLUP
      inverted: true
    name: "Button 1"
    id: button1
    on_press:
      then:
        - lambda: |-
            id(screen_index) = (id(screen_index) + 1) % 2;

# SPI settings for ST7789 LCD
spi:
  clk_pin: GPIO12
  mosi_pin: GPIO11

# Font settings
font:
  - file: "/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf"
    id: font1
    size: 24
  - file: "/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf"
    id: font2
    size: 18
  - file: "/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf"
    id: font3
    size: 14

# LCD Display Configuration
display:
  - platform: st7789v
    model: "TTGO_TDISPLAY_135X240"
    cs_pin: GPIO10
    dc_pin: GPIO8
    reset_pin: GPIO9
    rotation: 270
    update_interval: 1s
    data_rate: 40MHz
    lambda: |-
      it.fill(Color::BLACK);
      it.print(10, 20, id(font2), id(color_white), "WiFi: ");
      it.print(90, 20, id(font2), id(wifi_status).state == "Connected" ? id(color_green) : id(color_red), id(wifi_status).state.c_str());
      
      it.print(10, 60, id(font2), id(color_white), "HA: ");
      it.print(70, 60, id(font2), id(ha_status).state == "Connected" ? id(color_green) : id(color_red), id(ha_status).state.c_str());
      
      it.print(10, 100, id(font2), id(color_white), "Uptime: ");
      it.print(130, 100, id(font2), id(color_white), id(uptime_tracker_sensor).state.c_str());

color:
  - id: color_light_grey
    hex: BBBBBB
  - id: color_dark_grey
    hex: 5F5F5F
  - id: color_white
    hex: FFFFFF
  - id: color_red
    hex: FF0000
  - id: color_green
    hex: 14FF00
  - id: color_yellow
    hex: FFFF00
```
