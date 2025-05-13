
## Main Objective

Develop a firmware application for a BLE-UART Gateway that:
 Forwards UART messages to a connected BLE central.
 Minimizes power via inactivity-based sleep.
 Responds to command input for BLE control and power mode overrides.

-----------------------------------------------------------------------------------------------------------------

##  Platform

 Microcontroller: ESP32 (ESP-IDF)
 Framework: ESP-IDF (tested on v5.x)
 UART Interface: UART0 (GPIO1 TXD, GPIO3 RXD)
 BLE Interface: Custom GATT Server

---------------------------------------------------------------------------------------------------------------------------------

##  Features Implemented

###  BLE
- Initializes BLE stack with custom GATT service.
- 1 characteristic supporting read, write, and notify.
- Advertises on startup.
- Handles BLE connect/disconnect events.
- Sends notifications to BLE central on UART input.

###  UART
- UART0 initialized with default pins and interrupt-based RX.
- Reads up to 64-byte messages from PC terminal (e.g. I am using Serial Debug Assistant).
- Forwards each valid message to BLE central immediately.(valid msgs except commands)

###  Low-Power Mode
- Inactivity timer (5s) checks for no UART/BLE activity.
- Enters light sleep mode on timeout.
- Wake-up triggers:
  - UART RX via GPIO wake-up.(even 1 baud chsnge)
  - BLE connection/write events.
- BLE advertising resumes after wake.

###  UART Commands (Bonus)
 Command   Function                           
--------------------------------------------------------------------------------------------------------------------------------------------
 `BLEON`   Start BLE advertising              
 `BLEOFF`  Stop BLE advertising               
 `STATUS`  Log BLE and sleep status           
 `BYPASS`  Prevent sleep mode (active mode)   for testing purpose onlyy
 `ENABLE`  Resume automatic sleep mode for testing purposes only       

----------------------------------------------------------------------------------------------------------------------------------------------------------

##  Test Steps and steps to reproduce

1. Flash the code to your ESP32 using ESP-IDF (`idf.py -p PORT flash monitor`).
2. Open a serial terminal (Tera Term / PuTTY / Hercules/ Serial Debug Assistant).
3. Connect to BLE using nRF Connect mobile app:
   - Scan for device named `ESP_GATTS_DEMO`.
   - Connect and enable notifications.
4. send UART messages from terminal; observe BLE notifications on phone.
5. Test commands:
   - Type `BLEOFF` → Device stops advertising.
   - Type `BLEON` → Advertising restarts.
   - Type `BYPASS` → Sleep mode is bypassed.
   - Type `ENABLE` → Sleep resumes after 5s idle.
   - Type `STATUS` → Shows BLE connected state and bypass mode.

---

## Low-Power Strategy

- Uses FreeRTOS software timer `xTimer` to track inactivity.
- Light sleep is entered via `esp_light_sleep_start()` after 5s idle which is globally declared so interval can be managed easily.
- Wake-up on:
  - GPIO (UART RXD low so even if 1 bit data is sent over uart0 then it wakes up)
  - BLE connection or write activity
- BLE advertising restarts on wake-up.
- `BYPASS` command disables automatic sleep until `ENABLE` is issued.

---

## Build Instructions

1. Clone or download this repo.
2. Open terminal:
   idf.py set-target esp32
   idf.py build
   idf.py -p /dev/ttyUSB0 flash monitor
