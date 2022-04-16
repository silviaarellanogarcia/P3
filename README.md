PAV - P3: estimación de pitch
=============================

Esta práctica se distribuye a través del repositorio GitHub [Práctica 3](https://github.com/albino-pav/P3).
Siga las instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para realizar un `fork` de la
misma y distribuir copias locales (*clones*) del mismo a los distintos integrantes del grupo de prácticas.

Recuerde realizar el *pull request* al repositorio original una vez completada la práctica.

Ejercicios básicos
------------------

- Complete el código de los ficheros necesarios para realizar la estimación de pitch usando el programa
  `get_pitch`.

   * Complete el cálculo de la autocorrelación e inserte a continuación el código correspondiente.

      ```c++
      for (unsigned int l = 0; l < r.size(); ++l) {
        r[l] = 0;
        for (unsigned int n = l; n < x.size(); n++){
          r[l] += x[n]*x[n-l];
        }
        r[l] /= x.size();
      }

      if (r[0] == 0.0F) //to avoid log() and divide zero 
        r[0] = 1e-10; 
      }
      ```

   * Inserte una gŕafica donde, en un *subplot*, se vea con claridad la señal temporal de un segmento de
     unos 30 ms de un fonema sonoro y su periodo de pitch; y, en otro *subplot*, se vea con claridad la
	 autocorrelación de la señal y la posición del primer máximo secundario.

	 NOTA: es más que probable que tenga que usar Python, Octave/MATLAB u otro programa semejante para
	 hacerlo. Se valorará la utilización de la biblioteca matplotlib de Python.

      <img src="imgs/all.jpg" width="640" align="center">


   * Determine el mejor candidato para el periodo de pitch localizando el primer máximo secundario de la
     autocorrelación. Inserte a continuación el código correspondiente.

     Como se puede observar en la imagen superior, el mejor candidato para el periodo de pitch es 4.8125ms. Este valor puede localizarse perfectamente en ambos dominios, pues en la gráfica temporal se aprecia claramente que cada periodo ocupa aproximadamente 5s, y en el dominio de la autocorrelación se distingue claramente el máximo, obteniendo el valor gracias al código realizado con Python mostrado a continuación.

     Código utilizado para el cálculo de la autocorrelación y su máximo fuera del origen.

     ```python
      corr = np.correlate(data,data,'full') / len(data)
      corr = corr[int(corr.size/2):]

      min_index = np.argmin(corr)
      max_index = np.argmax(corr[min_index:])
      max_value = np.max(corr[min_index:])
      fig, axs = plt.subplots(2)
      
      axs[0].plot(t, data)
      axs[1].plot(t, corr)
      axs[1].plot((min_index+max_index)*1000/samplerate,max_value,'ro', label='Temporal index = {}ms'.format((min_index+max_index)*1000/samplerate)) #MOSTRAR EL MÁXIMO DE LA AUTOCORRELACIÓN

      axs[0].set_title('Voiced frame')
      axs[1].set_title('Voiced frame autocorrelation')
      axs[1].set_xlabel('time (ms)')
      axs[1].legend()

      fig.tight_layout()
      plt.show()
      ```

   * Implemente la regla de decisión sonoro o sordo e inserte el código correspondiente.
   
      La regla de decisión se ha basado en 3 parámetros: la autocorrelación, la relación R(1)/R(0) y el valor de la potencia.

      ```c++
      if(rmaxnorm>umaxnorm && r1norm > r1thr && pot > powthr) return false;
      return true;
      ```

- Una vez completados los puntos anteriores, dispondrá de una primera versión del estimador de pitch. El 
  resto del trabajo consiste, básicamente, en obtener las mejores prestaciones posibles con él.

  * Utilice el programa `wavesurfer` para analizar las condiciones apropiadas para determinar si un
    segmento es sonoro o sordo. 
	
	  - Inserte una gráfica con la estimación de pitch incorporada a `wavesurfer` y, junto a ella, los 
	    principales candidatos para determinar la sonoridad de la voz: el nivel de potencia de la señal
		(r[0]), la autocorrelación normalizada de uno (r1norm = r[1] / r[0]) y el valor de la
		autocorrelación en su máximo secundario (rmaxnorm = r[lag] / r[0]).

		Puede considerar, también, la conveniencia de usar la tasa de cruces por cero.

	    Recuerde configurar los paneles de datos para que el desplazamiento de ventana sea el adecuado, que
		en esta práctica es de 15 ms.

        <img src="imgs/medidas.png" width="640" align="center">
        <img src="imgs/wavesurfer.png" width="640" align="center">
       Las gráficas hacen referencia al nivel de potencia de la señal
		(r[0]), la autocorrelación normalizada de uno (r1norm = r[1] / r[0]), el valor de la
		autocorrelación en su máximo secundario (rmaxnorm = r[lag] / r[0]) y el estimador de pitch implementado en el programa wavesurfer, respectivamente. Observamos que valores altos de estos parámetros corresponden a tramos sonoros.

      - Use el estimador de pitch implementado en el programa `wavesurfer` en una señal de prueba y compare
	    su resultado con el obtenido por la mejor versión de su propio sistema.  Inserte una gráfica
		ilustrativa del resultado de ambos estimadores.

		<img src="imgs/wavesurfer.png" width="640" align="center">
		<img src="imgs/output.png" width="640" align="center">
      Podemos observar que el estimador de pitch realizado es bastante preciso, ya que el contorno de pitch obtenido es prácticamente igual al del estimador de wavesurfer.
 
	Aunque puede usar el propio Wavesurfer para obtener la representación, se valorará
	 	el uso de alternativas de mayor calidad (particularmente Python).
  
  * Optimice los parámetros de su sistema de estimación de pitch e inserte una tabla con las tasas de error
    y el *score* TOTAL proporcionados por `pitch_evaluate` en la evaluación de la base de datos 
	`pitch_db/train`..
  
    El resultado final es el siguiente:
    <img src="imgs/results.png" width="640" align="center">

Ejercicios de ampliación
------------------------

- Usando la librería `docopt_cpp`, modifique el fichero `get_pitch.cpp` para incorporar los parámetros del
  estimador a los argumentos de la línea de comandos.
  
  Esta técnica le resultará especialmente útil para optimizar los parámetros del estimador. Recuerde que
  una parte importante de la evaluación recaerá en el resultado obtenido en la estimación de pitch en la
  base de datos.

  * Inserte un *pantallazo* en el que se vea el mensaje de ayuda del programa y un ejemplo de utilización
    con los argumentos añadidos.
    <img src="imgs/get_pitch.png" width="640" align="center">

- Implemente las técnicas que considere oportunas para optimizar las prestaciones del sistema de estimación
  de pitch.

  Entre las posibles mejoras, puede escoger una o más de las siguientes:

  * Técnicas de preprocesado: filtrado paso bajo, diezmado, *center clipping*, etc.
  * Técnicas de postprocesado: filtro de mediana, *dynamic time warping*, etc.
  * Métodos alternativos a la autocorrelación: procesado cepstral, *average magnitude difference function*
    (AMDF), etc.
  * Optimización **demostrable** de los parámetros que gobiernan el estimador, en concreto, de los que
    gobiernan la decisión sonoro/sordo.
  * Cualquier otra técnica que se le pueda ocurrir o encuentre en la literatura.

  Encontrará más información acerca de estas técnicas en las [Transparencias del Curso](https://atenea.upc.edu/pluginfile.php/2908770/mod_resource/content/3/2b_PS%20Techniques.pdf)
  y en [Spoken Language Processing](https://discovery.upc.edu/iii/encore/record/C__Rb1233593?lang=cat).
  También encontrará más información en los anexos del enunciado de esta práctica.

  Incluya, a continuación, una explicación de las técnicas incorporadas al estimador. Se valorará la
  inclusión de gráficas, tablas, código o cualquier otra cosa que ayude a comprender el trabajo realizado.

  También se valorará la realización de un estudio de los parámetros involucrados. Por ejemplo, si se opta
  por implementar el filtro de mediana, se valorará el análisis de los resultados obtenidos en función de
  la longitud del filtro.

  Se han implementado dos mejoras: un filtro de center-clipping y un filtro de mediana.

  **Center Clipping sin offset**
  No se haa realizado con un valor fijo, igual para todas las señales, sino que se adapta dependiendo de la potencia máxima de cada una. Aplicamos un center-clipping sin offset porque en nuestro caso obtenemos unos resultados mejores.

  ``` c++
  float max = *std::max_element(x.begin(), x.end());
  for(int i = 0; i < (int)x.size(); i++) {
    if(abs(x[i]) < cclip1*max) {
      x[i] = 0.0F;
    } 
  }
  ```
   **Filtro de Mediana**

  En nuestro caso, el filtro de mediana trabaja mejor con 3 valores. Eso tiene sentido, pues hay fragmentos sonoros de la trama que son muy cortos, por lo que para que tenga efecto en ese caso es mejor comparar con muestras contiguas.
  ``` c++
  vector<float> f0_final(f0.size());
  vector<float> temp(3);
  int i;
  for(i = 1; i < (int)(f0.size() - 1); i++) {
    temp = {f0[i-1], f0[i], f0[i+1]};
    auto m = temp.begin() + temp.size()/2;
    std::nth_element(temp.begin(), m, temp.end());
    f0_final[i] = temp[temp.size()/2];
  }
  f0_final[i] = f0_final[i-1];
  f0_final[0] = f0_final[1];
  ```


Evaluación *ciega* del estimador
-------------------------------

Antes de realizar el *pull request* debe asegurarse de que su repositorio contiene los ficheros necesarios
para compilar los programas correctamente ejecutando `make release`.

Con los ejecutables construidos de esta manera, los profesores de la asignatura procederán a evaluar el
estimador con la parte de test de la base de datos (desconocida para los alumnos). Una parte importante de
la nota de la práctica recaerá en el resultado de esta evaluación.
