¡Excelente decisión! Pasar a un software de diseño esquemático (como EasyEDA, Proteus, KiCad o Fritzing) es el paso natural para un proyecto de nivel de 8vo semestre. Te permitirá tener un plano profesional, detectar errores de ruteo y, si en el futuro quieres mandar a fabricar la placa (PCB), ya tendrás el 90% del trabajo hecho.

Aquí tienes la **Guía Definitiva de Conexión**, estructurada lógicamente para que la vayas dibujando bloque por bloque en tu programa.

---

### BLOQUE 1: Gestión de Energía Solar (El Tanque Principal)

1.  **Panel Solar:**
    * Positivo $\rightarrow$ `IN+` (o `PV+`) del módulo **CN3791**.
    * Negativo $\rightarrow$ `IN-` (o `PV-`) del módulo **CN3791**.
2.  **Baterías (Banco de 3x 18650 en Paralelo):**
    * Todos los Positivos unidos $\rightarrow$ `BAT+` del **CN3791**.
    * Todos los Negativos unidos $\rightarrow$ `BAT-` del **CN3791**.
    * *Nota para el esquemático:* A este `BAT-` llámalo **"GND_COMUN"** (o usa el símbolo de tierra principal). A este `BAT+` llámalo **"V_BAT"** (aprox 3.7V - 4.2V).

---

### BLOQUE 2: El "Mundo Siempre Encendido" (Ultra Low Power)


1.  **Regulador LDO (HT7333):**
    * **Pata 1 (GND):** Conectar a **GND_COMUN**.
    * **Pata 2 (VIN):** Conectar a **V_BAT** (Positivo de las baterías).
    * **Pata 3 (VOUT):** Conectar al pin **3V3** del ESP32.
2.  **Capacitores de Filtro (CRÍTICO para el esquemático):**
    * Capacitor de 10µF: Entre **VIN** y **GND_COMUN** del HT7333.
    * Capacitor de 100µF: Entre **VOUT** y **GND_COMUN** del HT7333.
3.  **El Cerebro (ESP32):**
    * Pin **GND** del ESP32 $\rightarrow$ **GND_COMUN**.
4.  **El Vigilante (Sensor IR TCRT5000):**
    * Pin **VCC** $\rightarrow$ Pin **3V3** del ESP32.
    * Pin **GND** $\rightarrow$ **GND_COMUN**.
    * Pin **OUT** (o D0) $\rightarrow$ Pin **GPIO 33** del ESP32.

---

### BLOQUE 3: La "Puerta" de Potencia (El Switch MOSFET)
*Aquí creamos la tierra separada que se apagará para ahorrar energía.*

1.  **MOSFET IRLZ44N:**
    * **Pata 3 (Source):** Conectar a **GND_COMUN** (Tierra principal).
    * **Pata 1 (Gate):** Conectar a una resistencia de **330 $\Omega$**. El otro lado de la resistencia va al pin **GPIO 25** del ESP32.
    * **Resistencia Pull-down:** Conectar una resistencia de **10k $\Omega$** entre la **Pata 1 (Gate)** y la **Pata 3 (Source)**.
    * **Pata 2 (Drain):** A partir de aquí, nombra a esta red como **"GND_SWITCH"** (Tierra Conmutada).

---

### BLOQUE 4: Elevación y Reducción (Los Módulos DC-DC)
*Estos módulos y sus periféricos solo vivirán cuando el ESP32 cierre la puerta.*

1.  **Elevador (XL6009) - Ajustado a 12V:**
    * `IN+` $\rightarrow$ Conectar a **V_BAT** (Positivo directo de las baterías).
    * `IN-` $\rightarrow$ Conectar a **GND_SWITCH** (La Pata 2 de tu MOSFET).
    * *Salida:* Nombrar `OUT+` como **"12V_RED"** y `OUT-` será nuestra tierra para esta zona, conéctala a **GND_SWITCH**.
2.  **Reductor (Mini-360 / MP1584EN) - Ajustado a 5V:**
    * `IN+` $\rightarrow$ Conectar a **"12V_RED"** (`OUT+` del XL6009).
    * `IN-` $\rightarrow$ Conectar a **GND_SWITCH**.
    * *Salida:* Nombrar `OUT+` como **"5V_RED"** y `OUT-` a **GND_SWITCH**.

---

### BLOQUE 5: Los Trabajadores Pesados (Periféricos)

**A. Monedero Electrónico (CH-926 o similar - 12V):**
1.  **Cable Rojo (DC+):** Conectar a **"12V_RED"**.
2.  **Cable Negro (GND):** Conectar a **GND_SWITCH**.
3.  **Aislamiento de Señal (Optoacoplador PC817):**
    * **Pata 1 (Ánodo):** Conectar a resistencia de **1k $\Omega$**. El otro lado de la resistencia va al **Cable Blanco (Coin)** del monedero.
    * **Pata 2 (Cátodo):** Conectar a **GND_SWITCH** (Tierra del monedero).
    * **Pata 4 (Colector):** Conectar al pin **GPIO 27** del ESP32.
    * **Pata 3 (Emisor):** Conectar a **GND_COMUN** (Tierra del ESP32).

**B. Pantalla LCD I2C (5V):**
1.  **VCC:** Conectar a **"5V_RED"**.
2.  **GND:** Conectar a **GND_SWITCH**.
3.  **SDA:** Conectar al pin **GPIO 21** del ESP32.
4.  **SCL:** Conectar al pin **GPIO 22** del ESP32.

**C. Lector QR (GM65 - 5V):**
1.  **VCC:** Conectar a **"5V_RED"**.
2.  **GND:** Conectar a **GND_SWITCH**.
3.  **TX:** Conectar al pin **GPIO 16** (RX2) del ESP32.
4.  **RX:** Conectar al pin **GPIO 17** (TX2) del ESP32.

---

### Resumen de Pines del ESP32 para tus Etiquetas (Labels) en el Esquemático:

| Pin ESP32 | Nombre de Red / Etiqueta | Función |
| :--- | :--- | :--- |
| **3V3** | VCC_3V3 | Entrada de energía desde el HT7333. |
| **GND** | GND_COMUN | Tierra principal del sistema. |
| **GPIO 33** | IR_WAKEUP | Señal del sensor infrarrojo para despertar. |
| **GPIO 25** | PWR_EN | Señal (High) para activar el MOSFET. |
| **GPIO 27** | COIN_SIG | Señal aislada del monedero (Lectura de moneda). |
| **GPIO 21** | I2C_SDA | Datos para Pantalla LCD. |
| **GPIO 22** | I2C_SCL | Reloj para Pantalla LCD. |
| **GPIO 16** | QR_RX | Recepción Serial desde el lector QR. |
| **GPIO 17** | QR_TX | Transmisión Serial hacia el lector QR. |

Con esta estructura, puedes ir arrastrando los componentes en tu software y conectarlos usando etiquetas (Net Labels) para que el diagrama te quede limpio y sin cruces de cables innecesarios. ¿Qué software vas a utilizar para dibujarlo?