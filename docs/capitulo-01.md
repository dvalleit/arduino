---
layout: default
title: "Capítulo 1 — Introducción a Arduino"
nav_order: 2
---

# Capítulo 1 — Introducción a Arduino
{: .no_toc }

Antes de encender tu primer LED, vamos a entender qué es Arduino, cómo funciona y cómo preparar tu entorno de trabajo.
{: .fs-6 .fw-300 }

## Contenido
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 1.1 ¿Qué es Arduino?

Arduino es una **plataforma de electrónica de código abierto** compuesta por hardware (la placa) y software (el entorno de programación). Fue diseñada para que cualquier persona — sin importar su nivel de experiencia — pueda crear proyectos interactivos.

Con Arduino puedes leer sensores (temperatura, luz, distancia), controlar actuadores (LEDs, motores, pantallas) y comunicarte con otros dispositivos.

### ¿Por qué Arduino es tan popular?

- **Bajo costo**: las placas originales y los clones compatibles son accesibles.
- **Comunidad enorme**: millones de proyectos documentados en internet.
- **Lenguaje simple**: basado en C/C++, con muchísimas librerías listas para usar.
- **Hardware abierto**: los esquemáticos son públicos; puedes hacer tu propia placa.

---

## 1.2 Tipos de placas Arduino

Existen docenas de modelos. Estos son los más comunes para aprender:

| Placa | Microcontrolador | Pines digitales | Pines analógicos | Ideal para |
|-------|-----------------|-----------------|-----------------|------------|
| **Uno** | ATmega328P | 14 (6 PWM) | 6 | Aprender — es la referencia |
| **Nano** | ATmega328P | 14 (6 PWM) | 8 | Proyectos compactos |
| **Mega 2560** | ATmega2560 | 54 (15 PWM) | 16 | Proyectos grandes |
| **Leonardo** | ATmega32U4 | 20 (7 PWM) | 12 | Emulación USB (teclado/ratón) |

> **Esta guía usa el Arduino Uno como referencia**, pero todo es compatible con Nano y Mega salvo que se indique lo contrario.

### Anatomía del Arduino Uno

```
                    ┌─────────────────────────────┐
    Pines           │  ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ │  Pines digitales 0-13
    de              │  RESET    [LED pin 13]       │
    alimentación    │  3.3V     [USB]              │  Conector USB
    (3.3V, 5V,      │  5V       [POWER]            │  para programar
    GND, Vin)       │  GND                         │
                    │  GND                         │
                    │  Vin                         │
                    │  A0 A1 A2 A3 A4 A5           │  Pines analógicos
                    └─────────────────────────────┘
```

**Componentes clave:**
- **Microcontrolador ATmega328P**: el "cerebro" que ejecuta tu código.
- **Cristal de 16 MHz**: marca el ritmo del procesador.
- **Regulador de voltaje**: convierte el USB (5V) o el jack (7-12V) a 5V estables.
- **LED integrado**: conectado al pin 13 — muy útil para pruebas rápidas.
- **Pines digitales 0-13**: entrada o salida digital (0V = LOW, 5V = HIGH).
- **Pines analógicos A0-A5**: leen voltajes de 0 a 5V con resolución de 10 bits.
- **Pines PWM (~)**: 3, 5, 6, 9, 10, 11 — simulan salida analógica.

---

## 1.3 Instalación del Arduino IDE

### Paso a paso

1. Ve a [arduino.cc/en/software](https://www.arduino.cc/en/software).
2. Descarga la versión **Arduino IDE 2.x** para tu sistema operativo.
3. Instala el programa (siguiente → siguiente → instalar).
4. Conecta tu Arduino Uno con el cable USB.
5. Abre el IDE — deberías ver la placa detectada automáticamente.

### Configura tu placa

1. Menú **Herramientas → Placa → Arduino AVR Boards → Arduino Uno**.
2. Menú **Herramientas → Puerto** → selecciona el puerto donde está el Arduino  
   (en Windows: `COM3`, `COM4`...; en macOS/Linux: `/dev/ttyUSB0` o `/dev/cu.usbmodem...`).

> **Consejo:** Si no aparece el puerto, instala el driver CH340 (para clones chinos) o el driver FTDI.

---

## 1.4 La estructura de un sketch

Todo programa de Arduino se llama **sketch** y tiene dos funciones obligatorias:

```cpp
// setup() se ejecuta una sola vez al encender o reiniciar el Arduino
void setup() {
  // Aquí configuras pines, inicializas librerías, etc.
}

// loop() se ejecuta en bucle infinito mientras el Arduino esté encendido
void loop() {
  // Aquí va la lógica principal de tu programa
}
```

### Tipos de datos más usados

| Tipo | Descripción | Rango |
|------|-------------|-------|
| `int` | Entero | -32,768 a 32,767 |
| `long` | Entero largo | ±2,147,483,647 |
| `float` | Decimal | ~6-7 dígitos significativos |
| `bool` | Verdadero/Falso | `true` o `false` |
| `char` | Carácter | -128 a 127 |
| `String` | Cadena de texto | Variable |
| `byte` | Entero sin signo | 0 a 255 |

### Comentarios

```cpp
// Esto es un comentario de una línea

/*
  Esto es un comentario
  de múltiples líneas
*/
```

---

## 1.5 Funciones esenciales

Estas son las funciones que usarás en casi todos los sketches:

```cpp
// Configura un pin como ENTRADA o SALIDA
pinMode(pin, modo);       // modo: INPUT, OUTPUT, INPUT_PULLUP

// Escribe HIGH (5V) o LOW (0V) en un pin digital
digitalWrite(pin, valor); // valor: HIGH o LOW

// Lee el estado de un pin digital (HIGH o LOW)
int estado = digitalRead(pin);

// Lee el voltaje en un pin analógico (0 a 1023)
int valor = analogRead(pin);

// Escribe un valor PWM en un pin (0 a 255)
analogWrite(pin, valor);

// Pausa el programa durante 'ms' milisegundos
delay(ms);

// Devuelve el tiempo en milisegundos desde el inicio
unsigned long t = millis();
```

---

## 1.6 Ejercicio 1 — Verificar la instalación

**Objetivo:** Confirmar que el IDE y el Arduino funcionan correctamente.

### Pasos

1. Abre el IDE de Arduino.
2. Ve a **Archivo → Ejemplos → 01.Basics → BareMinimum**.
3. Verás el sketch más simple posible:

```cpp
void setup() {
  // No hace nada por ahora
}

void loop() {
  // No hace nada por ahora
}
```

4. Haz clic en el botón **Verificar** (✓) — debe compilar sin errores.
5. Haz clic en **Subir** (→) — debe transferirse al Arduino.

Si la carga fue exitosa, verás `Subido.` en la barra de estado. ¡Felicitaciones, tu entorno funciona!

---

## 1.7 Ejercicio 2 — El Monitor Serial

El Monitor Serial te permite **ver mensajes** desde el Arduino en tu computadora. Es tu mejor herramienta para depurar y explorar.

```cpp
void setup() {
  // Inicia la comunicación serial a 9600 bits por segundo
  Serial.begin(9600);
  Serial.println("¡Arduino listo!");
}

void loop() {
  Serial.println("Hola desde el loop");
  delay(1000); // Espera 1 segundo
}
```

**Sube este sketch** y abre el Monitor Serial con **Ctrl+Mayús+M** (o el ícono de lupa). Deberías ver los mensajes aparecer cada segundo.

> **Importante:** La velocidad (`9600`) en `Serial.begin()` debe coincidir con la velocidad seleccionada en el Monitor Serial (esquina inferior derecha).

---

## Resumen del capítulo

| Concepto | Lo que aprendiste |
|----------|------------------|
| Arduino | Plataforma de hardware y software de código abierto |
| Sketch | Programa de Arduino con `setup()` y `loop()` |
| IDE | Entorno de desarrollo para escribir y subir programas |
| Monitor Serial | Herramienta para ver mensajes del Arduino |

---

## Desafío

Modifica el Ejercicio 2 para que el Arduino imprima tu nombre y la fecha actual cada vez que se enciende. Pista: `setup()` se ejecuta una sola vez.

---

[← Inicio](index.html){: .btn } [Capítulo 2 — El Blink →](capitulo-02.html){: .btn .btn-primary }
