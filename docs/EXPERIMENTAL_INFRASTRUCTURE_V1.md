# EXPERIMENTAL_INFRASTRUCTURE_V1

## Propósito

Este documento describe la infraestructura experimental actualmente utilizada para generar datasets sintéticos, registrar Ground Truth y evaluar algoritmos de detección de artefactos de audio.

Su objetivo es documentar la metodología experimental utilizada hasta la fecha dentro del ecosistema Vostok.

No pretende definir modelos físicos definitivos de degradación ni representar de manera perfecta todos los artefactos existentes en grabaciones reales.

Los modelos descritos aquí deben entenderse como aproximaciones experimentales actualmente utilizadas por el laboratorio.

---

# Filosofía General

La infraestructura experimental fue desarrollada originalmente para validar y auditar los detectores DSP de Vostok Restoration.

Su función principal es permitir experimentos controlados donde:

* La ubicación de los artefactos es conocida.
* La intensidad de los artefactos es conocida.
* Los parámetros de degradación son conocidos.
* Los resultados pueden evaluarse automáticamente.

Esto permite construir benchmarks reproducibles y comparar diferentes enfoques bajo condiciones controladas.

---

# Arquitectura Experimental

El flujo experimental general es:

Audio Limpio
↓
Generador de Degradaciones
↓
Audio Degradado
+
Ground Truth
↓
Detector / Algoritmo
↓
Reporte de Detección
↓
Evaluador Automático
↓
Métricas

---

# Generador de Degradaciones Sintéticas

## Objetivo

Crear versiones degradadas de archivos de audio limpios mediante la inyección controlada de artefactos.

El objetivo no es reproducir exactamente todos los fenómenos físicos del mundo real.

El objetivo es generar escenarios reproducibles y parametrizables que permitan estudiar el comportamiento de algoritmos de detección.

---

# Modelos de Artefactos Actuales

Importante:

Los siguientes modelos representan la implementación experimental actual.

No deben interpretarse como definiciones físicas universales de cada artefacto.

Podrán modificarse o evolucionar conforme avance la investigación.

---

## Click

Modelo actual:

Impulso extremadamente breve.

Características:

* Duración aproximada de una muestra.
* Polaridad positiva o negativa.
* Distribución aleatoria.
* Posibilidad de aparición en clusters temporales.

Interpretación experimental:

Se utiliza como aproximación de transitorios impulsivos muy cortos.

---

## Pop

Modelo actual:

Transitorio amortiguado de baja frecuencia.

Características:

* Duración de decenas de milisegundos.
* Componente sinusoidal de baja frecuencia.
* Decaimiento exponencial.
* Asimetría controlada en la envolvente.

Interpretación experimental:

Busca aproximar eventos tipo "thump" o transitorios plosivos de baja frecuencia.

---

## Dropout

Modelo actual:

Pérdida temporal de señal.

Características:

* Atenuación extrema o silenciamiento.
* Duración breve.
* Posición aleatoria.

Interpretación experimental:

Representa interrupciones temporales de la señal.

---

## Clipping

Modelo actual:

Saturación artificial de segmentos específicos.

Características:

* Amplificación previa.
* Aplicación de umbral de recorte.
* Verificación de saturación efectiva.
* Transiciones suavizadas.

Interpretación experimental:

Busca aproximar procesos de saturación asociados a ADC, etapas analógicas o errores de ganancia.

---

## Hum

Modelo actual:

Interferencia tonal segmentada.

Características:

* Fundamental de 50 Hz.
* Inclusión de armónicos.
* Segmentos localizados.
* Fade-in y fade-out suaves.

Interpretación experimental:

Representa contaminación eléctrica de red y fenómenos relacionados.

---

## Hiss

Modelo actual:

Ruido de banda ancha.

Características:

* Ruido gaussiano.
* Segmentación temporal.
* Envolventes suaves.
* Intensidad variable.

Interpretación experimental:

Representa ruido de fondo continuo asociado a procesos analógicos o electrónicos.

---

# Superposición de Artefactos

La infraestructura actual permite la coexistencia de múltiples artefactos.

Ejemplos:

* Click dentro de una región con Hum.
* Dropout dentro de una región con Hiss.
* Hum y Hiss simultáneos.

Esta capacidad existe para reducir el riesgo de construir datasets excesivamente simples o artificiales.

---

# Ground Truth

## Objetivo

Registrar con precisión los eventos generados durante el proceso de degradación.

El Ground Truth constituye la referencia experimental utilizada para la evaluación automática de detectores.

---

# Ground Truth V2

Formato conceptual actual:

StartTime
EndTime
ArtifactType
Magnitude
Metadata

---

## StartTime

Tiempo de inicio del artefacto.

---

## EndTime

Tiempo de finalización.

Para artefactos instantáneos:

StartTime = EndTime

---

## ArtifactType

Clasificación del artefacto generado.

Ejemplos:

* click
* pop
* dropout
* clipping
* hum_50hz
* hiss

---

## Magnitude

Parámetro numérico asociado a la intensidad del evento.

Su significado depende del artefacto.

Ejemplos:

* amplitud
* umbral de clipping
* nivel de ruido
* nivel de hum

---

## Metadata

Información adicional específica del evento.

Ejemplos:

* duración
* frecuencia
* armónicos
* superposición
* parámetros de generación

---

# Evaluación Automática

La infraestructura dispone de herramientas capaces de comparar:

Ground Truth
vs
Reporte generado por un detector

A partir de esta comparación se calculan métricas de rendimiento.

---

# Métricas Utilizadas

Actualmente se utilizan principalmente:

* Precision
* Recall
* F1 Score
* False Positives
* False Negatives

Estas métricas fueron utilizadas extensivamente durante el desarrollo y auditoría de Vostok Restoration.

---

# Limitaciones Conocidas

## Realidad Sintética vs Realidad Física

La existencia de Ground Truth perfecto constituye una ventaja experimental importante.

Sin embargo:

Los artefactos sintéticos no son equivalentes a artefactos reales.

La investigación debe asumir permanentemente la existencia de una posible brecha entre:

* Datos sintéticos.
* Datos reales.

---

## Domain Shift

Un detector puede mostrar excelente desempeño sobre datos sintéticos y degradarse significativamente al enfrentarse a grabaciones reales.

Este riesgo debe considerarse en toda investigación futura.

---

## Evolución Continua

Los modelos de degradación utilizados actualmente no se consideran definitivos.

La infraestructura experimental es un sistema vivo.

Los modelos de generación podrán refinarse, reemplazarse o ampliarse conforme se obtenga nuevo conocimiento sobre el comportamiento real de los artefactos.

---

# Rol Dentro del Laboratorio

Esta infraestructura proporciona:

* Reproducibilidad experimental.
* Ground Truth controlado.
* Generación de benchmarks.
* Evaluación automática.
* Comparación objetiva entre enfoques.

Constituye actualmente uno de los principales activos metodológicos disponibles para Vostok ML Research Lab.
