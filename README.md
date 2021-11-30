PAV - P3: detección de pitch
============================

Esta práctica se distribuye a través del repositorio GitHub [Práctica 3](https://github.com/albino-pav/P3).
Siga las instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para realizar un `fork` de la
misma y distribuir copias locales (*clones*) del mismo a los distintos integrantes del grupo de prácticas.

Recuerde realizar el *pull request* al repositorio original una vez completada la práctica.

Ejercicios básicos
------------------

- Complete el código de los ficheros necesarios para realizar la detección de pitch usando el programa
  `get_pitch`.

   * Complete el cálculo de la autocorrelación e inserte a continuación el código correspondiente.

      La función de autocorrelación se define como la correlación cruzada de la señal consigo misma.

      ```cpp
      void PitchAnalyzer::autocorrelation(const vector<float> &x, vector<float> &r) const {

      for (unsigned int l = 0; l < r.size(); ++l) { //r es un vector perque te size
  		  r[l]=0;
        for (unsigned int n=0; n < x.size() -l; n++) {
          r[l] += x[n] * x[n+l];
        }
        r[1] = r[1]/x.size();
      }
      if (r[0] == 0.0F) //to avoid log() and divide zero 
        r[0] = 1e-10; 
      }
      ```

   * Inserte una gŕafica donde, en un *subplot*, se vea con claridad la señal temporal de un segmento de
     unos 30 ms de un fonema sonoro y su periodo de pitch; y, en otro *subplot*, se vea con claridad la autocorrelación de la señal y la posición del primer máximo secundario.

	 NOTA: es más que probable que tenga que usar Python, Octave/MATLAB u otro programa semejante para
	 hacerlo. Se valorará la utilización de la librería matplotlib de Python.

      ![WhatsApp Image 2021-11-30 at 16 01 13](https://user-images.githubusercontent.com/91891304/144074746-94464248-0e68-4cf0-80d1-89f1acf253a6.jpeg)

      El código utilizado, haciendo uso de la librería matplotlib de Python, para hacer las gráficas es el siguiente:
      
      ```cpp
      senyal, fm = sf.read('pav434_1.wav')

      t = np.arange(0, len(senyal)) / fm

      plt.subplot(2,1,1)
      plt.title('Señal temporal')
      plt.plot(t, senyal)

      plt.subplot(2,1,2)
      plt.title('Autocorrelación')
      plt.acorr(senyal, maxlags=20)

      plt.tight_layout(pad=3.0)
      plt.show()
      ```

   * Determine el mejor candidato para el periodo de pitch localizando el primer máximo secundario de la
     autocorrelación. Inserte a continuación el código correspondiente.
 
 	1. Recorremos todos los valores para encontrar el primer máximo secundario de la correlación.
     	2. Comprobamos si el valor almacenado es mayor al valor actual dentro del bucle.
     	3. Si esto se cumple, guardamos la posición actual como máxima y calculamos la potencia de la señal, sabiendo que la potencia màxima coincide con la posición 0 de la 		autocorrelación.
     
      ```cpp
      vector<float>::const_iterator iR = r.begin(), iRMax = iR + npitch_min;
  
        for(iR = r.begin() + npitch_min; iR < r.begin() + npitch_max; iR++){
          if(*iR>*iRMax) {
          iRMax = iR;
          }
        }
        unsigned int lag = iRMax - r.begin();
  
        float pot = 10 * log10(r[0]);
      ```

   * Implemente la regla de decisión sonoro o sordo e inserte el código correspondiente.

   1. Comprobamos la poténcia de la señal. Si el sonido es sonoro tendrá una potencia significativamente mayor a si es sordo.
   2. Identificamos que la función de autocorrelación tiene dos máximos superiores a los umbrales predeterminados. Esto nos ayuda a encontrar el pitch ya que solo tiene 	sentido calcularlo en los tramos sonoros.

    ```cpp
     bool PitchAnalyzer::unvoiced(float pot, float r1norm, float rmaxnorm) const {

      if((rmaxnorm > UMBRAL_RMAXNORM || r1norm > UMBRAL_R1NORM) && pot > -UMBRAL_POT){
        return false;
      }
      else{
        return true; 
      }
    }
    ```
    Usando los umbrales siguientes:
    
    ![WhatsApp Image 2021-11-30 at 14 38 32](https://user-images.githubusercontent.com/91891304/144060115-e64e4a9a-36be-4b1f-978d-15d66141827f.jpeg)
    
- Una vez completados los puntos anteriores, dispondrá de una primera versión del detector de pitch. El 
  resto del trabajo consiste, básicamente, en obtener las mejores prestaciones posibles con él.

  * Utilice el programa `wavesurfer` para analizar las condiciones apropiadas para determinar si un
    segmento es sonoro o sordo. 
	
	- Inserte una gráfica con la detección de pitch incorporada a `wavesurfer` y, junto a ella, los principales candidatos para determinar la sonoridad de la voz: el 	nivel de potencia de la señal (r[0]), la autocorrelación normalizada de uno (r1norm = r[1] / r[0]) y el valor de la autocorrelación en su máximo secundario 			(rmaxnorm = r[lag] / r[0]).

	Puede considerar, también, la conveniencia de usar la tasa de cruces por cero.

	Recuerde configurar los paneles de datos para que el desplazamiento de ventana sea el adecuado, que en esta práctica es de 15 ms.
	
	La gráfica que hemos obtenido con WaveSurfer es la siguiente:
	
	(foto)
	
	Las gráficas de la imagen superior están en el siguiente orden (de arriba a abajo):
	1. Tasa de cruces por cero (ZCR)
	2. El valor de la autocorrelación en su máximo secundario
	3. Autocorrelación normalizada de uno
	4. El nivel de potencia de la señal
	5. Detector de Pitch
	6. Waveform de la señal.
	
	Observamos que se ha detectado correctamente la sonoridad de la voz para los candidatos y la detección del pitch es correcta.

  - Use el detector de pitch implementado en el programa `wavesurfer` en una señal de prueba y compare su resultado con el obtenido por la mejor versión de su propio sistema.
  -Inserte una gráfica ilustrativa del resultado de ambos detectores.
  
     La gráfica que hemos obtenido con WaveSurfer es la siguiente:
     
     (foto)
     
     Comparando las dos gráficas vemos que son bastante parecidas.
  
  * Optimice los parámetros de su sistema de detección de pitch e inserte una tabla con las tasas de error
    y el *score* TOTAL proporcionados por `pitch_evaluate` en la evaluación de la base de datos 
	  `pitch_db/train`..

    ![WhatsApp Image 2021-11-30 at 14 36 46](https://user-images.githubusercontent.com/91891304/144060569-5a3978be-1cba-4796-8789-5dc41e01941d.jpeg)
    
    Observamos que el score obtenido es de 89,07%, es un resultado bastante bueno, practicamente 90% sin aún mejoras.

  * Inserte una gráfica en la que se vea con claridad el resultado de su detector de pitch junto al del
    detector de Wavesurfer. Aunque puede usarse Wavesurfer para obtener la representación, se valorará
	  el uso de alternativas de mayor calidad (particularmente Python).

   	La siguiente captura se ha realizado con la señal "pav434_1.wav" que ya usamos en la práctica anterior.
   
   	(foto)

   	Observamos que se ha detectado correctamente la sonoridad de la voz en todos los tramos que corresponde y la detección del pitch es correcta.
	 
Aún así, se obtendrían mejores resultados aplicando un método de preprocesado y postprocesado de la señal. El método de preprocesado de la señal más utilizado es el center-clipping, que consiste en recortar los picos de la señal, para disminuir los errores en la detección. El método más usado de postprocesado de la señal más utilizado es el aplicar un filtro de mediana, para disminuir saltos en el pitch y errores en la detección, asignando a cada punto el valor de la mediana local, por lo que solo cambian los valores que no corresponden a la mediana de la muestra.

Ambas técnicas de preprocesado y postprocesado las vamos a implementar en la ampliación de esta práctica.
   

Ejercicios de ampliación
------------------------

- Usando la librería `docopt_cpp`, modifique el fichero `get_pitch.cpp` para incorporar los parámetros del
  detector a los argumentos de la línea de comandos.
  
  Esta técnica le resultará especialmente útil para optimizar los parámetros del detector. Recuerde que
  una parte importante de la evaluación recaerá en el resultado obtenido en la detección de pitch en la
  base de datos.

  * Inserte un *pantallazo* en el que se vea el mensaje de ayuda del programa y un ejemplo de utilización
    con los argumentos añadidos.

- Implemente las técnicas que considere oportunas para optimizar las prestaciones del sistema de detección
  de pitch.

  Entre las posibles mejoras, puede escoger una o más de las siguientes:

  * Técnicas de preprocesado: filtrado paso bajo, *center clipping*, etc.

	  ```cpp
	  float pow = 0;
	  for (unsigned int i = 0; i < x.size(); i++)
	  {
	    pow += x[i]*x[i];
	  }
	  pow /= x.size();

	  float umbral_clipping = 0.75 * pow;

	  for (unsigned int i = 0; i < x.size(); i++)
	  {
	    if (x[i] >= umbral_clipping)
	    {
	      x[i] -= umbral_clipping;
	    }else if (abs(x[i]) < umbral_clipping)
	    {
	      x[i]=0;
	    }else{
	      x[i] += umbral_clipping;
	    }
	  }
	  ```
	  Se ha añadido la libreria <math.h>
	  
	  El score obtenido es el siguiente:
	 
	  ![WhatsApp Image 2021-11-30 at 14 37 36](https://user-images.githubusercontent.com/91891304/144060792-03dee75b-f040-462a-aa75-9ab793877ea5.jpeg)

  	 Como el score es de 90.06%, vemos que añadir el central clipping ha mejorado la obtención del pitch.


  * Técnicas de postprocesado: filtro de mediana, *dynamic time warping*, etc.
  
  El código realizado es el siguiente, para un filtro de longitud 3:
  
  ```cpp
  vector<float> median_filter(3);

  for (unsigned int i = 2; i < f0.size(); i++)
  {
    median_filter = {f0[i-1],f0[i],f0[i+1]};
    sort(median_filter.begin(), median_filter.end());
    f0[i] = median_filter[1];
  }
  ```
  
  El score obtenido es el siguiente:
  
  ![WhatsApp Image 2021-11-30 at 14 38 15](https://user-images.githubusercontent.com/91891304/144060861-eb9ce24e-05ed-4f80-b6a9-5f24fe3bebc8.jpeg)
  
  Observamos que el resultado ha mejorado con un 90.66%, por tanto aplicar este filtro también mejora la detección del pitch.
  
  Finalmente hemos generado las siguientes gráficas para comparar el efecto del preprocesado y del postprocesado que hemos implementado en dos señales.
  En ambas gráficas el orden es el siguiente: 
    1. Detector de pitch de WaveSurfer
    2. Detector de pitch básico
    3. Detector de pitch ampliación
    4. Waveform de la señal.

    Para la señal "P3_a.wav":

    (foto)

    Para la señal "pav434_1.wav":

    (foto)

    En las gráficas podemos ver que las ampliaciones suavizan el contorno de pitch y eliminan algunos picos que si se apareciaban en la primera versión.


  * Métodos alternativos a la autocorrelación: procesado cepstral, *average magnitude difference function*
    (AMDF), etc.
  * Optimización **demostrable** de los parámetros que gobiernan el detector, en concreto, de los que
    gobiernan la decisión sonoro/sordo.
  * Cualquier otra técnica que se le pueda ocurrir o encuentre en la literatura.

  Encontrará más información acerca de estas técnicas en las [Transparencias del Curso](https://atenea.upc.edu/pluginfile.php/2908770/mod_resource/content/3/2b_PS%20Techniques.pdf)
  y en [Spoken Language Processing](https://discovery.upc.edu/iii/encore/record/C__Rb1233593?lang=cat).
  También encontrará más información en los anexos del enunciado de esta práctica.

  Incluya, a continuación, una explicación de las técnicas incorporadas al detector. Se valorará la
  inclusión de gráficas, tablas, código o cualquier otra cosa que ayude a comprender el trabajo realizado.

  También se valorará la realización de un estudio de los parámetros involucrados. Por ejemplo, si se opta
  por implementar el filtro de mediana, se valorará el análisis de los resultados obtenidos en función de
  la longitud del filtro.
   

Evaluación *ciega* del detector
-------------------------------

Antes de realizar el *pull request* debe asegurarse de que su repositorio contiene los ficheros necesarios
para compilar los programas correctamente ejecutando `make release`.

Con los ejecutables construidos de esta manera, los profesores de la asignatura procederán a evaluar el
detector con la parte de test de la base de datos (desconocida para los alumnos). Una parte importante de
la nota de la práctica recaerá en el resultado de esta evaluación.
