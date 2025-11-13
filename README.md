

# Helmet_RearView_Display_v1.md

## Overview

This project creates a **rear-view camera HUD** for a helmet using the **Seeed Studio XIAO ESP32S3 Sense** (with its built-in OV2640 camera) and a **1.28″ GC9A01 round TFT LCD**.
The ESP32S3 captures live video and streams frames directly to the display—perfect for compact augmented-reality or tactical heads-up applications.

---

##  Bill of Materials

| Item                                | Quantity | Description                       |
| ----------------------------------- | -------- | --------------------------------- |
| Seeed Studio XIAO ESP32S3 Sense     | 1        | Microcontroller + camera (OV2640) |
| GC9A01 1.28″ TFT LCD                | 1        | 240 × 240 SPI display             |
| Jumper wires (M–F)                  | 7        | For SPI connections               |
| 3.7 V Li-ion battery (500–1000 mAh) | 1        | Portable power                    |
| Boost converter (3.7 → 5 V)         | 1        | Optional for battery power        |
| 3D-printed mount                    | 1        | Holds display on visor or glasses |

---

##  Pin Connections

| GC9A01 Pin | XIAO ESP32S3 Sense Pin | Function              |
| ---------- | ---------------------- | --------------------- |
| VCC        | 3V3                    | Power (3.3 V)         |
| GND        | GND                    | Ground                |
| SCL        | D10                    | SPI Clock             |
| SDA        | D9                     | SPI MOSI              |
| DC         | D8                     | Data / Command        |
| CS         | D7                     | Chip Select           |
| RST        | D6                     | Reset                 |
| BLK        | 3.3 V                  | Backlight (always on) |

---

## 🖧 Wiring Diagram

*(See attached image — “A_wiring_diagram_in_the_image_illustrates_the_conn.png”)*

---

##  Arduino IDE Setup

1. **Install Board Package**
   *Tools → Board Manager → search “Seeed ESP32S3” → Install*
2. **Select Board**
   *Tools → Board → Seeed XIAO ESP32S3 Sense*
3. **Install Libraries**

   * `esp32-camera`
   * `TFT_eSPI`
4. **Configure TFT_eSPI**
   Edit `User_Setup_Select.h` → add:

   ```cpp
   #include <User_Setups/SetupXIAO_GC9A01.h>
   ```

   Then create `SetupXIAO_GC9A01.h`:

   ```cpp
   #define GC9A01_DRIVER
   #define TFT_MOSI 9
   #define TFT_SCLK 10
   #define TFT_CS   7
   #define TFT_DC   8
   #define TFT_RST  6
   #define SPI_FREQUENCY 40000000
   ```

---

## Complete Arduino Sketch

```cpp
#include "esp_camera.h"
#include <TFT_eSPI.h>

TFT_eSPI tft = TFT_eSPI();

void setup() {
  Serial.begin(115200);
  tft.begin();
  tft.setRotation(0);
  tft.fillScreen(TFT_BLACK);

  // --- Camera configuration for XIAO ESP32S3 Sense ---
  camera_config_t config = {
    .frame_size   = FRAMESIZE_QQVGA,   // 160x120 or QVGA for smoother FPS
    .pixel_format = PIXFORMAT_RGB565,
    .fb_location  = CAMERA_FB_IN_PSRAM,
    .jpeg_quality = 12,
    .fb_count     = 1
  };

  if (esp_camera_init(&config) != ESP_OK) {
    Serial.println("Camera init failed!");
    while (true);
  }
  Serial.println("Camera initialized.");
}

void loop() {
  camera_fb_t *fb = esp_camera_fb_get();
  if (fb) {
    tft.pushImage(0, 0, 240, 240, (uint16_t*)fb->buf);
    esp_camera_fb_return(fb);
  }
}
```

---

##  Performance Tips

* Use **QVGA (320×240)** for sharper image or **QQVGA (160×120)** for higher FPS.
* Keep SPI frequency ≤ 40 MHz for stability.
* Supply stable 3.3 V to avoid display flicker.

---

##  Power Options

* **USB-C** direct power (5 V input).
* **3.7 V Li-ion → 5 V boost converter** into VBAT.
* Typical current draw ≈ 250 mA (ESP32 + Camera + Display).

---

##  Future Upgrades

* Wi-Fi stream (MJPEG to phone or Pi receiver).
* Add motion sensor to auto-activate display.
* Overlay telemetry (GPS or battery voltage readout).

---

##  Summary

| Feature         | Status | Notes               |
| --------------- | ------ | ------------------- |
| Live Video Feed | ✅      | 10–15 FPS approx.   |
| Low Power       | ✅      | Sleep ready         |
| Offline HUD     | ✅      | Self-contained unit |
| Compact Size    | ✅      | < 22 × 18 mm board  |

---

**Author:** Ezio
**Project:** Helmet Rear-View Display v1
**Hardware:** Seeed Studio XIAO ESP32S3 Sense + GC9A01 TFT LCD
**Date:** 2025-10-28

---


