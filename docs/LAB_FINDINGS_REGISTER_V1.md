# Registro Histórico Central de Descubrimientos de Laboratorio
**ID del Documento:** `LAB_FINDINGS_REGISTER_V1`  
**Consolidación de Conocimiento de Largo Plazo · Vostok ML Research Lab**  

Este documento sirve como el registro central institucional para preservar, catalogar y calificar la evidencia científica generada por las líneas de investigación de Vostok ML Research Lab. Su propósito es evitar la amnesia del laboratorio y guiar con rigor científico el desarrollo de los algoritmos de restauración de **Vostok Restoration**.

---

## Índice de Hallazgos

| ID | Hallazgo Científico | Origen | Confianza Científica | Estado |
| :--- | :--- | :--- | :---: | :---: |
| **FD-002-01** | Sesgo de Sobre-optimismo del Benchmark Dirac ($M_1$) | `ML-LAB-002` | **Alta** | Verificado |
| **FD-002-02** | Mimetismo Espectral de Magnitud por Clicks Analógicos | `ML-LAB-002` | **Alta** | Verificado |
| **FD-002-03** | Coherencia de Retardo de Grupo como Invariante de Fase | `ML-LAB-002` | **Moderada** | Verificado |
| **FD-002-04** | Sensibilidad Teórica del Retardo de Grupo en Baja Frecuencia | `ML-LAB-002` | **Baja** | Hipótesis |

---

## Catálogo Detallado de Hallazgos

### Hallazgo `FD-002-01`: Sesgo de Sobre-optimismo del Benchmark Dirac ($M_1$)
*   **Descripción:**  
    Calibrar y evaluar algoritmos de detección utilizando impulsos de Dirac ideales de una sola muestra crea una falsa sensación de alta separabilidad. El factor de cresta temporal cae drásticamente de $D_B \approx 218.9$ (Dirac) a $D_B \approx 3.23$ ($M_2$) y $D_B \approx 9.50$ ($M_4$) al considerar clicks amortiguados o con desfases físicos.
*   **Origen:** Experimento `ML-LAB-002` (Auditoría de Población Estocástica).
*   **Justificación:**
    *   **Qué conocimiento aporta:** Evidencia empírica de que la impulsividad temporal se diluye exponencialmente en sistemas electrónicos y analógicos de audio reales, reduciendo drásticamente la capacidad de detección basada en umbrales estáticos temporales clásicos.
    *   **Por qué merece persistencia:** Es una lección metodológica crucial. Impide que futuros ingenieros diseñen o calibren algoritmos comerciales utilizando señales sintéticas de juguete (Dirac), forzando el diseño de benchmarks realistas paramétricos.
*   **Nivel de Confianza Científica:** **ALTA**
    *   *Justificación de Confianza:* Demostración matemática y física directa: el decaimiento exponencial de la energía reparte el pico temporal a lo largo de múltiples muestras, lo que es un hecho físico inalterable para cualquier transitorio acústico o analógico.

---

### Hallazgo `FD-002-02`: Mimetismo Espectral de Magnitud por Clicks Analógicos
*   **Descripción:**  
    El decaimiento espectral de los clicks amortiguados ($M_2, M_3, M_4$) mimetiza de manera idéntica la atenuación espectral (*Spectral Slope*) de las cuerdas vocales humanas. Al aleatorizar la resonancia de los clicks, estos imitan los formantes legítimos de fricción, provocando que el solapamiento directo de histogramas espectrales colapse a valores inaceptables del **$26.0\%$ ($M_3$)** y **$30.0\%$ ($M_4$)**.
*   **Origen:** Experimento `ML-LAB-002` (Métrica de Magnitud Espectral).
*   **Justificación:**
    *   **Qué conocimiento aporta:** Demuestra que las características basadas en magnitud espectral pura (como la caída espectral o las sub-bandas de energía clásica) son insuficientes por sí solas para discriminar clicks analógicos reales complejos de la voz, induciendo una alta tasa de falsos positivos en consonantes fricativas.
    *   **Por qué merece persistencia:** Evita el desperdicio de ciclos de CPU y esfuerzo de desarrollo en refinamientos inútiles de detectores espectrales de magnitud en Vostok Restoration.
*   **Nivel de Confianza Científica:** **ALTA**
    *   *Justificación de Confianza:* Verificado empíricamente bajo una población estocástica aleatorizada de 250 clicks inyectados sobre ventanas vocales reales extraídas de `vozenoff.wav`. La similitud espectral entre el roll-off del filtro de click y la impedancia acústica de la boca y garganta es un hecho acústico repetible.

---

### Hallazgo `FD-002-03`: Coherencia de Retardo de Grupo como Invariante de Fase
*   **Descripción:**  
    La Varianza de Retardo de Grupo ($\sigma^2_{\text{GD}}$) calculada mediante la identidad matemática exacta en alta frecuencia ($4\text{ kHz a } 20\text{ kHz}$) se comporta como un invariante acústico definitivo, logrando un **$0.0\%$ de solapamiento directo** con la voz humana incluso ante clicks con dispersión de fase severa ($M_4$). Las excitaciones de voz son continuas y caóticas en fase ($\sigma^2_{\text{GD}} \gg 10^4$), mientras que los clicks inyectados, por más dispersos que estén analógicamente, conservan un alto acoplamiento y coherencia espacial de fase ($\sigma^2_{\text{GD}} < 10^3$).
*   **Origen:** Experimento `ML-LAB-002` (Implementación del operador de rampa temporal exacto).
*   **Justificación:**
    *   **Qué conocimiento aporta:** Identifica la coherencia de fase en alta frecuencia como la firma definitiva y matemáticamente demostrable de impulsividad física para erradicar falsos positivos frente a transitorios de voz real.
    *   **Por qué merece persistencia:** Constituye la piedra angular sobre la cual se diseñará el nuevo detector del motor DSP comercial de Vostok Restoration.
*   **Nivel de Confianza Científica:** **MODERADA**
    *   *Justificación de Confianza:* Aunque matemáticamente sólida y validada con un $0.0\%$ de solapamiento frente a la portadora real `vozenoff.wav`, su nivel de confianza se restringe a *moderada* debido a que la clase de control fue exclusivamente voz humana y clicks sintéticos. Aún no se ha verificado si instrumentos polifónicos de percusión metálica o ataques ultra-coherentes de alta frecuencia (como clavicémbalos o campanas) mimetizarán esta firma de fase compacta, provocando falsos positivos.

---

### Hallazgo `FD-002-04`: Sensibilidad Teórica del Retardo de Grupo en Baja Frecuencia
*   **Descripción:**  
    En entornos con relaciones señal-ruido bajas (SNR baja) o presencia de ruidos constantes de baja frecuencia (hum de red a $50\text{ Hz}$), el término del denominador de la identidad exacta de retardo de grupo:
    $$\tau_g(\omega) = \text{Re}\left\{ \frac{\text{DFT}\{n \cdot x[n]\}}{\text{DFT}\{x[n]\}} \right\}$$
    puede aproximarse críticamente a cero. Esto provocaría inestabilidades numéricas masivas por división (amplificando ruidos de cuantificación), destruyendo la estimación del retardo de grupo y esparciendo falsamente la firma coherente de los clicks reales.
*   **Origen:** Experimento `ML-LAB-002` (Análisis de Limitaciones y Amenazas a la Validez).
*   **Justificación:**
    *   **Qué conocimiento aporta:** Advierte sobre el límite operativo de la formulación directa de la identidad de rampa en baja frecuencia y la necesidad de aplicar acondicionamientos o regularizaciones.
    *   **Por qué merece persistencia:** Actúa como una alerta de ingeniería para futuros desarrolladores antes de la traducción de esta fórmula a código de producción de bajo nivel (como C++ o JSFX), previniendo crashes o divisiones por cero en el motor.
*   **Nivel de Confianza Científica:** **BAJA**
    *   *Justificación de Confianza:* Es una hipótesis matemática derivada de la estructura de la división en la fórmula y el análisis de ruido espectral de baja frecuencia, que aún no ha sido testeada experimentalmente mediante una campaña formal de inyección de ruido de hum en el laboratorio.

---

> [!NOTE]  
> **Directriz de Preservación:** Este registro se actualizará formalmente al cierre de cada hito experimental del laboratorio. Toda propuesta de cambio en el motor DSP de Vostok Restoration deberá consultar este registro para verificar su validez científica previa.
