PAV - P4: reconocimiento y verificación del locutor
===================================================

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 4](https://github.com/albino-pav/P4)
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 3](https://github.com/albino-pav/P3) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_spk.tgz](https://atenea.upc.edu/pluginfile.php/3008277/mod_assign/introattachment/0/db_spk.tgz?forcedownload=1)
con la base de datos oral que se utilizará en la parte experimental de la práctica.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde
que los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
  run_spkid mfcc train test classerr verify verifyerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A modo de memoria de la práctica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

## Ejercicios.

### Extracción de características.

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC), en su fichero <code>scripts/wav2lpcc.sh</code>:
 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh 
  sox $inputfile -t raw - dither -p12 | $X2X +sf | $FRAME -l 200 -p 40 | $WINDOW -l 200 -L 200 |
	$LPC -l 200 -m $lpc_order | $LPCC -m $lpc_order -M $cepstrum_order > $base.lpcc
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC), en
  su fichero <code>scripts/wav2mfcc.sh</code>:
  
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  sox $inputfile -t raw - dither -p12 | $X2X +sf | $FRAME -l 200 -p 40 | $WINDOW -l 200 -L 200 |
	$MFCC -l 200 -m $mfcc_order -s $frequency > $base.mfcc
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Indique qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC.

  #### LPCC:
    	LPC_order =  8
    	Cepstrum_order = 12
    
  #### MFCC:
    	Filters = 20
    	MFCC_order = 12
    	Sampling frequency = 8kHz

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para una señal de prueba.
  
  				LP - Dependencia entre los coeficientes 2 y 3
  <img src="img/lp_coefs.png" width="550" align="center">
  
  				LPCC - Dependencia entre los coeficientes 2 y 3 
  <img src="img/lpcc_coefs.png" width="550" align="center">
  
  				MFCC - Dependencia entre los coeficientes 2 y 3
  <img src="img/mfcc_coefs.png" width="550" align="center">
  
  + ¿Cuál de ellas le parece que contiene más información?
  
  >Podemos apreciar que, estos resultados nos muestran que los coeficientes LP están muy correlados por lo que no nos sirven 	para el reconocimiento del hablante. Los LPCC y MFCC sí que están incorrelados y por lo tanto sí que nos servirán. Indagando más en la pregunta, sabemos que el propósito principal del sistema es reconocer y verificar el hablante así pues nos interesará más resolución en las frecuencias bajas y consideramos que la distribución MFCC es experta en ello. Lo podemos fundamentar gracias a la gráfica inferior donde se muestra cómo funciona la MFCC en frecuencias bajas y cómo tiene así más resolución en frecuencias bajas:
  
  <img src="img/mfcc.png" width="450" align="center">
  

- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3, y rellene la tabla siguiente con los valores obtenidos.

  |                        |      LP     |    LPCC    |    MFCC    |
  |------------------------|:-----------:|:----------:|:----------:|
  | &rho;<sub>x</sub>[2,3] |  -0.467486  |  0.290027  |  0.317451  |
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.
  
  >Como vemos en la fórmula, figura 1, del programa <code>pearson</code> vemos que calcula el coeficiente de correlación entre componentes de un vector de características, &rho;<sub>i</sub> y &rho;<sub>j</sub>. En la segunda foto adjuntada, figura 2, vemos como funcionan los valores del coeficiente de <code>pearson</code>. Interpretando la figura vemos como los valores de &rho igual a 0 significan una incorrelación total y los que sean igual a 1 o -1 una correlación total dependiendo en la pendiente. Así pues, podemos concluir que los valores concuerdan a los de las gráficas. El de MFCC es muy cercano al 0 que significa que los puntos están casi incorrelados totalmente. El de LPCC está entre el 0 y el 1, cercanos más a 0 que tienden a mas incorrelación que se puede apreciar. Y finalmente, el LP ya se aprecia que tiene pendiente negativa y con algo de correlación lo que supone un valor entre 0 y -1.	 
	
  <img src="img/pearsonfig1.png" width="300" align="left">  
  <img src="img/pearsonfig2.png" width="300" align="right">
  <br/>
  <br/>
  <br/>
  <br/>
  <br/>
  <br/>
  
  
### Entrenamiento y visualización de los GMM.

- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.
  
  <img src="img/todoazul.png" width="400" align="center">
  
> Podemos observar como el modelado mediante GMM se adapta casi a la perfección a la distribución del locutor, obteniendo un resultado muy preciso.
  
- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos. Comente el
  resultado obtenido y discuta si el modelado mediante GMM permite diferenciar las señales de uno y otro.
  
  <img src="img/subir4.png" width="750" align="center">

> El modelado nos permite diferenciar entre dos locutores, ya que como vemos en los ejemplos de arriba, el 1 y el 9 nos muestran dos bloques principales donde se acaparan, en cada una, casi el 50% de todas las muestras. Así también podemos notar que no es del todo correcto, ya que los dos grandes bloques podemos decir que representan el silencio y la voz y la voz de cada locutor puede cambiar pero, está claro que, el silencio no depende entre locutores.
	
### Reconocimiento del locutor.

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.
  
  >Para estos resultados hemos utilizado 20 iteraciones y 32 gaussianas con inicialización con el algoritmo VQ (i = 1)
  
  |                        |    LP   |   LPCC  |   MFCC  |
  |------------------------|:-------:|:-------:|:-------:|
  | tasa de error	   |  4.97%  |  1.78%  |  0.64%  |
  

### Verificación del locutor.

- Inserte una tabla con el *score* obtenido con su mejor sistema de verificación del locutor en la tarea
  de verificación de SPEECON. La tabla debe incluir el umbral óptimo, el número de falsas alarmas y de
  pérdidas, y el score obtenido usando la parametrización que mejor resultado le hubiera dado en la tarea
  de reconocimiento.
 
  |   GMM USUARIO          |   LP   |   LPCC  |   MFCC  |
  |------------------------|:------:|:-------:|:-------:|
  | CD		       	   | 97.2   |  99.2   | 95.6    |
  | umbral óptimo      	   | -0.43  |  11.63  | -33.69  |
  | núm. falsas alarmas    | 0      |  0      | 0       |
  | pérdidas     	   | 0.97   |  0.99   | 0.95    |

### Test final y trabajo de ampliación.

- Recuerde adjuntar los ficheros `class_test.log` y `verif_test.log` correspondientes a la evaluación
  *ciega* final.

- Recuerde, también, enviar a Atenea un fichero en formato zip o tgz con la memoria con el trabajo
  realizado como ampliación, así como los ficheros `class_ampl.log` y/o `verif_ampl.log`, obtenidos como
  resultado del mismo.
