# Agitador CNC
Asesoría máquina CNC controlada con NodeRED.  
Automatizar proceso de agitación de un objeto dentro de recipientes con reactivos líquidos.  

## Hardware
Tarjeta controladora de motores **MKS_GEN_L_V1**.  
Se utiliza en impresoras 3D, pero también se puede usar en CNCs. Diseño basado en la tarjeta RAMPS.  
https://github.com/makerbase-mks/MKS-GEN_L/wiki/MKS_GEN_L_V1  

Controlador principal **Raspberry Pi 4**.  

Notas:
grbl Mega 5X pinout  
https://github.com/fra589/grbl-Mega-5X/wiki/grbl-Mega-5X-pinout  

## Software (y firmware)
La tarjeta MKS_GEN_L_V1 venía con el firmware **Marlin** instalado.
Se cambió al firmware **grbl Mega**  
https://github.com/gnea/grbl-Mega  
En este repo está el código fuente editado. El código se compila con el IDE de Arduino.  
Descargar el repo y copiar la carpeta "grbl" a "Documentos/Arduino/libraries". 
Abrir el IDE de Arduino y buscar el ejemplo "grbl upload". 
Seleccionar Arduino Mega, compilar y subir a la tarjeta.  

Los cambios en el código fuente son:
- comentar líneas 37 y 38 para no usar la placa por defecto
- descomentar líneas 41 y 42 para seleccionar la placa RAMPS
- editar líneas 110 a 112 para cambiar el orden de homing

Además se puede descomentar la línea 134 para habilitar el homing de ejes individuales.  

Se usa **Node-RED** para crear la interfaz gráfica y la lógica principal del programa.  
En este repo está el flujo de la máquina.  

Cómo instalar Node-RED en Raspberry Pi  
https://nodered.org/docs/getting-started/raspberrypi  

En Node-RED se deben instalar estos módulos desde "Manage palette":
- Dashboard - para poder crear la interfaz gráfica https://flows.nodered.org/node/node-red-dashboard
- Nodos de Raspberry Pi - para usar los GPIO 
- digital clock - para mostrar un reloj estilizado https://flows.nodered.org/node/node-red-contrib-ui-digital-clock

También se instaló un teclado virtual para poder escribir los valores con pantalla táctil  
https://flows.nodered.org/flow/7fb5bc5ae66e6bc1b1c1b8e800bdef51  

### Problema para pausar el flujo
Cuando se envía una línea de gcode hay que esperar a que la máquina termine de hacer el movimiento. 
El primer intento fue usar solo código de javascript con un ciclo while que termina cuando la 
máquina cambia al estado IDLE, pero no funcionó. Node-RED no permite la ejecución de esos ciclos. 
También se probó con los métodos setTimeout, setInterval y requestAnimationFrame, pero no funcionaron.  

Se revidaron algunos nodos para hacer la espera, pero son más complejos de lo que se necesita  
https://flows.nodered.org/flow/45cd96abdf9b965ec343e4e986f17c71  
https://flows.nodered.org/node/node-red-contrib-delay-gate  
https://discourse.nodered.org/t/wait-for-the-execution-of-a-flow-and-then-continoue-another/21560  
https://flows.nodered.org/node/node-red-contrib-state  
https://flows.nodered.org/node/node-red-contrib-wait-paths  

Los ciclos de espera se pueden hacer con bucles de nodos y el nodo delay en medio de los bucles.  
https://flowfuse.com/node-red/core-nodes/delay/  

## Iniciar dashboard en pantalla completa
Cuando la Raspberry enciende se inicia NodeRED y el navegador para mostrar el dashboard.  

Primero se instaló Raspberry OS (bookworm), pero el inicio automático del navegador no funcionaba bien.  
Este tutorial funcionó una vez, pero después ya no https://www.raspberrypi.com/tutorials/how-to-use-a-raspberry-pi-in-kiosk-mode/
Probamos otras opciones, pero tampoco funcionaron https://raspberrytips.es/iniciar-un-programa-raspberry-pi/
Otro tutorial https://pimylifeup.com/raspberry-pi-kiosk/

Se logró el inicio automático del navegador con FullPageOS.  
Al configurar FullPageOS es importante dejar el usuario "pi", porque usa scripts con rutas fijas dentro del sistema de archivos.  
Editar el archivo "/boot/fullpageos.txt"
Escribir la dirección del dashboard "http://localhost:1880/ui/"

## Ajustes y comandos de grbl
Descripción de los ajustes  
https://github.com/gnea/grbl/blob/master/doc/markdown/settings.md  

El comando ```$$``` muestra los ajustes
```
$0 = 10 (Step pulse time, microseconds)
$1 = 25 (Step idle delay, milliseconds)
$2 = 0 (Step pulse invert, mask)
$3 = 0 (Step direction invert, mask)
$4 = 0 (Invert step enable pin, boolean)
$5 = 1 (Invert limit pins, boolean)
$6 = 0 (Invert probe pin, boolean)
$10 = 1 (Status report options, mask)
$11 = 0.010 (Junction deviation, millimeters)
$12 = 0.002 (Arc tolerance, millimeters)
$13 = 1 (Report in inches, boolean)
$20 = 1 (Soft limits enable, boolean)
$21 = 0 (Hard limits enable, boolean)
$22 = 1 (Homing cycle enable, boolean)
$23 = 3 (Homing direction invert, mask)
$24 = 250.000 (Homing locate feed rate, mm/min)
$25 = 4000.000 (Homing search seek rate, mm/min)
$26 = 250 (Homing switch debounce delay, milliseconds)
$27 = 5.000 (Homing switch pull-off distance, millimeters)
$30 = 1000 (Maximum spindle speed, RPM)
$31 = 0 (Minimum spindle speed, RPM)
$32 = 0 (Laser-mode enable, boolean)
$100 = 80.000 (X-axis travel resolution, step/mm)
$101 = 80.000 (Y-axis travel resolution, step/mm)
$102 = 2267.717 (Z-axis travel resolution, step/mm)
$110 = 10000.000 (X-axis maximum rate, mm/min)
$111 = 10000.000 (Y-axis maximum rate, mm/min)
$112 = 500.000 (Z-axis maximum rate, mm/min)
$120 = 500.000 (X-axis acceleration, mm/sec^2)
$121 = 500.000 (Y-axis acceleration, mm/sec^2)
$122 = 300.000 (Z-axis acceleration, mm/sec^2)
$130 = 610.000 (X-axis maximum travel, millimeters)
$131 = 610.000 (Y-axis maximum travel, millimeters)
$132 = 85.000 (Z-axis maximum travel, millimeters)
```

## Gcode
http://linuxcnc.org/docs/html/gcode.html  
