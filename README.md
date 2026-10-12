# Helico-Harvester
Edge-AI Smart Micro-Energy Harvester for Rail Corridors &amp; Off-Grid Farms
/*
  ===============================================================================
  PROJECT: Helico Harvester 🚁⚡
  TRACK: Hack for Social Cause / Viksit Bharat Young Leaders Dialogue
  DEPLOYMENT: Dual-Use (Railway Corridors & Off-Grid Agricultural Telemetry)
  ===============================================================================
  Description:
  Edge-AI Micro-Energy Harvester simulating train-induced airflow & wind kinetic 
  energy harvesting with real-time turbine speed monitoring, dynamic power calculation, 
  predictive structural health scoring, and multi-screen serial telemetry output.
  ===============================================================================
*/

// --- PIN DEFINITIONS ---
const int POT_PIN = A0;       // Potentiometer simulating wind velocity / train displacement
const int MOTOR_PIN = 9;      // PWM output to NPN Transistor Base driving turbine/motor

// --- TIMING CONSTANTS (NON-BLOCKING millis) ---
const unsigned long TELEMETRY_INTERVAL = 1000;  // Update telemetry screen every 1 second
const unsigned long AI_DIAGNOSTIC_INTERVAL = 3000; // Run Edge-AI diagnostic cycle every 3 seconds

// --- GLOBAL VARIABLES ---
unsigned long previousTelemetryMillis = 0;
unsigned long previousAiMillis = 0;

int screenState = 0;          // Dynamic screen layout toggler
float estimatedPowerMw = 0.0;
int rawPotValue = 0;
int motorPwmValue = 0;
float healthScore = 100.0;

// --- FUNCTION DECLARATIONS ---
void readSensors();
void updateActuators();
void calculatePowerAndHealth();
void transmitTelemetry();

void setup() {
  // Initialize Serial Telemetry Baud Rate
  Serial.begin(9600);
  
  // Pin Configurations
  pinMode(MOTOR_PIN, OUTPUT);
  pinMode(POT_PIN, INPUT);

  // Initializing Banner
  Serial.println(F("=================================================="));
  Serial.println(F("   HELICO HARVESTER - EDGE-AI SMART POWER NODE   "));
  Serial.println(F("  Target: Railway Track Safety & Agri Telemetry  "));
  Serial.println(F("=================================================="));
  delay(1000);
}

void loop() {
  unsigned long currentMillis = millis();

  // 1. Continuous Sensor Reading & Actuation Control
  readSensors();
  updateActuators();
  calculatePowerAndHealth();

  // 2. Periodic Edge-AI Telemetry Output (Non-blocking)
  if (currentMillis - previousTelemetryMillis >= TELEMETRY_INTERVAL) {
    previousTelemetryMillis = currentMillis;
    transmitTelemetry();
  }
}

void readSensors() {
  rawPotValue = analogRead(POT_PIN); // Read simulated airflow intensity (0 - 1023)
}

void updateActuators() {
  // Map 10-bit analog input (0-1023) to 8-bit PWM output (0-255)
  motorPwmValue = map(rawPotValue, 0, 1023, 0, 255);
  analogWrite(MOTOR_PIN, motorPwmValue);
}

void calculatePowerAndHealth() {
  // Convert PWM value to simulated voltage (0V to 5.0V)
  float voltage = (motorPwmValue / 255.0) * 5.0;
  
  // Calculate simulated current (mA) & harvested micro-power (mW)
  float simulatedCurrentMa = voltage * 40.0; // Assume 40mA load at max 5V
  estimatedPowerMw = voltage * simulatedCurrentMa;

  // --- EDGE-AI PREDICTIVE HEALTH DIAGNOSTICS ---
  // Evaluates operational strain based on high rpm turbulence vs static idle strain
  if (motorPwmValue > 230) {
    // High wind turbulence degradation risk
    healthScore -= 0.15;
  } else if (motorPwmValue < 15 && healthScore > 50.0) {
    // Slight idle strain
    healthScore -= 0.02;
  } else if (motorPwmValue >= 50 && motorPwmValue <= 180 && healthScore < 99.5) {
    // Optimal operational efficiency zone recovery
    healthScore += 0.05;
  }

  // Constrain health score between 0% and 100%
  healthScore = constrain(healthScore, 0.0, 100.0);
}

void transmitTelemetry() {
  // Toggle screens for clean multi-screen serial dashboard view
  screenState = (screenState + 1) % 2;

  Serial.println(F("--------------------------------------------------"));
  
  if (screenState == 0) {
    // SCREEN 1: REAL-TIME HARVESTING METRICS
    Serial.println(F("[SYSTEM TELEMETRY: POWER GENERATION]"));
    Serial.print(F("Airflow Intensity (ADC): "));
    Serial.println(rawPotValue);
    
    Serial.print(F("Turbine PWM Drive Level: "));
    Serial.print(motorPwmValue);
    Serial.print(F(" / 255 ("));
    Serial.print((motorPwmValue / 255.0) * 100.0, 1);
    Serial.println(F("%)"));

    Serial.print(F("Estimated Generated Power: "));
    Serial.print(estimatedPowerMw, 2);
    Serial.println(F(" mW"));

  } else {
    // SCREEN 2: EDGE-AI DIAGNOSTICS & SYSTEM HEALTH
    Serial.println(F("[EDGE-AI DIAGNOSTICS & TELEMETRY]"));
    Serial.print(F("Predictive Equipment Health: "));
    Serial.print(healthScore, 1);
    Serial.println(F("%"));

    Serial.print(F("Status: "));
    if (healthScore > 80.0) {
      Serial.println(F("OPTIMAL - Normal Operation"));
    } else if (healthScore > 50.0) {
      Serial.println(F("WARNING - Maintenance Recommended"));
    } else {
      Serial.println(F("CRITICAL - High Bearing Strain Detected!"));
    }

    Serial.print(F("Target Deployment Mode: "));
    if (motorPwmValue > 150) {
      Serial.println(F("HIGH AIRFLOW (Passing Train Displacement)"));
    } else {
      Serial.println(F("AMBIENT MICRO-WIND (Off-Grid Farm Field)"));
    }
  }
  
  Serial.println(F("--------------------------------------------------"));
}
