# Temperature-Humidity-Control-THC-
A modern temperature and humidity controller for cooling systems, built on Arduino Nano with updated libraries. Provides stable DHT11 readings, flicker‑free LCD output, relay control, EEPROM‑saved setpoints, and reliable sequential timing. Designed for beginners reviving older Arduino projects.


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
