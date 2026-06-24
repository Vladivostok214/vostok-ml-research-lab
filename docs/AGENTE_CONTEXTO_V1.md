# AGENTE_CONTEXTO_V1

## Vostok ML Research Lab

Laboratorio de investigación dedicado al estudio de técnicas de análisis espectral, detección de artefactos de audio, Machine Learning y sistemas híbridos DSP + ML.

---

# Propósito

El laboratorio existe para comprender mejor los problemas asociados a la detección, clasificación y eventual restauración de artefactos de audio.

No existe una conclusión predeterminada respecto a la utilidad de DSP o Machine Learning.

Las decisiones deberán basarse en evidencia experimental.

---

# Estado Actual

Infraestructura operativa:

* GitHub
* Google Colab
* Google Drive
* Entorno local
* Antigravity CLI

Primer notebook activo:

* ML-LAB-001

Estado:

Exploración inicial y construcción de conocimiento.

---

# Estado del Conocimiento

Actualmente sabemos:

* **Invariante de Fase Definitivo:** La Varianza de Retardo de Grupo ($\sigma^2_{\text{GD}}$) calculada mediante la identidad exacta de la rampa temporal es un descriptor extremadamente robusto en alta frecuencia ($4\text{ kHz a } 20\text{ kHz}$), mostrando un solapamiento del $0.0\%$ entre transitorios de voz real y clicks analógicos complejos (incluyendo clicks dispersivos $M_4$).
* **Sesgo de Dirac:** Diseñar y evaluar algoritmos de restauración utilizando únicamente el modelo histórico de Dirac ($M_1$) sobreestima la separabilidad temporal de forma crítica (Crest Factor $D_B \approx 218.9$), induciendo un sesgo de sobre-optimismo severo en los benchmarks clásicos.
* **Vulnerabilidad de Magnitud Espectral:** Las métricas basadas en magnitud espectral pura (como *Spectral Slope*) y envolvente temporal (como *PE-Ratio*) sufren de un severo mimetismo acústico ante clicks analógicos resonantes y bi-exponenciales, con solapamientos directos que colapsan a valores inaceptables de hasta el $26\%$ y $30\%$.
* **Necesidad de Auditoría Estocástica:** Los benchmarks basados en formas de onda estáticas deterministas sufren de sesgos estadísticos de varianza nula. Solo una población de clicks con aleatorización paramétrica continua revela la verdadera física y separabilidad del sistema.
* Existen un motor DSP funcional desarrollado en Vostok Restoration y un ecosistema de simulación en el laboratorio.

Actualmente no sabemos:

* **Generalización Polifónica e Instrumental:** Si la varianza de retardo de grupo mantendrá su separabilidad absoluta del $100\%$ frente a transitorios de música polifónica legítimos de alta coherencia física (como pizzicatos, clavicémbalos o campanas).
* **Resiliencia al Ruido:** Cuál es el límite preciso de resiliencia del estimador exacto de retardo de grupo ante relaciones señal-ruido (SNR) severamente bajas o interferencias intensas de baja frecuencia (hum).
* **Generalidad de Umbrales Cuantitativos:** Qué tan bien generalizan los umbrales de decisión obtenidos en `ML-LAB-002` a un corpus masivo de locutores de voz (femenina, infantil) y diferentes lenguas o registros.
* Qué arquitectura híbrida (combinación de umbrales lineales de baja CPU frente a modelos profundos) resultará óptima para el motor en tiempo real.

---

# Referencia DSP

El laboratorio dispone de una línea base DSP documentada en:

docs/DSP_BASELINE_V1.md

Dicha línea base representa el conocimiento actual disponible.

No representa una validación definitiva del sistema.

---

# Infraestructura Experimental

El laboratorio dispone de herramientas de simulación de degradaciones y evaluación de detectores documentadas en:

docs/EXPERIMENTAL_INFRASTRUCTURE_V1.md

Este ecosistema permite generar datasets controlados con Ground Truth matemático y evaluar de forma automática el desempeño de los algoritmos.

---

# Rol de Antigravity

Antigravity actúa como investigador asistente del laboratorio.

Su función principal es:

* Formular preguntas relevantes.
* Identificar oportunidades de investigación.
* Detectar riesgos metodológicos.
* Proponer experimentos.
* Mantener documentación técnica.
* Ayudar a organizar el conocimiento acumulado.

No debe asumir conclusiones que aún no hayan sido demostradas experimentalmente.

---

# Herramientas del Laboratorio

## Google Colab

Entorno principal de experimentación.

## Google Drive

Almacenamiento de datasets, papers y notebooks.

## GitHub

Control de versiones y documentación.

## Entorno Local

Integración con Antigravity CLI.

---

# Filosofía de Trabajo

El laboratorio prioriza:

* Comprensión antes que implementación.
* Evidencia antes que opinión.
* Experimentación antes que conclusiones.
* Documentación antes que memoria.

Las líneas de investigación podrán cambiar conforme aparezcan nuevos resultados.

No existe una hoja de ruta rígida.

---

# Pregunta Principal

La pregunta central del laboratorio actualmente es:

¿De qué manera pueden combinarse análisis espectral, DSP y Machine Learning para comprender mejor los artefactos de audio y mejorar su detección o restauración?

---

# Documentos de Referencia

Prioridad alta:

* DSP_BASELINE_V1.md
* DSP_ARCHITECTURE_OVERVIEW_V1.md
* EXPERIMENTAL_INFRASTRUCTURE_V1.md
* reference/CURRENT_STATE_DSP.md

Prioridad media:

* Auditorías recientes de Vostok Restoration.

La documentación histórica debe utilizarse únicamente como contexto adicional cuando sea necesario.
