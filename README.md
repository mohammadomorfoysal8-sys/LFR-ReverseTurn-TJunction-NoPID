# LFR-ReverseTurn-TJunction-NoPID
A 6-sensor Arduino line follower robot using rule-based logic (No PID), optimized for low-speed accuracy.  Includes T-junction detection and reverse differential turning for reliable left/right navigation. 
// -------------------------------------
// OPTIMIZED LINE FOLLOWER (NO PID)
// PERFECT FOR CURVE / CIRCLE / 90° TURN
// T-JUNCTION SUPPORTED
// (Poster track: BLACK gives HIGH reading)
// -------------------------------------

int sensors[6] = {A0, A1, A2, A3, A4, A5};

// ✅ New thresholds for your DIGITAL PRINT poster track
// (Calculated from your white+black readings)
int thresholdVal[6] = {376, 462, 436, 444, 386, 513};

#define PWMA 9
#define AIN1 5
#define AIN2 6
#define PWMB 10
#define BIN1 7
#define BIN2 8
#define STBY 11

int baseSpeed = 40;
int turnSpeed = 65;
int sharpTurn = 100;   // boosted for curve & circle

void setup() {
  pinMode(PWMA, OUTPUT);
  pinMode(AIN1, OUTPUT);
  pinMode(AIN2, OUTPUT);

  pinMode(PWMB, OUTPUT);
  pinMode(BIN1, OUTPUT);
  pinMode(BIN2, OUTPUT);

  pinMode(STBY, OUTPUT);
  digitalWrite(STBY, HIGH);

  Serial.begin(9600);
}

// ------------------ MOTOR CONTROL ------------------

void leftMotor(int s) {
  if (s >= 0) { digitalWrite(AIN1, LOW); digitalWrite(AIN2, HIGH); }
  else        { digitalWrite(AIN1, HIGH); digitalWrite(AIN2, LOW); s = -s; }
  analogWrite(PWMA, s);
}

void rightMotor(int s) {
  if (s >= 0) { digitalWrite(BIN1, HIGH); digitalWrite(BIN2, LOW); }
  else        { digitalWrite(BIN1, LOW); digitalWrite(BIN2, HIGH); s = -s; }
  analogWrite(PWMB, s);
}

void stopMotors() {
  leftMotor(0);
  rightMotor(0);
}

// ------------------ MAIN LOOP ------------------

void loop() {

  int s[6];
  for (int i = 0; i < 6; i++) {
    int v = analogRead(sensors[i]);

    // ✅ Binary rule for your sensor readings:
    // BLACK line = HIGH value => 1
    s[i] = (v > thresholdVal[i]) ? 1 : 0;
  }

  // -------------------------------------
  // T-JUNCTION HANDLING
  // -------------------------------------
  if ((s[0] && s[1] && s[2] && s[3]) ||
      (s[2] && s[3] && s[4] && s[5]) ||
      (s[0] && s[1] && s[2] && s[3] && s[4] && s[5])) {

    stopMotors();
    delay(200);

    // LEFT side detected → turn RIGHT
    if (s[0] || s[1]) {
      leftMotor(sharpTurn);
      rightMotor(-sharpTurn / 2);
      delay(400);
      return;
    }

    // RIGHT side detected → turn LEFT
    if (s[4] || s[5]) {
      leftMotor(-sharpTurn / 2);
      rightMotor(sharpTurn);
      delay(400);
      return;
    }
  }

  // -------------------------------------
  // CENTER (STRAIGHT)
  // -------------------------------------
  if (s[2] && s[3]) {
    leftMotor(baseSpeed);
    rightMotor(baseSpeed);
  }

  // -------------------------------------
  // SLIGHT LEFT (1 SENSOR)
  // -------------------------------------
  else if (s[1]) {
    leftMotor(turnSpeed);
    rightMotor(baseSpeed);
  }

  // -------------------------------------
  // SLIGHT RIGHT (1 SENSOR)
  // -------------------------------------
  else if (s[4]) {
    leftMotor(baseSpeed);
    rightMotor(turnSpeed);
  }

  // -------------------------------------
  // SHARP LEFT (EXTREME SENSOR)
  // -------------------------------------
  else if (s[0]) {
    leftMotor(sharpTurn);
    rightMotor(baseSpeed - 10);
  }

  // -------------------------------------
  // SHARP RIGHT (EXTREME SENSOR)
  // -------------------------------------
  else if (s[5]) {
    leftMotor(baseSpeed - 10);
    rightMotor(sharpTurn);
  }

  // -------------------------------------
  // LOST LINE → AUTO RECOVERY
  // -------------------------------------
  else if (!s[0] && !s[1] && !s[2] && !s[3] && !s[4] && !s[5]) {
    leftMotor(-40);
    rightMotor(40);
    delay(120);
  }
}

