# 🌐 Práctica Avanzada: Red Zigbee Multi-Nodo con Direccionamiento IEEE y NVS

En esta práctica evolucionaremos nuestra red Zigbee básica a un sistema de control centralizado. Aprenderemos a manejar múltiples dispositivos finales (Focos) desde un solo Coordinador utilizando comandos por Consola Serial y direcciones físicas (IEEE Address). Además, implementaremos memoria no volátil (NVS) para que la red sobreviva a cortes de energía.

---

## 👨‍🏫 Datos de la Asignatura
* **Docente:** MGTI. Saul Isai Soto Ortiz


---

## 🛠️ 1. Lista de Materiales (Por Equipo)
* **3x Tarjetas de desarrollo ESP32-C6.** (1 Coordinador/Consola, 2 Actuadores/Focos).
* **2x Módulos Grove Relay (HLS8L-DC3V-S-C).**
* **2x Focos (Bombillas) con sus sockets.**
* **Cables jumper y clavijas de corriente alterna (AC).**

---

## ⚠️ 2. Reglas Estrictas de Seguridad AC
1. **Regla de Oro:** Conectar los cables de alta tensión (110V) al relevador ÚNICAMENTE con la clavija desenchufada de la pared.
2. **Cero cables sueltos:** Asegúrense de que al atornillar los cables en la bornera verde del relevador, no quede ningún filamento de cobre expuesto que pueda causar un cortocircuito.
3. **Inspección Docente:** El Mtro. Saul debe aprobar visualmente su circuito antes de energizar cualquier clavija en el laboratorio.

---

## 🔌 3. Diagrama de Conexiones

### Nodos Actuadores 1 y 2 (Los Focos):
**Baja Tensión (Control):**
* 🟨 **Cable Amarillo (Señal):** Al pin **GPIO 4**.
* 🟥 **Cable Rojo (VCC):** Al pin **3V3**.
* ⬛ **Cable Negro (GND):** Al pin **GND**.

**Alta Tensión (Potencia):**
El relevador actúa como un interruptor. Corten únicamente el cable de **Fase** de su clavija. Conecten la punta que viene del enchufe al tornillo **"C" (Común)** y la punta que va al foco al tornillo **"NO" (Normalmente Abierto)**. El cable Neutro pasa directo al foco.

### Nodo Coordinador (La Consola Maestra):
* Conectar un cable jumper al pin **GPIO 18**.
* Dejar la otra punta suelta. Este cable servirá como un "botón físico" al tocar cualquier pin **GND** de la placa.

---

## ⚙️ 4. Configuración del Arduino IDE
Para TODAS las placas, el menú `Herramientas` (Tools) debe estar configurado de la siguiente manera:
* **Board:** ESP32C6 Dev Module
* **Zigbee mode:** `Zigbee ZCZR (Coordinator/Router)` *(¡Importante: NUNCA usar End Device para evitar que los nodos se duerman!)*
* **Partition Scheme:** `Zigbee 4MB with spiffs`
* **Erase All Flash Before Sketch Upload:** `Enabled`

---

## 🧠 5. Conceptos Clave del Laboratorio

Para comprender el código de esta práctica, es vital dominar dos conceptos de ingeniería de telecomunicaciones y sistemas embebidos:

### 5.1 Los "Endpoints" (Puntos Finales) en Zigbee
En redes Wi-Fi, una placa se conecta al módem y listo. En Zigbee, la arquitectura es más profunda. 
Imagina que el microcontrolador ESP32-C6 es un **edificio de departamentos** (El Nodo físico) y el **Endpoint** es el **número de un departamento específico**.
* **¿Por qué los usamos en esta práctica?** Aunque hoy tenemos un ESP32 físico para cada foco, el estándar Zigbee está diseñado para escalar. Si mañana compramos un módulo de 4 relevadores para un solo ESP32, la red necesita saber qué foco prender. Le diremos a la red: *"Enciende el Nodo B, pero específicamente su Endpoint 10 (Sala), no el 11 (Cocina)"*.
* En nuestro código, todos los focos usan el `ENDPOINT_LUZ = 10`, pero la red los distingue usando su Dirección Física (MAC / IEEE Address).

### 5.2 La Memoria NVS (Non-Volatile Storage)
La memoria RAM de los microcontroladores es "volátil"; si se va la luz, todo lo que estaba ahí se borra. En un entorno industrial o de domótica inteligente, sería inaceptable tener que reconfigurar y emparejar toda la casa cada vez que hay un apagón.
* **¿Cuál es su propósito en este laboratorio?** En el código del Coordinador incluimos la librería `<Preferences.h>`. Esto nos permite guardar las direcciones IEEE de nuestros focos en una pequeña partición de memoria Flash (NVS) que sobrevive a los cortes de energía. Si desconectan el Coordinador de la computadora y lo conectan a un cargador de pared, ¡leerá la NVS, recordará quiénes son sus focos y la red seguirá funcionando inmediatamente!

---

## 🧪 6. Protocolo de Laboratorio: Vinculación y Consola

En esta práctica, nosotros seremos los administradores manuales de la red:

1. **Arranque:** Suban los códigos correspondientes y enciendan las 3 placas.
2. **Escaneo de Red:** En el **Coordinador**, abran el Monitor Serial a 115200 baudios. Toquen el pin **GND** con el cable conectado al pin 18. El Monitor Serial listará los dispositivos encontrados en el aire y mostrará una dirección larga (Ej. `04:08:8A:FF:FE:23:44:11`). ¡Esa es la MAC/IEEE Address de sus focos! Anótenla.
3. **Registro (Consola):** Usen la caja de texto del Monitor Serial para registrar el primer foco escribiendo el comando: `config`
4. El sistema les pedirá datos uno por uno. Respondan:
   * Número de luz: `1`
   * Endpoint: `10`
   * IEEE Address: `[Peguen la dirección exacta que anotaron]`
5. **Configurar el Foco 2:** Repitan el paso 3 y 4, pero asignando el número de luz `2` y la dirección IEEE del segundo foco.
6. **¡Pruebas de Control!** Escriban en la consola los siguientes comandos y observen sus relevadores:
   * `1on` (Enciende Foco 1)
   * `1off` (Apaga Foco 1)
   * `2toggle` (Alterna el Foco 2)
   * `toggle` (Alterna todos los focos registrados a la vez).
