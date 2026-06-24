\# ML-INV-000



Estado: Activo



Título:



Fundamentos y Preguntas Abiertas del Laboratorio



\---



\# Propósito



Este documento establece el marco conceptual inicial de Vostok ML Research Lab.



No representa una hoja de ruta rígida.



No representa una planificación cerrada.



Su función es registrar las preguntas abiertas, hipótesis, oportunidades de investigación y áreas de incertidumbre identificadas durante las primeras etapas del laboratorio.



Las líneas de investigación podrán modificarse, expandirse o desaparecer conforme se obtenga nueva evidencia.



\---



\# Contexto



Vostok ML Research Lab surge como una iniciativa paralela al desarrollo de Vostok Restoration.



Actualmente existe un motor DSP funcional cuyo estado de conocimiento se encuentra documentado en:



\* DSP\_BASELINE\_V1.md

\* CURRENT\_STATE\_DSP.md



Sin embargo, todavía existen preguntas abiertas relacionadas con:



\* Generalización.

\* Representaciones espectrales.

\* Clasificación automática.

\* Sistemas híbridos DSP + ML.

\* Construcción de datasets.

\* Metodologías de validación.



\---



\# Hipótesis Inicial



Los artefactos de audio contienen información observable que podría ser representada, analizada y eventualmente clasificada mediante distintos enfoques computacionales.



No se asume que Machine Learning sea necesariamente la mejor herramienta para resolver estos problemas.



El propósito del laboratorio es investigar qué enfoques resultan útiles y bajo qué condiciones.



\---



\# Preguntas Abiertas



\## Generalización DSP



\* ¿Qué tan bien generalizan los detectores DSP actuales?

\* ¿Cómo se comportan frente a grandes volúmenes de material desconocido?

\* ¿Qué tipos de errores aparecen con mayor frecuencia?



\---



\## Representaciones Espectrales



\* ¿Qué información contienen realmente los espectrogramas?

\* ¿Qué diferencias prácticas existen entre STFT, Mel Spectrogram, Log Spectrogram y CQT?

\* ¿Qué representaciones resaltan mejor distintos tipos de artefactos?



\---



\## Artefactos



\* ¿Existen firmas espectrales consistentes para cada artefacto?

\* ¿Qué artefactos son más fáciles de diferenciar?

\* ¿Qué artefactos presentan zonas de ambigüedad?



\---



\## Datasets



\* ¿Qué tamaño mínimo debería tener un dataset útil?

\* ¿Qué proporción debería existir entre datos sintéticos y datos reales?

\* ¿Cómo evitar sesgos y data leakage?



\---



\## Machine Learning



\* ¿Existen patrones que los métodos DSP actuales no estén capturando?

\* ¿Puede ML aportar información adicional?

\* ¿Puede ML mejorar clasificación o validación sin reemplazar DSP?



\---



\## Sistemas Híbridos



\* ¿Qué arquitectura híbrida tendría sentido investigar?

\* ¿DSP como primera etapa y ML como validación?

\* ¿ML como clasificador posterior?

\* ¿Modelos especializados por artefacto?



\---



\## Restauración



\* ¿Existen oportunidades para utilizar ML en reparación automática?

\* ¿Qué tareas siguen siendo más apropiadas para DSP clásico?



\---



\# Google Colab como Plataforma de Investigación



Actualmente Google Colab constituye el principal entorno experimental del laboratorio.



Áreas de interés:



\* Procesamiento espectral.

\* Visualización.

\* Construcción de datasets.

\* Entrenamiento experimental.

\* Exploración de modelos.

\* Automatización de análisis.



Las capacidades de Colab deberán revisarse continuamente conforme avance la investigación.



\---



\# Estado Actual



Infraestructura:



Completada.



\* GitHub operativo.

\* Google Drive operativo.

\* Google Colab operativo.

\* Sincronización local validada.



Investigación activa:



\* ML-LAB-001



Objetivo actual:



Comprender mejor las representaciones espectrales y explorar la información contenida en ellas antes de considerar modelos de Machine Learning.



\---



\# Criterio de Éxito



El éxito del laboratorio no se medirá por la cantidad de modelos entrenados.



El éxito se medirá por la calidad del conocimiento generado.



Una hipótesis descartada correctamente tiene tanto valor como una hipótesis validada.



El objetivo principal es comprender el problema antes de intentar resolverlo.



