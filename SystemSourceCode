#include <Keypad.h>
#include <Servo.h>

const byte ROWS = 4;
const byte COLS = 3;

char keys[ROWS][COLS] = {
  {'1','2','3'},
  {'4','5','6'},
  {'7','8','9'},
  {'*','0','#'}
};

byte rowPins[ROWS] = {2, 3, 4, 5};  // Adjust to your wiring
byte colPins[COLS] = {6, 7, 8};

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

Servo myServo;
const int servoPin = 9;

// RGB Pins
const int redPin = 10;
const int greenPin = 11;
const int bluePin = 12;

// Buzzer
const int buzzerPin = 13;

String inputCode = "";
String correctCode = "1234";
bool isUnlocked = false;
bool isChangingCode = false;
String newCode = "";

int wrongAttempts = 0;
unsigned long lockoutStart = 0;
bool isLockedOut = false;
const unsigned long lockoutDuration = 3 * 60 * 1000UL;

void setup() {
  Serial.begin(9600);
  myServo.attach(servoPin);
  myServo.write(0); // Locked

  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);

  setLockedLED();  // Start locked
  Serial.println("🔒 System started LOCKED.");
}

void loop() {
  if (isLockedOut && (millis() - lockoutStart >= lockoutDuration)) {
    isLockedOut = false;
    wrongAttempts = 0;
    Serial.println("🔓 Lockout ended.");
    setLockedLED();
  }

  if (isLockedOut) return;

  char key = keypad.getKey();

  if (key) {
    // Short beep for each key press
    tone(buzzerPin, 2000, 100);
    delay(100);

    Serial.print("Pressed: ");
    Serial.println(key);

    if (isChangingCode) {
      if (key == '#') {
        if (newCode.length() > 0) {
          correctCode = newCode;
          Serial.println("✅ Password changed.");
          playSuccessTone();
        } else {
          Serial.println("❌ Empty password.");
        }
        newCode = "";
        isChangingCode = false;
      } else if (key == '*') {
        newCode = "";
        playDeleteTone();
        Serial.println("🔁 New password cleared.");
      } else {
        newCode += key;
      }
      return;
    }

    if (key == '#') {
      if (!isUnlocked && inputCode == correctCode) {
        Serial.println("✅ Correct code. Unlocked.");
        myServo.write(90);
        isUnlocked = true;
        wrongAttempts = 0;
        setUnlockedLED();
        playSuccessTone();
      } else if (isUnlocked) {
        Serial.println("🔒 Locked again.");
        myServo.write(0);
        isUnlocked = false;
        setLockedLED();
        playLockTone();
      } else {
        wrongAttempts++;
        Serial.println("❌ Wrong code.");
        playErrorTone();

        if (wrongAttempts >= 3) {
          isLockedOut = true;
          lockoutStart = millis();
          Serial.println("🚫 Too many attempts. Locked for 3 minutes.");
          setWaitingLED();
          playLockoutTone();
        }
      }
      inputCode = "";
    } 
    else if (key == '*') {
      playDeleteTone(); // Long tone for delete

      if (isUnlocked) {
        Serial.println("🔧 Enter new password, then press # to save.");
        isChangingCode = true;
        newCode = "";
      } else {
        inputCode = "";
        Serial.println("🔁 Input cleared.");
      }
    } 
    else {
      inputCode += key;
    }
  }
}

// === LED Functions ===
void setLockedLED() {
  digitalWrite(redPin, LOW);
  digitalWrite(greenPin, LOW);
  digitalWrite(bluePin, HIGH);  // 🔵 Locked
}

void setUnlockedLED() {
  digitalWrite(redPin, LOW);
  digitalWrite(greenPin, HIGH); // 🟢 Unlocked
  digitalWrite(bluePin, LOW);
}

void setWaitingLED() {
  digitalWrite(redPin, HIGH);   // 🔴 Waiting
  digitalWrite(greenPin, LOW);
  digitalWrite(bluePin, LOW);
}

// === Tone Functions ===
void playSuccessTone() {
  tone(buzzerPin, 1000, 150);
  delay(200);
  tone(buzzerPin, 1500, 150);
  delay(200);
  noTone(buzzerPin);
}

void playLockTone() {
  tone(buzzerPin, 800, 200);
  delay(200);
  noTone(buzzerPin);
}

void playErrorTone() {
  tone(buzzerPin, 400, 300);
  delay(300);
  noTone(buzzerPin);
}

void playLockoutTone() {
  for (int i = 0; i < 3; i++) {
    tone(buzzerPin, 300, 200);
    delay(300);
  }
  noTone(buzzerPin);
}

void playDeleteTone() {
  tone(buzzerPin, 1000, 400); // Longer tone for delete
  delay(400);
  noTone(buzzerPin);
}
