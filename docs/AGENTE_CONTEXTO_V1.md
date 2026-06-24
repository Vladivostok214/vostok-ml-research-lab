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

* Los artefactos presentan características observables.
* Las representaciones espectrales contienen información útil.
* Existe un motor DSP funcional desarrollado en Vostok Restoration.
* Existen benchmarks prometedores sobre conjuntos de prueba controlados.

Actualmente no sabemos:

* Qué tan bien generaliza el motor DSP.
* Qué tan bien generalizarían futuros modelos ML.
* Qué representación espectral es la más adecuada.
* Qué arquitectura híbrida podría resultar más efectiva.
* Qué tamaño y composición debería tener un dataset de referencia.

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
