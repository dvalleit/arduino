---
layout: default
title: "Capítulo 9 — Sensores Avanzados"
nav_order: 10
---

# Capítulo 9 — Sensores Avanzados
{: .no_toc }

Aprende a usar el sensor de distancia HC-SR04 y el sensor de temperatura/humedad DHT11, dos de los más populares en proyectos Arduino.
{: .fs-6 .fw-300 }

## Contenido
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 9.1 El sensor ultrasónico HC-SR04

El **HC-SR04** mide distancias entre **2 cm y 400 cm** usando ultrasonido. Funciona como un sonar:

1. Envía un pulso ultrasónico de 40kHz por el pin **TRIG**.
2. El pulso rebota en el obstáculo y regresa.
3. El pin **ECHO** se pone en HIGH durante el tiempo que tardó el eco en regresar.
4. Con ese tiempo calculamos la distancia.

$$\text{Distancia (cm)} = \frac{\text{Tiempo de eco (µs)}}{58.2}$$

### Conexión

```
HC-SR04       Arduino Uno
  VCC ─────── 5V
  GND ─────── GND
  TRIG ─────── Pin 10
  ECHO ─────── Pin 11
```

---

## 9.2 Ejercicio 1 — Medidor de distancia

```cpp
// Medidor de distancia con HC-SR04
const int TRIG = 10;
const int ECHO = 11;

long leerDistancia() {
  // Genera el pulso de disparo (mínimo 10µs)
  digitalWrite(TRIG, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG, LOW);

  // Mide el tiempo del eco en microsegundos
  long duracion = pulseIn(ECHO, HIGH, 30000); // Timeout 30ms

  if (duracion == 0) return -1; // Sin eco → fuera de rango

  // Convierte a centímetros
  return duracion / 58.2;
}

void setup() {
  pinMode(TRIG, OUTPUT);
  pinMode(ECHO, INPUT);
  Serial.begin(9600);
  Serial.println("Medidor de distancia HC-SR04");
}

void loop() {
  long distancia = leerDistancia();

  if (distancia < 0) {
    Serial.println("Fuera de rango");
  } else {
    Serial.print("Distancia: ");
    Serial.print(distancia);
    Serial.println(" cm");
  }

  delay(300);
}
```

> **`pulseIn(pin, HIGH, timeout)`** mide el ancho de un pulso en microsegundos. El timeout evita que el programa se bloquee si no hay eco.

---

## 9.3 Ejercicio 2 — Radar de proximidad con LEDs

Indica la cercanía del obstáculo con 3 LEDs: verde (lejos), amarillo (cerca), rojo (muy cerca).

### Materiales
- HC-SR04
- 3 LEDs (rojo, amarillo, verde) + resistencias 220Ω
- 1 buzzer (opcional)

### Conexión

```
Arduino
  Pin 10 ─── TRIG (HC-SR04)
  Pin 11 ─── ECHO (HC-SR04)
  Pin  9 ─── R220Ω ─── LED Verde
  Pin  8 ─── R220Ω ─── LED Amarillo
  Pin  7 ─── R220Ω ─── LED Rojo
  Pin  6 ─── Buzzer (opcional)
```

### Código

```cpp
// Radar de proximidad con 3 LEDs y buzzer
const int TRIG  = 10;
const int ECHO  = 11;
const int LED_V = 9;  // Verde: lejos (> 50cm)
const int LED_A = 8;  // Amarillo: cerca (20-50cm)
const int LED_R = 7;  // Rojo: muy cerca (< 20cm)
const int BUZZER = 6;

long leerDistancia() {
  digitalWrite(TRIG, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG, LOW);
  long duracion = pulseIn(ECHO, HIGH, 30000);
  return (duracion == 0) ? -1 : duracion / 58.2;
}

void apagaTodos() {
  digitalWrite(LED_V, LOW);
  digitalWrite(LED_A, LOW);
  digitalWrite(LED_R, LOW);
  noTone(BUZZER);
}

void setup() {
  pinMode(TRIG, OUTPUT);
  pinMode(ECHO, INPUT);
  pinMode(LED_V, OUTPUT);
  pinMode(LED_A, OUTPUT);
  pinMode(LED_R, OUTPUT);
  pinMode(BUZZER, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  long dist = leerDistancia();
  apagaTodos();

  if (dist < 0) {
    Serial.println("Sin señal");
  } else if (dist > 50) {
    digitalWrite(LED_V, HIGH);           // Lejos → verde
    Serial.print("LEJOS: ");
  } else if (dist > 20) {
    digitalWrite(LED_A, HIGH);           // Cerca → amarillo
    tone(BUZZER, 500, 100);             // Pitido suave
    Serial.print("CERCA: ");
  } else {
    digitalWrite(LED_R, HIGH);           // Muy cerca → rojo
    tone(BUZZER, 1200);                 // Pitido continuo
    Serial.print("¡MUY CERCA!: ");
  }

  Serial.print(dist);
  Serial.println(" cm");
  delay(200);
}
```

---

## 9.4 El sensor DHT11 / DHT22

El **DHT11** y **DHT22** miden temperatura y humedad relativa. Usan un protocolo digital propio a través de **un solo cable de datos**.

| Sensor | Temperatura | Humedad | Precisión temp. |
|--------|-------------|---------|-----------------|
| DHT11 | 0–50°C | 20–80% | ±2°C |
| DHT22 | -40–80°C | 0–100% | ±0.5°C |

### Conexión (DHT11 y DHT22 son iguales)

```
DHT11/22       Arduino
  VCC ─────── 5V
  DATA ─────── Pin 2 (con resistencia pull-up 4.7kΩ a 5V)
  GND ─────── GND
```

> Algunos módulos ya incluyen la resistencia pull-up. Si compras el sensor solo, necesitas agregar una resistencia de **4.7kΩ** entre DATA y 5V.

### Instalar la librería DHT

1. **Herramientas → Administrar Bibliotecas...**
2. Busca `DHT sensor library`
3. Instala la de **Adafruit** (acepta instalar también `Adafruit Unified Sensor`)

---

## 9.5 Ejercicio 3 — Temperatura y humedad por Serial

```cpp
// Temperatura y humedad con DHT11
#include <DHT.h>

#define PIN_DHT 2
#define TIPO_DHT DHT11 // Cambia a DHT22 si usas ese modelo

DHT dht(PIN_DHT, TIPO_DHT);

void setup() {
  Serial.begin(9600);
  dht.begin();
  Serial.println("Sensor DHT11 listo");
  Serial.println("Tiempo(ms)\tTemp(°C)\tHumedad(%)");
}

void loop() {
  delay(2000); // El DHT11 necesita al menos 2 segundos entre lecturas

  float temperatura = dht.readTemperature(); // En Celsius
  float humedad     = dht.readHumidity();

  // Verifica si la lectura fue exitosa
  if (isnan(temperatura) || isnan(humedad)) {
    Serial.println("Error: no se pudo leer el sensor DHT");
    return;
  }

  // Calcula el índice de calor (sensación térmica)
  float indicCalor = dht.computeHeatIndex(temperatura, humedad, false);

  Serial.print(millis());
  Serial.print("\t");
  Serial.print(temperatura, 1);
  Serial.print("\t\t");
  Serial.println(humedad, 1);

  Serial.print("Sensación térmica: ");
  Serial.print(indicCalor, 1);
  Serial.println(" °C");
  Serial.println("---");
}
```

---

## 9.6 Ejercicio 4 — Estación meteorológica en LCD

Combina el DHT11 con una pantalla LCD para mostrar temperatura y humedad.

### Materiales
- Sensor DHT11 o DHT22
- LCD 16×2 con módulo I2C
- Protoboard y cables

### Código

```cpp
// Mini estación meteorológica: DHT11 + LCD I2C
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>

#define PIN_DHT  2
#define TIPO_DHT DHT11

LiquidCrystal_I2C lcd(0x27, 16, 2);
DHT dht(PIN_DHT, TIPO_DHT);

// Símbolos personalizados
byte gradoChar[8] = {0b00110,0b01001,0b01001,0b00110,0,0,0,0};
byte humedadChar[8] = {0b00100,0b01110,0b11111,0b11111,0b01110,0b00100,0,0};

void setup() {
  lcd.init();
  lcd.backlight();
  lcd.createChar(0, gradoChar);
  lcd.createChar(1, humedadChar);

  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print(" Estacion Meteo ");
  delay(2000);
  lcd.clear();
}

void loop() {
  float temp = dht.readTemperature();
  float hum  = dht.readHumidity();

  if (!isnan(temp) && !isnan(hum)) {
    // Fila 0: Temperatura
    lcd.setCursor(0, 0);
    lcd.print("Temp: ");
    lcd.print(temp, 1);
    lcd.write(0); // Símbolo de grado
    lcd.print("C  ");

    // Fila 1: Humedad
    lcd.setCursor(0, 1);
    lcd.write(1); // Símbolo de humedad
    lcd.print(" Hum: ");
    lcd.print(hum, 1);
    lcd.print("%   ");
  } else {
    lcd.setCursor(0, 0);
    lcd.print("Error sensor!   ");
  }

  delay(2000);
}
```

---

## 9.7 Ejercicio 5 — Sistema anticolisión

Detiene un motor DC cuando un obstáculo está a menos de 10 cm.

```cpp
// Sistema anticolisión: para el motor si hay obstáculo cerca
#include <Servo.h>

const int TRIG = 10;
const int ECHO = 11;
const int ENA  = 5;
const int IN1  = 6;
const int IN2  = 7;
const int LED_ROJO  = 9;
const int LED_VERDE = 8;

const int DISTANCIA_MINIMA = 10; // cm

long leerDistancia() {
  digitalWrite(TRIG, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG, LOW);
  long d = pulseIn(ECHO, HIGH, 30000);
  return (d == 0) ? 999 : d / 58.2;
}

void motorAdelante() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, 180);
  digitalWrite(LED_VERDE, HIGH);
  digitalWrite(LED_ROJO, LOW);
}

void motorDetener() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, 0);
  digitalWrite(LED_VERDE, LOW);
  digitalWrite(LED_ROJO, HIGH);
}

void setup() {
  pinMode(TRIG, OUTPUT);
  pinMode(ECHO, INPUT);
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(LED_ROJO, OUTPUT);
  pinMode(LED_VERDE, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  long dist = leerDistancia();

  Serial.print("Distancia: ");
  Serial.print(dist);
  Serial.print(" cm → ");

  if (dist < DISTANCIA_MINIMA) {
    motorDetener();
    Serial.println("¡DETENIDO!");
  } else {
    motorAdelante();
    Serial.println("Avanzando");
  }

  delay(100);
}
```

---

## Resumen del capítulo

| Sensor | Qué mide | Pines |
|--------|----------|-------|
| HC-SR04 | Distancia (2-400 cm) | TRIG + ECHO |
| DHT11 | Temperatura y humedad | 1 pin de datos |
| DHT22 | Temperatura y humedad (más preciso) | 1 pin de datos |

| Función | Descripción |
|---------|-------------|
| `pulseIn(pin, HIGH, timeout)` | Mide la duración de un pulso en µs |
| `dht.readTemperature()` | Lee temperatura en °C |
| `dht.readHumidity()` | Lee humedad relativa en % |
| `isnan(valor)` | Devuelve `true` si el valor no es un número válido |

---

## Desafío

Construye un **detector de presencia con alarma**:
- El HC-SR04 monitorea un área
- Si detecta algo a menos de 30 cm, hace sonar el buzzer y enciende el LED rojo
- Muestra "INTRUSO" en la LCD
- Si no hay nada, muestra "ZONA SEGURA" en la LCD

---

[← Capítulo 8](capitulo-08.html){: .btn } [Capítulo 10 — Proyecto Final →](capitulo-10.html){: .btn .btn-primary }
