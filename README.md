# Acoustic-Fire-Suppression-System-
An experimental fire suppression system that uses sound waves to disrupt the flame and reduce combustion. The project is built using an Arduino Nano, speaker/subwoofer, amplifier, and buzzer, demonstrating an alternative approach to conventional fire suppression using acoustic energy.
/*
 * ============================================================
 *          ACOUSTIC FIRE SUPPRESSION SYSTEM
 * ============================================================
 *
 * Controller : ESP32
 *
 * Sensors:
 *   - Flame Sensor
 *   - IR Sensor
 *
 * Outputs:
 *   - Buzzer
 *   - Status LED
 *   - Amplifier -> Subwoofer
 *
 * Working:
 *   The flame and IR sensors continuously monitor the area.
 *   When both sensors indicate a fire condition, the ESP32:
 *
 *      1. Activates the warning LED
 *      2. Activates the buzzer
 *      3. Generates an acoustic signal
 *         through the amplifier and subwoofer
 *
 * IMPORTANT:
 *   The ESP32 GPIO must NOT be connected directly to a
 *   subwoofer. Use a suitable amplifier/driver circuit.
 *
 *   This is an experimental prototype. Actual acoustic fire
 *   suppression depends on acoustic power, frequency,
 *   distance, flame characteristics and system design.
 * ============================================================
 */


// ============================================================
// PIN DEFINITIONS
// ============================================================

// Flame sensor digital output
#define FLAME_SENSOR_PIN 34

// IR sensor digital output
#define IR_SENSOR_PIN 35

// Buzzer
#define BUZZER_PIN 25

// Status LED
#define LED_PIN 2

// Audio signal output
// Connect this to the input of an amplifier,
// NOT directly to the subwoofer.
#define AUDIO_PIN 26


// ============================================================
// AUDIO / PWM SETTINGS
// ============================================================

// ESP32 LEDC channel
#define AUDIO_CHANNEL 0

// Experimental acoustic frequency
#define FIRE_FREQUENCY 1000

// PWM resolution
#define PWM_RESOLUTION 8


// ============================================================
// SENSOR SETTINGS
// ============================================================

// Most common flame/IR modules give LOW when detection occurs.
// If your sensor works opposite, change LOW to HIGH.

#define FLAME_DETECTED LOW
#define IR_DETECTED LOW


// ============================================================
// SYSTEM SETTINGS
// ============================================================

#define SENSOR_DELAY 100


// ============================================================
// VARIABLES
// ============================================================

bool flameDetected = false;
bool irDetected = false;
bool fireDetected = false;


// ============================================================
// SETUP
// ============================================================

void setup()
{
  // Start Serial Monitor
  Serial.begin(115200);

  // ----------------------------------------------------------
  // SENSOR PINS
  // ----------------------------------------------------------

  pinMode(FLAME_SENSOR_PIN, INPUT);
  pinMode(IR_SENSOR_PIN, INPUT);


  // ----------------------------------------------------------
  // OUTPUT PINS
  // ----------------------------------------------------------

  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(LED_PIN, OUTPUT);


  // Initially turn OFF outputs
  digitalWrite(BUZZER_PIN, LOW);
  digitalWrite(LED_PIN, LOW);


  // ----------------------------------------------------------
  // CONFIGURE ESP32 AUDIO PWM
  // ----------------------------------------------------------

  ledcSetup(
    AUDIO_CHANNEL,
    FIRE_FREQUENCY,
    PWM_RESOLUTION
  );

  ledcAttachPin(
    AUDIO_PIN,
    AUDIO_CHANNEL
  );


  // Make sure audio is OFF initially
  ledcWriteTone(
    AUDIO_CHANNEL,
    0
  );


  // ----------------------------------------------------------
  // STARTUP MESSAGE
  // ----------------------------------------------------------

  Serial.println();
  Serial.println("======================================");
  Serial.println("   ACOUSTIC FIRE SUPPRESSION SYSTEM");
  Serial.println("======================================");

  Serial.println("Controller : ESP32");
  Serial.println("Flame Sensor : Connected");
  Serial.println("IR Sensor    : Connected");
  Serial.println("Audio Output : Connected");

  Serial.println("--------------------------------------");
  Serial.println("System Initializing...");

  delay(2000);

  Serial.println("System Ready");
  Serial.println("Monitoring for fire...");
  Serial.println("--------------------------------------");
}


// ============================================================
// MAIN LOOP
// ============================================================

void loop()
{
  // ----------------------------------------------------------
  // READ FLAME SENSOR
  // ----------------------------------------------------------

  int flameSensorValue =
    digitalRead(FLAME_SENSOR_PIN);


  // ----------------------------------------------------------
  // READ IR SENSOR
  // ----------------------------------------------------------

  int irSensorValue =
    digitalRead(IR_SENSOR_PIN);


  // ----------------------------------------------------------
  // DETERMINE SENSOR STATUS
  // ----------------------------------------------------------

  flameDetected =
    (flameSensorValue == FLAME_DETECTED);

  irDetected =
    (irSensorValue == IR_DETECTED);


  // ----------------------------------------------------------
  // FIRE CONFIRMATION
  // ----------------------------------------------------------

  /*
   * Fire is confirmed only when both sensors
   * detect the condition.
   *
   * This helps reduce false triggering from
   * a single sensor.
   */

  if (flameDetected && irDetected)
  {
    fireDetected = true;
  }
  else
  {
    fireDetected = false;
  }


  // ----------------------------------------------------------
  // DISPLAY SENSOR STATUS
  // ----------------------------------------------------------

  Serial.print("Flame: ");

  if (flameDetected)
  {
    Serial.print("DETECTED");
  }
  else
  {
    Serial.print("NORMAL");
  }


  Serial.print(" | IR: ");

  if (irDetected)
  {
    Serial.print("DETECTED");
  }
  else
  {
    Serial.print("NORMAL");
  }


  // ----------------------------------------------------------
  // FIRE DETECTED
  // ----------------------------------------------------------

  if (fireDetected)
  {
    fireSuppressionMode();
  }


  // ----------------------------------------------------------
  // NO FIRE
  // ----------------------------------------------------------

  else
  {
    normalMonitoringMode();
  }


  delay(SENSOR_DELAY);
}


// ============================================================
// FIRE SUPPRESSION MODE
// ============================================================

void fireSuppressionMode()
{
  Serial.println(" | FIRE DETECTED!");
  Serial.println("Activating acoustic suppression system...");


  // ----------------------------------------------------------
  // WARNING LED
  // ----------------------------------------------------------

  digitalWrite(
    LED_PIN,
    HIGH
  );


  // ----------------------------------------------------------
  // BUZZER
  // ----------------------------------------------------------

  digitalWrite(
    BUZZER_PIN,
    HIGH
  );


  // ----------------------------------------------------------
  // ACOUSTIC OUTPUT
  // ----------------------------------------------------------

  /*
   * Generate the experimental acoustic signal.
   *
   * AUDIO_PIN -> Amplifier -> Subwoofer
   */

  ledcWriteTone(
    AUDIO_CHANNEL,
    FIRE_FREQUENCY
  );
}


// ============================================================
// NORMAL MONITORING MODE
// ============================================================

void normalMonitoringMode()
{
  // ----------------------------------------------------------
  // LED OFF
  // ----------------------------------------------------------

  digitalWrite(
    LED_PIN,
    LOW
  );


  // ----------------------------------------------------------
  // BUZZER OFF
  // ----------------------------------------------------------

  digitalWrite(
    BUZZER_PIN,
    LOW
  );


  // ----------------------------------------------------------
  // AUDIO OFF
  // ----------------------------------------------------------

  ledcWriteTone(
    AUDIO_CHANNEL,
    0
  );
}
