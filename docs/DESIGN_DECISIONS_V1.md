# DESIGN_DECISIONS_V1

## Propósito

Este documento registra las principales decisiones conceptuales y metodológicas que dieron forma a la infraestructura experimental y a los detectores DSP de Vostok Restoration.

No pretende ser una cronología completa del proyecto.

Su objetivo es explicar por qué ciertas decisiones fueron tomadas y qué problemas intentaban resolver.

---

# Filosofía General

Muchas de las características actuales del sistema no surgieron a partir de teoría pura.

Fueron el resultado de múltiples ciclos de auditoría, experimentación y validación práctica.

Por esta razón, algunas decisiones representan soluciones pragmáticas a problemas observados durante el desarrollo y no necesariamente modelos físicos definitivos del mundo real.

---

# Evolución del Generador Sintético

## Objetivo Original

Inicialmente el generador sintético fue creado para validar detectores DSP.

No fue diseñado originalmente para entrenar modelos de Machine Learning.

Su propósito principal era permitir:

* Generación reproducible de artefactos.
* Ground Truth conocido.
* Comparación objetiva entre versiones de algoritmos.

---

# Click

## Situación Inicial

Las primeras versiones del generador utilizaban modelos de click más extensos.

Estos eventos podían ocupar varias muestras consecutivas.

---

## Problema Observado

Durante las auditorías DSP se observó que clicks más largos comenzaban a compartir características con otros artefactos.

Particularmente:

* Pops.
* Clipping.
* Transitorios complejos.

Esto generaba ambigüedad durante la evaluación de detectores.

---

## Decisión Adoptada

El modelo evolucionó hacia impulsos extremadamente breves.

La versión actual utiliza aproximadamente una muestra de duración.

---

## Interpretación Correcta

Esta decisión no implica que todos los clicks reales tengan una muestra de duración.

Representa únicamente el modelo sintético actualmente utilizado para validación experimental.

Continúa siendo una pregunta abierta cómo deberían modelarse clicks más complejos para futuras investigaciones.

---

# Hum

## Situación Inicial

Las primeras aproximaciones consideraban hum continuo durante toda la duración del archivo.

---

## Problema Observado

Los escenarios resultaban excesivamente simples.

El detector enfrentaba condiciones poco representativas respecto a material real.

---

## Decisión Adoptada

Se introdujeron:

* Segmentación temporal.
* Fade-in.
* Fade-out.
* Armónicos.

---

## Objetivo

Aumentar la complejidad experimental y aproximar escenarios más realistas.

---

# Hiss

## Situación Inicial

El ruido podía modelarse como una capa uniforme presente durante todo el archivo.

---

## Problema Observado

Los eventos eran demasiado fáciles de aislar.

---

## Decisión Adoptada

Se incorporaron segmentos localizados y variaciones temporales.

---

## Objetivo

Introducir variabilidad y reducir escenarios excesivamente artificiales.

---

# Clipping

## Situación Inicial

La saturación podía aplicarse de forma abrupta.

---

## Problema Observado

Los bordes generaban artefactos secundarios que no representaban adecuadamente procesos reales de saturación.

---

## Decisión Adoptada

Se incorporaron:

* Ganancia previa controlada.
* Verificación de saturación efectiva.
* Transiciones suavizadas.

---

## Objetivo

Representar de forma más consistente eventos de clipping observables en grabaciones reales.

---

# Superposición de Artefactos

## Situación Inicial

Los artefactos eran generados de forma independiente.

---

## Problema Observado

Los escenarios resultaban demasiado limpios.

Los detectores eran evaluados en condiciones simplificadas.

---

## Decisión Adoptada

Permitir la coexistencia de múltiples artefactos.

Ejemplos:

* Click sobre Hum.
* Dropout sobre Hiss.
* Hum y Hiss simultáneos.

---

## Objetivo

Incrementar la complejidad del benchmark y aproximarlo a situaciones reales.

---

# Evolución del Ground Truth

## Ground Truth Inicial

Las primeras versiones estaban orientadas principalmente a eventos breves.

---

## Problema Observado

Artefactos continuos como:

* Hum
* Hiss
* Dropouts prolongados

requerían una descripción más rica.

---

## Decisión Adoptada

Evolución hacia Ground Truth V2.

Campos conceptuales:

* StartTime
* EndTime
* ArtifactType
* Magnitude
* Metadata

---

## Objetivo

Permitir una descripción más precisa de eventos complejos y facilitar evaluaciones automatizadas.

---

# Filosofía de Benchmarking

Durante el desarrollo de Vostok Restoration se adoptó una filosofía iterativa.

El objetivo nunca fue maximizar métricas aisladas.

El objetivo fue comprender:

* Por qué ocurrían errores.
* Qué tipos de falsos positivos aparecían.
* Qué tipos de falsos negativos persistían.
* Qué hipótesis explicaban dichos comportamientos.

Esta filosofía continúa vigente dentro de Vostok ML Research Lab.

---

# Lecciones Aprendidas

Las métricas por sí solas rara vez explican el comportamiento de un sistema.

Comprender las causas de los errores suele aportar más conocimiento que mejorar marginalmente una métrica.

Los modelos sintéticos son herramientas experimentales útiles, pero no sustituyen la validación sobre material real.

Las decisiones de diseño deben entenderse dentro del contexto histórico en el que fueron tomadas.

Muchas decisiones actuales representan soluciones pragmáticas a problemas concretos observados durante el desarrollo.

---

# Estado Actual

Las decisiones descritas en este documento no se consideran definitivas.

Todas ellas podrán ser revisadas, refinadas o reemplazadas conforme aparezca nueva evidencia experimental.

El propósito de este documento es preservar el razonamiento detrás de las decisiones actuales para evitar la pérdida de conocimiento acumulado.
