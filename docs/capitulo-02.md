---
layout: default
title: "Capítulo 2 — El Blink (¡Hola, LED!)"
nav_order: 3
---

# Capítulo 2 — El Blink (¡Hola, LED!)
{: .no_toc }

El Blink es el "Hola Mundo" de la electrónica. Aprenderás a controlar pines digitales y a crear secuencias de luz.
{: .fs-6 .fw-300 }

## Contenido
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 2.1 El LED y la resistencia

Un **LED** (Light Emitting Diode) es un componente que emite luz cuando pasa corriente por él. Tiene dos patas:

- **Ánodo (+)**: pata más larga — va al voltaje positivo.
- **Cátodo (−)**: pata más corta — va a GND.

> **¡Nunca conectes un LED directo al pin sin una resistencia!** La corriente destruirá el LED en segundos.

### ¿Qué resistencia usar?

La fórmula es la **Ley de Ohm**:

$$R = \frac{V_{fuente} - V_{LED}}{I_{LED}}$$

Con un Arduino Uno (5V), un LED rojo (V_LED ≈ 2V) y una corriente de 20mA:

$$R = \frac{5V - 2V}{0.02A} = 150\Omega$$

En la práctica, **220Ω o 330Ω** funcionan perfectamente con cualquier LED de colores básicos.

---

## 2.2 Pines digitales

Los pines digitales del Arduino solo conocen dos estados:

| Estado | Voltaje | Constante |
|--------|---------|-----------|
| **HIGH** | 5V | `HIGH` o `1` |
| **LOW** | 0V | `LOW` o `0` |

Para usar un pin como salida (que envíe voltaje), lo configuras con `pinMode(pin, OUTPUT)`.

---

## 2.3 Ejercicio 1 — Blink con el LED integrado

El Arduino Uno tiene un **LED soldado en la placa** conectado al **pin 13**. No necesitas ningún componente externo.

### Código

```cpp
// Blink - Parpadeo del LED integrado
// Compatible con Arduino Uno, Nano, Mega

void setup() {
  // Configura el pin 13 como salida
  pinMode(13, OUTPUT);
}

void loop() {
  digitalWrite(13, HIGH); // Enciende el LED
  delay(1000);            // Espera 1 segundo
  digitalWrite(13, LOW);  // Apaga el LED
  delay(1000);            // Espera 1 segundo
}
```

### Pasos
1. Copia el código en el IDE.
2. Sube al Arduino.
3. Observa el LED amarillo/naranja en la placa parpadear cada segundo.

> **Truco:** Puedes usar la constante `LED_BUILTIN` en lugar de `13` — es más portable entre placas: `pinMode(LED_BUILTIN, OUTPUT)`.

---

## 2.4 Ejercicio 2 — LED externo

Ahora vamos a conectar un LED externo.

### Materiales
- 1 LED (cualquier color)
- 1 resistencia de 220Ω
- Protoboard y cables

### Diagrama de conexión

```
Arduino Uno          Protoboard
   Pin 9  ───────── Resistencia 220Ω ──── Ánodo LED (+)
   GND    ─────────────────────────────── Cátodo LED (−)
```

### Código

```cpp
// LED externo en pin 9
const int PIN_LED = 9;

void setup() {
  pinMode(PIN_LED, OUTPUT);
}

void loop() {
  digitalWrite(PIN_LED, HIGH); // Enciende
  delay(500);                  // 0.5 segundos
  digitalWrite(PIN_LED, LOW);  // Apaga
  delay(500);
}
```

**Experimenta:** Cambia los valores de `delay()` y observa la diferencia. ¿Qué pasa si pones `delay(50)`?

---

## 2.5 Ejercicio 3 — Múltiples LEDs

Vamos a controlar 3 LEDs de forma independiente.

### Materiales
- 3 LEDs (rojo, amarillo, verde)
- 3 resistencias de 220Ω
- Protoboard y cables

### Diagrama de conexión

```
Arduino Uno
   Pin 11 ─── R220Ω ─── LED Rojo    (+)─── GND
   Pin 10 ─── R220Ω ─── LED Amarillo(+)─── GND
   Pin  9 ─── R220Ω ─── LED Verde   (+)─── GND
```

### Código — Secuencia de semáforo básico

```cpp
// Semáforo básico con 3 LEDs
const int LED_ROJO    = 11;
const int LED_AMARILLO = 10;
const int LED_VERDE   = 9;

void setup() {
  pinMode(LED_ROJO, OUTPUT);
  pinMode(LED_AMARILLO, OUTPUT);
  pinMode(LED_VERDE, OUTPUT);
}

void loop() {
  // ROJO — Detente (4 segundos)
  digitalWrite(LED_ROJO, HIGH);
  digitalWrite(LED_AMARILLO, LOW);
  digitalWrite(LED_VERDE, LOW);
  delay(4000);

  // AMARILLO — Prepárate (1 segundo)
  digitalWrite(LED_ROJO, LOW);
  digitalWrite(LED_AMARILLO, HIGH);
  digitalWrite(LED_VERDE, LOW);
  delay(1000);

  // VERDE — Avanza (4 segundos)
  digitalWrite(LED_ROJO, LOW);
  digitalWrite(LED_AMARILLO, LOW);
  digitalWrite(LED_VERDE, HIGH);
  delay(4000);

  // AMARILLO — Prepárate para detenerse (1 segundo)
  digitalWrite(LED_ROJO, LOW);
  digitalWrite(LED_AMARILLO, HIGH);
  digitalWrite(LED_VERDE, LOW);
  delay(1000);
}
```

---

## 2.6 Ejercicio 4 — Código Morse: SOS

El código Morse usa puntos (·) y rayas (—). La señal de socorro **SOS** es:

```
S = · · ·   (tres puntos)
O = — — —   (tres rayas)
S = · · ·   (tres puntos)
```

### Código

```cpp
// SOS en código Morse
const int LED = 13;

// Duraciones (en milisegundos)
const int PUNTO = 200;
const int RAYA  = 600;
const int PAUSA_LETRA = 400;
const int PAUSA_PALABRA = 1400;

void encender(int duracion) {
  digitalWrite(LED, HIGH);
  delay(duracion);
  digitalWrite(LED, LOW);
  delay(PUNTO); // Pausa entre señales
}

void setup() {
  pinMode(LED, OUTPUT);
}

void loop() {
  // S (· · ·)
  encender(PUNTO);
  encender(PUNTO);
  encender(PUNTO);
  delay(PAUSA_LETRA);

  // O (— — —)
  encender(RAYA);
  encender(RAYA);
  encender(RAYA);
  delay(PAUSA_LETRA);

  // S (· · ·)
  encender(PUNTO);
  encender(PUNTO);
  encender(PUNTO);
  delay(PAUSA_PALABRA); // Pausa antes de repetir
}
```

---

## 2.7 La función `millis()` — Sin bloquear el programa

`delay()` detiene completamente el Arduino. Para hacer varias cosas a la vez (parpadear un LED *y* leer un botón), usamos `millis()`:

```cpp
// Blink sin delay() — el LED parpadea pero el programa sigue corriendo
const int LED = 13;
const long INTERVALO = 1000; // milisegundos

unsigned long tiempoAnterior = 0;
bool estadoLED = LOW;

void setup() {
  pinMode(LED, OUTPUT);
}

void loop() {
  unsigned long tiempoActual = millis();

  // Si pasó el intervalo, cambia el estado del LED
  if (tiempoActual - tiempoAnterior >= INTERVALO) {
    tiempoAnterior = tiempoActual;
    estadoLED = !estadoLED;          // Invierte el estado
    digitalWrite(LED, estadoLED);
  }

  // Aquí puedes hacer otras cosas mientras el LED parpadea
}
```

> **Nota:** `millis()` retorna un `unsigned long`. Después de ~50 días se desborda y vuelve a 0, pero la resta `tiempoActual - tiempoAnterior` sigue siendo correcta gracias a la aritmética de enteros sin signo.

---

## Resumen del capítulo

| Función | Descripción |
|---------|-------------|
| `pinMode(pin, OUTPUT)` | Configura el pin como salida |
| `digitalWrite(pin, HIGH)` | Pone el pin a 5V (enciende el LED) |
| `digitalWrite(pin, LOW)` | Pone el pin a 0V (apaga el LED) |
| `delay(ms)` | Pausa el programa N milisegundos |
| `millis()` | Tiempo transcurrido desde el inicio (ms) |

---

## Desafío

Crea tu propia secuencia de Morse con las iniciales de tu nombre. Usa la función `encender()` del Ejercicio 4 y el [código Morse internacional](https://es.wikipedia.org/wiki/C%C3%B3digo_Morse).

---

[← Capítulo 1](capitulo-01.html){: .btn } [Capítulo 3 — Entradas Digitales →](capitulo-03.html){: .btn .btn-primary }
