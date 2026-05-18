---
layout: default
title: "Capítulo 7 — Motores y Servos"
nav_order: 8
---

# Capítulo 7 — Motores y Servos
{: .no_toc }

Aprende a controlar servos y motores DC para crear proyectos con movimiento.
{: .fs-6 .fw-300 }

## Contenido
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 7.1 Tipos de motores

| Tipo | Descripción | Control |
|------|-------------|---------|
| **Servo** | Gira a un ángulo preciso (0°-180°) | Señal PWM especial |
| **Motor DC** | Gira continuamente | Dirección + velocidad (PWM) |
| **Motor paso a paso** | Giros precisos por pasos | Secuencias de pulsos |
| **Motor brushless** | Alta eficiencia, alta velocidad | ESC |

---

## 7.2 El servo SG90

El servo **SG90** es el más común para principiantes. Puede girar entre **0° y 180°** con gran precisión.

### Conexión

```
Servo SG90
  Cable marrón/negro ─── GND
  Cable rojo          ─── 5V
  Cable naranja/amarillo ── Pin digital Arduino (señal)
```

> **Nunca** alimentes más de 1-2 servos directamente desde el Arduino. Para muchos servos, usa una fuente externa de 5V.

### Cómo funciona

El servo responde a pulsos de entre **1ms y 2ms** cada 20ms:
- 1ms → 0°
- 1.5ms → 90° (centro)
- 2ms → 180°

La librería `Servo.h` te abstrae de estos detalles.

---

## 7.3 Ejercicio 1 — Servo básico (barrido automático)

### Código

```cpp
// Barrido automático de servo
#include <Servo.h>

Servo miServo;          // Crea el objeto servo
const int PIN_SERVO = 9;

void setup() {
  miServo.attach(PIN_SERVO); // Conecta el servo al pin
}

void loop() {
  // Barre de 0° a 180°
  for (int angulo = 0; angulo <= 180; angulo++) {
    miServo.write(angulo);
    delay(15); // 15ms por grado → ~2.7 segundos en total
  }

  // Barre de 180° a 0°
  for (int angulo = 180; angulo >= 0; angulo--) {
    miServo.write(angulo);
    delay(15);
  }
}
```

**Funciones de la librería Servo:**

```cpp
Servo s;
s.attach(pin);           // Asocia el servo al pin
s.attach(pin, min, max); // Con pulsos personalizados (µs)
s.write(angulo);         // Mueve al ángulo (0-180)
s.read();                // Devuelve el ángulo actual
s.detach();              // Libera el pin
```

---

## 7.4 Ejercicio 2 — Servo controlado por potenciómetro

### Materiales
- 1 servo SG90
- 1 potenciómetro 10kΩ
- Protoboard y cables

### Diagrama de conexión

```
Arduino
  A0   ─── Potenciómetro (señal)
  Pin 9─── Servo (señal)
  5V   ─── Servo (rojo) + Potenciómetro (extremo)
  GND  ─── Servo (marrón) + Potenciómetro (extremo)
```

### Código

```cpp
// Servo controlado por potenciómetro
#include <Servo.h>

Servo miServo;
const int PIN_POT   = A0;
const int PIN_SERVO = 9;

void setup() {
  miServo.attach(PIN_SERVO);
  Serial.begin(9600);
}

void loop() {
  int lectura = analogRead(PIN_POT);
  int angulo  = map(lectura, 0, 1023, 0, 180);

  miServo.write(angulo);

  Serial.print("Potenciómetro: ");
  Serial.print(lectura);
  Serial.print("  → Ángulo: ");
  Serial.print(angulo);
  Serial.println("°");

  delay(20);
}
```

---

## 7.5 Ejercicio 3 — Servo controlado por Serial

```cpp
// Controla el servo enviando el ángulo por Serial (0-180)
#include <Servo.h>

Servo miServo;

void setup() {
  miServo.attach(9);
  Serial.begin(9600);
  Serial.println("Envía un ángulo (0-180):");
}

void loop() {
  if (Serial.available() > 0) {
    int angulo = Serial.parseInt();

    // Valida el rango
    angulo = constrain(angulo, 0, 180);

    miServo.write(angulo);

    Serial.print("Moviendo a: ");
    Serial.print(angulo);
    Serial.println("°");
  }
}
```

---

## 7.6 Motor DC con el puente H L298N

Un motor DC necesita más corriente de la que puede dar el Arduino (máximo 40mA por pin). Usamos un **driver L298N** que actúa como puente H para controlar:
- **Dirección**: adelante o atrás
- **Velocidad**: PWM

### Descripción del L298N

```
L298N
  ENA  ─── Pin PWM del Arduino (velocidad motor A)
  IN1  ─── Pin digital (dirección)
  IN2  ─── Pin digital (dirección)
  OUTA ─── Terminal del motor
  OUTB ─── Terminal del motor
  12V  ─── Fuente externa (7-12V para el motor)
  GND  ─── GND común con Arduino
  5V   ─── Puede alimentar el Arduino (si jumper activo)
```

### Tabla de control de dirección

| IN1 | IN2 | Motor |
|-----|-----|-------|
| HIGH | LOW | Adelante |
| LOW | HIGH | Atrás |
| LOW | LOW | Freno (libre) |
| HIGH | HIGH | Freno (fijo) |

---

## 7.7 Ejercicio 4 — Motor DC adelante y atrás

### Materiales
- 1 motor DC (6-12V)
- 1 módulo L298N
- Fuente de alimentación para el motor (p.ej. 4x pilas AA = 6V)
- Protoboard y cables

### Diagrama de conexión

```
Arduino         L298N
  Pin 5 (PWM) ─── ENA
  Pin 6       ─── IN1
  Pin 7       ─── IN2
  GND         ─── GND
```

### Código

```cpp
// Control de motor DC con L298N
const int ENA = 5; // PWM - velocidad
const int IN1 = 6;
const int IN2 = 7;

// Mueve el motor hacia adelante a una velocidad (0-255)
void adelante(int velocidad) {
  velocidad = constrain(velocidad, 0, 255);
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, velocidad);
}

// Mueve el motor hacia atrás a una velocidad (0-255)
void atras(int velocidad) {
  velocidad = constrain(velocidad, 0, 255);
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  analogWrite(ENA, velocidad);
}

// Detiene el motor
void detener() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, 0);
}

void setup() {
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  Serial.begin(9600);
  Serial.println("Motor DC listo");
}

void loop() {
  Serial.println("→ Adelante (velocidad media)");
  adelante(150);
  delay(3000);

  Serial.println("■ Deteniendo...");
  detener();
  delay(1000);

  Serial.println("← Atrás (velocidad alta)");
  atras(255);
  delay(3000);

  Serial.println("■ Deteniendo...");
  detener();
  delay(1000);

  // Aceleración gradual
  Serial.println("↑ Acelerando...");
  for (int v = 0; v <= 255; v += 5) {
    adelante(v);
    delay(50);
  }
  delay(2000);

  Serial.println("↓ Frenando...");
  for (int v = 255; v >= 0; v -= 5) {
    adelante(v);
    delay(50);
  }
  detener();
  delay(1000);
}
```

---

## 7.8 Ejercicio 5 — Brazo robótico con 2 servos

Un brazo simple con dos servos controlado por dos potenciómetros.

### Materiales
- 2 servos SG90
- 2 potenciómetros 10kΩ

### Código

```cpp
// Brazo robótico: 2 servos controlados por 2 potenciómetros
#include <Servo.h>

Servo servoBase;    // Rotación horizontal
Servo servoHombro;  // Elevación

const int POT_BASE    = A0;
const int POT_HOMBRO  = A1;
const int PIN_BASE    = 9;
const int PIN_HOMBRO  = 10;

void setup() {
  servoBase.attach(PIN_BASE);
  servoHombro.attach(PIN_HOMBRO);
  Serial.begin(9600);
}

void loop() {
  int anguloBase    = map(analogRead(POT_BASE),   0, 1023, 0, 180);
  int anguloHombro  = map(analogRead(POT_HOMBRO), 0, 1023, 0, 90);

  servoBase.write(anguloBase);
  servoHombro.write(anguloHombro);

  Serial.print("Base: ");
  Serial.print(anguloBase);
  Serial.print("°  Hombro: ");
  Serial.print(anguloHombro);
  Serial.println("°");

  delay(20);
}
```

---

## Resumen del capítulo

| Concepto | Lo que aprendiste |
|----------|------------------|
| Servo | Giro de 0° a 180° con `Servo.h` |
| `miServo.write(angulo)` | Mueve el servo al ángulo indicado |
| Motor DC | Necesita driver (L298N) para corriente y dirección |
| L298N | Puente H que controla dirección y velocidad por PWM |

---

## Desafío

Agrega un botón al brazo del Ejercicio 5 para **guardar una posición** (anota los ángulos) y un segundo botón para **reproducirla** automáticamente. Usa un array para guardar hasta 5 posiciones.

---

[← Capítulo 6](capitulo-06.html){: .btn } [Capítulo 8 — Pantalla LCD →](capitulo-08.html){: .btn .btn-primary }
