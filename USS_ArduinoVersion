// define pin numbers
const int trigPin = 13; // P13 -> PB5 (Also LED "L")
const int echoPin = 3;  // P13 -> PD3
const int d3LedPin = 11; // PB3 (OC2A) -> "D3" LED

// define variables
const int d1 = 12;
const int d2 = 49;

long duration;
int distance;

// target brightness: 100% at ≤D1, 5% at ≥D2
const uint8_t PWM_100 = 255;
const uint8_t PWM_5   = (uint8_t)((5UL * 255UL + 50) / 100UL); // ~ 13
uint8_t duty;

// blink "L" with T=0.75 s => toggle every 375 ms when outside the window
const unsigned blinkHalfMs = 375;
unsigned long nextBlinkMs = 0;
bool blinkState = false;


void setup() {

  pinMode(trigPin, OUTPUT); // Sets the trigPin as an Output
  pinMode(echoPin, INPUT); // Sets the echoPin as an Input
  pinMode(d3LedPin, OUTPUT); // Sets the D3 LED as an Output
  Serial.begin(9600); // Starts the serial communication

}


void loop() {

  // Clears the trigPin
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  analogWrite(d3LedPin, 255); // keep D3 off at start of cycle (active-LOW)

  // Sets the trigPin on HIGH state for 10 micro seconds to generate the ultrasound wave
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  // Reads the echoPin, returns the sound wave travel time in microseconds (time until Echo pin receives the reflected soundwave from Trig pin)
  duration = pulseIn(echoPin, HIGH, 30000UL);

  // Calculating the distance
  if (duration == 0)
    distance = -1;
  else
    distance = (int)(duration * 0.034 / 2); // speed of sound = 0.034 cm/microsec

  if (distance <= d1 && distance >= 0)
      duty = PWM_100;
  else if (distance >= d2) 
      duty = PWM_5;
  else if (distance > d1 && distance < d2)
      duty = (uint8_t)(255 - 6.54054054f * (distance - 12) + 0.5f); // equation calculated through linear interpolation: duty 255 at 12 cm, duty 12 at 49 cm

  // The board’s LEDs are active-LOW, so invert for analogWrite.
  analogWrite(d3LedPin, 255 - duty);

  uint8_t brightness = (uint8_t)(((unsigned)duty * 100U + 127U) / 255U);

  // Blink LED "L" (on PB5) with T=0.75 s when outside [D1;D2]
  bool outside = (distance < 0) || (distance < d1) || (distance > d2);
  if (!outside) {
    // keep line LOW so TRIG stays idle and LED L is OFF
    digitalWrite(trigPin, LOW);
  } 
  else {
    unsigned long now = millis();
    if (now >= nextBlinkMs) {
      blinkState = !blinkState;
      // blink by driving PB5 high/low; high will also keep TRIG high during this half-period
      digitalWrite(trigPin, blinkState ? HIGH : LOW);
      nextBlinkMs = now + blinkHalfMs;
    }
  }

  // Prints the echo time, distance, and pwm on the Serial Monitor
  Serial.print("Echo Time: ");
  Serial.print(duration);
  Serial.print(", Distance: ");
  Serial.print(distance);
  Serial.print(", Brightness: ");
  Serial.print(brightness);
  Serial.print("%, PWM=");
  Serial.println(duty);

  delay(400);
}

