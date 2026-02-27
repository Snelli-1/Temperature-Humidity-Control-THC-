# Temperature-Humidity-Control-THC-

<p><a href="https://www.youtube.com/shorts/djUbr9WUUEY">
<img width="800" height="677" alt="Designer" src="https://github.com/user-attachments/assets/1f6c9440-8e78-48ed-bddb-1785dc36e684" />
</a></p>
<img width="800" height="677" alt="image" src="https://github.com/user-attachments/assets/cd992089-b05d-4779-87a3-a25ff39baac4" />

Optocoupler‑isolated MCU relay driver with flyback diode and motor transient suppression.

<img width="800" height="677" alt="relay_opto_relay_schematic_padded_2x_label_up" src="https://github.com/user-attachments/assets/963dcb07-b3d7-49f9-b348-4454f027c5e8" />

A modern temperature and humidity controller for cooling systems, built on Arduino Nano with updated libraries. Provides stable DHT11 readings, flicker‑free LCD output, relay control, EEPROM‑saved setpoints, and reliable sequential timing. Designed for beginners reviving the older Arduino project.

<img width="800" height="677" alt="image" src="https://github.com/user-attachments/assets/58ba843e-809a-4ae6-b3f9-71b7cf7af34e" />

Here are the Library Manager names to search + the preferred official links:

Built-in (no install needed)

EEPROM — EEPROM (built-in Arduino library)

Link: Arduino EEPROM Library docs [docs.arduino.cc]

LiquidCrystal — LiquidCrystal (often included with Arduino IDE)

Link: Arduino LiquidCrystal library docs [docs.arduino.cc]

Install via Arduino Library Manager

EasyButton — search: EasyButton (by Evert Arias)

Links: Arduino library page  • Official GitHub [docs.arduino.cc] [github.com]

Adafruit Unified Sensor — search: Adafruit Unified Sensor (provides <Adafruit_Sensor.h>)

Links: Arduino library page  • Official GitHub [docs.arduino.cc] [github.com]

DHT sensor library — search: DHT sensor library (provides <DHT.h> and <DHT_U.h>)

Link: Official GitHub [github.com]

Don’t install (deprecated)

Adafruit_DHT_Unified — deprecated/merged into DHT sensor library

Link: Deprecated repo notice [github.com], [github.com]

YouTube demonstration video:
https://www.youtube.com/shorts/djUbr9WUUEY

Step‑by‑Step Guide (Plain Text Version)
This guide explains how to build, wire, upload, and test the updated temperature and humidity controller using modern Arduino tools and libraries.

1. Install the Required Software
Arduino IDE (latest version):
https://www.arduino.cc/en/software

Board setup in Arduino IDE:

Tools → Board → Arduino AVR Boards → Arduino Nano

Tools → Processor → ATmega328P

If upload fails: ATmega328P (Old Bootloader)

Tools → Port → select the correct COM port

2. Install the Required Libraries
Open Library Manager:
Sketch → Include Library → Manage Libraries…

Install these:

EasyButton
https://github.com/evert-arias/EasyButton

Adafruit DHT Sensor Library
https://github.com/adafruit/DHT-sensor-library

Adafruit Unified Sensor
https://github.com/adafruit/Adafruit_Sensor

LiquidCrystal
(built into Arduino IDE)
https://www.arduino.cc/en/Reference/LiquidCrystal

3. Wire the Hardware (Plain Text)
DHT11 Sensor Wiring
VCC → 5V

DATA → D8

GND → GND

LCD 16×2 Wiring (Parallel)
RS → D7

E  → D6

D4 → D5

D5 → D4

D6 → D3

D7 → D2

LCD VCC → 5V

LCD GND → GND

Optional: contrast potentiometer between V0 and GND

LCD reference:
https://www.arduino.cc/en/Tutorial/HelloWorld

Button Wiring
Temperature Up → D10

Temperature Down → D11

Humidity Up → D12

Humidity Down → D13

Use pull‑down or pull‑up resistors depending on wiring style

Relay Wiring
Relay 1 input → A0

Relay 2 input → A1

Relay VCC → 5V

Relay GND → GND

Relays are active‑LOW (LOW = ON, HIGH = OFF)

Relay reference:
https://lastminuteengineers.com/relay-module-arduino-tutorial/

4. Add the Updated Source Code
Create a new sketch: File → New

Paste the full updated .ino file

Save the sketch

This version is compatible with modern libraries.

5. Upload the Code
Click Verify

Click Upload

Wait for “Done uploading”

If upload fails:
Tools → Processor → ATmega328P (Old Bootloader)

6. First Startup Test
Power the Arduino

LCD shows “Starting…”

After ~3 seconds, the first sensor reading appears:

Example:
T:25.3 Ts:28
H:45% Hs:63 R0

If the sensor is not ready:
T: --- Ts:28
H: --- Hs:63 R0

7. Adjusting Setpoints
Temperature setpoint (Ts):

Increase: D10

Decrease: D11

Humidity setpoint (Hs):

Increase: D12

Decrease: D13

Setpoints are saved to EEPROM automatically.

EEPROM reference:
https://www.arduino.cc/en/Reference/EEPROM

8. Relay Operation
Relays turn ON when:

Temperature > Ts
OR

Humidity > Hs

Relays turn OFF otherwise.

9. Troubleshooting (Plain Text)
LCD shows nothing:

Check wiring

Adjust contrast

Confirm pin mapping

LCD shows "---" for a long time:

Check DHT11 wiring

Ensure DATA is on D8

Confirm 5V and GND

Relays never switch:

Check if valid T/H values appear

Verify relay wiring

Remember: active‑LOW

Upload errors:

Try Old Bootloader

Check USB cable and COM port

10. Project Sources and References
Original project by Enrico Simonetti:

Article: https://enricosimonetti.com/arduino-temperature-driven-fan/

Original code:
https://gist.githubusercontent.com/esimonetti/b886736d753dda9933b6/raw/bdcd2cc057a613e67d30e0f5d6d9a8786718eb0f/arduino_temperature_driven_fan.ino

Updated libraries:

EasyButton: https://github.com/evert-arias/EasyButton

Adafruit DHT Sensor Library: https://github.com/adafruit/DHT-sensor-library

Adafruit Unified Sensor: https://github.com/adafruit/Adafruit_Sensor

LiquidCrystal: https://www.arduino.cc/en/Reference/LiquidCrystal

Arduino documentation:

IDE: https://www.arduino.cc/en/software

EEPROM: https://www.arduino.cc/en/Reference/EEPROM

LCD tutorial: https://www.arduino.cc/en/Tutorial/HelloWorld
LCD tutorial:
🔗 https://www.arduino.cc/en/Tutorial/HelloWorld


Percentage of Original Project Preserved
This updated version is based on the original temperature‑driven fan project by Enrico Simonetti. Most of the wiring remains identical to the original design. Only one wiring change was required to make the system compatible with modern libraries and the updated code.

Wiring preserved from the original project: about 95%
All wiring is the same except for one pin change.

Actual wiring change:

Original project: DHT sensor on A0

Updated project: DHT sensor on D8  
Everything else in the wiring remains identical.

Original concept and idea: about 40%
The purpose of controlling temperature with relays is unchanged.

Original wiring logic: about 95%
Only the DHT pin changed. All other wiring matches the original project.

Original code structure: about 10%
The old structure no longer works with modern libraries. Only the general sequence (read sensor → update display → control relays) remains.

Original code lines reused: about 5%
Most code had to be rewritten because the DHT library API changed, LiquidCrystal timing changed, EasyButton requires a different approach, EEPROM handling needed offset logic, and the old fast loop caused display flicker.

Overall original content preserved: about 25%
The wiring is mostly original, but the logic, timing, libraries, and stability features are almost entirely new.

Summary: the project is roughly 25% original and 75% modernized, rewritten, or expanded.
