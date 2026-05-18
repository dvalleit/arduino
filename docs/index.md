---
layout: home
title: Inicio
nav_order: 1
---

# Aprende Arduino desde Cero
{: .fs-9 }

Una guía completa en español para aprender Arduino paso a paso, con ejercicios prácticos y proyectos reales — desde encender tu primer LED hasta construir una estación meteorológica.
{: .fs-6 .fw-300 }

[Comenzar con el Capítulo 1](capitulo-01.html){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Ver en GitHub](https://github.com/dvalleit/arduino){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## ¿Qué vas a aprender?

Esta guía está organizada como un libro, capítulo por capítulo. No necesitas experiencia previa en electrónica ni programación. Cada capítulo incluye teoría, ejercicios con código completo y desafíos para practicar.

| Capítulo | Tema | Habilidades |
|----------|------|-------------|
| [1 — Introducción](capitulo-01.html) | ¿Qué es Arduino? Instalación del IDE | Conocer el hardware y el entorno |
| [2 — El Blink](capitulo-02.html) | Tu primer programa | `pinMode`, `digitalWrite`, `delay` |
| [3 — Entradas Digitales](capitulo-03.html) | Botones y pull-up/pull-down | `digitalRead`, debounce |
| [4 — Señales Analógicas](capitulo-04.html) | Potenciómetros, LDR, NTC | `analogRead`, sensores |
| [5 — PWM](capitulo-05.html) | Control de brillo y velocidad | `analogWrite`, LED RGB |
| [6 — Comunicación Serial](capitulo-06.html) | Hablar con tu computadora | `Serial.print`, `Serial.read` |
| [7 — Motores y Servos](capitulo-07.html) | Servo y motor DC | Librería `Servo`, L298N |
| [8 — Pantalla LCD](capitulo-08.html) | Mostrar información | Librería `LiquidCrystal` |
| [9 — Sensores Avanzados](capitulo-09.html) | HC-SR04, DHT11 | Librerías externas |
| [10 — Proyecto Final](capitulo-10.html) | Estación Meteorológica | Todo lo aprendido |

---

## Lo que necesitas

### Hardware
- **Arduino Uno** (o Nano / Mega — compatible con todos los ejercicios)
- Protoboard y cables dupont
- LEDs, resistencias (220Ω, 10kΩ)
- Potenciómetro 10kΩ
- Pulsadores / botones
- Sensor LDR, NTC o LM35
- Servo SG90
- Pantalla LCD 16×2 con módulo I2C
- Sensor HC-SR04
- Sensor DHT11 o DHT22

> Muchos ejercicios de los primeros capítulos sólo necesitan el Arduino y el LED integrado.

### Software
- [Arduino IDE 2.x](https://www.arduino.cc/en/software) — gratuito para Windows, macOS y Linux
- Cable USB para conectar el Arduino a la computadora

---

## ¿Cómo usar esta guía?

1. **Lee la teoría** de cada capítulo — son explicaciones cortas y directas.
2. **Arma el circuito** siguiendo el diagrama de conexiones.
3. **Copia o escribe el código** en el IDE y súbelo al Arduino.
4. **Experimenta** cambiando valores y observando qué pasa.
5. **Completa el desafío** al final de cada capítulo para afianzar lo aprendido.

---

## Recursos adicionales

- [Referencia oficial de Arduino](https://www.arduino.cc/reference/en/) (en inglés)
- [Foro de Arduino en español](https://forum.arduino.cc/c/international/espanol/69)
- [Fritzing](https://fritzing.org/) — herramienta para diagramas de circuitos
