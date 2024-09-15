# PS3-Bitcoin-Mining
Programa de prueba de minería de bitcoins para PS3 en ensamblador para el procesador Cell/BE.

(Sé que una PS3 es una plataforma de minería de Bitcoin lenta según los estándares actuales, pero quería ver si podía hacerla funcionar)

Este programa está escrito para el procesador Cell/BE que está en la consola de juegos PS3. El procesador Cell/BE también está disponible en el IBM QS20 Blade, que está implementado en el chasis IBM Blade Center. Este código fuente debería poder usarse en el QS20; es posible que sea necesario adaptar la cadena de herramientas, ya que esta PS3 ejecuta una versión muy antigua de Linux y el kit de herramientas del compilador GNU.

Si alguien tiene un Blade Center con un blade QS20 y me permite acceder a él por SSH, estaré encantado de portar, probar y escalar el código.

El procesador Cell/BE es un chip PPC (Power PC) de doble núcleo, con una frecuencia de reloj de 3,2 GHz. Hay un Cell/BE en el PS3, hay 2 en un blade IBM QS20. El PS3 tiene 256 Mb de memoria, que no es una gran cantidad de memoria, pero más que suficiente para la minería de bitcoins. Los chips Cell/BE se fabrican con 8 SPE (elementos de procesamiento sinérgico) conectados. En el PS3, Linux identifica 7 de los SPE, reserva uno para el sistema y deja a los desarrolladores de aplicaciones con acceso a 6 de los SPE.

Cada SPE es en realidad un procesador vectorial de 4 vías que se puede programar en el modelo SIMD (instrucción única, múltiples datos). El procesador carga y almacena datos en palabras cuaternarias (palabras de 4 x 32 bits). SIMD se refiere al hecho de que cada operación, por ejemplo, una suma o una multiplicación, se aplica a los 4 elementos de la palabra cuaternaria simultáneamente. Cada una de las palabras de 32 bits de la palabra cuaternaria se denomina carriles y cada instrucción se ejecuta simultáneamente en los 4 carriles. Dado el problema, el algoritmo y la implementación correctos, el procesamiento vectorial en Cell/BE ejecuta 4 veces más trabajo por instrucción de CPU que un procesador escalar estándar.

La programación del procesador SPE presenta varios desafíos. Los SPE no tienen acceso directo a la memoria PPC. El SPE tiene una cantidad muy pequeña de memoria local: solo 128k. Los datos se mueven de la memoria PPC a la memoria SPE con una especie de programación DMA. La minería de Bitcoin evita este problema. Al minar, todos los datos necesarios para los cálculos se ensamblan en una estructura C y se transfieren por DMA desde la memoria del sistema PPC a la memoria local SPE en un solo fragmento a medida que se invoca la función de lenguaje ensamblador.

En la minería de Bitcoin, la suma de comprobación SHA256 se calcula a partir de un fragmento relativamente pequeño de datos. Si el SHA256 no cumple con el nivel de dificultad, se incrementa un campo entero de 32 bits en los datos y se calcula nuevamente el SHA256. Continúe iterando de esta manera hasta que se hayan probado todas las combinaciones 2B o se encuentre un SHA256 ganador, uno que sea menor que la dificultad.

Existen algunos aspectos desafiantes de la programación SIMD. Un punto a tener en cuenta es que realmente debe tener un problema vergonzosamente paralelo para obtener el efecto completo de la programación SIMD: procesar 4 elementos de datos a la vez. Si tiene mucha lógica de decisión u otro código escalar, no verá un gran beneficio de la programación SIMD.

Resulta que calcular SHA256 para la minería de Bitcoin es una gran combinación para la arquitectura Cell/BE.

El cálculo de SHA256 se realiza en el código ensamblador y las instrucciones deben programarse de manera efectiva. Obviamente, el compilador de C puede reordenar las instrucciones y acelerar la ejecución, esto debe hacerse a mano al escribir el código ensamblador.

-- habrá más información.
Controlador de prueba actual:
este es un programa C con el segundo bloque (bloque número 1), básicamente codificado en el programa. El bloque se formatea en una estructura y se pasa a la rutina de ensamblaje.

La rutina de ensamblaje puede llamar código de las bibliotecas C, básicamente todo se depuró mediante printf(s), ya que no hay un depurador que admita los SPE disponibles en la versión anterior de Yellow Dog Linux (6.2) que estoy usando.

Referencias:

TODO:
* obtener mejor información de tiempo para una versión de proceso único del programa. Bitcoin es un "hash SHA256 doble" donde el SHA-256 resultante se vuelve a codificar. Un solo procesador puede hacer 2,5 millones de hashes por segundo, lo que da 1,25 millones de hashes dobles de Bitcoin por segundo.
* escribir un controlador de prueba que pueda alimentar más casos de prueba. Actualmente, un caso de prueba está codificado en drive.c
* escribir un controlador que pueda ejecutar los casos de prueba en paralelo.
* continuar observando la programación de instrucciones con /opt/cell/sdk/usr/bin/spu_timing
* mejores comentarios, a medida que mezclamos las instrucciones para las diferentes partes del algoritmo, _será_ confuso.