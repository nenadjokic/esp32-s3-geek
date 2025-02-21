# 🚀 ESP32-S3 GEEK - Bluetooth Proxy with Display for Home Assistant  

Transform your **ESP32-S3 GEEK** into a powerful **Bluetooth Proxy** with a real-time display for **Home Assistant**!  
This ESPHome configuration allows you to extend Bluetooth coverage while displaying **WiFi status, HA connection status, and BLE device detection** on the built-in **1.14" ST7789V display**.

---

## 📌 Features  

✅ **Bluetooth Proxy for Home Assistant** – Extends BLE device range and forwards BLE signals.  
✅ **WiFi & HA Status Monitoring** – Displays **Connected/Disconnected** states with color coding.  
✅ **Last Detected BLE Device** – Shows the most recently detected BLE device.  
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
   
---

# 🎛️ How to Use  

## 🖥️ On-Screen Information  
- **WiFi:** Displays connection status (**Green = Connected, Red = Disconnected**).  
- **HA:** Shows if Home Assistant API is connected.  
- **Last BLE:** Displays the most recently detected BLE device.  
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

---

# 📥 Download & Contribute  

This project is **open-source**! Feel free to modify and improve it.  
If you encounter issues or have suggestions, open a **GitHub Issue** or **Pull Request**.  

📌 **Download the latest ESPHome configuration from this repository and enhance your smart home today!** 🚀  


# Configuration
```
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
    key: "YOUR ENCRYPTION KEY"
  on_client_connected:
    - lambda: |-
        id(ha_status) = "Connected";
  on_client_disconnected:
    - lambda: |-
        id(ha_status) = "Disconnected";

ota:
  platform: esphome
  password: "YOUR PASSWORD"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true
  on_connect:
    - lambda: |-
        id(wifi_status) = "Connected";
  on_disconnect:
    - lambda: |-
        id(wifi_status) = "Disconnected";

# Bluetooth Proxy
bluetooth_proxy:
  active: true

# Global variables for display
globals:
  - id: screen_index
    type: int
    restore_value: no
    initial_value: '0'
  - id: wifi_status
    type: std::string
    restore_value: no
    initial_value: '"Unknown"'
  - id: ha_status
    type: std::string
    restore_value: no
    initial_value: '"Unknown"'
  - id: ble_device_name
    type: std::string
    restore_value: no
    initial_value: '"--"'

# Bluetooth Tracker
esp32_ble_tracker:

binary_sensor:
  - platform: gpio
    pin: 
      number: GPIO1
      inverted: true
    name: "Button 1"
    id: button1
    on_press:
      then:
        - lambda: |-
            id(screen_index) = (id(screen_index) + 1) % 2;

sensor:
  - platform: wifi_signal
    name: "WiFi Signal"
    id: wifi_strength
    update_interval: 30s

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
    update_interval: 500ms
    lambda: |-
      it.fill(Color::BLACK);
      if (id(screen_index) == 0) {
        it.print(10, 20, id(font2), id(color_white), "WiFi: ");
        it.print(90, 20, id(font2), id(wifi_status) == "Connected" ? id(color_green) : id(color_red), id(wifi_status).c_str());
        
        it.print(10, 60, id(font2), id(color_white), "HA: ");
        it.print(70, 60, id(font2), id(ha_status) == "Connected" ? id(color_green) : id(color_red), id(ha_status).c_str());
        
        it.print(10, 100, id(font2), id(color_white), "Last BLE: ");
        it.print(130, 100, id(font2), id(color_white), id(ble_device_name).c_str());
      } else {
        it.print(10, 20, id(font2), id(color_white), "WiFi Signal: ");
        it.printf(150, 20, id(font2), id(color_white), "%d dBm", (int)id(wifi_strength).state);
        it.print(10, 60, id(font2), id(color_white), "Press Btn to return");
      }

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
