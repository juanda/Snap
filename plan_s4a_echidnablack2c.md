# Plan to Implement

## Plan: Crear librería **S4A EchidnaBlack2C**

---

## Contexto

Snap! es un entorno de programación basado en bloques. Tiene un mecanismo de extensión que puedes consultar en docs/Extensions.md. EchidnaBlack y EchidnaBlack2C son placas de robótica educativa basadas en Arduino Nano que, mediante firmata, pueden comunicarse con Snap!. El archivo libraries/s4aConn/s4aConn.js contine las primitivas javascript para realizar la comunicación con placas que entienden firmata.

Se necesita crear un archivo de proyecto **Snap!** (`S4A EchidnaBlack2C.xml`) para la placa **EchidnaBlack2C** (variante **black-i2c**).  
Esta placa usa un acelerómetro **I2C (LIS3DH en 0x18)** en lugar del analógico, y tiene un pinout ligeramente diferente a la **Echidna Black original**.

El archivo reutilizará las primitivas JS ya existentes en `libraries/s4aConn/s4aConn.js` (por ejemplo: `digitalWrite`, `pwmWrite`, `servoWrite`, `analogReading`, `digitalReading`, `i2c*`, `tone`, etc.) **sin necesidad de crear nuevo código JS**.

El archivo `S4A Echidna.xml` es un archivo de proyecto **Snap!** para la placa **EchidnaBlack**, un modelo anterior a **EchidnaBlack2C**, pero parecido, por lo que puede usarse como modelo y ejemplo para saber como se definen los bloques y lo que debe contener el nuevo proyecto para funcionar correctamenete.

El Pinout de la nueva placa EchidnaBlack2C viene dado por la siguiente tabla:

| Componente | Pin | Tipo |
|---|---|---|
| LED rojo | D13 | Digital out |
| LED naranja | D12 | Digital out |
| LED verde | D11 | Digital out |
| RGB rojo | D9 | PWM |
| RGB verde | D6 | PWM |
| RGB azul | D5 | PWM |
| Buzzer | D10 | PWM/Digital |
| Botón SR | D2 | Digital in |
| Botón SL | D3 | Digital in |
| Joystick X | A0 | Analog in |
| Joystick Y | A1 | Analog in |
| Luz (LDR) | A3 | Analog in |
| Temperatura | A6 | Analog in |
| Micrófono | A7 | Analog in |
| Acelerómetro | I2C (0x18, LIS3DH) | I2C (A4=SDA, A5=SCL) |
| I/O genéricos | A2, D4, D7, D8 | Digital in/out + servo |
| MakeyMakey | A0,A1,A2,A3,A6,A7,D2,D3 | Digital in |

---

## Diferencias clave vs S4A Echidna (black original)

- **Luz:** A3 (era A5)
- **Acelerómetro:** I2C LIS3DH en 0x18 (era analógico en A2/A3)
- **MkMk pins:** A0,A1,A2,A3,A6,A7,D2,D3 (era A0-A5,D2,D3)
- **Pines servo/digitales extra:** A2,D4,D7,D8 (era D4,D7,D8,A4)
- **No hay A4/A5 analógicos** (usados por I2C)

---

## Bloques a crear

Hay que crear los siguientes bloques, en este mismo orden y respetando los nombres y la traducción, usa S4A Echidna.xml solo para saber cómo se declaran los bloques y cómo debe implementarse S4A EchidnaBlack2C.xml, no copies los nombres, usa los que se especifican a continuación. Los bloque deben estar visibles, con icono 🅴.

1. **set LED %color to %state** (command)
   - Menú color: 🔴 (D13), 🍊 (D12), 🍏 (D11)  
   - Input booleano on/off  
   - Usa: `s4a_digitalWrite(pin, value)`
   - traducción al español: **poner LED %color a %state**

2. **set LED RGB to R: %red G: %green B: %blue** (command)
   - R→D9, G→D6, B→D5  
   - Usa: `s4a_pwmWrite(pin, value)` x3
   - Traducción al español: **poner LED RGB a %rojo G: %verde B: %azul**

3. **green LED %intensity** (command)
   - PWM en D11
   - Usa: `s4a_pwmWrite(pin, value)`  
   - Traducción al español: **LED verde %intensity%**

4. **%switch buzzer** (command) 
   - D10 on/off  
   - Usa: `s4a_digitalWrite(pin, value)` con pin=10
   - Traducción al español: **%switch zumbador**

5. **servo %pin to %direction speed %speed** (command)
   - Menú pin: D4, D7, D8, A2  
   - Menú dirección: clockwise, counter-clockwise  
   - Velocidad 0-100%  
   - Usa: `s4a_servoWrite(pin, value)` con cálculo de velocidad
   - Traducción al español: **servo [PIN] sentido %direction velocidad %speed**

6. **servo %pin to angle %angle** (command) 
   - Menú pin: D4, D7, D8, A2  
   - Ángulo 0-180  
   - Usa: `s4a_servoWrite(pin, value)`
   - Traducción al español: **servo %pin al ángulo %angle**

7. **%switch digital pin %pin** (command)
   - Menú pin: D4(4), D7(7), D8(8), A2(16)  
   - Usa: `s4a_digitalWrite(pin, value)`
   - Traducción al español: **%switch pin digital %pin**

8. **button %button on?** (predicate)
   - Menú: SR (D2), SL (D3)  
   - Usa: `s4a_reportDigitalReading(pin)`
   - Traducción al español: **¿botón %button pulsado?**

9. **MkMk %pin on?** (predicate)
   - Menú: A0,A1,A2,A3,A6,A7,D2,D3  
   - Pines GPIO: 14,15,16,17,20,21,2,3  
   - Usa: `s4a_reportDigitalReading(pin)`
   - Traducción al español: **¿MkMk [PIN] activado?**

10. **digital input %pin on?** (predicate)
   - Menú: D2,D3,D4,D7,D8  
   - Usa: `s4a_reportDigitalReading(pin)`
   - Traducción al español: **¿entrada digital %pin activada?**

11. **read accelerometer %axis** (reporter) 
   - Menú: x, y, z  
   - Usa: I2C para leer LIS3DH (0x18)  
   - Requiere: script con `s4a_i2cwrite`, `s4a_i2cread1/2/3` para configurar y leer registros del LIS3DH
   - Traducción al español: **leer acelerómetro %axis**

12. **read joystick %xy** (reporter)  
   - Menú: x, y  
   - X=A0, Y=A1  
   - Usa: `s4a_reportAnalogReading(pin)` + cálculo -1..1
   - Traducción al español: **leer joystick %xy**

13. **read light sensor** (reporter)
   - A3 (pin analógico 3)  
   - Usa: `s4a_reportAnalogReading(3)
   - Traducción al español: **leer sensor de luz**


14. **read temperature** (reporter)
   - A6 (pin analógico 6)  
   - Usa: `s4a_reportAnalogReading(6)` → `(value * 0.4658) - 50.0`
   - Traducción al español: **leer temperatura**

15. **read microphone** (reporter)  
   - A7 (pin analógico 7)  
   - Usa: `s4a_reportAnalogReading(7)`
   - Traducción al español: **leer micrófono**

16. **read analog input %pin** (reporter)
   - Menú: A0,A1,A2,A3,A6,A7  
   - Usa: `s4a_reportAnalogReading(pin)`
   - Traducción al español: **leer entrada analógica %pin**

---

## Bloques helper (ocultos, heredados de S4A Connector genérico)

Se copiarán del `S4A Echidna.xml` original:

- board connected ?

---

## Archivo a crear

- `/home/juanda/Proyectos/Echidna/desarrollo/Snap/S4A EchidnaBlack2C.xml`

---

## Estructura del XML

Se tomará como base `S4A Echidna.xml` cob `<project name="S4A EchidnaBlack2C">`
---

## Implementación del acelerómetro LIS3DH

El bloque de acelerómetro necesita:

1. Configurar I2C con el LIS3DH (dirección 0x18)
2. Escribir al registro `CTRL_REG1 (0x20)` el valor `0x57` para activar 100Hz, todos los ejes
3. Leer registros de salida:
   - `OUT_X_L (0x28)` / `OUT_X_H (0x29)`
   - `OUT_Y_L (0x2A)` / `OUT_Y_H (0x2B)`
   - `OUT_Z_L (0x2C)` / `OUT_Z_H (0x2D)`
4. Combinar bytes y convertir a valor con signo (complemento a 2, 16 bits)

Se implementará como un script Snap! que use las primitivas I2C existentes (`s4a_i2csend`, `s4a_i2cwrite`, `s4a_i2cread1/2/3`).

---

## Verificación

1. Abrir el archivo en Snap! (`snap.html`)
2. Verificar que la categoría **S4A Connector** aparece con sus bloques
3. Verificar que los botones **Connect / Disconnect** aparecen
4. Conectar una EchidnaBlack2C y probar:
   - LEDs, buzzer, servo
   - Lectura de sensores (joystick, luz, temperatura, sonido)
   - Acelerómetro I2C
   - MakeyMakey
