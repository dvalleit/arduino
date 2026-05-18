---
layout: default
title: "Capítulo 4 — Señales Analógicas"
nav_order: 5
---

# Capítulo 4 — Señales Analógicas
{: .no_toc }

Aprende a leer sensores del mundo real: potenciómetros, sensores de luz y de temperatura.
{: .fs-6 .fw-300 }

## Contenido
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 4.1 El convertidor analógico-digital (ADC)

El Arduino Uno tiene 6 pines analógicos (A0–A5). Cada uno puede leer voltajes entre **0V y 5V** y convertirlos en un número entero de **0 a 1023** usando un ADC de 10 bits.

$$\text{Valor} = \frac{V_{entrada}}{5V} \times 1023$$

| Voltaje | Valor leído |
|---------|-------------|
| 0V | 0 |
| 1.25V | 255 |
| 2.5V | 511 |
| 3.75V | 767 |
| 5V | 1023 |

La función para leer es:

```cpp
int valor = analogRead(A0); // Devuelve 0-1023
```

---

## 4.2 Ejercicio 1 — Leer un potenciómetro

Un potenciómetro es una resistencia variable. Al girar su eje, cambia la resistencia y por lo tanto el voltaje en su patilla central.

### Materiales
- 1 potenciómetro de 10kΩ
- Protoboard y cables

### Diagrama de conexión

```
Potenciómetro
  Pata izquierda  ─── 5V  (Arduino)
  Pata central    ─── A0  (Arduino)
  Pata derecha    ─── GND (Arduino)
```

### Código

```cpp
// Leer potenciómetro y mostrar en Serial
void setup() {
  Serial.begin(9600);
  Serial.println("Lectura de potenciómetro:");
}

void loop() {
  int valor = analogRead(A0);      // 0 a 1023

  // Convierte a voltaje
  float voltaje = valor * (5.0 / 1023.0);

  Serial.print("Valor crudo: ");
  Serial.print(valor);
  Serial.print("  |  Voltaje: ");
  Serial.print(voltaje, 2); // 2 decimales
  Serial.println(" V");

  delay(200);
}
```

Abre el Monitor Serial y gira el potenciómetro — verás los valores cambiar en tiempo real.

### La función `map()`

`map()` reescala un rango a otro. Por ejemplo, de 0-1023 a 0-100 (porcentaje):

```cpp
int porcentaje = map(analogRead(A0), 0, 1023, 0, 100);
```

Sintaxis: `map(valor, fromLow, fromHigh, toLow, toHigh)`

---

## 4.3 Ejercicio 2 — LED controlado por potenciómetro

```cpp
// Controla el brillo de un LED con el potenciómetro
const int PIN_POT = A0;
const int PIN_LED = 9; // Pin PWM

void setup() {
  pinMode(PIN_LED, OUTPUT);
}

void loop() {
  int lectura = analogRead(PIN_POT);          // 0 a 1023
  int brillo  = map(lectura, 0, 1023, 0, 255); // 0 a 255 (PWM)
  analogWrite(PIN_LED, brillo);
  delay(10);
}
```

> `analogWrite()` lo veremos en detalle en el Capítulo 5. Por ahora, sabe que controla el brillo del LED.

---

## 4.4 El sensor LDR (fotorresistencia)

Una **LDR** (Light Dependent Resistor) es una resistencia que cambia su valor según la luz:
- **Mucha luz** → baja resistencia (~100Ω)
- **Oscuridad** → alta resistencia (~1MΩ)

### Divisor de voltaje con LDR

Para convertir la variación de resistencia en voltaje, usamos un divisor:

```
5V ─── LDR ─── A0
                │
               R10kΩ
                │
               GND
```

$$V_{A0} = 5V \times \frac{R_{fija}}{R_{LDR} + R_{fija}}$$

### Código — Sensor de luz con umbral

```cpp
// Enciende el LED cuando hay oscuridad
const int PIN_LDR = A0;
const int PIN_LED = 9;
const int UMBRAL  = 400; // Ajusta según tu ambiente

void setup() {
  pinMode(PIN_LED, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int luz = analogRead(PIN_LDR);

  Serial.print("Luz: ");
  Serial.println(luz);

  if (luz < UMBRAL) {
    digitalWrite(PIN_LED, HIGH); // Oscuro → LED encendido
  } else {
    digitalWrite(PIN_LED, LOW);  // Hay luz → LED apagado
  }

  delay(100);
}
```

**Calibración:** Abre el Monitor Serial, tapa el LDR con la mano y anota el valor. Luego ilumínalo con tu celular y anota el otro. Pon el umbral entre los dos.

---

## 4.5 El sensor NTC (termistor)

Un **NTC** (Negative Temperature Coefficient) es una resistencia que disminuye con la temperatura.

### Diagrama de conexión (igual que el LDR)

```
5V ─── NTC ─── A0
                │
               R10kΩ
                │
               GND
```

### Código — Termómetro básico

```cpp
// Termómetro con NTC de 10kΩ (B=3950)
#include <math.h>

const int PIN_NTC = A0;
const float BETA  = 3950.0; // Constante B del NTC
const float R_NOM = 10000.0; // Resistencia nominal (10kΩ a 25°C)
const float R_FIJ = 10000.0; // Resistencia de la serie
const float T_NOM = 298.15;  // 25°C en Kelvin

void setup() {
  Serial.begin(9600);
}

void loop() {
  int adc = analogRead(PIN_NTC);

  // Calcula la resistencia del NTC
  float voltaje = adc * (5.0 / 1023.0);
  float resistencia = R_FIJ * (5.0 / voltaje - 1.0);

  // Ecuación de Steinhart-Hart simplificada
  float kelvin = 1.0 / (log(resistencia / R_NOM) / BETA + 1.0 / T_NOM);
  float celsius = kelvin - 273.15;

  Serial.print("Temperatura: ");
  Serial.print(celsius, 1);
  Serial.println(" °C");

  delay(1000);
}
```

---

## 4.6 El sensor LM35 (temperatura lineal)

El **LM35** es más sencillo: su salida es directamente proporcional a la temperatura.

$$T(°C) = \frac{V_{salida}}{10mV}$$

### Diagrama de conexión

```
LM35
  VCC ─── 5V
  OUT ─── A0
  GND ─── GND
```

### Código — Termómetro LM35

```cpp
// Termómetro con LM35
const int PIN_LM35 = A0;

void setup() {
  Serial.begin(9600);
}

void loop() {
  int adc = analogRead(PIN_LM35);

  // LM35: 10mV por grado Celsius
  // Con ADC de 10 bits y 5V de referencia:
  // voltaje (mV) = adc * (5000 / 1023)
  float temperatura = adc * (500.0 / 1023.0);

  Serial.print("Temperatura: ");
  Serial.print(temperatura, 1);
  Serial.println(" °C");

  delay(1000);
}
```

---

## 4.7 Ejercicio avanzado — Nightlight automático

Combina el LDR con los LEDs para crear una lamparita automática que se enciende gradualmente según la oscuridad.

```cpp
// Nightlight: brillo inversamente proporcional a la luz ambiente
const int PIN_LDR = A0;
const int PIN_LED = 9; // PIN PWM

void setup() {
  pinMode(PIN_LED, OUTPUT);
}

void loop() {
  int luz    = analogRead(PIN_LDR);              // 0 (oscuro) - 1023 (luz plena)
  int brillo = map(luz, 0, 1023, 255, 0);        // Invierte: más oscuro = más brillo
  brillo     = constrain(brillo, 0, 255);        // Asegura que esté en rango

  analogWrite(PIN_LED, brillo);
  delay(50);
}
```

> **`constrain(valor, min, max)`** limita el valor entre mínimo y máximo.

---

## Resumen del capítulo

| Función/Concepto | Descripción |
|-----------------|-------------|
| `analogRead(A0)` | Lee un pin analógico, devuelve 0-1023 |
| `map(v, 0, 1023, 0, 100)` | Reescala un rango de valores |
| `constrain(v, min, max)` | Limita un valor a un rango |
| LDR | Resistencia que varía con la luz |
| NTC / LM35 | Sensores de temperatura analógicos |

---

## Desafío

Construye un **termómetro visual con LEDs**:
- 1 LED verde: temperatura < 20°C
- 1 LED amarillo: temperatura entre 20°C y 30°C
- 1 LED rojo: temperatura > 30°C

---

[← Capítulo 3](capitulo-03.html){: .btn } [Capítulo 5 — PWM →](capitulo-05.html){: .btn .btn-primary }
