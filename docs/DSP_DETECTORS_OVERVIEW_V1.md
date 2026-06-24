# DSP_DETECTORS_OVERVIEW_V1

Este documento desglosa y deconstruye el "modelo mental" y la lógica de razonamiento del motor DSP de **Vostok Restoration** de manera conceptual, traduciendo la implementación en Rust (`detectors.rs`) hacia principios físicos, acústicos y matemáticos universales. Su objetivo es servir como manual teórico de referencia para los investigadores de Vostok ML Research Lab.

---

# 1. Detector: Click

## Identidad del Detector
*   **Nombre:** Detector de Clicks e Impulsos Rápidos.
*   **Tipo de artefacto objetivo:** Clicks y transitorios impulsivos ultracortos.
*   **Naturaleza física:** Ruido impulsivo de banda ancha provocado por descargas electrostáticas, partículas de polvo en soportes mecánicos (vinilos) o pérdidas locales de sincronización digital de muy corta duración. Físicamente, se manifiesta como una discontinuidad abrupta que rompe la continuidad de fase y amplitud del audio de fondo.

## Modelo Mental del Detector
*   **Qué asume sobre el artefacto:** Asume que un click es una anomalía de alta energía de muy corta duración (típicamente $< 6$ muestras) que no puede ser predicha por la historia inmediata del audio armónico (música o habla).
*   **Qué evidencia busca:** Busca un incremento severo y abrupto en el **error de predicción lineal (LPC)**.
*   **Qué características considera relevantes:**
    *   La impredictibilidad estadística de la amplitud de la muestra.
    *   La "blancura espectral" (distribución de energía plana en altas frecuencias).
    *   La dirección del gradiente del impulso (cambio repentino del signo del gradiente).
*   **Qué características ignora:** Ignora transitorios musicales coherentes (como percusión armónica fina o sibilancias naturales de la voz) mediante compuertas de fase y regularización por cruces por cero.

## Dominio de Análisis
*   **Híbrido (Temporal y Frecuencial):** Utiliza un modelo autoregresivo temporal (LPC) para capturar la impredictibilidad de la muestra, pero valida la sospecha en el dominio espectral (STFT) evaluando la distribución de energía en alta frecuencia para evitar falsos positivos en instrumentos musicales transitorios.

## Flujo Conceptual de Detección
1.  **Filtrado por Energía RMS:** Ignora bloques de audio con energía extremadamente baja (silencios o colas de ruido por debajo de $-80\text{ dB}$) para evitar falsos positivos en el vacío.
2.  **Modelado Autoregresivo (LPC):** Analiza el bloque de audio utilizando un filtro adaptativo de predicción lineal de orden 16. Este filtro "aprende" la trayectoria continua del habla o la música.
3.  **Cálculo del Residuo:** Calcula la señal residual $e[n]$ (la diferencia entre la muestra real y la predicción del modelo). En audio limpio, el residuo es cercano a cero.
4.  **Detección de Anomalías Estadísticas:** Si el residuo supera un umbral adaptativo local (basado en la desviación estándar del bloque multiplicada por un factor dinámico de sensibilidad), se marca la muestra como "sospechosa".
5.  **Filtro de Coherencia Armónica y Sibilancia (Voz):**
    *   Si está en modo *Voz*, mide la tasa de cruce por cero (ZCR). Si el ZCR es alto y coincide con sibilancias naturales (como el fonema /s/), descarta el sospechoso.
    *   Si está en modo *Música*, analiza la coherencia de fase para descartar ataques percusivos legítimos.
6.  **Validación de Blancura Espectral:** Compara la energía por encima de $4\text{ kHz}$ contra la energía espectral total. Un click real debe poseer un espectro de banda ancha casi plano (alta blancura espectral). Si la energía de alta frecuencia es insuficiente, el sospechoso es descartado por "coloreado".
7.  **Cómputo del Score de Confianza:** Acumula evidencia sobre tres ejes: estadística LPC ($40\%$), blancura espectral ($30\%$) y discontinuidad de gradiente de fase ($30\%$). Solo si la confianza acumulada es $\ge 70\%$, el evento es emitido.

## Relación con la Infraestructura Experimental
*   **Interacción con el Degradador:** El click sintético es un impulso de una sola muestra, lo que genera un error de predicción LPC matemáticamente infinito y una blancura espectral perfecta del $100\%$. El detector está sobreajustado para encontrar este caso de laboratorio ideal.
*   **Diferencias frente a lo Real:** Un click analógico real sufre dispersión de fase y atenuación de alta frecuencia debido a la capacitancia física del soporte de reproducción. Su energía no es una Delta de Dirac pura, sino que posee "colas" oscilatorias cortas. El modelo de predicción LPC de orden 16 actual podría modelar erróneamente un click analógico "ancho" como parte legítima de la señal de audio, provocando Falsos Negativos.

## Fortalezas Aparentes
*   **Excelente precisión en voces limpias** y grabaciones con transitorios musicales suaves (instrumentos de cuerda frotada, flautas).
*   El sistema de **Score de Confianza de triple entrada** reduce de forma masiva los falsos positivos transitorios.

## Limitaciones Aparentes
*   Puede presentar **Falsos Negativos masivos** en clicks analógicos anchos o deformados por filtros pasabajos de equipos de cinta históricos.
*   En música percusiva densa (baterías, metales), las constantes mágicas de "blancura espectral" pueden confundir platillos legítimos con clicks de alta sensibilidad.

## Preguntas Abiertas para Investigación
*   *¿Podemos usar un clasificador convolucional ligero (CNN) para aprender la forma espectral analógica 2D de un click real en lugar de depender de la suposición rígida de blancura espectral plana por encima de 4 kHz?*

---

# 2. Detector: Pop

## Identidad del Detector
*   **Nombre:** Detector de Pops y Thumps de Baja Frecuencia.
*   **Tipo de artefacto objetivo:** Pops, impactos físicos de baja frecuencia y plosivas de micrófono.
*   **Naturaleza física:** Energía acústica concentrada en el espectro sub-grave ($< 120\text{ Hz}$) provocada por golpes mecánicos en el giradiscos, cortes físicos en cinta magnética o explosiones de aire sobre el diafragma de un micrófono (plosivas vocales). Físicamente, se comporta como un pulso sinusoidal amortiguado de baja frecuencia con un rápido decaimiento exponencial.

## Modelo Mental del Detector
*   **Qué asume sobre el artefacto:** Asume que un pop es una erupción súbita y localizada de energía subsónica que dura entre $15\text{ ms}$ y $60\text{ ms}$, la cual rompe de forma transitoria la continuidad de la base rítmica o tonal grave de fondo.
*   **Qué evidencia busca:** Busca un incremento relativo violento en la energía espectral acumulada entre los $20\text{ Hz}$ y los $120\text{ Hz}$ al compararla con la historia espectral inmediata.
*   **Qué características considera relevantes:**
    *   La concentración de energía espectral sub-grave.
    *   La duración temporal estricta medida en frames de la STFT (debe durar entre 1 y 3 frames de la STFT).
    *   La tasa de cruce por cero (ZCR) en modo voz para descartar sibilancias explosivas.
*   **Qué características ignora:** Ignora la fase detallada y la energía espectral por encima de los $200\text{ Hz}$.

## Dominio de Análisis
*   **Frecuencial (STFT):** Utiliza bins específicos de baja frecuencia de la STFT para aislar el fenómeno, y evalúa el historial dinámico espectral frame a frame para evitar falsos positivos con graves continuos de la música.

## Flujo Conceptual de Detección
1.  **Cálculo de Energía Sub-grave:** Para cada frame de la STFT, calcula la suma de energía espectral acumulada estrictamente entre los $20\text{ Hz}$ y los $120\text{ Hz}$ (omitiendo el bin 0/DC para evitar desviaciones por continua).
2.  **Evaluación de Contexto Histórico:** Calcula la energía promedio en el mismo rango espectral grave de los 8 frames precedentes (ventana de contexto de aproximadamente $185\text{ ms}$).
3.  **Cálculo del Ratio de Incremento:** Compara la energía grave del frame actual contra el promedio histórico. Si el ratio supera un umbral adaptativo regulado por la sensibilidad (ej: de $3.0\text{x}$ a $5.0\text{x}$), el frame se marca como "sospechoso de pop".
4.  **Filtro de Descarte de Plosivas de Voz:** Si el motor está en modo *Voz*, evalúa la tasa de cruce por cero (ZCR). Si el ZCR es alto, el sospechoso es descartado por considerarse una sibilancia natural de la voz o liberación normal de aire del tracto vocal, y no un pop de cinta magnética.
5.  **Validación de Duración Temporal:** Agrupa frames contiguos sospechosos. Si el evento acumulado dura entre 1 y 3 frames de la STFT (apróx. de $15\text{ ms}$ a $60\text{ ms}$), se emite como un Pop. Si la duración es mayor, se descarta por considerarse una nota de bajo sostenida, un bombo regular o un hum persistente.

## Relación con la Infraestructura Experimental
*   **Interacción con el Degradador:** El pop sintético es modelado precisamente como una sinusoide amortiguada de baja frecuencia con decaimiento exponencial que dura decenas de milisegundos.
*   **Alineación:** El detector de pops está perfectamente alineado con este modelo de inyección sintética, lo que explica su desempeño sobresaliente en pruebas de laboratorio.
*   **Diferencias frente a lo Real:** En grabaciones de campo complejas, un golpe o pop físico real puede inducir clipping o saturación secundaria en las frecuencias altas. El detector de pops, al ignorar la energía por encima de los $120\text{ Hz}$, es inmune a esta distorsión colateral, pero podría fallar en clasificar correctamente un impacto de banda ancha complejo (un golpe que abarca todo el espectro), marcándolo erróneamente solo como click o ignorándolo debido a la restricción rígida de duración corta de frames.

## Fortalezas Aparentes
*   **Alta inmunidad a bajos e instrumentos graves continuos** (como contrabajos o sintetizadores) gracias a la restricción rígida de duración temporal máxima de 3 frames ($60\text{ ms}$).
*   La diferenciación en modo *Voz* basada en ZCR es sumamente efectiva para evitar falsos positivos con plosivas sibilantes naturales.

## Limitaciones Aparentes
*   Incapacidad de detectar pops reales masivos de larga duración (ej: golpes mecánicos que provocan una resonancia acústica larga de más de $150\text{ ms}$ en la sala de grabación), los cuales son descartados sistemáticamente por superar la restricción de duración de 3 frames.
*   Depende de un tamaño de ventana STFT fijo de 4096 muestras; a tasas de muestreo distintas (ej: $96\text{ kHz}$ o $192\text{ kHz}$), el mapeo físico de los bins de frecuencia se altera, requiriendo recalibración.

## Preguntas Abiertas para Investigación
*   *¿Cómo se comportan las firmas de Pops cuando coexisten simultáneamente con Clipping? ¿Se distorsiona el balance de baja frecuencia de la STFT provocando Falsos Negativos?*

---

# 3. Detector: Hum

## Identidad del Detector
*   **Nombre:** Detector de Hum Electromagnético y Tonos de Red.
*   **Tipo de artefacto objetivo:** Hum estacionario en 50/60 Hz y armónicos principales.
*   **Naturaleza física:** Interferencia electromagnética continua inducida por la corriente alterna de la red eléctrica en los cables y preamplificadores (típicamente $50\text{ Hz}$ en Europa/América del Sur y $60\text{ Hz}$ en América del Norte, junto con sus armónicos superiores en $100/120\text{ Hz}$, $150/180\text{ Hz}$, etc.). Físicamente, se comporta como un conjunto de ondas sinusoidales puras, ultraestables y de muy larga duración temporal.

## Modelo Mental del Detector
*   **Qué asume sobre el artefacto:** Asume que el hum es una interferencia sinusoidal pura y estática en frecuencias exactas prefijadas, cuya amplitud permanece excepcionalmente estable a lo largo del tiempo y supera drásticamente al ruido de fondo de las frecuencias vecinas.
*   **Qué evidencia busca:** Busca "picos" espectrales hiper-pronunciados y estables en los bins correspondientes a 50, 60, 100, 120, 150 y $180\text{ Hz}$.
*   **Qué características considera relevantes:**
    *   La relación de amplitud entre el bin objetivo y sus bins vecinos (excluyendo la fuga espectral inmediata).
    *   La estabilidad temporal de la amplitud (la fluctuación debe ser mínima a lo largo de los frames).
    *   La duración mínima (debe durar al menos $1.5$ segundos de forma ininterrumpida).
*   **Qué características ignora:** Ignora por completo cualquier fenómeno dinámico rápido o transitorio que ocurra en esas mismas frecuencias (como notas musicales rápidas de paso).

## Dominio de Análisis
*   **Frecuencial (STFT):** Utiliza bins ultra-específicos de la STFT de ventana larga ($N_{fft} = 4096$) para lograr la resolución de frecuencia necesaria en graves, evaluando la estabilidad en la dimensión temporal de la matriz.

## Flujo Conceptual de Detección
1.  **Mapeo de Bins Objetivo:** Calcula cuáles son los bins de frecuencia exactos en la STFT correspondientes a $50, 60, 100, 120, 150\text{ y } 180\text{ Hz}$ según la tasa de muestreo actual.
2.  **Cálculo de Contraste Espectral (Vecindario):** Para cada frame, evalúa la magnitud del bin objetivo y la compara contra el promedio de sus vecinos espectrales lejanos (ej: $bin-3$, $bin-2$, $bin+2$ y $bin+3$), ignorando deliberadamente los vecinos inmediatos ($bin-1$ y $bin+1$) para descartar la fuga espectral natural de la ventana de Hann.
3.  **Filtrado por Relación de Pico:** Si el bin objetivo supera a sus vecinos por un factor dinámico de contraste (de $3.0\text{x}$ a $6.0\text{x}$, equivalente a un resalte de $+9.5\text{ dB}$ a $+15.6\text{ dB}$) y tiene una amplitud audible mínima ($> 0.001$), se marca el frame como "sospechoso de hum".
4.  **Consolidación y Tolerancia de Huecos (Gap Tolerance):** Agrupa los frames sospechosos continuos permitiendo una tolerancia de pérdida (dropout temporal del hum) de hasta 15 frames ($\sim 348\text{ ms}$) antes de segmentar el evento.
5.  **Filtro de Duración Estacionaria:** Exige que el evento dure al menos 70 frames continuos ($\sim 1.5$ segundos). Esto discrimina de forma infalible notas de paso de instrumentos musicales reales (como bajos o violonchelos).
6.  **Filtro de Estabilidad de Amplitud (Vibrato/Voz):** Si está en modo *Voz*, calcula el ratio entre la amplitud mínima y máxima del hum en el tiempo. Un hum real es estático (ratio de estabilidad $> 25\%$-$40\%$). Si hay vibrato o fluctuaciones naturales del habla, se descarta por considerarse una vocal sostenida y no ruido eléctrico.

## Relación con la Infraestructura Experimental
*   **Interacción con el Degradador:** El hum sintético es inyectado como una fundamental exacta de $50\text{ Hz}$ con armónicos estáticos y envolventes suaves de fade-in/out.
*   **Alineación:** El detector de hum está completamente alineado con la física del inyector sintético, lo que resulta en una precisión del $100\%$ sobre las pruebas controladas de laboratorio.
*   **Diferencias frente a lo Real:** En grabaciones históricas reales de cinta, la frecuencia de red puede sufrir variaciones lentas de velocidad debido a la inestabilidad de los motores analógicos (*Wow & Flutter* o deriva térmica). Esto dispersa la frecuencia del hum fuera de los bins fijos del detector, lo que provocaría que el pico espectral de $50\text{ Hz}$ "baile" lateralmente y el detector clásico de bins fijos sufra de **Falsos Negativos** severos al no encontrar el pico en el bin central exacto.

## Fortalezas Aparentes
*   **Excepcional precisión** al aislar hum eléctrico real frente a notas graves musicales de instrumentos gracias al filtro doble de duración de $1.5$ segundos y al ratio de estabilidad de amplitud.
*   Inmune a falsos positivos por voces humanas graves (vocales continuas) gracias al filtro de estabilidad que detecta el vibrato natural del habla.

## Limitaciones Aparentes
*   **Ceguera ante el "Wow & Flutter" (Fluctuaciones de velocidad):** Si el hum real fluctúa incluso en $\pm 1.5\text{ Hz}$ debido al arrastre de cinta analógica, el detector perderá el pico espectral central.
*   Ignora hums de frecuencias diferentes a los presets fijos (ej: interferencias de fuentes conmutadas a alta frecuencia o interferencias en $400\text{ Hz}$ típicas de equipos de aviación antiguos).

## Preguntas Abiertas para Investigación
*   *¿Podemos entrenar un clasificador ML que aprenda la firma "armónica paralela" del Hum, detectándolo de manera adaptativa incluso si su frecuencia fundamental fluctúa dinámicamente en el tiempo (Tracking adaptativo de Hum con Wow & Flutter)?*

---

# 4. Detector: Hiss

## Identidad del Detector
*   **Nombre:** Detector de Hiss y Soplido de Banda Ancha.
*   **Tipo de artefacto objetivo:** Siseo continuo y ruido térmico de alta frecuencia.
*   **Naturaleza física:** Ruido térmico constante generado por los componentes analógicos activos (válvulas, transistores) y la fricción de partículas magnéticas en cintas analógicas de archivo. Físicamente, se comporta como un ruido gaussiano de distribución espectral casi plana (ruido blanco/rosa) concentrado de forma masiva por encima de los $5\text{ kHz}$.

## Modelo Mental del Detector
*   **Qué asume sobre el artefacto:** Asume que el hiss es un ruido constante de banda ancha de alta frecuencia, cuya amplitud es uniforme en el tiempo y presenta una planeidad espectral (*spectral flatness*) excepcionalmente alta en las frecuencias agudas.
*   **Qué evidencia busca:** Busca una planeidad espectral alta simultánea a nivel global de frame y localmente en el rango de $5\text{ kHz}$ a $22\text{ kHz}$.
*   **Qué características considera relevantes:**
    *   La planeidad espectral en alta frecuencia (cercana a 1.0, correspondiente a ruido blanco puro).
    *   La energía RMS global del frame (debe ser audible, $> 0.01$).
    *   La duración ininterrumpida de muy larga escala (debe durar al menos $2.0$ segundos).
*   **Qué características ignora:** Ignora transitorios agudos e impulsivos rápidos (como platillos transitorios o sibilancias cortas).

## Dominio de Análisis
*   **Frecuencial (STFT):** Evalúa de forma estricta los bins espectrales de alta frecuencia ($> 5\text{ kHz}$) y calcula la relación matemática entre su media geométrica y su media aritmética (definición de planeidad espectral) frame a frame.

## Flujo Conceptual de Detección
1.  **Segmentación Espectral Aguda:** Aísla de forma estricta los bins de la STFT por encima de los $5\text{ kHz}$ (frecuencias agudas).
2.  **Cálculo de Planeidad Espectral Local:** Para cada frame, calcula la planeidad espectral (*spectral flatness*) local de este segmento agudo (el cociente entre la media geométrica y la aritmética de las magnitudes). Una planeidad alta ($> 0.35$-$0.45$) indica que el espectro es plano y carece de tonos musicales armónicos coloreados en agudos.
3.  **Filtro de Energía Audible:** Verifica que el frame tenga energía RMS global mínima ($> 0.01$) para no marcar falsos positivos en zonas de silencio digital puro de cinta.
4.  **Detección de Coincidencia Doble:** Si tanto la planeidad espectral local (agudos) como la planeidad global del frame superan el umbral adaptativo regulado por la sensibilidad, el frame se marca como "sospechoso de hiss".
5.  **Consolidación con Gran Tolerancia:** Agrupa frames contiguos sospechosos permitiendo una tolerancia de pérdida (enmascaramiento del hiss por voces o música fuerte) de hasta 30 frames ($\sim 696\text{ ms}$). Esto consolida el hiss como un ruido de fondo que de hecho sigue existiendo por debajo de los instrumentos.
6.  **Filtro de Larga Duración:** Exige que el evento consolidado tenga una duración mínima de al menos 90 frames ($\sim 2.0$ segundos). Esto discrimina de forma infalible platillos de batería transitorios, sibilancias de voz naturales prolongadas o respiraciones.

## Relación con la Infraestructura Experimental
*   **Interacción con el Degradador:** El hiss sintético se inyecta como ruido gaussiano con segmentaciones y envolventes suaves de fade-in/out.
*   **Alineación:** El detector de hiss está altamente alineado con este modelo de inyección sintética, lo que explica su desempeño del $100\%$ en benchmarks de laboratorio.
*   **Diferencias frente a lo Real:** El hiss de cinta analógica real rara vez es puramente gaussiano plano (blanco); suele sufrir atenuaciones dependientes de la ecualización de cinta (ruido rosa o coloreado por curvas NAB/CCIR). El cálculo rígido de la planeidad espectral matemática pura podría fallar ante ruidos de hiss de cinta real que tengan una pendiente de caída espectral muy pronunciada, marcándolos como "coloreados" y provocando **Falsos Negativos**.

## Fortalezas Aparentes
*   **Excepcional discriminación** de sibilancias del habla y transitorios musicales agudos (platillos, campanas) gracias al riguroso filtro de duración de $2.0$ segundos y doble planeidad espectral.
*   La tolerancia de huecos de casi $700\text{ ms}$ es acústicamente idónea para mantener el reporte de hiss continuo incluso cuando es enmascarado temporalmente por transitorios fuertes.

## Limitaciones Aparentes
*   **Vulnerabilidad al Hiss Coloreado (Ruido Rosa/Marrón):** Si el siseo analógico real está fuertemente filtrado por la respuesta acústica del equipo y presenta una pendiente de caída, su planeidad espectral geométrica/aritmética decaerá por debajo del umbral, provocando que el detector sea ciego ante él.

## Preguntas Abiertas para Investigación
*   *¿Podemos usar técnicas de estimación de piso de ruido basadas en autoencoders de ML no supervisados para aprender el perfil de ruido espectral continuo de una cinta real, sin asumir que el hiss deba ser plano de alta frecuencia?*

---

# 5. Detector: Dropout

## Identidad del Detector
*   **Nombre:** Detector de Caídas de Señal y Dropouts.
*   **Tipo de artefacto objetivo:** Dropouts, pérdidas temporales de soporte y silenciamientos accidentales.
*   **Naturaleza física:** Pérdida súbita, abrupta y severa de la amplitud de la señal de audio provocada por el desprendimiento físico de la capa de óxido magnético de la cinta analógica, arrugas físicas en el soporte, o fallas en la transmisión del flujo de datos digital. Físicamente, se manifiesta como un decaimiento extremo de la energía global que ocurre a escala de milisegundos y que posteriormente se recupera de manera igualmente abrupta.

## Modelo Mental del Detector
*   **Qué asume sobre el artefacto:** Asume que un dropout es una pérdida transitoria de energía profunda que dura entre $4\text{ ms}$ y $150\text{ ms}$, la cual ocurre única y exclusivamente en medio de una zona que previamente estaba activa y con volumen normal (excluyendo así silencios naturales de la interpretación).
*   **Qué evidencia busca:** Busca una caída abrupta del RMS de corto plazo en comparación con la energía del contexto precedente de mediano plazo.
*   **Qué características considera relevantes:**
    *   La relación de energía (RMS relativo) entre un bloque corto ($2.5\text{ ms}$) y su contexto precedente ($150\text{ ms}$).
    *   La energía absoluta del bloque corto (debe ser menor a $0.001$, es decir, silencio casi absoluto).
    *   La energía absoluta del contexto precedente (debe ser una zona de señal activa, $> 10\%$ de la energía global y $> 0.0025$ absoluta).
    *   La duración temporal estricta (debe durar entre $4\text{ ms}$ y $150\text{ ms}$).
*   **Qué características ignora:** Ignora las propiedades espectrales y el contenido de fase de la señal.

## Dominio de Análisis
*   **Temporal Puro:** Utiliza el cálculo de energía RMS en el dominio del tiempo. Utiliza una implementación altamente optimizada de **suma cuadrática acumulada** (*integral image* temporal) para calcular el RMS de cualquier tamaño de bloque en tiempo constante $O(1)$, maximizando el rendimiento computacional.

## Flujo Conceptual de Detección
1.  **Cálculo del Historial de Suma Cuadrática:** Genera un vector acumulado del cuadrado de las muestras de audio. Esto permite calcular el RMS de cualquier segmento de audio restando solo dos índices del vector y aplicando raíz cuadrada.
2.  **Monitoreo del Bloque de Corto Plazo:** Desliza una ventana de análisis ultracorta de $2.5\text{ ms}$ a lo largo del audio.
3.  **Monitoreo del Contexto Precedente:** Evalúa una ventana de contexto de $150\text{ ms}$ inmediatamente anterior al bloque de corto plazo.
4.  **Verificación de Región Activa:** Valida si el contexto precedente era una "región activa" legítima (el RMS del contexto debe superar el $10\%$ del RMS global de todo el archivo y ser físicamente mayor a $0.0025$). Esto evita falsos positivos durante silencios musicales, pausas de habla o zonas de decaimiento natural de la reverberación.
5.  **Evaluación de la Caída Abrupta:** Compara el RMS del bloque de $2.5\text{ ms}$ contra el de su contexto de $150\text{ ms}$. Si la energía cae de forma abrupta por debajo de un umbral relativo regulado por la sensibilidad (ej: de un $3\%$ a un $10\%$ de la energía del contexto) y el RMS absoluto del bloque es menor a $0.001$ (un silencio profundo), el bloque se marca como "dropout activo".
6.  **Validación de Duración Temporal:** Al finalizar la región de caída, mide su duración. Si dura entre $4\text{ ms}$ y $150\text{ ms}$, se reporta como un Dropout. Si dura más, se asume que es una pausa musical o de locución legítima prolongada y se descarta.
7.  **Banda de Resguardo de Fin de Archivo (EOF Guard Band):** Si el dropout llega al final del archivo, solo se emite si hay un margen de resguardo de al menos $150\text{ ms}$ antes del final real de las muestras de audio. Esto evita falsos positivos provocados por el desvanecimiento (*fade-out*) natural al final de las canciones.

## Relación con la Infraestructura Experimental
*   **Interacción con el Degradador:** El dropout sintético es inyectado como una pérdida total de señal de corta duración con posiciones aleatorias.
*   **Alineación:** La lógica del detector está perfectamente alineada con este modelo de inyección matemática de caída vertical de ganancia, logrando una efectividad del $100\%$ en benchmarks controlados.
*   **Diferencias frente a lo Real:** En dropouts reales de cinta magnética analógica, la pérdida de señal no siempre es total ni vertical; a veces ocurre como una pérdida progresiva de altas frecuencias (*spectral dropout* debido a la pérdida de contacto del cabezal con la cinta por una arruga o mota de polvo) donde los graves sobreviven parcialmente. El detector temporal puro actual, al exigir una caída RMS absoluta profunda por debajo de $0.001$, es **completamente ciego ante dropouts espectrales reales** donde la amplitud global disminuye solo a la mitad pero los agudos se extinguen por completo.

## Fortalezas Aparentes
*   **Extrema eficiencia computacional** gracias al cálculo de RMS en $O(1)$ usando sumas acumuladas.
*   La lógica de **región activa** y la **banda de resguardo (EOF Guard Band)** de $150\text{ ms}$ evitan con éxito falsos positivos en pausas vocales y desvanecimientos finales de canciones de forma robusta.

## Limitaciones Aparentes
*   **Ceguera ante dropouts parciales o espectrales:** Incapaz de detectar pérdidas de contacto de cabezal que degraden severamente la brillantez del audio pero mantengan la energía de baja frecuencia por encima del umbral rígido de $0.001$.

## Preguntas Abiertas para Investigación
*   *¿Podemos utilizar descriptores espectrales (como el decaimiento abrupto del Spectral Centroid o Spectral Flux) para identificar dropouts parciales de alta frecuencia que el RMS temporal puro no logra capturar?*

---

# 6. Detector: Clipping

## Identidad del Detector
*   **Nombre:** Detector de Clipping y Saturación Digital.
*   **Tipo de artefacto objetivo:** Clipping duro, recorte de picos de señal y distorsión digital.
*   **Naturaleza física:** Limitación severa de la amplitud máxima de la señal provocada por un exceso de ganancia analógica o digital que supera el rango dinámico del sistema (por ejemplo, superar los $0\text{ dBFS}$ en un convertidor ADC o de punto fijo). Físicamente, se manifiesta en el dominio temporal como un "achatamiento" o meseta plana horizontal en la cúspide de la forma de onda.

## Modelo Mental del Detector
*   **Qué asume sobre el artefacto:** Asume que el clipping es la aparición consecutiva de picos planos de amplitud ultra-estable (mesetas horizontales) localizados en el nivel máximo absoluto de ganancia de la ventana de audio actual.
*   **Qué evidencia busca:** Busca secuencias consecutivas de muestras (al menos 2 consecutivas) que tengan exactamente la misma amplitud absoluta con una tolerancia infinitesimal.
*   **Qué características considera relevantes:**
    *   La aparición de muestras consecutivas con amplitudes casi idénticas (tolerancia $\le 0.015$).
    *   La proximidad al límite máximo de la ventana actual (el umbral adaptativo local debe estar por encima de $0.70$ y cerca del pico absoluto de la ventana: `max(0.70, peak * 0.95)`).
    *   La consolidación de múltiples picos distorsionados con una ventana de tolerancia a la separación (gap) de $100\text{ ms}$.
*   **Qué características ignora:** Ignora por completo el comportamiento armónico espectral de alta frecuencia derivado de la distorsión.

## Dominio de Análisis
*   **Temporal Puro:** Analiza la amplitud y diferencia matemática muestra a muestra en el dominio del tiempo de forma secuencial.

## Flujo Conceptual de Detección
1.  **Segmentación por Ventanas de 1 Segundo:** Divide el archivo de audio en bloques continuos de 1 segundo de duración.
2.  **Cálculo de Pico Absoluto Local:** Determina el pico absoluto de amplitud máxima dentro de la ventana de 1 segundo.
3.  **Cálculo de Límite Adaptativo Dinámico:** Establece el umbral de clipping adaptativo para esa ventana como el valor máximo entre $0.70$ absoluto y el $95\%$ del pico local (`max(0.70, local_peak * 0.95)`). Esto permite capturar clipping analógico o digital que haya sido atenuado en ganancia de volumen general en procesos posteriores.
4.  **Identificación de Muestras Planas (Mesetas):** Recorre las muestras. Si una muestra supera el umbral adaptativo, evalúa las siguientes. Si hay al menos 2 o más muestras consecutivas cuya diferencia absoluta de amplitud sea casi nula ($\le 0.015$), se marca el segmento como "muestras recortadas de clipping".
5.  **Consolidación por Gap de 100 ms:** Agrupa los picos de clipping individuales detectados. Si la distancia entre dos eventos de recorte es menor o igual a $100\text{ ms}$ de muestras, se consolidan dentro de una única región continua de distorsión (`GlitchType::Distortion`). Esto evita reportar miles de micro-eventos individuales de clipping (un reporte por cada ciclo de onda saturada), entregando en su lugar regiones lógicas continuas de distorsión.

## Relación con la Infraestructura Experimental
*   **Interacción con el Degradador:** El clipping sintético se genera aplicando ganancia previa e inyectando un umbral de recorte plano con transiciones suavizadas.
*   **Alineación:** El detector actual, al utilizar una tolerancia de meseta plana de $0.015$, logra absorber de manera exitosa el "suavizado" artificial del inyector sintético, reportando con precisión del $100\%$ el rango completo de la distorsión en pruebas de laboratorio.
*   **Diferencias frente a lo Real:** En escenarios reales, el audio saturado puede haber pasado posteriormente por un proceso de ecualización analógica (como un filtro pasabajos de cabezal o un preamplificador). La ecualización destruye la "planeidad horizontal" de las mesetas de clipping en el dominio temporal, inclinando y curvando las superficies planas de la onda. El detector temporal puro actual, al depender críticamente de la planeidad absoluta de muestras consecutivas, es **severamente vulnerable ante clipping ecualizado real**, sufriendo de **Falsos Negativos** masivos al no encontrar mesetas planas horizontales perfectas.

## Fortalezas Aparentes
*   **Umbral adaptativo local sumamente robusto:** Permite detectar clipping digital incluso si el archivo de audio fue bajado de volumen general posteriormente (ej: un clipping que ahora reside físicamente a $-6\text{ dB}$ de amplitud).
*   La **consolidación temporal de 100 ms** es excelente para agrupar micro-saturaciones de ciclos de onda continuos en una sola región lógica de distorsión para el usuario.

## Limitaciones Aparentes
*   **Vulnerabilidad a la Ecualización Posterior:** Si la distorsión digital fue filtrada o ecualizada analógicamente, las mesetas planas se deforman y curvan, haciendo que el detector de coincidencia temporal muestra a muestra falle por completo.

## Preguntas Abiertas para Investigación
*   *¿Podemos usar la distorsión armónica en altas frecuencias detectable en el espectrograma (aparición de armónicos de distorsión impares densos) como una firma híbrida para validar el clipping ecualizado que ha perdido su planeidad temporal en el osciloscopio?*

---

# Reflexión del Investigador

### 1. ¿Qué filosofía general parece existir detrás del conjunto completo de detectores?
La filosofía dominante detrás del motor DSP actual es una **heurística de base física de caja blanca, guiada por reglas deterministas fijas y parametrizada de manera pragmática**. 

El sistema asume que cada artefacto de audio posee una "firma física fundamental" y discreta que puede ser aislada de forma limpia modelando las propiedades acústicas teóricas ideales del evento. Es una filosofía de **ingeniería de control clásica**: se prefiere comprender físicamente la variable (LPC, ZCR, Flatness, RMS) antes que modelar de forma estadística de caja negra. Cada detector actúa como un filtro de decisiones estricto que intenta "descartar" de forma agresiva todo lo que parezca comportamiento normal de la música o la voz.

### 2. ¿Observas patrones comunes entre ellos?
Sí, existen tres patrones arquitectónicos y algorítmicos transversales muy marcados en el código de `detectors.rs`:
*   **Consolidación Temporal por Tolerancia de Vacíos (Gap Tolerance):** Prácticamente todos los detectores de largo alcance (Hum, Hiss, Clipping, Dropouts) implementan una máquina de estados con un contador de "huecos" (*gap counter* o *gap tolerance*). Esto reconoce acústicamente que el ruido de fondo es continuo y que puede ser "enmascarado" temporalmente por transitorios de alta energía de la música o del habla, por lo que el detector debe "mantener la memoria de detección" durante cientos de milisegundos antes de declarar que el artefacto ha terminado.
*   **Adaptabilidad Local en Ventanas de Corto Plazo:** Los detectores evitan usar umbrales globales fijos sobre todo el archivo de audio. En su lugar, todos calculan picos locales, RMS de bloque corto, o desviación estándar en ventanas móviles (ej: bloques de 1 segundo en Clipping, bloques de $2.5\text{ ms}$ y $150\text{ ms}$ en Dropouts, bloques de 8 frames en Pops). Esto garantiza que el motor se auto-calibre de forma dinámica ante las variaciones de volumen de la señal portadora.
*   **Diferenciación de Escenarios de Voz/Música (`AudioMode`):** Los detectores más propensos a falsos positivos (Clicks, Pops y Hum) bifurcan su lógica de umbrales y filtrado basándose en si el material es voz o música, utilizando el cruce por cero (ZCR) y la regularidad de amplitud para proteger las sibilancias y las variaciones naturales de las cuerdas vocales humanas.

### 3. ¿El sistema parece orientado a maximizar Recall, Precision o un equilibrio entre ambos?
El motor DSP actual está **fuertemente sesgado a maximizar la Precisión (*Precision*), sacrificando activamente el *Recall* (Sensibilidad)**.

**Evidencia en el código:**
El diseño de los detectores incluye múltiples etapas sucesivas de "compuertas de descarte agresivo" (ej: descartar clicks armónicamente coherentes, descartar clicks coloreados sin blancura espectral, exigir un score de confianza estricto de $\ge 70\%$, descartar pops de más de 3 frames, descartar hums que no duren $1.5$ segundos, exigir estabilidad estricta de magnitud, etc.).

**Justificación comercial/acústica:**
En la restauración de audio profesional, un falso positivo ($FP$) es catastrófico: significa "reparar quirúrgicamente" (dañar y atenuar) un pasaje de música o voz perfectamente limpio, introduciendo artefactos de fase o distorsión donde no había problemas. Por lo tanto, el diseño prefiere tener Falsos Negativos ($FN$, dejar pasar un click suave e inaudible) antes que cometer un Falso Positivo que mutile la señal original limpia. El benchmark oficial que reporta una precisión global de **$96.88\%$** y un recall de **$86.11\%$** es el reflejo directo de esta decisión metodológica de diseño.

### 4. ¿Qué detectores parecen más maduros conceptualmente?
Los detectores más maduros conceptualmente son:
*   **Click (LPC Autorregresivo):** Es una joya de procesamiento digital de señales de nivel SOTA. El uso del error de predicción de un filtro lineal autoregresivo de orden 16 mediante Levinson-Durbin es un enfoque clásico e impecable para modelar señales continuas y resaltar anomalías transitorias. El sistema de score de confianza multivariante añade una robustez científica ejemplar.
*   **Dropout (Suma Cuadrática Acumulada):** Conceptualmente maduro por su elegancia matemática. El uso de cálculo de RMS en $O(1)$ mediante un vector integral demuestra un profundo entendimiento de la optimización del rendimiento temporal en DSP, y la lógica de "región activa de contexto" está acústicamente muy bien fundamentada.

### 5. ¿Qué detectores parecen depender más de umbrales empíricos?
Los detectores que dependen más de constantes "mágicas" o umbrales empíricos son:
*   **Hiss (Detector Espectral):** Su lógica depende críticamente de que el soplido de cinta sea plano y de que se localice estrictamente por encima de los $5\text{ kHz}$. El umbral de planeidad espectral estático (ej: `0.45 - sensitivity * 0.15`) y la constante de $5\text{ kHz}$ son decisiones empíricas que asumen un ruido blanco de laboratorio, colapsando ante ruidos coloreados o de banda más estrecha.
*   **Clipping (Meseta Temporal):** Depende por completo de que el clipping sea perfectamente plano en muestras consecutivas (con una tolerancia empírica fija de $\le 0.015$). Es una heurística muy simplificada que ignora la deformación física real que sufre la meseta al pasar por cualquier filtro analógico o convertidor AC posterior.

### 6. ¿Qué oportunidades de Machine Learning observas?
El análisis completo de `detectors.rs` nos abre un abanico colosal de oportunidades para la investigación híbrida de Machine Learning dentro de nuestro laboratorio:

```text
                  [ SISTEMA HÍBRIDO DSP + ML ]
                               │
       ┌───────────────────────┴───────────────────────┐
       ▼                                               ▼
[ Clasificadores de Segunda Etapa ]           [ Aprendizaje de Firmas No Lineales ]
Usar redes ligeras (MLP o CNN) para            Modelar el Clipping Curvado o el Hum
validar los "Sospechosos" del DSP,             con Wow & Flutter que los filtros fijos
reduciendo FP sin perder Recall.               del DSP no pueden capturar por reglas.
```

1.  **Clasificadores Espectrales de Segunda Etapa (Validación de Sospechosos):**
    Dado que el DSP clásico sacrifica Recall para evitar falsos positivos mediante compuertas rígidas de descarte, podemos redefinir el sistema: relajar los umbrales del DSP clásico para maximizar el *Recall* (permitiendo que entren más sospechosos a la red) y utilizar un modelo de clasificación convolucional (CNN) o perceptrón multicapa (MLP) muy ligero para evaluar únicamente los segmentos sospechosos reportados por el DSP. El ML evaluaría el parche de espectrograma 2D del evento sospechoso para clasificarlo con alta precisión, logrando lo mejor de ambos mundos: alto recall sin penalizar la precisión.
2.  **Aprendizaje de Representaciones Espectrales de Clipping Ecualizado:**
    Entrenar una red neuronal para que aprenda a identificar las firmas de distorsión armónica de picos curvados (clipping ecualizado), una tarea que es imposible para el detector de mesetas temporales rígido del DSP pero muy viable para una CNN que analiza el espectrograma de magnitud de alta frecuencia.
3.  **Tracking Adaptativo de Hum Dinámico (Wow & Flutter Tracker):**
    Utilizar redes neuronales recurrentes (LSTM) o filtros de Kalman de ML para realizar el rastreo continuo de la frecuencia fundamental de hum electromagnético variante en el tiempo, adaptando dinámicamente los coeficientes del filtro Notch del restaurador clásico sin requerir presets estáticos de bins de frecuencia.
4.  **Uso de `FrameFeatures` de Baja Dimensionalidad como Features de Entrada:**
    En lugar de alimentar modelos pesados con espectrogramas de 1 MB, podemos usar el vector de características de 7 descriptores ya calculado por el backend (`FrameFeatures`) para entrenar modelos de clasificación o detección de anomalías basados en arquitecturas de ML ultraligeras (como Random Forests o SVMs), logrando una integración de bajísimo consumo de CPU idónea para sistemas embebidos o plugins de audio en tiempo real.
