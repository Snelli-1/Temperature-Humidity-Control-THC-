#include <EEPROM.h>
#include <LiquidCrystal.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <DHT_U.h>

// -------------------- Hardware --------------------
#define DHT_PIN 8
#define DHTTYPE DHT11
DHT dht(DHT_PIN, DHTTYPE);

LiquidCrystal lcd(7, 6, 5, 4, 3, 2);

// Buttons (ACTIVE HIGH: PIN -- button -- +5V, external pulldown resistor)
const int BTN_T_UP   = 10;
const int BTN_T_DOWN = 11;
const int BTN_H_UP   = 12;
const int BTN_H_DOWN = 13;

// Relays
const int RELAY1 = 14; // A0
const int RELAY2 = 15; // A1

// -------------------- Setpoints --------------------
int T_set = 25;
int H_set = 55;

const int T_MIN = -20;
const int T_MAX = 60;
const int H_MIN = 0;
const int H_MAX = 100;

const int EEPROM_T_ADDR = 0;
const int EEPROM_H_ADDR = 1;
const int T_OFFSET = 20;   // -20..60 → 0..80

// -------------------- Timing --------------------
unsigned long lastSensor  = 0;
unsigned long lastProcess = 0;
unsigned long lastLCD     = 0;

const unsigned long SENSOR_MS  = 3000;
const unsigned long PROCESS_MS = 500;
const unsigned long LCD_MS     = 500;

// -------------------- Sensor data --------------------
float curT = NAN;
float curH = NAN;
bool haveValidReading = false;

// ========================================
// Robust button handling for ACTIVE HIGH
// - One short press = exactly one step (on PRESS edge)
// - Long press repeat starts after 1s
// - Strong debounce for real hardware
// ========================================
const unsigned long DEBOUNCE_MS   = 60;     // kicsit erősebb, hogy a pattogás ne csináljon 5 élt
const unsigned long LONG_START_MS = 1000;   // 1s után indul a repeat
const unsigned long REPEAT_MS     = 180;    // ismétlés sebesség (állítható)

struct Button {
  int pin;

  bool raw;                  // pillanatnyi olvasás (pressed? true/false)
  bool stable;               // debounced állapot
  unsigned long rawChangedAt;

  bool pressed;              // stable == true (HIGH)
  unsigned long nextRepeatAt;
};

Button bTup = {BTN_T_UP,   false, false, 0, false, 0};
Button bTdn = {BTN_T_DOWN, false, false, 0, false, 0};
Button bHup = {BTN_H_UP,   false, false, 0, false, 0};
Button bHdn = {BTN_H_DOWN, false, false, 0, false, 0};

// returns true exactly when ONE step should occur
bool processButton(Button &b, unsigned long now) {
  bool step = false;

  // ACTIVE HIGH: pressed when digitalRead == HIGH
  bool r = (digitalRead(b.pin) == HIGH);

  if (r != b.raw) {
    b.raw = r;
    b.rawChangedAt = now;
  }

  // Debounce: accept a state change only if stable for DEBOUNCE_MS
  if ((now - b.rawChangedAt) >= DEBOUNCE_MS) {
    if (b.stable != b.raw) {
      b.stable = b.raw;

      // PRESS edge (becomes HIGH)
      if (b.stable) {
        b.pressed = true;

        // short press: EXACTLY one step on press
        step = true;

        // long press: start repeating after 1s
        b.nextRepeatAt = now + LONG_START_MS;
      }
      // RELEASE edge (becomes LOW)
      else {
        b.pressed = false;
      }
    }
  }

  // While held: long press repeat (no "burst catch-up")
  if (b.pressed && b.stable) {
    if (now >= b.nextRepeatAt) {
      step = true;
      b.nextRepeatAt = now + REPEAT_MS;
    }
  }

  return step;
}

void clampInt(int &v, int mn, int mx) {
  if (v < mn) v = mn;
  if (v > mx) v = mx;
}

// EEPROM wear reduction: update writes only if different
void saveIfChanged(int addr, int oldv, int newv, int offset = 0) {
  int storedOld = oldv + offset;
  int storedNew = newv + offset;
  if (storedOld != storedNew) EEPROM.update(addr, (byte)storedNew);
}

// -------------------- LCD update --------------------
void updateLCD() {
  lcd.setCursor(0, 0);
  if (haveValidReading) {
    lcd.print("T:");
    lcd.print(curT, 1);
    lcd.print(" Ts:");
    if (T_set < 10 && T_set >= 0) lcd.print("0");
    lcd.print(T_set);
    lcd.print("   ");
  } else {
    lcd.print("T: --- Ts:");
    if (T_set < 10 && T_set >= 0) lcd.print("0");
    lcd.print(T_set);
    lcd.print("   ");
  }

  lcd.setCursor(0, 1);
  if (haveValidReading) {
    lcd.print("H:");
    lcd.print(curH, 0);
    lcd.print("% Hs:");
    if (H_set < 10) lcd.print("0");
    lcd.print(H_set);
    lcd.print(" R");
    lcd.print((curT > T_set || curH > H_set) ? '1' : '0');
    lcd.print(" ");
  } else {
    lcd.print("H: --- Hs:");
    if (H_set < 10) lcd.print("0");
    lcd.print(H_set);
    lcd.print(" R0 ");
  }
}

// -------------------- Setup --------------------
void setup() {
  lcd.begin(16, 2);
  lcd.print("Starting...");

  pinMode(RELAY1, OUTPUT);
  pinMode(RELAY2, OUTPUT);
  digitalWrite(RELAY1, HIGH);
  digitalWrite(RELAY2, HIGH);

  // IMPORTANT for your wiring: external pulldown -> plain INPUT (no internal pullup!)
  pinMode(BTN_T_UP,   INPUT);
  pinMode(BTN_T_DOWN, INPUT);
  pinMode(BTN_H_UP,   INPUT);
  pinMode(BTN_H_DOWN, INPUT);

  dht.begin();
  delay(1500);

  byte vT = EEPROM.read(EEPROM_T_ADDR);
  byte vH = EEPROM.read(EEPROM_H_ADDR);

  // sanity checks (avoid 255 -> nonsense)
  if (vT <= (byte)(T_MAX + T_OFFSET)) T_set = (int)vT - T_OFFSET;
  else T_set = 25;

  if (vH <= 100) H_set = (int)vH;
  else H_set = 55;

  clampInt(T_set, T_MIN, T_MAX);
  clampInt(H_set, H_MIN, H_MAX);

  updateLCD();
}

// -------------------- Loop --------------------
void loop() {
  unsigned long now = millis();

  int oldT = T_set;
  int oldH = H_set;

  // Buttons
  if (processButton(bTup, now)) T_set++;
  if (processButton(bTdn, now)) T_set--;
  if (processButton(bHup, now)) H_set++;
  if (processButton(bHdn, now)) H_set--;

  clampInt(T_set, T_MIN, T_MAX);
  clampInt(H_set, H_MIN, H_MAX);

  saveIfChanged(EEPROM_T_ADDR, oldT, T_set, T_OFFSET);
  saveIfChanged(EEPROM_H_ADDR, oldH, H_set);

  // Sensor read
  if (now - lastSensor >= SENSOR_MS) {
    float t = dht.readTemperature();
    float h = dht.readHumidity();

    if (!isnan(t) && !isnan(h)) {
      curT = t;
      curH = h;
      haveValidReading = true;
    } else {
      haveValidReading = false;
    }
    lastSensor = now;
  }

  // Relay processing
  if (now - lastProcess >= PROCESS_MS) {
    bool relay = false;
    if (haveValidReading) relay = (curT > T_set) || (curH > H_set);

    digitalWrite(RELAY1, relay ? LOW : HIGH);
    digitalWrite(RELAY2, relay ? LOW : HIGH);

    lastProcess = now;
  }

  // LCD update
  if (now - lastLCD >= LCD_MS) {
    updateLCD();
    lastLCD = now;
  }
}
