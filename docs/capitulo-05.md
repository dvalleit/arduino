---
layout: default
title: "Capítulo 5 — PWM y Salidas Analógicas"
nav_order: 6
---

# Capítulo 5 — PWM y Salidas Analógicas
{: .no_toc }

Aprende a controlar el brillo de un LED, el color de un LED RGB y a generar señales analógicas con PWM.
{: .fs-6 .fw-300 }

## Contenido
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 5.1 ¿Qué es PWM?

**PWM** significa *Pulse Width Modulation* (Modulación por Ancho de Pulso). Es una técnica para simular una salida analógica usando señales digitales.

Un pin PWM alterna rápidamente entre HIGH (5V) y LOW (0V). El **ciclo de trabajo** (*duty cycle*) determina cuánto tiempo está en HIGH:

```
100% duty cycle (máximo):
▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀   Voltaje promedio = 5V

50% duty cycle (mitad):
▀▀▀▀▀▀▀▀▀▀            Voltaje promedio = 2.5V
          ▀▀▀▀▀▀▀▀▀▀
0% duty cycle (apagado):
                        Voltaje promedio = 0V
```

En Arduino, `analogWrite(pin, valor)` controla el duty cycle:
- `0` → 0% (siempre LOW, 0V)
- `128` → 50% (mitad del tiempo HIGH)
- `255` → 100% (siempre HIGH, 5V)

### Pines PWM en el Arduino Uno

Los pines con capacidad PWM están marcados con **~** en la placa: **3, 5, 6, 9, 10, 11**.

> La frecuencia PWM por defecto es ~490 Hz (pines 3, 9, 10, 11) o ~980 Hz (pines 5, 6). Es suficientemente rápida para que el ojo humano no perciba el parpadeo.

---

## 5.2 Ejercicio 1 — Dimmer de LED con potenciómetro

### Materiales
- 1 LED + resistencia 220Ω
- 1 potenciómetro 10kΩ
- Protoboard y cables

### Diagrama de conexión

```
Arduino
  A0   ─── Potenciómetro (patilla central)
  5V   ─── Potenciómetro (patilla izquierda)
  GND  ─── Potenciómetro (patilla derecha)
  Pin 9─── R220Ω ─── LED ─── GND
```

### Código

```cpp
// Dimmer: potenciómetro controla el brillo del LED
const int PIN_POT = A0;
const int PIN_LED = 9;

void setup() {
  // No es necesario configurar pines analógicos
  Serial.begin(9600);
}

void loop() {
  int lectura = analogRead(PIN_POT);          // 0 a 1023
  int brillo  = map(lectura, 0, 1023, 0, 255); // Escala a 0-255

  analogWrite(PIN_LED, brillo);

  Serial.print("Potenciómetro: ");
  Serial.print(lectura);
  Serial.print("  → Brillo PWM: ");
  Serial.println(brillo);

  delay(50);
}
```

Gira el potenciómetro y observa el brillo del LED cambiar suavemente de apagado a máximo.

---

## 5.3 Ejercicio 2 — LED que "respira"

El efecto "respiración" consiste en subir y bajar el brillo suavemente de forma continua (como un MacBook en reposo).

```cpp
// LED respirando con función seno
const int PIN_LED = 9;

void setup() {
  pinMode(PIN_LED, OUTPUT);
}

void loop() {
  // Usando la función seno para suavizar la transición
  for (int angulo = 0; angulo < 360; angulo++) {
    // sin() devuelve -1 a 1; lo convertimos a 0-255
    float s = sin(angulo * PI / 180.0);
    int brillo = (int)(((s + 1.0) / 2.0) * 255.0);
    analogWrite(PIN_LED, brillo);
    delay(8); // ~3 segundos por ciclo completo
  }
}
```

**Versión alternativa más simple (sin seno):**

```cpp
// Respiración con fade lineal
const int PIN_LED = 9;

void setup() {
  pinMode(PIN_LED, OUTPUT);
}

void loop() {
  // Sube de 0 a 255
  for (int b = 0; b <= 255; b++) {
    analogWrite(PIN_LED, b);
    delay(8);
  }
  // Baja de 255 a 0
  for (int b = 255; b >= 0; b--) {
    analogWrite(PIN_LED, b);
    delay(8);
  }
}
```

---

## 5.4 El LED RGB

Un **LED RGB** contiene tres LEDs en un solo encapsulado: **R**ojo (Red), **V**erde (Green) y **A**zul (Blue). Mezclando los tres colores puedes obtener cualquier color.

### Tipos
- **Ánodo común**: la pata larga se conecta a 5V; escritura invertida (0 = máximo, 255 = apagado).
- **Cátodo común**: la pata larga se conecta a GND; escritura normal (255 = máximo). ← *Más común*

### Diagrama de conexión (cátodo común)

```
LED RGB (cátodo común)
  R ─── R220Ω ─── Pin 11 (PWM)
  G ─── R220Ω ─── Pin 10 (PWM)
  B ─── R220Ω ─── Pin  9 (PWM)
  GND ─────────── GND Arduino
```

---

## 5.5 Ejercicio 3 — LED RGB con 3 potenciómetros

### Materiales
- 1 LED RGB (cátodo común)
- 3 resistencias 220Ω
- 3 potenciómetros 10kΩ
- Protoboard y cables

### Diagrama de conexión

```
Arduino
  A0 ─── Potenciómetro 1 (Rojo)
  A1 ─── Potenciómetro 2 (Verde)
  A2 ─── Potenciómetro 3 (Azul)
  Pin 11 ─── R220Ω ─── LED R
  Pin 10 ─── R220Ω ─── LED G
  Pin  9 ─── R220Ω ─── LED B
             GND ─── Cátodo LED
```

### Código

```cpp
// Mezclador de colores RGB con 3 potenciómetros
const int PIN_R = 11;
const int PIN_G = 10;
const int PIN_B = 9;

const int POT_R = A0;
const int POT_G = A1;
const int POT_B = A2;

void setup() {
  pinMode(PIN_R, OUTPUT);
  pinMode(PIN_G, OUTPUT);
  pinMode(PIN_B, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int rojo  = map(analogRead(POT_R), 0, 1023, 0, 255);
  int verde = map(analogRead(POT_G), 0, 1023, 0, 255);
  int azul  = map(analogRead(POT_B), 0, 1023, 0, 255);

  analogWrite(PIN_R, rojo);
  analogWrite(PIN_G, verde);
  analogWrite(PIN_B, azul);

  // Muestra el color en formato HEX
  Serial.print("Color: #");
  if (rojo  < 16) Serial.print("0");
  Serial.print(rojo, HEX);
  if (verde < 16) Serial.print("0");
  Serial.print(verde, HEX);
  if (azul  < 16) Serial.print("0");
  Serial.println(azul, HEX);

  delay(100);
}
```

### Mezcla de colores

| R | G | B | Color |
|---|---|---|-------|
| 255 | 0 | 0 | Rojo |
| 0 | 255 | 0 | Verde |
| 0 | 0 | 255 | Azul |
| 255 | 255 | 0 | Amarillo |
| 255 | 0 | 255 | Magenta |
| 0 | 255 | 255 | Cian |
| 255 | 255 | 255 | Blanco |
| 128 | 0 | 128 | Morado |

---

## 5.6 Ejercicio 4 — Ciclo de arco iris automático

```cpp
// Arco iris automático en LED RGB
const int PIN_R = 11;
const int PIN_G = 10;
const int PIN_B = 9;

// Función para establecer el color
void setColor(int r, int g, int b) {
  analogWrite(PIN_R, r);
  analogWrite(PIN_G, g);
  analogWrite(PIN_B, b);
}

// Interpolación lineal entre dos valores
int lerp(int inicio, int fin, float t) {
  return (int)(inicio + (fin - inicio) * t);
}

// Define los colores del arco iris
const int NUM_COLORES = 7;
const int COLORES[7][3] = {
  {255,   0,   0}, // Rojo
  {255, 128,   0}, // Naranja
  {255, 255,   0}, // Amarillo
  {  0, 255,   0}, // Verde
  {  0,   0, 255}, // Azul
  { 75,   0, 130}, // Índigo
  {148,   0, 211}  // Violeta
};

void setup() {
  pinMode(PIN_R, OUTPUT);
  pinMode(PIN_G, OUTPUT);
  pinMode(PIN_B, OUTPUT);
}

void loop() {
  for (int i = 0; i < NUM_COLORES; i++) {
    int siguiente = (i + 1) % NUM_COLORES;

    // Transición suave entre colores
    for (int paso = 0; paso <= 100; paso++) {
      float t = paso / 100.0;
      int r = lerp(COLORES[i][0], COLORES[siguiente][0], t);
      int g = lerp(COLORES[i][1], COLORES[siguiente][1], t);
      int b = lerp(COLORES[i][2], COLORES[siguiente][2], t);
      setColor(r, g, b);
      delay(20);
    }
  }
}
```

---

## Resumen del capítulo

| Concepto | Descripción |
|----------|-------------|
| PWM | Simulación de salida analógica con pulsos digitales |
| `analogWrite(pin, 0-255)` | Establece el duty cycle en un pin PWM |
| Pines PWM (Uno) | 3, 5, 6, 9, 10, 11 (marcados con ~) |
| LED RGB | Mezcla de R, G, B para obtener cualquier color |

---

## Desafío

Crea un **amanecer simulado**: un LED RGB empieza de negro (apagado), pasa por tonos naranja-rojizos y termina en blanco (luz de día). El proceso debe durar 30 segundos y ejecutarse al encender el Arduino.

---

[← Capítulo 4](capitulo-04.html){: .btn } [Capítulo 6 — Comunicación Serial →](capitulo-06.html){: .btn .btn-primary }
