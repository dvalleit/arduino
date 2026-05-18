---
layout: default
title: "Capítulo 10 — Proyecto Final: Estación Meteorológica"
nav_order: 11
---

# Capítulo 10 — Proyecto Final: Estación Meteorológica
{: .no_toc }

Integra todo lo aprendido en un proyecto completo: una estación meteorológica con LCD, sensor de temperatura/humedad, sensor de luz y control serial.
{: .fs-6 .fw-300 }

## Contenido
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 10.1 Descripción del proyecto

La **Estación Meteorológica Arduino** muestra en tiempo real:

- 🌡️ Temperatura (DHT11/22)
- 💧 Humedad relativa (DHT11/22)
- ☀️ Nivel de luz (LDR)
- 🕐 Tiempo desde el encendido (reloj interno)

### Funciones

| Función | Descripción |
|---------|-------------|
| LCD 16×2 | Muestra temperatura, humedad y luz en pantallas rotativas |
| LED RGB | Indica si hace calor (rojo), frío (azul) o temperatura ideal (verde) |
| Monitor Serial | Exporta datos en formato CSV para graficar en Excel |
| Botón | Cambia la pantalla que se muestra en la LCD |
| Potenciómetro | Ajusta el umbral de temperatura "ideal" |

---

## 10.2 Lista de componentes

| Componente | Cantidad | Notas |
|-----------|----------|-------|
| Arduino Uno (o Nano) | 1 | |
| Sensor DHT11 o DHT22 | 1 | |
| Sensor LDR | 1 | |
| LCD 16×2 con módulo I2C | 1 | Dirección 0x27 o 0x3F |
| LED RGB (cátodo común) | 1 | |
| Resistencias 220Ω | 3 | Para el LED RGB |
| Resistencia 10kΩ | 2 | Pull-up DHT + divisor LDR |
| Potenciómetro 10kΩ | 1 | Umbral de temperatura |
| Pulsador | 1 | Cambio de pantalla |
| Protoboard grande | 1 | |
| Cables dupont | ~20 | |
| Cable USB | 1 | Para programar y alimentar |

---

## 10.3 Diagrama de conexiones completo

```
Arduino Uno
│
├── 5V ────────── VCC (LCD I2C)
│             ── VCC (DHT11)
│             ── Potenciómetro extremo izq.
│             ── Extremo superior (divisor LDR)
│
├── GND ───────── GND (LCD I2C)
│              ── GND (DHT11)
│              ── GND (LED RGB, cátodo)
│              ── GND (Botón)
│              ── GND (Potenciómetro extremo der.)
│              ── Extremo inferior (divisor LDR)
│
├── A4 (SDA) ─── SDA (LCD I2C)
├── A5 (SCL) ─── SCL (LCD I2C)
│
├── A0 ────────── LDR (punto medio del divisor con R10kΩ a GND)
├── A1 ────────── Potenciómetro (patilla central)
│
├── Pin 2 ─────── DHT11/22 (DATA, con R4.7kΩ a 5V)
├── Pin 3 ─────── Botón ──── GND  (INPUT_PULLUP)
│
├── Pin 9  ─────── R220Ω ─── LED R
├── Pin 10 ─────── R220Ω ─── LED G
├── Pin 11 ─────── R220Ω ─── LED B
```

---

## 10.4 Código completo — Paso a paso

### Paso 1: Incluir librerías y definir pines

```cpp
// ============================================
// ESTACIÓN METEOROLÓGICA CON ARDUINO
// Capítulo 10 — Proyecto Final
// Aprende Arduino desde Cero
// ============================================

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>

// ── Pines ────────────────────────────────────
#define PIN_DHT    2
#define TIPO_DHT   DHT11    // Cambia a DHT22 si corresponde
#define PIN_BOTON  3

const int PIN_LDR  = A0;
const int PIN_POT  = A1;
const int PIN_R    = 9;
const int PIN_G    = 10;
const int PIN_B    = 11;

// ── Objetos ──────────────────────────────────
LiquidCrystal_I2C lcd(0x27, 16, 2);
DHT dht(PIN_DHT, TIPO_DHT);
```

### Paso 2: Variables globales

```cpp
// ── Variables de datos ────────────────────────
float temperatura   = 0;
float humedad       = 0;
int   nivelLuz      = 0;
float umbralTemp    = 25.0; // Temperatura "ideal" por defecto

// ── Pantallas ────────────────────────────────
const int NUM_PANTALLAS = 3;
int pantallaActual = 0;

// ── Control del tiempo ────────────────────────
unsigned long ultimaLecturaDHT = 0;
unsigned long ultimaActLCD     = 0;
const long INTERVALO_DHT = 2000;   // Leer DHT cada 2s
const long INTERVALO_LCD = 500;    // Actualizar LCD cada 500ms

// ── Botón ─────────────────────────────────────
bool botonAntes = HIGH;

// ── Caracteres LCD personalizados ─────────────
byte gradoChar[8]    = {0b00110,0b01001,0b01001,0b00110,0,0,0,0};
byte humChar[8]      = {0b00100,0b01110,0b11111,0b11111,0b01110,0b00100,0,0};
byte solChar[8]      = {0b00100,0b10101,0b01110,0b11111,0b01110,0b10101,0b00100,0};
byte corazonChar[8]  = {0,0b01010,0b11111,0b11111,0b01110,0b00100,0,0};
```

### Paso 3: Funciones auxiliares

```cpp
// ── Funciones de LED RGB ──────────────────────
void setColor(int r, int g, int b) {
  analogWrite(PIN_R, r);
  analogWrite(PIN_G, g);
  analogWrite(PIN_B, b);
}

void actualizarLED() {
  if (temperatura > umbralTemp + 5) {
    setColor(255, 0, 0);      // Rojo: caliente
  } else if (temperatura < umbralTemp - 5) {
    setColor(0, 80, 255);     // Azul: frío
  } else {
    setColor(0, 200, 0);      // Verde: ideal
  }
}

// ── Convierte nivel LDR en texto ──────────────
String descripcionLuz(int nivel) {
  if (nivel > 800) return "Oscuro      ";
  if (nivel > 500) return "Penumbra    ";
  if (nivel > 200) return "Claro       ";
  return "Muy claro   ";
}

// ── Imprime número con cero a la izquierda ────
void imprimirDos(int n) {
  if (n < 10) lcd.print("0");
  lcd.print(n);
}

// ── Tiempo en HH:MM:SS ────────────────────────
String tiempoFormateado() {
  unsigned long seg = millis() / 1000;
  int h = seg / 3600;
  int m = (seg % 3600) / 60;
  int s = seg % 60;

  String t = "";
  if (h < 10) t += "0";
  t += h; t += ":";
  if (m < 10) t += "0";
  t += m; t += ":";
  if (s < 10) t += "0";
  t += s;
  return t;
}
```

### Paso 4: Mostrar pantallas en la LCD

```cpp
// ── Pantalla 0: Temperatura y Humedad ────────
void mostrarPantalla0() {
  lcd.setCursor(0, 0);
  lcd.write(0);           // Símbolo de grado (reutilizado como termómetro)
  lcd.print(" Temp: ");
  lcd.print(temperatura, 1);
  lcd.write(0);
  lcd.print("C  ");

  lcd.setCursor(0, 1);
  lcd.write(1);           // Símbolo de humedad
  lcd.print(" Hum:  ");
  lcd.print(humedad, 1);
  lcd.print("%   ");
}

// ── Pantalla 1: Luz y Umbral ──────────────────
void mostrarPantalla1() {
  lcd.setCursor(0, 0);
  lcd.write(2);           // Símbolo de sol
  lcd.print(" Luz: ");
  lcd.print(descripcionLuz(nivelLuz));

  lcd.setCursor(0, 1);
  lcd.write(3);           // Símbolo de corazón (umbral ideal)
  lcd.print(" Ideal: ");
  lcd.print(umbralTemp, 1);
  lcd.write(0);
  lcd.print("C ");
}

// ── Pantalla 2: Tiempo activo ─────────────────
void mostrarPantalla2() {
  lcd.setCursor(0, 0);
  lcd.print("Tiempo activo:  ");
  lcd.setCursor(3, 1);
  lcd.print(tiempoFormateado());
  lcd.print("  ");
}

// ── Selecciona qué pantalla mostrar ──────────
void actualizarLCD() {
  lcd.clear();
  switch (pantallaActual) {
    case 0: mostrarPantalla0(); break;
    case 1: mostrarPantalla1(); break;
    case 2: mostrarPantalla2(); break;
  }
}
```

### Paso 5: Enviar datos por Serial (CSV)

```cpp
// ── Logger de datos en formato CSV ───────────
void enviarCSV() {
  // Cabecera (solo si es la primera vez)
  static bool cabeceraEnviada = false;
  if (!cabeceraEnviada) {
    Serial.println("tiempo_ms,temperatura,humedad,luz,umbral");
    cabeceraEnviada = true;
  }

  Serial.print(millis());
  Serial.print(",");
  Serial.print(temperatura, 2);
  Serial.print(",");
  Serial.print(humedad, 2);
  Serial.print(",");
  Serial.print(nivelLuz);
  Serial.print(",");
  Serial.println(umbralTemp, 1);
}
```

### Paso 6: setup() y loop()

```cpp
void setup() {
  // ── Serial ───────────────────────────────────
  Serial.begin(9600);
  Serial.println("# Estacion Meteorologica - Arduino");
  Serial.println("# Esperando lecturas...");

  // ── LCD ──────────────────────────────────────
  lcd.init();
  lcd.backlight();
  lcd.createChar(0, gradoChar);
  lcd.createChar(1, humChar);
  lcd.createChar(2, solChar);
  lcd.createChar(3, corazonChar);

  // Pantalla de bienvenida
  lcd.setCursor(0, 0);
  lcd.print(" Estacion Meteo ");
  lcd.setCursor(0, 1);
  lcd.print("  Iniciando...  ");
  delay(2000);
  lcd.clear();

  // ── DHT ──────────────────────────────────────
  dht.begin();

  // ── Botón ────────────────────────────────────
  pinMode(PIN_BOTON, INPUT_PULLUP);

  // ── LED RGB ───────────────────────────────────
  pinMode(PIN_R, OUTPUT);
  pinMode(PIN_G, OUTPUT);
  pinMode(PIN_B, OUTPUT);
  setColor(0, 0, 255); // Azul en arranque
  delay(500);
  setColor(0, 0, 0);
}

void loop() {
  unsigned long ahora = millis();

  // ── 1. Leer botón (cambiar pantalla) ─────────
  bool botonAhora = digitalRead(PIN_BOTON);
  if (botonAhora == LOW && botonAntes == HIGH) {
    delay(50); // Debounce
    pantallaActual = (pantallaActual + 1) % NUM_PANTALLAS;
    actualizarLCD();
  }
  botonAntes = botonAhora;

  // ── 2. Leer potenciómetro (umbral de temp.) ───
  umbralTemp = map(analogRead(PIN_POT), 0, 1023, 15, 40);

  // ── 3. Leer LDR ───────────────────────────────
  nivelLuz = analogRead(PIN_LDR);

  // ── 4. Leer DHT (cada 2 segundos) ────────────
  if (ahora - ultimaLecturaDHT >= INTERVALO_DHT) {
    ultimaLecturaDHT = ahora;

    float t = dht.readTemperature();
    float h = dht.readHumidity();

    if (!isnan(t) && !isnan(h)) {
      temperatura = t;
      humedad     = h;
      actualizarLED();   // Actualiza color del LED
      enviarCSV();       // Envía datos por Serial
    }
  }

  // ── 5. Actualizar LCD (cada 500ms) ───────────
  if (ahora - ultimaActLCD >= INTERVALO_LCD) {
    ultimaActLCD = ahora;
    actualizarLCD();
  }
}
```

---

## 10.5 Código completo (todo junto)

Para tu comodidad, aquí está todo el código en un solo bloque listo para copiar:

```cpp
// ============================================
// ESTACIÓN METEOROLÓGICA CON ARDUINO
// ============================================
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>

#define PIN_DHT    2
#define TIPO_DHT   DHT11
#define PIN_BOTON  3

const int PIN_LDR = A0, PIN_POT = A1;
const int PIN_R = 9, PIN_G = 10, PIN_B = 11;

LiquidCrystal_I2C lcd(0x27, 16, 2);
DHT dht(PIN_DHT, TIPO_DHT);

float temperatura = 0, humedad = 0, umbralTemp = 25.0;
int nivelLuz = 0, pantallaActual = 0;
bool botonAntes = HIGH;
unsigned long ultimaLecturaDHT = 0, ultimaActLCD = 0;

byte gradoChar[8]   = {0b00110,0b01001,0b01001,0b00110,0,0,0,0};
byte humChar[8]     = {0b00100,0b01110,0b11111,0b11111,0b01110,0b00100,0,0};
byte solChar[8]     = {0b00100,0b10101,0b01110,0b11111,0b01110,0b10101,0b00100,0};
byte corazonChar[8] = {0,0b01010,0b11111,0b11111,0b01110,0b00100,0,0};

void setColor(int r, int g, int b) {
  analogWrite(PIN_R, r); analogWrite(PIN_G, g); analogWrite(PIN_B, b);
}

void actualizarLED() {
  if (temperatura > umbralTemp + 5)      setColor(255, 0, 0);
  else if (temperatura < umbralTemp - 5) setColor(0, 80, 255);
  else                                   setColor(0, 200, 0);
}

String descripcionLuz(int n) {
  if (n > 800) return "Oscuro      ";
  if (n > 500) return "Penumbra    ";
  if (n > 200) return "Claro       ";
  return "Muy claro   ";
}

String tiempoFormateado() {
  unsigned long s = millis() / 1000;
  String t = "";
  int h = s/3600, m = (s%3600)/60, sec = s%60;
  if (h<10) t+="0"; t+=h; t+=":";
  if (m<10) t+="0"; t+=m; t+=":";
  if (sec<10) t+="0"; t+=sec;
  return t;
}

void mostrarPantalla0() {
  lcd.setCursor(0,0); lcd.write(0); lcd.print(" Temp: ");
  lcd.print(temperatura,1); lcd.write(0); lcd.print("C  ");
  lcd.setCursor(0,1); lcd.write(1); lcd.print(" Hum:  ");
  lcd.print(humedad,1); lcd.print("%   ");
}

void mostrarPantalla1() {
  lcd.setCursor(0,0); lcd.write(2); lcd.print(" Luz: ");
  lcd.print(descripcionLuz(nivelLuz));
  lcd.setCursor(0,1); lcd.write(3); lcd.print(" Ideal: ");
  lcd.print(umbralTemp,1); lcd.write(0); lcd.print("C ");
}

void mostrarPantalla2() {
  lcd.setCursor(0,0); lcd.print("Tiempo activo:  ");
  lcd.setCursor(3,1); lcd.print(tiempoFormateado()); lcd.print("  ");
}

void actualizarLCD() {
  lcd.clear();
  switch (pantallaActual) {
    case 0: mostrarPantalla0(); break;
    case 1: mostrarPantalla1(); break;
    case 2: mostrarPantalla2(); break;
  }
}

void enviarCSV() {
  static bool cab = false;
  if (!cab) { Serial.println("tiempo_ms,temperatura,humedad,luz,umbral"); cab=true; }
  Serial.print(millis()); Serial.print(",");
  Serial.print(temperatura,2); Serial.print(",");
  Serial.print(humedad,2); Serial.print(",");
  Serial.print(nivelLuz); Serial.print(",");
  Serial.println(umbralTemp,1);
}

void setup() {
  Serial.begin(9600);
  lcd.init(); lcd.backlight();
  lcd.createChar(0, gradoChar); lcd.createChar(1, humChar);
  lcd.createChar(2, solChar);   lcd.createChar(3, corazonChar);
  lcd.setCursor(0,0); lcd.print(" Estacion Meteo ");
  lcd.setCursor(0,1); lcd.print("  Iniciando...  ");
  delay(2000); lcd.clear();
  dht.begin();
  pinMode(PIN_BOTON, INPUT_PULLUP);
  pinMode(PIN_R, OUTPUT); pinMode(PIN_G, OUTPUT); pinMode(PIN_B, OUTPUT);
  setColor(0, 0, 255); delay(500); setColor(0, 0, 0);
}

void loop() {
  unsigned long ahora = millis();
  bool botonAhora = digitalRead(PIN_BOTON);
  if (botonAhora == LOW && botonAntes == HIGH) {
    delay(50);
    pantallaActual = (pantallaActual + 1) % 3;
    actualizarLCD();
  }
  botonAntes = botonAhora;
  umbralTemp = map(analogRead(PIN_POT), 0, 1023, 15, 40);
  nivelLuz   = analogRead(PIN_LDR);
  if (ahora - ultimaLecturaDHT >= 2000) {
    ultimaLecturaDHT = ahora;
    float t = dht.readTemperature(), h = dht.readHumidity();
    if (!isnan(t) && !isnan(h)) {
      temperatura = t; humedad = h;
      actualizarLED(); enviarCSV();
    }
  }
  if (ahora - ultimaActLCD >= 500) {
    ultimaActLCD = ahora;
    actualizarLCD();
  }
}
```

---

## 10.6 Cómo graficar los datos en Excel / Google Sheets

1. Abre el Monitor Serial con velocidad **9600**.
2. Deja el Arduino funcionando por el tiempo que quieras registrar.
3. Selecciona todo el texto del Monitor Serial y cópialo.
4. Pégalo en un archivo de texto y guárdalo como `datos.csv`.
5. Ábrelo en **Excel** o **Google Sheets** → los datos se separarán automáticamente por columnas.
6. Selecciona las columnas y crea un **gráfico de líneas**.

---

## 10.7 Mejoras posibles

Una vez que el proyecto básico funcione, aquí hay ideas para ampliarlo:

| Mejora | Componente adicional |
|--------|---------------------|
| Reloj real (no pierde la hora) | Módulo RTC DS3231 |
| Guardar historial | Módulo SD Card |
| Enviar datos por WiFi | ESP8266 / ESP32 |
| Pantalla más grande | OLED 128×64 I2C |
| Sensor de presión atmosférica | BMP280 |
| Sensor de calidad del aire | MQ-135 |
| Carga solar | Panel solar + TP4056 |
| Caja impresa en 3D | Archivo STL personalizado |

---

## 10.8 ¿Y ahora qué?

¡Felicitaciones por llegar hasta aquí! Has aprendido:

- ✅ Programación en C++ para microcontroladores
- ✅ Control de entradas y salidas digitales y analógicas
- ✅ PWM y control de brillo/velocidad
- ✅ Comunicación serial con la computadora
- ✅ Control de servos y motores DC
- ✅ Uso de pantallas LCD
- ✅ Sensores de distancia, temperatura y humedad
- ✅ Integrar todo en un proyecto completo

### Próximos pasos recomendados

1. **ESP32 / ESP8266** — Arduino con WiFi integrado para proyectos IoT
2. **Comunicación I2C y SPI** — para conectar múltiples sensores avanzados
3. **MQTT y Home Assistant** — domótica con Arduino
4. **PCB personalizada** — diseña tu propia placa con KiCad (gratuito)
5. **FreeRTOS** — programación multitarea en microcontroladores

### Recursos para seguir aprendiendo

- [arduino.cc/en/Tutorial/HomePage](https://www.arduino.cc/en/Tutorial/HomePage) — tutoriales oficiales
- [randomnerdtutorials.com](https://randomnerdtutorials.com) — proyectos prácticos (inglés)
- [programarfacil.com](https://programarfacil.com) — tutoriales en español
- [hackster.io](https://www.hackster.io) — comunidad de proyectos
- [GitHub — arduino/arduino-examples](https://github.com/arduino/arduino-examples) — ejemplos oficiales

---

[← Capítulo 9](capitulo-09.html){: .btn } [↑ Inicio](index.html){: .btn .btn-primary }
