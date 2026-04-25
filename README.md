# ESP32-S USB Dongle (Auto-WiFi & LED Enhanced)

> **Adapted by Ivan Svarkovsky, 2026**  
> Based on Espressif's official [esp-iot-solution/examples/usb/device/usb_dongle](https://github.com/espressif/esp-iot-solution)

A fully autonomous USB Wi-Fi dongle solution for ESP32-S2/S3/P4.  
**Zero console configuration required.** Just plug in, wait 4 seconds, and configure via the ESP-Touch mobile app. Credentials are saved to FLASH and auto-restored on next boot.

## ✨ Key Modifications
| Feature | Description |
|---------|-------------|
|  **Auto-Reconnect** | Automatically connects to the last known Wi-Fi network on power-up. |
| 💾 **Persistent Storage** | Wi-Fi credentials are saved to `NVS/FLASH` (`WIFI_STORAGE_FLASH`). Survives reboots & firmware updates. |
| 📱 **SmartConfig Fallback** | If no saved network is found or connection fails within **4 seconds**, the device automatically enters ESP-Touch mode. |
| 🎨 **LED Status Indication** | Real-time feedback via addressable RGB LED (WS2812/SK6812). |
| ⌨️ **Zero-CLI Setup** | No need to use `sta -s <ssid>` commands. All configuration is handled wirelessly via mobile app. |

##  LED Indication Guide
| Color | State | Meaning |
|-------|-------|---------|
| **Red** | `20, 0, 0` | Initializing / Disconnected / Connection Error |
| **Green** | `0, 20, 0` | Successfully connected to Wi-Fi & Internet ready |
| **Blue** | `0, 0, 20` | Waiting for ESP-Touch configuration (SmartConfig active) |

## 🛠 Hardware Requirements
- **MCU:** ESP32-S2, ESP32-S3, or ESP32-P4 board with USB-OTG support
- **LED (Optional):** Addressable RGB LED (WS2812/SK6812) for status indication.
  - *Default Pin:* **GPIO48** (ESP32-S3 DevKitC/UNO), **GPIO18** (ESP32-S2 Saola).
  - *Note:* If your board uses a different pin or has no LED, the firmware will automatically detect this, log a warning, and continue working normally without visual indication. No code changes required.
- **USB Cable:** Data-capable USB cable for power & enumeration
- **Mobile App:** [ESP-Touch](https://github.com/EspressifApp/EsptouchForAndroid) (Android) or [ESP-Touch](https://github.com/EspressifApp/EsptouchForIOS) (iOS)
  
> ⚠️ **Note on LED Pin:** The code defaults to `GPIO48`. If your board uses a different pin, modify `#define BLINK_GPIO` in `main/cmd_wifi.c`. The initialization routine safely handles missing LEDs without crashing.

## 📦 Software & Build
Requires **ESP-IDF v5.4+**. The `led_strip` dependency is automatically managed via `idf_component.yml`.

```bash
# 1. Clean project artifacts
idf.py fullclean

# 2. Add LED dependency
idf.py add-dependency "espressif/led_strip^2.0.0"

# 3. Erase flash memory (clears old Wi-Fi passwords)
esptool.py --chip esp32s3 --port /dev/ttyUSB0 erase_flash

# 4. Set target chip (esp32s2 / esp32s3 / esp32p4)
idf.py set-target esp32s3

# 5. Build and Flash
idf.py build flash
```

## 🚀 Quick Start Guide

### ⚠️ Important Hardware Note
**USB Port Conflict:** On most ESP32-S2/S3 development boards, the same USB connector is used for both **UART debugging/flashing** and **USB-OTG (Network)**.  
- **During Flashing:** Connect via USB for UART.  
- **During Operation:** You must **disconnect the UART cable** (or switch to a dedicated UART adapter if available) and ensure the PC connects to the device purely as a **USB Network Device**. If you keep the UART console active, the USB Ethernet adapter may not enumerate correctly or will have conflicts.

### Step-by-Step Setup

1. **Flash the Firmware**  
   Connect the board via USB and flash using `idf.py flash`. Wait for the process to complete.

2. **Switch to USB-OTG Mode**  
   - If your board has a physical switch for "UART/USB", toggle it to **USB**.  
   - If not, simply **unplug and replug** the USB cable after flashing to reset the USB enumeration state.  
   - Your PC should now detect a new **Ethernet Adapter** (`usb0` on Linux, `RNDIS/Ethernet Gadget` on Windows).

3. **Wait for SmartConfig Fallback**  
   Plug in the dongle. Wait ~4 seconds.  
   - If no saved Wi-Fi is found, the LED will turn **Blue** 🔵.  
   - This indicates the device is ready to receive credentials via ESP-Touch.

4. **Configure via ESP-Touch App**  
   - Open the [ESP-Touch app](https://github.com/EspressifApp/EsptouchForAndroid) on your smartphone.  
   - Ensure your phone is connected to the target **2.4GHz Wi-Fi network**.  
   - Enter the Wi-Fi password and tap **Confirm**.  
   - The app broadcasts credentials via UDP. The dongle will receive them, connect, save to FLASH, and the LED will turn **Green** 🟢.

5. **Automatic Reconnection**  
   On subsequent power-ups, the device will automatically reconnect to the saved network. If the network is unavailable, it will fall back to **Blue/SmartConfig** mode again.

---

### 📖 Original Features & Advanced Configuration
This project is based on Espressif's official [esp-iot-solution/examples/usb/device/usb_dongle](https://github.com/espressif/esp-iot-solution/tree/master/examples/usb/device/usb_dongle).  
For details on:
- Enabling Bluetooth over USB (BTH)
- Using DFU for firmware upgrades
- Advanced TinyUSB configuration (ECM/RNDIS/CDC combinations)
- Troubleshooting driver issues on Windows/macOS/Linux

Please refer to the **original documentation** linked above. Our modification focuses on **autonomous Wi-Fi setup** and **LED status indication**, keeping the core USB stack intact.




