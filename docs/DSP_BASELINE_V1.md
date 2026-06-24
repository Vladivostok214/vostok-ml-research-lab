# DSP_BASELINE_V1

## Propósito

Este documento resume el conocimiento actual disponible sobre el motor DSP de Vostok Restoration y establece el punto de partida conceptual para las investigaciones realizadas dentro de Vostok ML Research Lab.

No debe interpretarse como una especificación técnica del motor DSP ni como una validación definitiva de su desempeño.

Su objetivo es proporcionar contexto para futuras investigaciones relacionadas con análisis espectral, detección de artefactos y sistemas híbridos DSP + Machine Learning.

---

# Contexto

Vostok Restoration V1 incorpora un conjunto de detectores DSP desarrollados y refinados mediante múltiples ciclos de auditoría, experimentación y validación.

Actualmente existen detectores funcionales para:

* Clicks
* Pops
* Hum
* Hiss
* Dropouts
* Clipping

Durante el desarrollo de Vostok Restoration se realizaron múltiples investigaciones destinadas a mejorar la precisión, reducir falsos positivos y corregir errores de clasificación.

Como resultado, el motor DSP alcanzó un estado de madurez considerable sobre los conjuntos de prueba utilizados durante su desarrollo.

---

# Estado del Conocimiento Actual

La evidencia disponible sugiere que los detectores DSP muestran un desempeño sólido sobre los benchmarks controlados e históricos del laboratorio.

Sin embargo, a partir de las auditorías científicas de **`ML-LAB-002`**, se ha demostrado una brecha fundamental entre:
*   El desempeño artificialmente optimista en benchmarks que usan impulsos ideales (Dirac).
*   La vulnerabilidad del motor clásico frente a clicks analógicos reales con amortiguación, resonancia y corrimiento de fase.

Actualmente el laboratorio considera que:
1.  **La Coherencia de Fase es Superior:** El cálculo de la **Varianza de Retardo de Grupo ($\sigma^2_{\text{GD}}$)** mediante la identidad de la rampa temporal es la herramienta más robusta para discriminar clicks complejos, eliminando falsos positivos en consonantes fricativas.
2.  **La Magnitud Espectral es Vulnerable:** Los detectores basados exclusivamente en la magnitud del espectro (como la caída espectral o pendientes espectrales) son ineficaces frente a clicks reales amortiguados debido al mimetismo acústico que ejercen con la voz humana.
3.  **Los Benchmarks Deben Ser Estocásticos:** Se prohíbe el uso de clicks deterministicos estáticos en las evaluaciones del motor DSP, adoptando obligatoriamente poblaciones paramétricas aleatorizadas para simular dispersiones físicas reales.

---

# Lo Que Sabemos

Actualmente existe evidencia experimental de que:
*   **La Fase es un Invariante Robusto:** La Varianza de Retardo de Grupo ($\sigma^2_{\text{GD}}$) mantiene un **solapamiento del $0.0\%$** frente a la voz, comportándose como una firma física confiable e indestructible en alta frecuencia ($4\text{ kHz a } 20\text{ kHz}$).
*   **Identidad Matemática Exacta:** Es viable calcular el retardo de grupo en tiempo real de manera limpia, evitando las inestabilidades del desempaquetado de fase clásica, mediante el operador de rampa temporal en Fourier:
    $$\tau_g(\omega) = \text{Re}\left\{ \frac{\text{DFT}\{n \cdot x[n]\}}{\text{DFT}\{x[n]\}} \right\}$$
*   **Futilidad de la Magnitud Pura:** Clicks con decaimientos exponenciales reales ($M_2, M_3, M_4$) imitan los decaimientos de energía y formantes de la voz, produciendo solapamientos de densidad del **$26\%$ al $30\%$** en descriptores de magnitud espectral (*Spectral Slope*).
*   **La Impulsividad se Atenúa:** El factor de cresta de un click amortiguado decae drásticamente en comparación con un delta de Dirac ($D_B \approx 218.9 \to 3.23$), lo que invalida los umbrales de energía de pico fijos en el dominio temporal.

---

# Lo Que No Sabemos

Actualmente no existe evidencia suficiente para responder con confianza preguntas como:
*   **Comportamiento Polifónico:** ¿Cómo se comportará la varianza de retardo de grupo frente a música polifónica rica en instrumentos con transitorios ultra-rápidos de alta coherencia física (pizzicato, clavicémbalo, percusiones de metal)?
*   **Estabilidad en Baja Frecuencia:** ¿Cómo afecta la presencia de ruido ambiental masivo o hum de baja frecuencia ($50\text{ Hz}$) a la estabilidad numérica de la división en la identidad exacta del retardo de grupo?
*   **Generalización de Umbrales:** ¿Son los umbrales cuantitativos determinados sobre `vozenoff.wav` óptimos para una población diversa de locutores y lenguas en Vostok Restoration?

Estas preguntas permanecen abiertas y guiarán las futuras líneas de investigación.

---

# Relación con Machine Learning

El laboratorio no asume que Machine Learning sea superior a DSP.

Tampoco asume que DSP sea necesariamente la solución definitiva.

La existencia del motor DSP proporciona:

* Un punto de comparación.
* Una fuente de conocimiento acumulado.
* Un conjunto inicial de hipótesis.
* Un marco experimental sobre el cual construir nuevas investigaciones.

Cualquier resultado obtenido mediante Machine Learning deberá interpretarse en el contexto de este conocimiento previo.

---

# Rol Dentro del Laboratorio

DSP_BASELINE_V1 no representa una meta a superar.

Representa el estado actual del conocimiento.

Su función es proporcionar contexto para formular preguntas de investigación más precisas.

El propósito inicial de Vostok ML Research Lab no es reemplazar el motor DSP existente.

El propósito inicial es comprender mejor:

* Los artefactos.
* Sus representaciones espectrales.
* La capacidad de generalización de los métodos actuales.
* Las oportunidades potenciales para sistemas híbridos DSP + Machine Learning.

---

# Referencias

Para detalles técnicos específicos consultar:

Fuente principal de referencia:

* reference/CURRENT_STATE_DSP.md

Documentación técnica actualizada de auditorías e implementaciones:

* C:\Users\WLADI\Vostok Plugins\DesktopApps\Vostok Restoration V1\docs

Importante:

La documentación histórica, investigaciones antiguas y reportes intermedios del DSP Research Lab no deben considerarse fuentes primarias de verdad para este laboratorio.

Cuando exista discrepancia entre documentación histórica y documentación actual, deberá considerarse válida la información más reciente disponible en la carpeta docs de Vostok Restoration.
