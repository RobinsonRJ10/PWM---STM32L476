# PWM -  STM32L476

Se pretende realizar un ejercicio ilustrativo para hacer PWM en la tarjeta STM32L476

## Que es  un PWM?
La modulación por ancho o de pulso (en inglés pulse width modulation PWM) es un tipo de señal de voltaje utilizada para enviar información o para modificar la cantidad de energía que se envía a una carga. Esta acción tiene en cuenta la modificación del ciclo de trabajo de una señal de tipo periódico, como tener el control de la energía que se proporciona a una carga o llevar a cabo la transmisión de dato.

### Resumen del ejercicio
El siguiente programa consiste en desvanecer la intensidad luminica de un LED usando la funcionalidad PWM en timmer 4, los LED se encuentran conectados a los pines 12 y 15 del puerto GPIOD, el GPIOD 12 esta conectado al timmer 4 y canal 1.

####  Como se hará?
Como primer paso, se incluyen las librerías y las variables que utilizaremos en el código. La librería _math.h_ es un archivo de la biblioteca de funciones del Lenguaje de programación C  diseñado para operaciones matemáticas básicas sobre valores de tipo double.

![](imagenes/1.PNG)

Ahora definiremos algunas variables _static_ y borramos el  estado de interrupcion para el tim4 de la siguiente manera:

'' 'C
/*************************************************
 function declarations
*************************************************/
int main(void);

/*************************************************
* timer 4 interrupt handler
*************************************************/
void TIM4_IRQHandler(void)
{
    static uint32_t t = 0; //Número entero sin signo de 4 bytes
    static uint16_t duty = 0; //Número entero sin signo de 2 bytes
'' '


