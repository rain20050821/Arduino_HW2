# Arduino_HW2
// --- 1. 腳位定義 ---
const int buttonPin = 8;
const int RledPin = 9;
const int GledPin = 10;
const int BledPin = 11;

// --- 2. 參數設定 ---
int mood = 0;
const int maxMood = 20;

bool buttonPressed = false;

unsigned long lastTouchTime = 0;
unsigned long lastReduceTime = 0;

const unsigned long unTouchInterval = 5000; // 5 秒冷落
const unsigned long reduceInterval  = 1000; // 每秒扣 1 分

void setup() {
  pinMode(buttonPin, INPUT_PULLUP);
  pinMode(RledPin, OUTPUT);
  pinMode(GledPin, OUTPUT);
  pinMode(BledPin, OUTPUT);

  showLEDState(mood);
}

void loop() {
  unsigned long currentTime = millis();

  // ---------- 按鈕互動 ----------
  int buttonState = digitalRead(buttonPin);
  if (buttonState == LOW && !buttonPressed) {
    mood++;
    if (mood > maxMood) mood = maxMood;

    buttonPressed = true;
    lastTouchTime = currentTime;   // 更新最後互動時間
  }
  if (buttonState == HIGH) {
    buttonPressed = false;
  }

  // ---------- 冷落 → 生氣 ----------
  if (currentTime - lastTouchTime > unTouchInterval) {
    if (currentTime - lastReduceTime > reduceInterval) {
      if (mood > 0) mood--;
      lastReduceTime = currentTime;
    }
  }

  showLEDState(mood);
}

// ---------- 顏色邏輯 ----------
void showLEDState(int state) {
  int r = 0, g = 0, b = 0;

  if (state == 0) {
    // 純紅（最生氣）
    r = 255; g = 0; b = 0;
  }
  else if (state <= 10) {
    // 1~10：紅 → 黃（紅滿，綠慢慢加）
    r = 255;
    g = map(state, 1, 10, 0, 255);
    b = 0;
  }
  else {
    // 11~20：黃 → 藍（紅降、藍升，綠固定滿）
    r = map(state, 11, 20, 255, 0);
    g = 255;
    b = map(state, 11, 20, 0, 255);
  }

  analogWrite(RledPin, 255 - constrain(r, 0, 255));
analogWrite(GledPin, 255 - constrain(g, 0, 255));
analogWrite(BledPin, 255 - constrain(b, 0, 255));

}
