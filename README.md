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
