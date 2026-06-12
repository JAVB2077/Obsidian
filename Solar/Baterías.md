==cuantas baterias 18650 seria lo recomendable para mantener vivo el sistemas previendo que no haya sol por unos 2 o 3 dias y un uso considerable, habria cambio en algun componente?==
==Esta es la lista de materiales definitiva para la versión **"Alto Rendimiento / Carga Solar Eficiente"**.==

Con estos componentes, pasarás de aprovechar el 70% de la energía solar (sistema estándar) a aprovechar cerca del **95%**, y reducirás el consumo en reposo drásticamente.

### 1. Sistema de Carga Solar (MPPT)

_El corazón de la optimización._

- **Módulo Cargador MPPT: CN3791 (Versión para 6V).**
    
    - _La Optimización:_ [[Reemplaza al TP4056]]. Rastrea el punto de máxima potencia del panel, cargando la batería incluso en días nublados donde el TP4056 se cortaría.
        
- **Panel Solar: 6V - 6W (o superior) Monocristalino.**
    
    - _La Optimización:_ Al ser "Monocristalino" (color negro uniforme) capta mejor la radiación difusa (nubes) que los policristalinos (azules). El aumento de potencia (6W) asegura carga rápida en ventanas cortas de sol.
        
- **Batería: 18650 de Marca (LG HG2, Samsung 25R o Sony VTC).**
    
    - _La Optimización:_ Tienen menor resistencia interna que las genéricas. Aceptan la carga más rápido y se calientan menos. Necesitarás su portabaterías (Holder).
        

### 2. Regulación de Energía (Ultra Low Power)

_El secreto para que la batería dure semanas, no días._

- **Regulador de Voltaje LDO: Holtek HT7333-A (o MCP1700-3302E).**
    
    - _La Optimización:_ Este componente es **crítico**. Reemplaza al regulador que trae integrado la placa ESP32.
        
    - _Por qué:_ El regulador del ESP32 gasta energía aunque no hagas nada. El HT7333 consume solo unos pocos _microamperios_. Alimentarás el pin `3.3V` del ESP32 con esto, saltándote el pin `VIN`.
        
- **Capacitores Electrolíticos:** 10µF y 100µF (16V o más).
    
    - _Para qué:_ Son necesarios para acompañar al regulador HT7333 y estabilizar la energía, evitando que el ESP32 se reinicie por picos de consumo.
        

### 3. Control y Sensores

- **Microcontrolador: ESP32 DevKit V1.**
    
    - Seguimos usando este por su capacidad de Deep Sleep, pero lo alimentaremos de forma especial (ver punto 2).
        
- **Sensor Infrarrojo: Módulo TCRT5000.**
    
    - _Modificación recomendada:_ Si te animas, desuélda el LED de "Power" (suele ser rojo) que trae el módulo. Ese LED gasta energía inútilmente las 24 horas.
        

### 4. Etapa de Potencia (El Switch)

- **Transistor MOSFET: IRLZ44N (Canal N - Logic Level).**
    
    - Esencial que sea la serie **"L"** (Logic) para que abra completamente con los 3.3V del ESP32.
        
- **Resistencias:**
    
    - **10k Ohm:** Para mantener el MOSFET cerrado (apagado) por defecto.
        
    - **330 Ohm:** Para proteger el pin del ESP32 que va al MOSFET.
        

### 5. Periféricos y Elevación

- **Convertidor DC-DC Step-Up: MT3608.**
    
    - Eleva los 3.7V de la batería a los 5V necesarios para la pantalla y el lector. Al conectarlo _después_ del MOSFET, su consumo es **0** cuando el sistema duerme.
        
- **Lector QR: GM65 o GM66.**
    
- **Pantalla:** LCD 16x2 I2C o OLED 0.96".
    

---

### Resumen de Compra (Lista rápida para mostrador/tienda):

1. **Panel Solar:** 6V, 6 Watts, Monocristalino.
    
2. **Cargador:** Módulo CN3791 (verifica que diga "6V" en la placa).
    
3. **Batería:** 1x Célula 18650 (Samsung/LG) + Portabaterías.
    
4. **Regulador LDO:** Chip HT7333-A (TO-92) + 2 capacitores electrolíticos de 10µF.
    
5. **Microcontrolador:** ESP32 DevKit V1.
    
6. **Sensor:** Módulo TCRT5000.
    
7. **MOSFET:** Transistor IRLZ44N.
    
8. **Elevador:** Módulo MT3608.
    
9. **Lector/Pantalla:** Tu lector QR y pantalla LCD habituales.
    
10. **Varios:** Resistencias (10k, 330Ω), cables, placa perforada para soldar (PCB) y caja estanca.