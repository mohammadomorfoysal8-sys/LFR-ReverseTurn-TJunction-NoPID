# LFR-ReverseTurn-TJunction-NoPID
A 6-sensor Arduino line follower robot using rule-based logic (No PID), optimized for low-speed accuracy.  Includes T-junction detection and reverse differential turning for reliable left/right navigation. 

// -------------------------------------
// T-SHAPE LEFT/RIGHT TURN ENABLED
// PERFECT FOR LOW SPEED (40–70)
// NO PID
// -------------------------------------

int sensors[6] = {A0, A1, A2, A3, A4, A5};
int thresholdVal[6] = {552, 583, 620, 555, 529, 665};

#define PWMA 9
#define AIN1 5
#define AIN2 6
#define PWMB 10
#define BIN1 7
#define BIN2 8
#define STBY 11

int baseSpeed = 40;
int turnSpeed = 70;
int sharpTurn = 75;

void setup() {
  pinMode(PWMA, OUTPUT);
  pinMode(AIN1, OUTPUT);
  pinMode(AIN2, OUTPUT);

  pinMode(PWMB, OUTPUT);
  pinMode(BIN1, OUTPUT);
  pinMode(BIN2, OUTPUT);

  pinMode(STBY, OUTPUT);
  digitalWrite(STBY, HIGH);
}

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

void loop() {

  int s[6];
  for (int i = 0; i < 6; i++) {
    int v = analogRead(sensors[i]);
    s[i] = (v > thresholdVal[i]) ? 1 : 0;
  }

  // ---------- T-SHAPE TURN (FIXED REVERSED DIRECTION) ----------
if ((s[0] && s[1] && s[2] && s[3]) ||    
    (s[2] && s[3] && s[4] && s[5]) ||
    (s[0] && s[1] && s[2] && s[3] && s[4] && s[5])) {

    stopMotors();
    delay(200);

    // LEFT side detected → RIGHT turn (Corrected)
    if (s[0] == 1 || s[1] == 1) {
        leftMotor(sharpTurn);
        rightMotor(-sharpTurn/2);
        delay(350);
        return;
    }

    // RIGHT side detected → LEFT turn (Corrected)
    if (s[4] == 1 || s[5] == 1) {
        leftMotor(-sharpTurn/2);
        rightMotor(sharpTurn);
        delay(350);
        return;
    }
}

  // ---------- NORMAL STRAIGHT ----------
  if (s[2]==1 && s[3]==1) {
    leftMotor(baseSpeed);
    rightMotor(baseSpeed);
  }

  // ---------- SLIGHT LEFT ----------
  else if (s[1]==1 && s[2]==1) {
    leftMotor(turnSpeed);
    rightMotor(baseSpeed);
  }

  // ---------- SLIGHT RIGHT ----------
  else if (s[3]==1 && s[4]==1) {
    leftMotor(baseSpeed);
    rightMotor(turnSpeed);
  }

  // ---------- SHARP LEFT ----------
  else if (s[0]==1) {
    leftMotor(sharpTurn);
    rightMotor(baseSpeed);
  }

  // ---------- SHARP RIGHT ----------
  else if (s[5]==1) {
    leftMotor(baseSpeed);
    rightMotor(sharpTurn);
  }

  // ---------- LOST LINE: STOP ----------
  else if (s[0]==0 && s[1]==0 && s[2]==0 && s[3]==0 && s[4]==0 && s[5]==0) {
    stopMotors();
  }
}
