[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/KzqfxGd5)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=22828603&assignment_repo_type=AssignmentRepo)
# Lab02 - Caracterización de osciladores (externo vs. interno)


## 1. Integrantes

* [Samuel Forero](https://github.com/Sam232510)

* [Danna Pineda](https://github.com/Danna-pineda)

## 2. Documentación

### 2.1 Descripción del laboratorio
En este laboratorio realizamos tres diferentes montajes para probar el oscilador interno, el oscilador externo y circuito RC en el PIC18F45K22 para obtener una señal con una frecuencia aproximada de 500 Hz, primero se programó el oscilador interno del microcontrolador y se verificó su funcionamiento midiendo la señal generada en un osciloscopio, despues se monto un oscilador con un cristal para que funcionara con oscilación externa, realizando nuevamente la medición de la frecuencia obtenida y finalmente se montó un circuito RC como otra forma de oscilación externa, medimos hasta obtener igualmente los 500Hz.

En cada una de las configuraciones se realizaron mediciones con el fin de comprobar que la señal generada se aproximara a los 500 Hz requeridos en la guia de laboratorio, permitiendo así comparar el comportamiento de los diferentes métodos de generación de reloj utilizados en el microcontrolador.

### 2.2 Explicación del código implementado

### - Librerias

    #include <xc.h>

    #include <stdint.h>

En esta parte del código tenemos las librerías necesarias para trabajar con el microcontrolador.

- xc.h: permite acceder a los registros y configuraciones del microcontrolador.

- stdint.h: define tipos de datos enteros con tamaño específico, como uint16_t.

### - Configuración Modos

    // 1 = INTOSC interno
    // 2 = Cristal externo HS
    // 3 = RC externo
    #define MODE 1  

    #if MODE == 1
        #pragma config FOSC = INTIO67   // Oscilador    interno
        #define USE_PLL 0
    #elif MODE == 2
        #pragma config FOSC = HSHP     // Cristal HS
        #define USE_PLL 0
    #elif MODE == 3
        #pragma config FOSC = RC       // RC externo
        #define USE_PLL 0
    #else
        #error "Modo de oscilador inválido"
        #endif

Este bloque del programa permite seleccionar el tipo de oscilador que utilizará el microcontrolador PIC18F5022. Para ello se define una constante llamada MODE, la cual determina que oscilador estaremos usando.

El valor de MODE puede ser cualquiera de las tres configuraciones que tenemos:

MODE = 1: se utiliza el oscilador interno del microcontrolador.

MODE = 2: se utiliza un cristal externo.

MODE = 3: se utiliza un oscilador RC externo formado por una resistencia y un capacitor.

Mediante bloques como #if, #elif y #else el programa selecciona automáticamente la configuración correspondiente. En cada caso se utiliza #pragma config FOSC para establecer el tipo de oscilador que controlará el oscilador del sistema.

También se define la variable USE_PLL, la cual permite habilitar o deshabilitar el PLL para multiplicar la frecuencia del oscilador si se requiere mayor velocidad de operación.

Y por ultimo si se pone un valor inválido para MODE, el compilador generará un error con la parte del bloque #error, evitando que el programa se compile con una configuración incorrecta.

### - Configuración de frecuencia

    #if MODE == 1 || MODE == 2
        #if USE_PLL
            #define _XTAL_FREQ 64000000UL // 16 MHz * 4
        #else
            #define _XTAL_FREQ 16000000UL
        #endif
    #else
        #define _XTAL_FREQ 16000000UL // Ajustar según resistencia + condensador
    #endif

Este bloque define la frecuencia con la que va a operar el microcontrolador, la cual es almacenada en la constante _XTAL_FREQ, esta constante es utilizada por el programa para calcular correctamente los retardos generados mediante funciones como __delay_ms().

La frecuencia se establece dependiendo del modo de oscilador seleccionado, cuando se utiliza el oscilador interno o un cristal externo, la frecuencia base se define como 16 MHz en el caso del oscilador interno en cambio para el cristal la frecuencia que utilizamos fue de 800000 Hz y en el caso del RC la frecuencia a utilizar fue calculada mediante la formula de frecuencia en un circuito RC.

### - Funciones

    void delay_ms(uint16_t ms) {
        while(ms--) {
            __delay_ms(1);
        }
    }

    void init_pins(void) {
        // RC0 salida blinker
        TRISCbits.TRISC0 = 0;
        LATCbits.LATC0 = 0;

    // RA6 salida CLKO solo si modo lo permite
        if(MODE == 1 || (MODE == 2 && USE_PLL)) {
            TRISAbits.TRISA6 = 0;
            LATAbits.LATA6 = 0;
        }
    }

    void init_oscillator(void) {
    #if USE_PLL
        OSCCONbits.SPLLEN = 1;  // habilita PLL
    #endif
    }

Este bloque del programa define varias funciones que permiten organizar y facilitar la configuración del microcontrolador, la función delay_ms(uint16_t ms) se encarga de generar retardos en milisegundos mediante un ciclo while que ejecuta repetidamente la instrucción __delay_ms(1) hasta completar el tiempo indicado, esto permite controlar los tiempos de espera dentro del programa. 

La función init_pins() configura los pines que serán utilizados durante la ejecución del códigp, en este caso, el pin RC0 se establece como salida digital mediante el registro TRISC, iniciando en nivel bajo usando LATC, este pin es el que se utiliza para generar la señal que posteriormente se mide con el osciloscopio.

Finalmente la función init_oscillator() permite habilitar el PLL si está activado en la configuración del programa, utilizando el registro OSCCONbits.SPLLEN, el PLL permite multiplicar la frecuencia del oscilador y aumentar la velocidad de operación del microcontrolador. En conjunto estas funciones preparan el sistema antes de que se ejecute el programa principal.

### - Programa Principal

    void main(void) {
        init_pins();
        init_oscillator();

    while(1) {
        // RC0 toggle ≈ 500 Hz
        LATCbits.LATC0 = 1;
        delay_ms(1);
        LATCbits.LATC0 = 0;
        delay_ms(1);
        }
    }

Este bloque corresponde al programa principal que se ejecuta en el microcontrolador PIC18F5022. Inicialmente se llaman a las funciones init_pins() e init_oscillator(), las cuales se encargan de configurar los pines de entrada y salida y establecer la configuración del sistema de oscilación. Posteriormente se ejecuta un ciclo infinito while(1), el cual permite que el programa se repita continuamente mientras el microcontrolador esté encendido. 
    
Dentro de este ciclo se genera una señal digital en el pin RC0, colocando primero el pin en nivel alto (LATCbits.LATC0 = 1), esperando 1 ms mediante la función delay_ms(1), y luego colocando el pin en nivel bajo (LATCbits.LATC0 = 0) seguido de otro retardo de 1 ms. Este proceso se repite constantemente, produciendo una señal cuadrada con un período aproximado de 2 ms, lo que corresponde a una frecuencia cercana a 500 Hz, la cual puede ser observada y verificada utilizando el osciloscopio durante el laboratorio.

### 2.3 Análisis y comparación

A continuación se presentan diferentes tablas las cuales nos dan a entender cual fue el comportamiento de cada una de las señales generadas y comparando su porcentaje de error en cuanto a lo teorico a lo obtenido. Al igual realizando unas pequeñas variaciones como la temperatura a ver como puede influir dentro de las oscilaciones que se generan.

#### Tabla 1: Medición en frío

| Modo de oscilador | Freq. teórica Fosc | RA6 medible (CLKO)? | Freq. medida RA6 (Hz) | Freq. teórica RC0 (Hz)| Freq. medida RC0 (Hz) | Error RC0 (%) |  
|------------------|------------------|---------------------|---------------|---------------------|---------------|---------------|
| INTOSC (interno) | 16,000,000       | Sí                 |  31.4                   |                500                 | 496              |    0.8           | |
| HS (cristal externo 16 MHz) | 16,000,000 | No |     NA      |              500                 |  501.2             |            0.24   |
| RC externo       | ~16,000,000*     | No                                    |       N/A        | 500                 |    494           |      1.2         | |

#### Tabla 2: Medición con calor

| Modo de oscilador | Freq. teórica Fosc | RA6 medible (CLKO)? | Freq. medida RA6 (Hz) | Freq. teórica RC0 (Hz)| Freq. medida RC0 (Hz) | Error RC0 (%) |  
|------------------|------------------|---------------------|---------------|---------------------|---------------|---------------|
| INTOSC (interno) | 16,000,000       | Sí                 |  Sin señal                   |                500                 |   523            |     4.6          | |
| HS (cristal externo 16 MHz) | 16,000,000 | No |     NA      |               500                 |     499          |       0.2        |
| RC externo       | ~16,000,000*     | No                                    |       N/A        | 500                 |   528            |   5.6            | |

#### Tabla 3: Deriva

| Modo de oscilador |RC0 deriva (Hz) |
|------------------|--------------------|
| INTOSC (interno) |      4              |                
| HS (cristal externo 16 MHz) |       1.2         |                |
| RC externo       |     6            |                


<!-- Agregar tablas para valores usando PLL -->

<!-- Complemente con análisis de lo registrado en tablas -->

## 2.4 Diagramas

A partir de los siguientes diagramas se observo el comportamiento de cada uno de los circuitos de manera simulada para así verificar y asegurar que a el momento de la implementación fuera cierta y sin ningun tipo de problema. Las simulaciones ceuntan con el codigo de implementación en formato .hex dentro de el software proteus para observar los diferentes comportamientos.

* Simulación Oscilador Interno

![alt text](LedP.png) Figura 1. Oscilador Interno

* Simulación Cristal 16MHz

![alt text](CristalP.png) Figura 2. Cristal 16MHz

* Simulación Circuito RC

![alt text](RCp.png) Figura 3. Circuito RC

## 2.5 Formas de onda

### INTOSC (interno) 
![alt text](Interno.png) Figura 4. Oscilador interno

En esta imagen se observa la señal que nos genera el oscilador interno de el PIC18F45K22, siempre y cuando tenemos el modo 1 dentro de el codigo el cual nos activa un señal de 900000UL ya que es el que hace que funcione el oscilador interno. 
### HS
![alt text](Cristal.png) Figura 5. Oscilador externo cristal

En esta imagen se logra observar la señal que se genera gracias a la conexión entre dos condensadores y el cristal de 16MHz.
## RC
![alt text](RC.png) Figura 6. RC

En el tercer modo se observa la señal de 500Hz que se logra generar a traves de el circuito RC.
## 3. Evidencias de implementación

![alt text](Ledreal.png) Figura 7. Montaje Oscilador interno

En esta imagen se evidencia el diseño de el circuito en la realidad, en donde se realiza la oncexión a traves de el PICKIT4 junto a el PIC18F45K22 para que de esta forma se aliemnte y pueda compilar el codigo establecido. En el cual la señal que se obtiene dentro de el pic es llevada a el pin 15 (RC0) para así lograr observar lo que establece dentro del programa,

![alt text](CristalReal.png) Figura 8. Montaje Oscilador externo cristal

En esta imagen se evidencia el diseño de el circuito en la realidad, en donde se realiza la oncexión a traves de el PICKIT4 junto a el PIC18F45K22 en donde el oscilador de cristal junto a los 2 condensadores de 22pf en donde se envia una señal a la entrada (pin13) y la otra a la salida (pin14) para que de esta forma se genere una unica señal de salida y no se afecte entre el cristal y el oscilador interno de el PIC.

![alt text](RCReal.png) Figura 9. Montaje circuito RC

Para el circuito RC lo que se realiza es por medio de la formula de la frecuencia calcular los valores de los componentes que se necesitan, en nuestro caso se asumio un condensador de 22pf y se calculo la resistencia de la siguiente forma.

$$
f = \frac{1}{2\pi R C}
$$

En donde lo que se debe hacer es el despeje de la R ya que se conoce los demas datos.

$$
R = \frac{1}{2\pi f C}
$$

Ahora remplazamos y obtenemos.

$$
R = \frac{1}{2\pi (500)(22pf) }
$$

$$
R = 14MΩ  
$$

De esta forma la señal que nos segenera el RC se manda a el pin 13 el cual es una entrada de oscilaciones y en el pin 14 se observa la salida de estas oscilaciones obteniendo el resultado esperado.
## 4. Preguntas

* ¿En qué modo se obtuvo la medición más cercana a la frecuencia teórica?

Dentro de todos los modos se obtuvo una respuesta muy cercana a la teorica, pero en el que más nos acercamos fue en el de Cristal ya que esta se pasaba por 1Hz, pero esto fue gracias a la buena efectividad dentro de los componentes y la frecuencia decidida que se propone dentro de el codigo.

* ¿Fue posible evidenciar el fenómeno de deriva? ¿Qué factores podrían explicar la variación de frecuencia al calentar el PIC?

Sí, fue posible evidenciar el fenómeno de deriva. Al calentar el microcontrolador, la frecuencia de la señal generada presentó pequeñas variaciones respecto al valor inicial. Esto se puede observar en el osciloscopio cuando el periodo cambia ligeramente, lo que implica que la frecuencia ya no es exactamente la misma.

Este comportamiento ocurre porque la frecuencia del oscilador interno del microcontrolador no es perfectamente estable, y puede variar debido a cambios en las condiciones físicas, como la temperatura

* ¿Cuál es más preciso en cuanto a frecuencia teórica vs. medida?

El modo de oscilación más preciso fue el oscilador con cristal externo (HS), en las mediciones realizadas, este modo presentó el menor porcentaje de error tanto en condiciones normales como cuando se aplicó calor. En frío el error fue aproximadamente 0.24 %, y con temperatura elevada fue cercano a 0.2 %, lo cual indica que la frecuencia medida se mantiene muy cercana a la frecuencia teórica de 500 Hz.


* Explique cómo usar RC0 para estimar la frecuencia del oscilador cuando RA6 no está disponible.

Cuando el pin RA6 (CLKO) no está disponible para medir directamente la frecuencia del reloj del sistema, es posible estimarla utilizando una señal generada por software en otro pin, como RC0.

En el programa se genera una señal cuadrada en RC0 alternando el pin entre nivel alto y bajo con un retardo de 1 ms entre cada cambio. Esto produce un período total de aproximadamente 2 ms, lo que corresponde a una frecuencia cercana a 500 Hz.

* Si se quisiera duplicar la frecuencia del PIC usando PLL, ¿en qué modos se podría aplicar?

Se puede usar en la configuración de el oscilador interno y el con el oscilador del cristal en estos casos el PLL puede multiplicar la frecuencia base (por ejemplo 16 MHz) para obtener frecuencias mayores, sin embargo el modo RC externo normalmente no se utiliza con PLL, ya que la frecuencia generada por este tipo de oscilador no es lo suficientemente estable para un correcto funcionamiento del multiplicador de frecuencia.

* Enliste ventajas y desventajas de cada modo.

### Oscilador interno 

### Ventajas

- No requiere componentes externos.

- Reduce costo y tamaño del circuito.

- Fácil configuración.

### Desventajas

- Menor precisión en la frecuencia.

- Mayor variación con temperatura y voltaje.

### Oscilador del cristal

### Ventajas

- Alta precisión en la frecuencia.

- Muy buena estabilidad frente a cambios de temperatura.

- Ideal para aplicaciones que requieren sincronización exacta.

### Desventajas

- Requiere componentes externos (cristal y capacitores).

- Aumenta el costo y el espacio en el circuito.

### Oscilador RC

### Ventajas

- Implementación simple con pocos componentes.

- Bajo costo.

### Desventajas

- Baja precisión.

- Alta sensibilidad a temperatura y tolerancias de componentes.

- Frecuencia menos estable que las otras opciones.

## Conclusiones 

- En este laboratorio se analizaron diferentes modos de oscilación del microcontrolador PIC18F45K22 con el objetivo de generar y medir una señal cercana a 500 Hz utilizando un osciloscopio, a partir de las mediciones realizadas se pudo observar que el oscilador con cristal externo presentó la mayor precisión y estabilidad, ya que el error entre la frecuencia teórica y la medida fue el más bajo tanto en condiciones normales como cuando se aplicó calor.

- El oscilador interno mostró un desempeño aceptable en condiciones normales, pero presentó una variación mayor cuando la temperatura aumentó, lo que evidencia que este tipo de oscilador es más sensible a factores externos, el oscilador RC externo presentó la mayor variación en la frecuencia medida, debido a que su funcionamiento depende directamente de los valores de la resistencia y el capacitor, así como de las condiciones ambientales.

- El laboratorio permitió comprender las diferencias entre los distintos métodos de generación de reloj en un microcontrolador, así como la importancia de seleccionar el tipo de oscilador adecuado dependiendo de la precisión y estabilidad requeridas en una aplicación electrónica.

## 5. Referencias

- Microchip Technology. (2023). PIC18F4520 Family Reference Manual. Microchip Technology Inc. https://www.microchip.com

- Texas Instruments. (2020). Introduction to Oscillators and Crystal Circuits. Texas Instruments. https://www.ti.com
