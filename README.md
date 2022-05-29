PAV - P4: reconocimiento y verificación del locutor
===================================================

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 4](https://github.com/albino-pav/P4)
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_8mu.tgz](https://atenea.upc.edu/mod/resource/view.php?id=3508877?forcedownload=1)
con la base de datos oral que se utilizará en la parte experimental de la práctica.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde
que los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
  run_spkid mfcc train test classerr verify verifyerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Recuerde que, además de los trabajos indicados en esta parte básica, también deberá realizar un proyecto
de ampliación, del cual deberá subir una memoria explicativa a Atenea y los ficheros correspondientes al
repositorio de la práctica.

A modo de memoria de la parte básica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

## Ejercicios.

### SPTK, Sox y los scripts de extracción de características.

- Analice el script `wav2lp.sh` y explique la misión de los distintos comandos involucrados en el *pipeline*
  principal (`sox`, `$X2X`, `$FRAME`, `$WINDOW` y `$LPC`). Explique el significado de cada una de las 
  opciones empleadas y de sus valores.

El programa sox sirve para generar una señal del formato adecuado a partir de una señal con otro formato
El comando '$X2X' en nuestro caso como utilizamos ubunto, representa el comando sptk x2x que permite la conversión entre distintos formatos de datos.
El comando $FRAME representa el comando 'sptk frame' que permite extraer las ventanas de la sequencia de datos amb un frame lenght de 240 i una frame period de 80.
El comando $WINDOW representa el comando 'sptk window' nos da los detalles de la ventana. Vemos que el frame length of input es de 240, el frame length of output tambien es de 240.
El comando $LPC representa el comando 'sptk lpc' que nos da el LPC analisis usando el metodo de Levinson-Durbin con frame length de 240 y el ordre de LPC es el que le pases por parametro.

- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros de
  salida de SPTK (líneas 45 a 47 del script `wav2lp.sh`).
  
Para obtener un fichero fmatrix, primero se calcula el número de columnas se calcula a partir del orden del predictor
lineal (uno más el orden del predictor) ya que en el primer elemento del vector se almacena la ganancia de
predicción. 
El número de filas (igual al número de tramas) depende de la longitud de la señal, la longitud y desplazamiento de la ventana, y la cadena
de comandos que se ejecutan para obtener la parametrización. Por eso, extraeremos la información del fichero obtenido. Lo hacemos convirtiendo la señal parametrizada a 
texto, usando sox +fa, y contando el número de líneas, con el comando de UNIX wc -l.


  * ¿Por qué es conveniente usar este formato (u otro parecido)? Tenga en cuenta cuál es el formato de
    entrada y cuál es el de resultado.
    
  Este formato nos permite tener bien organizados los datos del fichero de tal manera que cada fila es una trama y las columnas son 
  los coeficientes de las señales. De esta manera tambien nos sera facil comparar los resultados distintos para un mismo coeficiente.
    
- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:
  ```
  sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 |
	$LPC -l 240 -m $lpc_order | $LPCC -m $lpc_order -M $lpcc_order > $base.lpcc
  ```
  
- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en su
  fichero <code>scripts/wav2mfcc.sh</code>:
```
sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 |
	$MFCC -l 240 -m $mfcc_order -n $melbank_order > $base.mfcc
```


### Extracción de características.

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para todas las señales de un locutor.
  
  ![GMM_LP](https://user-images.githubusercontent.com/100692201/170885927-d52860c4-ece3-4e55-a0e0-84165e6756e4.jpeg)
![GMM_LPCC](https://user-images.githubusercontent.com/100692201/170885977-ac50d4ee-65a4-4b22-aa6f-b0ec343ee4d7.jpeg)
![GMM_MFCC](https://user-images.githubusercontent.com/100692201/170885984-bae382ba-4fe1-4a4e-b373-e8b68642437a.jpeg)

  + Indique **todas** las órdenes necesarias para obtener las gráficas a partir de las señales 
    parametrizadas.
    ```
     plot_gmm_feat -x 2 -y 3 -p 99,50,10,1 -g green work/gmm/lp/SES011.gmm work/lp/BLOCK01/SES011/*
      plot_gmm_feat -x 2 -y 3 -p 99,50,10,1 -g green work/gmm/lpcc/SES011.gmm work/lpcc/BLOCK01/SES011/*
      plot_gmm_feat -x 2 -y 3 -p 99,50,10,1 -g green work/gmm/mfcc/SES011.gmm work/mfcc/BLOCK01/SES011/*
    ```
  + ¿Cuál de ellas le parece que contiene más información?

- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3 para un locutor, y rellene la tabla siguiente con los valores obtenidos.

  |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | &rho;<sub>x</sub>[2,3] |      |      |      |
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.
  
- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?

### Entrenamiento y visualización de los GMM.

Complete el código necesario para entrenar modelos GMM.

- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.
  
- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.

### Reconocimiento del locutor.

Complete el código necesario para realizar reconociminto del locutor y optimice sus parámetros.

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.

### Verificación del locutor.

Complete el código necesario para realizar verificación del locutor y optimice sus parámetros.

- Inserte una tabla con el *score* obtenido con su mejor sistema de verificación del locutor en la tarea
  de verificación de SPEECON. La tabla debe incluir el umbral óptimo, el número de falsas alarmas y de
  pérdidas, y el score obtenido usando la parametrización que mejor resultado le hubiera dado en la tarea
  de reconocimiento.
 
### Test final

- Adjunte, en el repositorio de la práctica, los ficheros `class_test.log` y `verif_test.log` 
  correspondientes a la evaluación *ciega* final.

### Trabajo de ampliación.

- Recuerde enviar a Atenea un fichero en formato zip o tgz con la memoria (en formato PDF) con el trabajo 
  realizado como ampliación, así como los ficheros `class_ampl.log` y/o `verif_ampl.log`, obtenidos como 
  resultado del mismo.
