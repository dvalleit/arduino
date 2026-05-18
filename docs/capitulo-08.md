---
layout: default
title: "Capítulo 8 — Pantalla LCD"
nav_order: 9
---

# Capítulo 8 — Pantalla LCD
{: .no_toc }

Aprende a mostrar texto, números y símbolos en una pantalla LCD 16×2 usando la librería LiquidCrystal.
{: .fs-6 .fw-300 }

## Contenido
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 8.1 La pantalla LCD 16×2

Una LCD 16×2 tiene **16 columnas y 2 filas**, lo que permite mostrar 32 caracteres a la vez.

### Dos formas de conectarla

| Conexión | Cables | Pines usados |
|----------|--------|-------------|
| **Paralela directa** | 6 + alimentación | 6 pines digitales |
| **Módulo I2C (PCF8574)** | 4 cables | SDA (A4) y SCL (A5) |

> **Recomendamos el módulo I2C** — mucho más simple y deja libres los pines para otras cosas.

---

## 8.2 Conexión con módulo I2C

El adaptador I2C (PCF8574) se suelda o se conecta directamente a la parte posterior de la LCD.

### Diagrama

```
Arduino Uno        Módulo I2C LCD
  5V     ─────── VCC
  GND    ─────── GND
  A4 (SDA)─────── SDA
  A5 (SCL)─────── SCL
```

### Instalar la librería

1. Menú **Herramientas → Administrar Bibliotecas...**
2. Busca `LiquidCrystal I2C`
3. Instala la de **Frank de Brabander** (la más popular)

### Encontrar la dirección I2C

Si no sabes la dirección de tu módulo (normalmente `0x27` o `0x3F`):

```cpp
// Escáner I2C - encuentra los dispositivos conectados
#include <Wire.h>

void setup() {
  Wire.begin();
  Serial.begin(9600);
  Serial.println("Escaneando I2C...");

  for (byte addr = 1; addr < 127; addr++) {
    Wire.beginTransmission(addr);
    if (Wire.endTransmission() == 0) {
      Serial.print("Dispositivo encontrado en: 0x");
      Serial.println(addr, HEX);
    }
  }
  Serial.println("Escaneo completo.");
}

void loop() {}
```

---

## 8.3 Ejercicio 1 — Hola Mundo en LCD (I2C)

```cpp
// Hola Mundo en LCD 16x2 con I2C
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// LiquidCrystal_I2C(dirección, columnas, filas)
LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  lcd.init();        // Inicializa la LCD
  lcd.backlight();   // Enciende la retroiluminación

  lcd.setCursor(0, 0); // Columna 0, Fila 0
  lcd.print("Hola, Mundo!");

  lcd.setCursor(0, 1); // Columna 0, Fila 1
  lcd.print("Arduino :)");
}

void loop() {
  // El mensaje se muestra una vez; no necesitamos el loop
}
```

### Funciones principales

```cpp
lcd.init();            // Inicializa la LCD
lcd.backlight();       // Enciende la luz de fondo
lcd.noBacklight();     // Apaga la luz de fondo
lcd.setCursor(col, fila); // Posiciona el cursor (0-indexed)
lcd.print("texto");    // Muestra texto en la posición actual
lcd.clear();           // Borra toda la pantalla
lcd.home();            // Cursor a posición (0,0)
lcd.cursor();          // Muestra el cursor (guión bajo)
lcd.noCursor();        // Oculta el cursor
lcd.blink();           // Cursor parpadeante
lcd.noBlink();         // Sin parpadeo
lcd.scrollDisplayLeft();  // Desplaza el contenido a la izquierda
lcd.scrollDisplayRight(); // Desplaza el contenido a la derecha
```

---

## 8.4 Ejercicio 2 — Contador regresivo

```cpp
// Contador regresivo de 10 a 0
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  lcd.init();
  lcd.backlight();

  lcd.setCursor(0, 0);
  lcd.print("Cuenta regresiva");

  for (int i = 10; i >= 0; i--) {
    lcd.setCursor(6, 1); // Centro de la segunda fila
    lcd.print("  ");     // Borra el número anterior
    lcd.setCursor(6, 1);

    if (i > 0) {
      lcd.print(i);
    } else {
      lcd.print("¡YA!");
    }
    delay(1000);
  }

  delay(2000);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Fin.");
}

void loop() {}
```

---

## 8.5 Ejercicio 3 — Reloj simple con millis()

```cpp
// Reloj digital simple usando millis()
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

int horas   = 0;
int minutos = 0;
int segundos = 0;

unsigned long ultimoTick = 0;

// Imprime número con cero a la izquierda
void imprimirDos(int n) {
  if (n < 10) lcd.print("0");
  lcd.print(n);
}

void setup() {
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("    Reloj:");
}

void loop() {
  if (millis() - ultimoTick >= 1000) {
    ultimoTick = millis();
    segundos++;

    if (segundos >= 60) {
      segundos = 0;
      minutos++;
    }
    if (minutos >= 60) {
      minutos = 0;
      horas++;
    }
    if (horas >= 24) {
      horas = 0;
    }

    // Muestra la hora en formato HH:MM:SS
    lcd.setCursor(4, 1);
    imprimirDos(horas);
    lcd.print(":");
    imprimirDos(minutos);
    lcd.print(":");
    imprimirDos(segundos);
  }
}
```

> **Nota:** Este reloj no es muy preciso a largo plazo. Para proyectos de reloj real, usa un módulo RTC como el DS3231.

---

## 8.6 Ejercicio 4 — Termómetro digital en LCD

Muestra la temperatura de un sensor LM35 en la pantalla.

### Materiales
- LCD 16×2 con módulo I2C
- Sensor LM35
- Protoboard y cables

### Código

```cpp
// Termómetro LM35 con LCD I2C
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);
const int PIN_LM35 = A0;

// Carácter personalizado: símbolo de grado °
byte gradoSimbolo[8] = {
  0b00110,
  0b01001,
  0b01001,
  0b00110,
  0b00000,
  0b00000,
  0b00000,
  0b00000
};

void setup() {
  lcd.init();
  lcd.backlight();
  lcd.createChar(0, gradoSimbolo); // Registra el carácter en posición 0

  lcd.setCursor(0, 0);
  lcd.print("  Termometro  ");
}

void loop() {
  int adc = analogRead(PIN_LM35);
  float temperatura = adc * (500.0 / 1023.0);

  lcd.setCursor(2, 1);
  lcd.print("Temp: ");
  lcd.print(temperatura, 1);
  lcd.write(0);        // Imprime el símbolo de grado
  lcd.print("C  ");

  delay(1000);
}
```

---

## 8.7 Ejercicio 5 — Texto desplazante

```cpp
// Texto que se desplaza por la pantalla
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

String mensaje = "  Arduino desde Cero  ";

void setup() {
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("  Texto scroll:");
}

void loop() {
  // Desplaza el texto por la segunda fila
  for (int i = 0; i < mensaje.length(); i++) {
    lcd.setCursor(0, 1);
    lcd.print(mensaje.substring(i, i + 16)); // Muestra 16 caracteres
    delay(250);
  }
}
```

---

## Conexión directa (sin I2C) — Referencia

Si no tienes módulo I2C, puedes conectar la LCD directamente con 6 pines:

```cpp
// Librería estándar (sin I2C)
#include <LiquidCrystal.h>

// LiquidCrystal(RS, EN, D4, D5, D6, D7)
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

// RS  ─── Pin 12
// EN  ─── Pin 11
// D4  ─── Pin 5
// D5  ─── Pin 4
// D6  ─── Pin 3
// D7  ─── Pin 2
// VO  ─── Potenciómetro para contraste
// VSS ─── GND
// VDD ─── 5V
// A   ─── 5V (retroiluminación)
// K   ─── GND (retroiluminación)

void setup() {
  lcd.begin(16, 2);
  lcd.print("Hola sin I2C!");
}
```

---

## Resumen del capítulo

| Función | Descripción |
|---------|-------------|
| `lcd.init()` | Inicializa la pantalla |
| `lcd.backlight()` | Enciende la retroiluminación |
| `lcd.setCursor(col, fila)` | Posiciona el cursor |
| `lcd.print()` | Muestra texto o números |
| `lcd.clear()` | Borra la pantalla |
| `lcd.createChar(id, datos)` | Crea un carácter personalizado |

---

## Desafío

Construye una **estación de monitoreo** que muestre en la LCD:
- Fila 1: temperatura del LM35
- Fila 2: valor del potenciómetro como porcentaje (0% a 100%)

El valor debe actualizarse cada 500ms.

---

[← Capítulo 7](capitulo-07.html){: .btn } [Capítulo 9 — Sensores Avanzados →](capitulo-09.html){: .btn .btn-primary }
