# PWM -  STM32L476

Se pretende realizar un ejercicio ilustrativo para hacer PWM en la tarjeta STM32L476

## Que es  un PWM?
La modulación por ancho o de pulso (en inglés pulse width modulation PWM) es un tipo de señal de voltaje utilizada para enviar información o para modificar la cantidad de energía que se envía a una carga. Esta acción tiene en cuenta la modificación del ciclo de trabajo de una señal de tipo periódico, como tener el control de la energía que se proporciona a una carga o llevar a cabo la transmisión de dato.

### Resumen del ejercicio
El siguiente programa consiste en desvanecer la intensidad luminica de un LED usando la funcionalidad PWM en timmer 4, los LED se encuentran conectados a los pines 12 y 15 del puerto GPIOD, el GPIOD 12 esta conectado al timmer 4 y canal 1.

####  Como se hará?
Como primer paso, se incluyen las librerías y las variables que utilizaremos en el código. La librería _math.h_ es un archivo de la biblioteca de funciones del Lenguaje de programación C  diseñado para operaciones matemáticas básicas sobre valores de tipo double.

```C
#include "stm32l476xx.h"
#include "system_stm32l476xx.h"
#include <math.h>

#define PWMPERIOD 100
#define SAMPLE 100
```

Ahora definiremos algunas variables _static_ y borramos el  estado de interrupcion para el tim4 de la siguiente manera:

```C
/*************************************************
 function declarations
*************************************************/
int main(void);

/*************************************************
timer 4 interrupt handler
*************************************************/
void TIM4_IRQHandler(void)
{
    static uint32_t t = 0; //Número entero sin signo de 4 bytes
    static uint16_t duty = 0; //Número entero sin signo de 2 bytes
```

![](https://github.com/RobinsonRJ10/PWM---STM32L476/blob/master/Imagenes/TIM4_DIER.png)

![](https://github.com/RobinsonRJ10/PWM---STM32L476/blob/master/Imagenes/TIM4_SR.png)

```C
// clear interrupt status
    if (TIM4->DIER & 0x01){
        if (TIM4->SR & 0x01){
            TIM4->SR &= ~(1U << 0);
        }
    }
```
Con esto se actualizara la bandera de interrupción, este bit lo establece el hardware en un evento de actualización cuando se actualizan los registros.

Los temporizadores de hardware STM32 son bloques de hardware separados que pueden contar desde 0 hasta un valor dado que desencadena algunos eventos intermedios. En el modo PWM, el temporizador controla la salida de 1 o más canales de salida. Cuando el valor del contador alcanza 0, máximo o un valor de comparación definido para cada canal, se puede cambiar el valor de salida del canal. Varias opciones de configuración definen qué eventos cambian el valor y cómo se cambia.

![](https://github.com/RobinsonRJ10/PWM---STM32L476/blob/master/Imagenes/pwm.png)

Ahora se establece un ciclo de trabajo.

```C
duty =(uint16_t)(PWMPERIOD/2.0 * (sin(2*M_PI*(double)t/(SAMPLE)) + 1.0));
    ++t;
    if (t == SAMPLE) t = 0;
    // Se establece un nuevo ciclo de trabajo
    TIM4->CCR1 = duty;
}
```
TIMx_CCR1 es de solo lectura y no se puede programar. CCR1 es el valor a cargar en el registro real de captura / comparación

Ahora se procedera a establecer el reloj del sistema a 168 Mhz, Habilitamos el reloj para el puerto GPIOD y modificamos los registros para configurar el pin 12 de este puerto a modo 10 (_Alternate function mode_), donde el TIM4 se establecera como (_Alternate function mode_) para el pin 12 (LED).

```C
/*************************************************
El código principal comienza desde aquí
*************************************************/
int main(void)
{
    /* set system clock to 168 Mhz */
    set_sysclk_to_168;
```

![](https://github.com/RobinsonRJ10/PWM---STM32L476/blob/master/Imagenes/RCC.png)


```C
    RCC->AHB2ENR |= 0x00000008; //(1 << 3);
```


![](https://github.com/RobinsonRJ10/PWM---STM32L476/blob/master/Imagenes/MODER.png)


```C
    GPIOD->MODER &= 0xFFFFFFFF;
    GPIOD->MODER &= 0xFEFFFFFF; //(0x2 << 24);
    GPIOD->AFR[1] |= (0x2 << 16);
```

Ahora se habilitara el relij del TIM4 y guardaremos PSC el valor que se va a cargar en el registro del preescaler activo en cada evento, en ARR el valor que se va a cargar en el registro de recarga automática real, el contador estará bloqueado mientras que el valor de recarga automatica sea nulo, establecemos el ciclo de trabajo en el canal 1.

![](https://github.com/RobinsonRJ10/PWM---STM32L476/blob/master/Imagenes/APB1ENR1.png)

```C
    RCC->APB1ENR1 |= 0x00000004; //(1 << 2);
    // fCK_PSC / (PSC[15:0] + 1)
    // 84 Mhz / 8399 + 1 = 10 khz timer clock speed
    TIM4->PSC = 8399;
    // set period
    TIM4->ARR = PWMPERIOD;
    TIM4->CCR1 = 1;
```

Ahora procedemos a establecer OC1 en modo PWM y habilitamos el canal 1 como salida en el registro de captura/comparacion y OC1 se emite en el pin de salida correspondiente

![](https://github.com/RobinsonRJ10/PWM---STM32L476/blob/master/Imagenes/OC1.png)

```C
    TIM4->CCMR1 |= (0x6 << 4);
    // enable oc1 preload bit 3
    TIM4->CCMR1 |= (1 << 3);
    // enable capture/compare ch1 output
    TIM4->CCER |= (1 << 0);
```

Por ultimo habilitamos la interrupcion de actualizacion , seleccionamos la prioridad, qur paras este caso sera de nivel 2 y habilitamos el modulo del TIM4 (CEN, bit0), hecho esto el programa estara terminado.

![](https://github.com/RobinsonRJ10/PWM---STM32L476/blob/master/Imagenes/DIER.png)

```C
    TIM4->DIER |= (1 << 0);
    NVIC_SetPriority(TIM4_IRQn, 2); // Priority level 2
    // enable TIM4 IRQ from NVIC
    NVIC_EnableIRQ(TIM4_IRQn);
```

![](https://github.com/RobinsonRJ10/PWM---STM32L476/blob/master/Imagenes/CR1.png)

```C
    TIM4->CR1 |= (1 << 0);

    while(1)
    {
        // Do nothing.
    }

    return 0;
}
```
