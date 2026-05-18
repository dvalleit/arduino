---
layout: default
title: "Capítulo 6 — Comunicación Serial"
nav_order: 7
---

# Capítulo 6 — Comunicación Serial
{: .no_toc }

Aprende a enviar y recibir datos entre el Arduino y tu computadora para monitorear, controlar y depurar tus proyectos.
{: .fs-6 .fw-300 }

## Contenido
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 6.1 ¿Qué es la comunicación Serial?

La comunicación serial (UART) transmite datos **bit a bit** a través de dos cables:
- **TX** (pin 1): transmite datos del Arduino a la computadora.
- **RX** (pin 0): recibe datos de la computadora al Arduino.

El cable USB convierte la señal serial al protocolo USB que entiende tu computadora.

> **Cuidado:** Nunca conectes nada a los pines 0 y 1 cuando estés usando el Serial — lo puedes interrumpir.

### Velocidades de comunicación (baud rate)

La velocidad debe ser **la misma** en el Arduino y en el Monitor Serial:

| Velocidad | Uso típico |
|-----------|-----------|
| 9600 | Proyectos simples (por defecto) |
| 115200 | Proyectos avanzados, más rápido |
| 57600 | Comunicación con módulos Bluetooth |

---

## 6.2 Funciones de Serial

```cpp
Serial.begin(velocidad);     // Inicia la comunicación
Serial.print("texto");       // Envía sin salto de línea
Serial.println("texto");     // Envía con salto de línea (\n)
Serial.print(numero);        // Envía un número
Serial.print(numero, DEC);   // Decimal (por defecto)
Serial.print(numero, HEX);   // Hexadecimal
Serial.print(numero, BIN);   // Binario
Serial.print(valor, 2);      // Float con 2 decimales

// Recibir datos
int disponibles = Serial.available(); // Bytes esperando ser leídos
char c = Serial.read();              // Lee 1 byte
String s = Serial.readString();       // Lee hasta timeout
int n = Serial.parseInt();           // Lee un entero
float f = Serial.parseFloat();       // Lee un float
```

---

## 6.3 Ejercicio 1 — Monitor de sensores

Muestra los valores de múltiples pines en el Serial con un formato legible.

```cpp
// Monitor de sensores en tiempo real
const int PIN_POT = A0;
const int PIN_LDR = A1;

void setup() {
  Serial.begin(9600);
  // Encabezado de tabla
  Serial.println("Tiempo(ms)\tPotenciometro\tLuz");
}

void loop() {
  unsigned long t   = millis();
  int pot           = analogRead(PIN_POT);
  int luz           = analogRead(PIN_LDR);

  // Formato de tabla con tabulaciones
  Serial.print(t);
  Serial.print("\t");
  Serial.print(pot);
  Serial.print("\t");
  Serial.println(luz);

  delay(500);
}
```

> Abre el **Serial Plotter** (Herramientas → Serial Plotter) para ver los datos en un gráfico en tiempo real.

---

## 6.4 Ejercicio 2 — Echo (repetir lo que escribes)

El Arduino recibe lo que escribes en el Monitor Serial y lo devuelve.

```cpp
// Echo: repite lo que el usuario envía
void setup() {
  Serial.begin(9600);
  Serial.println("Escribe algo y presiona Enter:");
}

void loop() {
  // Si hay datos disponibles para leer
  if (Serial.available() > 0) {
    char caracter = Serial.read();
    Serial.print("Recibido: ");
    Serial.println(caracter);
  }
}
```

**Versión con string completa:**

```cpp
void setup() {
  Serial.begin(9600);
  Serial.println("Eco de mensajes. Escribe y presiona Enter:");
}

void loop() {
  if (Serial.available() > 0) {
    String mensaje = Serial.readStringUntil('\n'); // Lee hasta Enter
    mensaje.trim(); // Elimina espacios y \r

    Serial.print("Eco → ");
    Serial.println(mensaje);
  }
}
```

---

## 6.5 Ejercicio 3 — Control por comandos de texto

Controla LEDs y el buzzer enviando comandos desde el Monitor Serial.

### Materiales
- 2 LEDs + resistencias 220Ω
- 1 buzzer (opcional)
- Protoboard y cables

### Comandos disponibles
- `ROJO ON` / `ROJO OFF` — controla el LED rojo
- `VERDE ON` / `VERDE OFF` — controla el LED verde
- `BEEP` — hace sonar el buzzer
- `ESTADO` — muestra el estado de los LEDs

### Código

```cpp
// Control por comandos de texto vía Serial
const int LED_ROJO  = 9;
const int LED_VERDE = 10;
const int BUZZER    = 8;

bool estadoRojo  = false;
bool estadoVerde = false;

void procesarComando(String cmd) {
  cmd.trim();
  cmd.toUpperCase();

  if (cmd == "ROJO ON") {
    estadoRojo = true;
    digitalWrite(LED_ROJO, HIGH);
    Serial.println("LED ROJO encendido");

  } else if (cmd == "ROJO OFF") {
    estadoRojo = false;
    digitalWrite(LED_ROJO, LOW);
    Serial.println("LED ROJO apagado");

  } else if (cmd == "VERDE ON") {
    estadoVerde = true;
    digitalWrite(LED_VERDE, HIGH);
    Serial.println("LED VERDE encendido");

  } else if (cmd == "VERDE OFF") {
    estadoVerde = false;
    digitalWrite(LED_VERDE, LOW);
    Serial.println("LED VERDE apagado");

  } else if (cmd == "BEEP") {
    tone(BUZZER, 1000, 200); // 1000 Hz durante 200ms
    Serial.println("¡Beep!");

  } else if (cmd == "ESTADO") {
    Serial.print("Rojo: ");
    Serial.println(estadoRojo ? "ON" : "OFF");
    Serial.print("Verde: ");
    Serial.println(estadoVerde ? "ON" : "OFF");

  } else {
    Serial.print("Comando no reconocido: '");
    Serial.print(cmd);
    Serial.println("'");
    Serial.println("Comandos: ROJO ON/OFF, VERDE ON/OFF, BEEP, ESTADO");
  }
}

void setup() {
  pinMode(LED_ROJO, OUTPUT);
  pinMode(LED_VERDE, OUTPUT);
  pinMode(BUZZER, OUTPUT);
  Serial.begin(9600);
  Serial.println("=== Control por Serial ===");
  Serial.println("Comandos: ROJO ON/OFF, VERDE ON/OFF, BEEP, ESTADO");
}

void loop() {
  if (Serial.available() > 0) {
    String comando = Serial.readStringUntil('\n');
    procesarComando(comando);
  }
}
```

> **Tip:** En el Monitor Serial, asegúrate de seleccionar **"Nueva línea"** o **"Ambos NL y CR"** en el menú desplegable de abajo a la derecha.

---

## 6.6 Ejercicio 4 — Calculadora Serial

Recibe dos números y una operación, devuelve el resultado.

```cpp
// Calculadora por Serial: "10 + 5", "20 * 3", "100 / 4"
void setup() {
  Serial.begin(9600);
  Serial.println("=== Calculadora Arduino ===");
  Serial.println("Formato: numero operador numero");
  Serial.println("Ejemplo: 10 + 5");
}

void loop() {
  if (Serial.available() > 0) {
    float a = Serial.parseFloat(); // Lee primer número
    char op  = 0;

    // Espera el operador
    while (Serial.available() == 0);
    // Salta espacios
    while (Serial.peek() == ' ') Serial.read();
    op = Serial.read();

    float b = Serial.parseFloat(); // Lee segundo número

    Serial.print(a, 2);
    Serial.print(" ");
    Serial.print(op);
    Serial.print(" ");
    Serial.print(b, 2);
    Serial.print(" = ");

    switch (op) {
      case '+': Serial.println(a + b, 4); break;
      case '-': Serial.println(a - b, 4); break;
      case '*': Serial.println(a * b, 4); break;
      case '/':
        if (b != 0) Serial.println(a / b, 4);
        else        Serial.println("Error: división por cero");
        break;
      default:
        Serial.print("Operador no válido: ");
        Serial.println(op);
    }
  }
}
```

---

## 6.7 Depuración con Serial

El Serial es tu mejor amigo para encontrar errores. Técnicas útiles:

```cpp
// Imprimir el valor de una variable en un punto específico
Serial.print("valor en este punto: ");
Serial.println(miVariable);

// Marcar secciones del código
Serial.println("[DEBUG] Entrando al if");

// Medir tiempo de ejecución
unsigned long inicio = micros();
// ... código a medir ...
unsigned long fin = micros();
Serial.print("Tiempo: ");
Serial.print(fin - inicio);
Serial.println(" microsegundos");
```

---

## Resumen del capítulo

| Función | Descripción |
|---------|-------------|
| `Serial.begin(9600)` | Inicia la comunicación a 9600 baud |
| `Serial.print()` / `println()` | Envía datos (sin/con salto de línea) |
| `Serial.available()` | Bytes pendientes de leer |
| `Serial.read()` | Lee 1 byte |
| `Serial.readStringUntil('\n')` | Lee una línea completa |
| `Serial.parseInt()` | Lee un entero |

---

## Desafío

Crea un **logger de temperatura**: cada segundo, lee un sensor analógico y envía la temperatura por Serial en formato CSV (`tiempo_ms,temperatura_celsius`). Luego copia los datos al Bloc de notas y guárdalos en un archivo `.csv` — puedes abrirlo con Excel o Google Sheets para ver la gráfica.

---

[← Capítulo 5](capitulo-05.html){: .btn } [Capítulo 7 — Motores y Servos →](capitulo-07.html){: .btn .btn-primary }
