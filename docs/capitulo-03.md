---
layout: default
title: "Capítulo 3 — Entradas Digitales"
nav_order: 4
---

# Capítulo 3 — Entradas Digitales
{: .no_toc }

Aprenderás a leer botones y pulsadores para que tu Arduino reaccione al mundo exterior.
{: .fs-6 .fw-300 }

## Contenido
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 3.1 Configurar un pin como entrada

Un pin configurado como **INPUT** puede leer si hay 5V (HIGH) o 0V (LOW) aplicado externamente.

```cpp
pinMode(pin, INPUT);         // Entrada normal (puede quedar flotante)
pinMode(pin, INPUT_PULLUP);  // Entrada con resistencia interna pull-up
```

---

## 3.2 Pull-up y Pull-down

Cuando un botón está abierto (sin presionar), el pin queda **flotante** — puede leer HIGH o LOW aleatoriamente. Para fijar el estado, usamos resistencias:

### Pull-down (resistencia a GND)

```
Pin ─── Botón ─── 5V
 │
 └─── R10kΩ ─── GND

Sin presionar: pin lee LOW
Con presionar: pin lee HIGH
```

### Pull-up (resistencia a 5V)

```
Pin ─── Botón ─── GND
 │
 └─── R10kΩ ─── 5V

Sin presionar: pin lee HIGH
Con presionar: pin lee LOW   ← lógica invertida
```

### Pull-up interno del Arduino

El Arduino tiene resistencias pull-up internas (~50kΩ). Con `INPUT_PULLUP` no necesitas resistencia externa:

```
Pin ─── Botón ─── GND

Sin presionar: pin lee HIGH
Con presionar: pin lee LOW
```

> **Recomendación:** Usa siempre `INPUT_PULLUP` — es más simple y ahorra componentes.

---

## 3.3 Ejercicio 1 — Botón enciende LED

### Materiales
- 1 LED + resistencia 220Ω
- 1 pulsador
- Protoboard y cables

### Diagrama de conexión

```
Arduino
  Pin 2  ─── Botón ─── GND      (INPUT_PULLUP)
  Pin 9  ─── R220Ω ─── LED ─── GND
```

### Código

```cpp
// Botón controla LED con INPUT_PULLUP
const int PIN_BOTON = 2;
const int PIN_LED   = 9;

void setup() {
  pinMode(PIN_BOTON, INPUT_PULLUP); // Pull-up interno
  pinMode(PIN_LED, OUTPUT);
}

void loop() {
  int estado = digitalRead(PIN_BOTON);

  // Con INPUT_PULLUP: LOW = presionado, HIGH = suelto
  if (estado == LOW) {
    digitalWrite(PIN_LED, HIGH); // Enciende mientras se presiona
  } else {
    digitalWrite(PIN_LED, LOW);  // Apaga al soltar
  }
}
```

**Observa:** El LED se enciende mientras mantienes el botón presionado y se apaga al soltar.

---

## 3.4 El problema del "rebote" (Bounce)

Cuando presionas un botón mecánico, los contactos metálicos rebotan durante 5-50 ms antes de estabilizarse. El Arduino es tan rápido que detecta estos rebotes como múltiples pulsaciones.

```
Señal real con rebote:
        ___     ___________
_______|   |___|
        ↑ rebotes

Señal que queremos:
        ___________
_______|
```

### Solución por software — Debounce

```cpp
// Debounce por software
const int PIN_BOTON = 2;
const int PIN_LED   = 9;
const int DEBOUNCE_MS = 50; // Tiempo de debounce

int estadoLED = LOW;
int lectura;
int ultimaLectura = HIGH;
unsigned long tiempoUltimoRebote = 0;

void setup() {
  pinMode(PIN_BOTON, INPUT_PULLUP);
  pinMode(PIN_LED, OUTPUT);
}

void loop() {
  lectura = digitalRead(PIN_BOTON);

  // Si la lectura cambió, reinicia el contador
  if (lectura != ultimaLectura) {
    tiempoUltimoRebote = millis();
  }

  // Si la señal lleva más de DEBOUNCE_MS ms estable, es válida
  if ((millis() - tiempoUltimoRebote) > DEBOUNCE_MS) {
    // Si el botón fue presionado (flanco descendente)
    if (lectura == LOW && ultimaLectura == HIGH) {
      estadoLED = !estadoLED; // Cambia el estado del LED
      digitalWrite(PIN_LED, estadoLED);
    }
  }

  ultimaLectura = lectura;
}
```

---

## 3.5 Ejercicio 2 — Toggle con debounce

El Toggle es el modo "interruptor": cada pulsación cambia el estado del LED.

El código del apartado anterior (3.4) ya implementa el toggle con debounce. Súbelo y prueba: cada vez que presiones el botón, el LED cambia de estado.

---

## 3.6 Ejercicio 3 — Semáforo interactivo con botón

Un semáforo que avanza de fase cuando se presiona el botón (como los semáforos peatonales).

### Materiales
- 3 LEDs (rojo, amarillo, verde) + 3 resistencias 220Ω
- 1 pulsador
- Protoboard y cables

### Diagrama de conexión

```
Arduino
  Pin 2  ─── Botón ─── GND
  Pin 11 ─── R220Ω ─── LED Rojo    ─── GND
  Pin 10 ─── R220Ω ─── LED Amarillo─── GND
  Pin  9 ─── R220Ω ─── LED Verde   ─── GND
```

### Código

```cpp
// Semáforo controlado por botón peatonal
const int BOTON    = 2;
const int LED_ROJO    = 11;
const int LED_AMARILLO = 10;
const int LED_VERDE   = 9;

// Fases del semáforo
enum Fase { ROJO, AMARILLO, VERDE };
Fase faseActual = VERDE; // Comienza en verde

bool botonPresionadoAntes = false;

void apagaTodos() {
  digitalWrite(LED_ROJO, LOW);
  digitalWrite(LED_AMARILLO, LOW);
  digitalWrite(LED_VERDE, LOW);
}

void mostrarFase(Fase f) {
  apagaTodos();
  if (f == ROJO)     digitalWrite(LED_ROJO, HIGH);
  if (f == AMARILLO) digitalWrite(LED_AMARILLO, HIGH);
  if (f == VERDE)    digitalWrite(LED_VERDE, HIGH);
}

void setup() {
  pinMode(BOTON, INPUT_PULLUP);
  pinMode(LED_ROJO, OUTPUT);
  pinMode(LED_AMARILLO, OUTPUT);
  pinMode(LED_VERDE, OUTPUT);
  mostrarFase(faseActual);
}

void loop() {
  bool botonPresionado = (digitalRead(BOTON) == LOW);

  // Detecta flanco de subida (acaba de presionarse)
  if (botonPresionado && !botonPresionadoAntes) {
    delay(50); // Debounce simple

    // Avanza a la siguiente fase
    if (faseActual == VERDE) {
      faseActual = AMARILLO;
    } else if (faseActual == AMARILLO) {
      faseActual = ROJO;
    } else {
      faseActual = VERDE;
    }

    mostrarFase(faseActual);
  }

  botonPresionadoAntes = botonPresionado;
  delay(10);
}
```

---

## 3.7 Ejercicio 4 — Piano de 4 notas

Conecta 4 botones y un buzzer para crear un piano simple.

### Materiales
- 4 pulsadores
- 1 buzzer pasivo (o altavoz pequeño)
- Protoboard y cables

### Diagrama de conexión

```
Arduino
  Pin 2 ─── Botón 1 ─── GND   (Nota DO)
  Pin 3 ─── Botón 2 ─── GND   (Nota MI)
  Pin 4 ─── Botón 3 ─── GND   (Nota SOL)
  Pin 5 ─── Botón 4 ─── GND   (Nota DO agudo)
  Pin 8 ─── Buzzer   ─── GND
```

### Código

```cpp
// Piano de 4 notas con buzzer
const int BOTONES[] = {2, 3, 4, 5};
const int BUZZER = 8;

// Frecuencias en Hz (escala de Do mayor)
const int NOTAS[] = {
  262,  // DO  (C4)
  330,  // MI  (E4)
  392,  // SOL (G4)
  523   // DO  (C5)
};

void setup() {
  for (int i = 0; i < 4; i++) {
    pinMode(BOTONES[i], INPUT_PULLUP);
  }
  pinMode(BUZZER, OUTPUT);
}

void loop() {
  bool algunaNotaSonando = false;

  for (int i = 0; i < 4; i++) {
    if (digitalRead(BOTONES[i]) == LOW) {
      tone(BUZZER, NOTAS[i]); // Toca la nota correspondiente
      algunaNotaSonando = true;
      break; // Solo una nota a la vez
    }
  }

  if (!algunaNotaSonando) {
    noTone(BUZZER); // Silencio si no hay botón presionado
  }
}
```

> **`tone(pin, frecuencia)`** genera una onda cuadrada en el pin especificado.  
> **`noTone(pin)`** detiene la reproducción.

---

## Resumen del capítulo

| Función | Descripción |
|---------|-------------|
| `pinMode(pin, INPUT_PULLUP)` | Pin de entrada con resistencia pull-up interna |
| `digitalRead(pin)` | Lee HIGH o LOW en el pin |
| `tone(pin, Hz)` | Genera un sonido en un buzzer |
| `noTone(pin)` | Detiene el sonido |

---

## Desafío

Amplía el piano del Ejercicio 4 para que tenga **8 notas** (una octava completa: Do Re Mi Fa Sol La Si Do). Necesitarás 8 botones y usar los pines 2–9.

---

[← Capítulo 2](capitulo-02.html){: .btn } [Capítulo 4 — Señales Analógicas →](capitulo-04.html){: .btn .btn-primary }
