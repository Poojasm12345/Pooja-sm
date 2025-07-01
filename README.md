// LED pin assignments
int green[] = {2, 5, 8, 11};
int yellow[] = {3, 6, 9, 12};
int red[] = {4, 7, 10, 13};

// IR sensor pins
int irPins[] = {A0, A1, A2, A3};

// Mode control
int mode = 0; // 0 = auto, 1–4 = manual road

void setup() {
  Serial.begin(9600); // Bluetooth communication

  for (int i = 0; i < 4; i++) {
    pinMode(green[i], OUTPUT);
    pinMode(yellow[i], OUTPUT);
    pinMode(red[i], OUTPUT);
    pinMode(irPins[i], INPUT);
    digitalWrite(red[i], HIGH); // Default all red
  }
}

void loop() {
  // Read Bluetooth command
  if (Serial.available()) {
    char command = Serial.read();

    if (command >= '0' && command <= '4') {
      Serial.println(command);
      mode = command - '0';  // Convert char to int
    }
  }

  if (mode >= 1 && mode <= 4) {
    // Manual mode: give green to specified road
    giveGreenTo(mode - 1, 2000);
  } else {
    // Auto mode: check IR sensors first
    int vehicleDetected = -1;
    for (int i = 0; i < 4; i++) {
      if (digitalRead(irPins[i]) == LOW) {
        vehicleDetected = i;
        break;
      }
    }

    if (vehicleDetected != -1) {
      giveGreenTo(vehicleDetected, 2000);
    } else {
      // Normal sequence
      for (int i = 0; i < 4; i++) {
        giveGreenTo(i, 2000);
      }
    }
  }
}

// Function to control LEDs
void giveGreenTo(int roadIndex, int greenDuration) {
  // All RED initially
  for (int i = 0; i < 4; i++) {
    digitalWrite(red[i], HIGH);
    digitalWrite(green[i], LOW);
    digitalWrite(yellow[i], LOW);
  }

  digitalWrite(red[roadIndex], LOW);
  digitalWrite(green[roadIndex], HIGH);
  delay(greenDuration);

  digitalWrite(green[roadIndex], LOW);
  digitalWrite(yellow[roadIndex], HIGH);
  delay(1000);

  digitalWrite(yellow[roadIndex], LOW);
  digitalWrite(red[roadIndex], HIGH);
}
