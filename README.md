# esp32-s3-geek





''
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
    key: "ADD_YOUR_ENCRYPTION_KEY_HERE"  # Users must add their own key
  on_client_connected:
    - lambda: |-
        id(ha_status) = "HA: Connected";
  on_client_disconnected:
    - lambda: |-
        id(ha_status) = "HA: Disconnected";

ota:
  platform: esphome
  password: "ADD_YOUR_OTA_PASSWORD_HERE"  # Users must add their own password

wifi:
  ssid: !secret wifi_ssid  # Ensure you have this defined in your secrets.yaml
  password: !secret wifi_password  # Ensure you have this defined in your secrets.yaml
  fast_connect: true
  on_connect:
    - lambda: |-
        id(wifi_status) = "WiFi: Connected";
  on_disconnect:
    - lambda: |-
        id(wifi_status) = "WiFi: Disconnected";

# Bluetooth Proxy
bluetooth_proxy:
  active: true

# Global variables for displaying information
globals:
  - id: screen_index
    type: int
    restore_value: no
    initial_value: '0'
  - id: wifi_status
    type: std::string
    restore_value: no
    initial_value: '"WiFi: Unknown"'
  - id: ha_status
    type: std::string
    restore_value: no
    initial_value: '"HA: Unknown"'
  - id: ble_device_name
    type: std::string
    restore_value: no
    initial_value: '"Last BLE: ---"'

# Bluetooth scanner for tracking all BLE devices
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
            id(screen_index) += 1;
            if (id(screen_index) > 1) id(screen_index) = 0;

sensor:
  - platform: wifi_signal
    name: "WiFi Signal"
    id: wifi_strength
    update_interval: 10s

# SPI configuration for ST7789 LCD display
spi:
  clk_pin: GPIO12
  mosi_pin: GPIO11

# Font configuration for text display
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

# LCD display for information
display:
  - platform: st7789v
    model: "TTGO_TDISPLAY_135X240"
    cs_pin: GPIO10
    dc_pin: GPIO8
    reset_pin: GPIO9
    rotation: 270
    update_interval: 1s
    lambda: |-
      it.fill(Color::BLACK);
      if (id(screen_index) == 0) {
        it.printf(10, 20, id(font2), id(wifi_status) == "WiFi: Connected" ? id(color_green) : id(color_red), "WiFi: %s", id(wifi_status).c_str());
        it.printf(10, 60, id(font2), id(ha_status) == "HA: Connected" ? id(color_green) : id(color_red), "HA: %s", id(ha_status).c_str());
        it.printf(10, 100, id(font2), id(color_white), "Last BLE: %s", id(ble_device_name).c_str());
      } else {
        it.printf(10, 20, id(font2), id(color_white), "WiFi Signal: %d dBm", (int)id(wifi_strength).state);
        it.printf(10, 60, id(font2), id(color_white), "Press Btn to return");
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
""
