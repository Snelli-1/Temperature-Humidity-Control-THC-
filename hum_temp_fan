#include <EEPROM.h>
#include <EasyButton.h>
#include <LiquidCrystal.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <DHT_U.h>

// -------------------- Hardware --------------------
#define DHT_PIN 8
#define DHTTYPE DHT11
DHT dht(DHT_PIN, DHTTYPE);

LiquidCrystal lcd(7, 6, 5, 4, 3, 2);

// Buttons
const int BTN_T_UP   = 10;
const int BTN_T_DOWN = 11;
const int BTN_H_UP   = 12;
const int BTN_H_DOWN = 13;

EasyButton bTup(BTN_T_UP);
EasyButton bTdn(BTN_T_DOWN);
EasyButton bHup(BTN_H_UP);
EasyButton bHdn(BTN_H_DOWN);

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
unsigned long lastSensor = 0;
unsigned long lastProcess = 0;
unsigned long lastLCD = 0;

const unsigned long SENSOR_MS  = 3000;
const unsigned long PROCESS_MS = 500;
const unsigned long LCD_MS     = 500;

// -------------------- Sensor data --------------------
float curT = NAN;
float curH = NAN;
bool haveValidReading = false;

// -------------------- Button state machine --------------------
struct BtnState {
  bool prev;
  bool held;
  unsigned long start;
  unsigned long lastRpt;
};

BtnState stTup = {false,false,0,0};
BtnState stTdn = {false,false,0,0};
BtnState stHup = {false,false,0,0};
BtnState stHdn = {false,false,0,0};

const unsigned long LONG_PRESS_MS = 1000;
const unsigned long REPEAT_MS     = 500;

bool handleButton(BtnState &st, bool pressed, unsigned long now) {
  bool act = false;

  if (pressed && !st.prev) {
    st.start = now;
    st.lastRpt = now;
    st.held = false;
  }
  else if (pressed && st.prev) {
    if (!st.held && (now - st.start >= LONG_PRESS_MS)) {
      st.held = true;
      act = true;
      st.lastRpt = now;
    }
    else if (st.held && (now - st.lastRpt >= REPEAT_MS)) {
      act = true;
      st.lastRpt += REPEAT_MS;
    }
  }
  else if (!pressed && st.prev) {
    if (!st.held && (now - st.start < LONG_PRESS_MS)) {
      act = true;
    }
    st.held = false;
  }

  st.prev = pressed;
  return act;
}

void clamp(int &v, int mn, int mx) {
  if (v < mn) v = mn;
  if (v > mx) v = mx;
}

void saveIfChanged(int addr, int oldv, int newv, int offset = 0) {
  int storedOld = oldv + offset;
  int storedNew = newv + offset;
  if (storedOld != storedNew) EEPROM.write(addr, (byte)storedNew);
}

// -------------------- LCD update (ALWAYS REFRESHES CORRECTLY) --------------------
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

  bTup.begin();
  bTdn.begin();
  bHup.begin();
  bHdn.begin();

  dht.begin();
  delay(3000);

  byte vT = EEPROM.read(EEPROM_T_ADDR);
  byte vH = EEPROM.read(EEPROM_H_ADDR);

  T_set = (int)vT - T_OFFSET;
  H_set = vH;

  clamp(T_set, T_MIN, T_MAX);
  clamp(H_set, H_MIN, H_MAX);

  updateLCD();
}

// -------------------- Loop --------------------
void loop() {
  unsigned long now = millis();

  // -------------------- BUTTONS --------------------
  bTup.read();
  bTdn.read();
  bHup.read();
  bHdn.read();

  int oldT = T_set;
  int oldH = H_set;

  if (handleButton(stTup, bTup.isPressed(), now)) T_set++;
  if (handleButton(stTdn, bTdn.isPressed(), now)) T_set--;
  if (handleButton(stHup, bHup.isPressed(), now)) H_set++;
  if (handleButton(stHdn, bHdn.isPressed(), now)) H_set--;

  clamp(T_set, T_MIN, T_MAX);
  clamp(H_set, H_MIN, H_MAX);

  saveIfChanged(EEPROM_T_ADDR, oldT, T_set, T_OFFSET);
  saveIfChanged(EEPROM_H_ADDR, oldH, H_set);

  // -------------------- SENSOR READ --------------------
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

  // -------------------- PROCESSING --------------------
  if (now - lastProcess >= PROCESS_MS) {
    bool relay = false;

    if (haveValidReading) {
      relay = (curT > T_set) || (curH > H_set);
    }

    digitalWrite(RELAY1, relay ? LOW : HIGH);
    digitalWrite(RELAY2, relay ? LOW : HIGH);

    lastProcess = now;
  }

  // -------------------- LCD UPDATE --------------------
  if (now - lastLCD >= LCD_MS) {
    updateLCD();
    lastLCD = now;
  }
}
