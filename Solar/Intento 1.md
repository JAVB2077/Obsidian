Esta es una excelente aplicación para un **sistema de bajo consumo (Low Power)**. Dado que ya tienes experiencia con el ESP32 y sensores (como vimos en tus proyectos anteriores con LCDs), la clave aquí no es "apagar" el ESP32 totalmente, sino usar su modo de **Deep Sleep (Sueño Profundo)**.

El reto principal que planteas es que un sensor infrarrojo estándar consume energía constantemente. Para optimizar el voltaje del panel solar, usaremos una arquitectura de **"Wake-up Interrupt" (Interrupción por Despertador)**.

Aquí tienes la propuesta técnica para lograr que el sistema solo consuma energía real cuando se acerque el teléfono.

---

### 1. El Sensor Correcto: TCRT5000 (Reflectivo)

Para detectar un teléfono a "muy corta distancia" (casi tocando), el sensor ultrasónico no sirve (tienen un punto ciego cercano) y el PIR detecta movimiento corporal, no objetos estáticos.

Necesitas un **Módulo Sensor Infrarrojo de Obstáculos (basado en TCRT5000 o similar)**.

- **Por qué:** Emite una luz IR; si el teléfono está cerca (1cm - 5cm), la luz rebota y el receptor la capta, enviando un `LOW` (o `HIGH`) digital.
    
- **El Truco de Energía:** Este sensor debe estar siempre encendido, pero consume poco (aprox 10-20mA). Si necesitas optimizar al extremo, puedes quitarle el LED indicador de "Power" con un soldador para ahorrar batería.
    

### 2. La Arquitectura del Circuito

Para que el sistema sea eficiente con el panel solar, dividiremos el circuito en dos bloques de energía:

1. **Bloque "Siempre Alerta" (Bajo Consumo):**
    
    - Batería Li-Ion (cargada por el panel).
        
    - Sensor IR (alimentado directo).
        
    - ESP32 (en modo Deep Sleep, consumiendo solo ~10 µA).
        
2. **Bloque "Carga Pesada" (Switcheable):**
    
    - Lector QR / Pantalla LCD / Pantalla OLED.
        
    - Estos **NO** se conectan directo a la batería. Se conectan a través de un **Transistor (MOSFET o BJT)** controlado por el ESP32. Así, cuando el ESP32 duerme, estos dispositivos están **físicamente apagados** (0 consumo).
        

---

### 3. Diagrama de Conexiones

Aquí te muestro cómo conectar todo para lograr la interrupción externa (`ext0` o `ext1` wakeup).

#### Descripción de Pines:

1. **Sensor IR (TCRT5000):**
    
    - `VCC` -> 3.3V (Salida del regulador o batería).
        
    - `GND` -> GND.
        
    - `OUT` -> **GPIO 33** (o cualquier pin RTC del ESP32). _Nota: Los pines RTC son necesarios para despertar al ESP32 del Deep Sleep._
        
2. **Periféricos (Lector QR / Pantalla):**
    
    - `VCC` de la Pantalla -> **Drain** de un MOSFET (Canal P) o Colector de un transistor PNP (como un 2N2222 para cargas pequeñas, o un IRF9540 para cargas grandes). _O más fácil: Un módulo Relé o MOSFET prefabricado._
        
    - `GND` -> GND común.
        
    - `Control` -> **GPIO 25** (Cualquier pin de salida).
        
3. **Sistema Solar:**
    
    - Panel Solar -> Módulo TP4056 (Cargador) -> Batería 18650 -> ESP32.
        

---

### 4. Lógica de Funcionamiento (Algoritmo)

El flujo para tu código en Arduino IDE / PlatformIO sería:

1. **Estado Inicial (Deep Sleep):** El ESP32 está "dormido". Solo el procesador de ultra bajo consumo (ULP) está vigilando el **GPIO 33**.
    
2. **Evento:** Alguien acerca el teléfono al sensor IR.
    
3. **Disparo:** El pin del sensor cambia de estado (ej. de HIGH a LOW).
    
4. **Despertar:** El ESP32 recibe la señal, se despierta y ejecuta el `setup()`.
    
5. **Activación de Periféricos:**
    
    - El ESP32 pone el **GPIO 25 en HIGH**.
        
    - Esto activa el MOSFET/Transistor.
        
    - La Pantalla/Lector QR reciben energía y se encienden.
        
6. **Acción:** El ESP32 lee el código QR o muestra datos en la pantalla.
    
7. **Temporizador:** Si el teléfono se retira o pasan 30 segundos, el ESP32 apaga el GPIO 25 (cortando la energía de la pantalla) y vuelve a entrar en `Deep Sleep`.
    

### 5. Ejemplo de Código (Fragmento Clave)

Como trabajas con ESP32, aquí tienes la estructura para la interrupción:

C++

```
#define PIN_SENSOR_IR 33  // Debe ser un pin RTC (0, 2, 4, 12-15, 25-27, 32-39)
#define PIN_MOSFET_PERIFERICOS 25

void setup() {
  // 1. Configurar control de energía
  pinMode(PIN_MOSFET_PERIFERICOS, OUTPUT);
  digitalWrite(PIN_MOSFET_PERIFERICOS, HIGH); // Encendemos la pantalla/QR

  Serial.begin(115200);
  Serial.println("Sistema Despierto: Escaneando...");

  // --- AQUÍ VA TU CÓDIGO DEL PROYECTO (Leer QR, Mostrar en LCD) ---
  delay(5000); // Simulamos tiempo de operación
  // ----------------------------------------------------------------

  Serial.println("Entrando en modo sueño...");
  
  // 2. Apagar periféricos antes de dormir
  digitalWrite(PIN_MOSFET_PERIFERICOS, LOW); 

  // 3. Configurar el despertador (Wake up source)
  // Suponiendo que el sensor IR da LOW cuando detecta objeto:
  esp_sleep_enable_ext0_wakeup((gpio_num_t)PIN_SENSOR_IR, 0); 

  // 4. A dormir
  esp_deep_sleep_start();
}

void loop() {
  // En Deep Sleep, el loop nunca se ejecuta. Todo ocurre en setup.
}
```

### Consideración Importante: "Luz Solar e Infrarrojos"

Hay un detalle físico crítico: **La luz del sol es una fuente masiva de radiación infrarroja.**

Si tu panel solar y tu dispositivo están al aire libre bajo el sol directo, el sensor IR podría "cegarse" o dar falsos positivos (creer que hay un teléfono cuando solo es el sol pegando fuerte).

- **Solución:** Debes montar el sensor IR dentro de una pequeña carcasa o tubo oscuro (como un pequeño túnel de 2cm) para que solo reaccione cuando algo tape físicamente la entrada de luz y rebote su propia señal, protegiéndolo de la luz ambiente directa.
    

¿Te gustaría que profundicemos en cómo conectar el transistor MOSFET para que la pantalla no te drene la batería mientras el sistema duerme?

------------
Aquí tienes la lista de materiales detallada, pensada específicamente para **maximizar la eficiencia energética** y soportar la carga solar. He seleccionado componentes comunes que son fáciles de conseguir en tiendas de electrónica (como Steren, Mercado Libre o tiendas locales de robótica en México).

### 1. El Cerebro y el Sensor (Low Power)

Estos son los únicos componentes que estarán conectados directamente a la batería todo el tiempo.

- **Microcontrolador: ESP32 DevKit V1 (30 pines o 38 pines).**
    
    - _Por qué:_ Tiene el modo "Deep Sleep" necesario y pines capacitivos/RTC para despertar.
        
- **Sensor Infrarrojo de Obstáculos: Módulo TCRT5000.**
    
    - _Por qué:_ Es barato, digital y funciona perfecto para detectar proximidad (1cm a 5cm).
        
    - _Ojo:_ Compra el **módulo** (el que trae el chip comparador LM393 y el potenciómetro azul), no solo el LED suelto, para que sea fácil ajustar la sensibilidad.
        

### 2. Sistema de Energía Solar (Carga y Batería)

Este bloque se encarga de recibir la energía del sol y guardarla.

- **Panel Solar: 6V (Voltios) a 3W o 5W.**
    
    - _Nota:_ No uses uno de 1W o menos; tardará demasiado en cargar. Con 3W-5W aseguras carga incluso en días nublados.
        
- **Módulo de Carga: TP4056 (versión con protección).**
    
    - _Importante:_ Busca el que tiene **protección de descarga** (tiene 4 pines de salida: OUT+, OUT-, B+, B-). Esto evitará que mates tu batería si se descarga demasiado.
        
- **Batería: 18650 Li-Ion (3.7V).**
    
    - _Recomendación:_ Una celda de **2500mAh a 3000mAh**. Si el sistema va a estar en un lugar muy transitado, considera poner dos baterías en paralelo (usando un portabaterías doble paralelo).
        
- **Portabaterías:** Para 18650 (simple o doble, según decidas).
    

### 3. El "Switch" de Ahorro (Corte de Energía)

Este es el componente **crítico** para tu proyecto. Es el que cortará la corriente a la pantalla y al lector QR cuando no se usen.

- **Transistor MOSFET de Nivel Lógico (Canal N): IRLZ44N.**
    
    - _Alternativa:_ Si no encuentras ese, un módulo **MOSFET IRF520** pre-armado servirá, aunque el IRLZ44N es más eficiente para 3.3V.
        
    - _Función:_ Actuará como un interruptor controlado por el ESP32 para apagar "físicamente" los periféricos tragones.
        

### 4. Periféricos (Los "Consumidores")

Estos se conectarán al MOSFET, no directo a la batería.

- **Lector de Código QR: Módulo GM66 o GM65.**
    
    - _Por qué:_ Son los estándares para Arduino/ESP32. Leen códigos 1D y 2D (QR) y se comunican por Serial (RX/TX).
        
- **Pantalla (Opciones):**
    
    - _Opción A (Bajo consumo):_ **OLED 0.96 pulgadas (I2C)**. Consume muy poco, pero es pequeña.
        
    - _Opción B (Tu conocida):_ **LCD 16x2 con interfaz I2C**. Ya la has usado antes. Es más grande y visible, pero consume más luz de fondo (backlight).
        

### 5. Regulación de Voltaje (Muy Importante)

El lector QR (GM66) y la pantalla LCD 16x2 suelen necesitar **5V** para funcionar bien. Tu batería solo da 3.7V. Necesitas elevar el voltaje **solo** para esa parte del circuito.

- **Convertidor DC-DC Step-Up (Elevador): Módulo MT3608.**
    
    - _Función:_ Tomará los 3.7V de la batería y los convertirá a 5V estables para alimentar el Lector QR y la Pantalla.
        
    - _Truco de ahorro:_ Conecta la entrada del MT3608 al MOSFET, así el elevador de voltaje TAMBIÉN se apaga cuando el sistema duerme.
        

### 6. Materiales Varios

- **Protoboard** (para pruebas).
    
- **Cables Jumper** (Macho-Macho, Macho-Hembra).
    
- **Resistencias:** Un kit variado, necesitarás principalmente de **10k Ohms** (para asegurar que el MOSFET se apague - pulldown) y **330 Ohms**.
    
- **Caja estanca o Gabinete:** Indispensable para proteger la electrónica de la intemperie si va con panel solar.
    

### Resumen de compra rápida:

1. ESP32 DevKit V1
    
2. Sensor Infrarrojo TCRT5000 (Módulo)
    
3. Panel Solar 6V 5W
    
4. Cargador TP4056 (con protección)
    
5. Batería 18650 + Portabaterías
    
6. MOSFET IRLZ44N (o módulo IRF520)
    
7. Módulo Step-Up MT3608
    
8. Lector QR GM65/GM66
    
9. Pantalla LCD 16x2 I2C (o OLED)
    

¿Quieres que te haga un pequeño esquema de cómo conectar el **MT3608** y el **MOSFET** juntos? Es la parte donde más gente se confunde y si se conecta mal, el ahorro de energía no funciona.