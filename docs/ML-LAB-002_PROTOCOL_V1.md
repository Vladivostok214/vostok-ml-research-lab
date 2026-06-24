# Protocolo de Investigación Científica: ML-LAB-002
**Análisis de Separabilidad Transitoria y Sensibilidad de Modelado Físico (Clicks vs. Transitorios Vocales)**  
**ID del Documento:** `ML-LAB-002_PROTOCOL_V1`  
**Estado:** Propuesta Formal (Aprobada Conceptualmente)  
**Autor:** Investigador Principal Asistente (Antigravity)  
**Proyecto:** Vostok ML Research Lab

---

## 1. Introducción y Justificación Científica

En los sistemas históricos de restauración de audio de **Vostok Restoration V1**, la principal fuente de falsas alarmas (Falsos Positivos) y el factor limitante del F1-Score global es la **confusión espectro-temporal** entre los transitorios naturales rápidos de la voz (fricciones, oclusivas /p, t, k/ y ataques glotales) y los transitorios destructivos espurios (clicks).

El benchmark clásico (`evaluar_vostok_v2.py`) asume un modelo de click sintético de tipo delta de Dirac de una sola muestra. Aunque matemáticamente conveniente, este modelo es físicamente inverosímil en sistemas de audio analógicos u ópticos, donde los clicks sufren dispersión de fase, resonancias mecánicas y saturaciones electrónicas que suavizan su envolvente temporal y moldean su fase.

Este protocolo establece el diseño experimental formal de **`ML-LAB-002`** para:
1.  **Cuantificar el sesgo de sobre-optimismo del benchmark histórico** causado por la simplificación del modelo de click de Dirac (Incertidumbre #1).
2.  **Mapear el límite físico-matemático de separabilidad espectro-temporal** de los transitorios acústicos vocales reales frente a clicks de complejidad física creciente (Incertidumbre #3), sin entrenar ningún modelo.

---

## 2. Hipótesis Científicas

*   **Hipótesis Primaria ($H_1$ - Sensibilidad de Modelado):**  
    La separabilidad estadística (medida a través de la distancia de Bhattacharyya $D_B$) entre los transitorios vocales legítimos de la señal portadora (`vozenoff.wav`) y las anomalías de tipo click decrece de forma monótona y significativa a medida que se incrementa la complejidad física del modelo de click inyectado.  
    $$\text{Es decir: } D_B(\text{Voz} \parallel M_1) > D_B(\text{Voz} \parallel M_2) > D_B(\text{Voz} \parallel M_3) > D_B(\text{Voz} \parallel M_4)$$

*   **Hipótesis Secundaria ($H_2$ - Atributo de Confusión Definitivo):**  
    La introducción de dispersión de fase no lineal mediante redes todo-paso (APF) y la distorsión asimétrica de corriente directa (offset de DC) en el **Modelo 4** elimina la separabilidad lineal en los descriptores clásicos, forzando un solapamiento estadístico crítico ($D_B < 0.5$ o solapamiento de histogramas $>30\%$).

---

## 3. Variables Experimentales

*   **Variable Independiente:**  
    El grado de complejidad del modelo matemático del click inyectado (variable cualitativa nominal con 4 niveles):
    *   **$M_1$**: Click de Dirac puro (1 muestra).
    *   **$M_2$**: Click bi-exponencial (con amortiguación y tiempo de ataque/caída).
    *   **$M_3$**: Click resonante sinusoidal amortiguado (resonancia mecánica aguja-vinilo).
    *   **$M_4$**: Click dispersivo no lineal con desvío residual de DC (resonancia + dispersión de fase APF + acoplamiento capacitivo lento).

*   **Variables Dependientes:**  
    Las métricas de separabilidad matemática unidimensional y multidimensional calculadas sobre el espacio de descriptores extraídos:
    *   **Distancia de Bhattacharyya ($D_B$)** entre la clase de control y cada clase de click.
    *   **Porcentaje de Solapamiento de Histogramas ($SO$)**.

*   **Variables de Control:**  
    *   **Amplitud de Pico ($A_{peak}$):** La amplitud máxima de cada click inyectado se normaliza rigurosamente para que coincida con el nivel pico-a-pico del transitorio vocal equivalente, evitando la discriminación trivial basada en el volumen.
    *   **Señal Portadora:** Se utiliza la misma pista de referencia de voz (`vozenoff.wav`) a una frecuencia de muestreo constante de $f_s = 44100 \text{ Hz}$.
    *   **Tamaño de Ventana de Análisis ($N$):** Fijado estrictamente en $1024$ muestras.

---

## 4. Justificación del Tamaño de Ventana ($N = 1024$)

El tamaño de la ventana de análisis se fija en $N = 1024$ muestras ($23.22 \text{ ms}$ a $44.1 \text{ kHz}$) por motivos físicos y de teoría de señales:

1.  **Frontera de Heisenberg-Gabor (Compromiso Tiempo-Frecuencia):**  
    Los transitorios rápidos de la voz (como la fase de explosión de una oclusiva) tienen una duración física de entre $5 \text{ y } 15 \text{ ms}$. Una ventana de $23.2 \text{ ms}$ es lo suficientemente estrecha para aislar el transitorio de la señal de voz estacionaria circundante, impidiendo que su energía espectral se "diluya" por promediado temporal.
2.  **Resolución Espectral Mínima para Offset de DC:**  
    Para analizar el desvío exponencial de DC en el **Modelo 4**, requerimos resolución en la banda ultra-baja. Con $N=1024$, el ancho de banda de cada bin de frecuencia es $\Delta f = \frac{44100}{1024} \approx 43 \text{ Hz}$. Esto permite caracterizar con suficiente precisión el comportamiento de la fase y magnitud en los bins más cercanos a 0 Hz (DC), donde el acoplamiento capacitivo analógico se manifiesta.

---

## 5. Descriptores Seleccionados (Justificación Científica)

Para caracterizar los transitorios, se seleccionan tres descriptores de alta sensibilidad, cada uno mapeando una dimensión física independiente de la señal de audio:

### A. Dimensión Temporal: Factor de Cresta Local y Pre-onset Energy Ratio ($PE\_Ratio$)
*   **Definición:** El factor de cresta mide la relación entre el pico y el valor RMS de la ventana. El $PE\_Ratio$ mide la proporción de energía contenida en las 10 muestras anteriores al pico versus las 10 muestras posteriores:
    $$PE\_Ratio = \frac{\sum_{i=-10}^{-1} x^2(n_{pico} + i)}{\sum_{i=1}^{10} x^2(n_{pico} + i)}$$
*   **Justificación:** Un click digital de Dirac tiene un ataque instantáneo de 0 muestras y una caída instantánea, resultando en un $PE\_Ratio \approx 0$ y un Factor de Cresta extremo. Los transitorios de la voz humana requieren tiempo físico para acumular presión acústica y tienen una cola de decaimiento elástico, presentando una estructura temporal asimétrica con $PE\_Ratio > 0.05$ y menor factor de cresta.

### B. Dimensión Espectral de Magnitud: Pendiente Espectral de Alta Frecuencia ($\gamma$)
*   **Definición:** Pendiente de la recta ajustada por mínimos cuadrados ordinarios (MCO) sobre el log-espectro de potencia en la banda de alta frecuencia ($4 \text{ kHz}$ a $20 \text{ kHz}$):
    $$\log_{10}(|X(f)|^2) \approx \gamma \cdot f + b$$
*   **Justificación:** Los transitorios reales de la voz están físicamente limitados por la inercia mecánica de las cuerdas vocales y la atenuación del aire (actúan como filtros acústicos), exhibiendo un decaimiento espectral rápido ($\gamma \ll 0$). Un click de Dirac tiene un espectro plano uniforme ($\gamma \approx 0$). Este descriptor cuantificará el decaimiento espectral inducido por la amortiguación analógica en los Modelos 2, 3 y 4.

### C. Dimensión de Fase: Varianza del Retardo de Grupo en Alta Frecuencia ($\sigma^2_{GD}$)
*   **Definición:** El Retardo de Grupo $\tau_g(\omega)$ es la derivada negativa de la fase desenrollada $\theta(\omega)$ respecto a la frecuencia angular:
    $$\tau_g(\omega) = -\frac{d\theta(\omega)}{d\omega}$$
    La variable dependiente es la varianza $\sigma^2_{GD}$ de $\tau_g(\omega)$ calculada sobre el rango de $4 \text{ kHz}$ a $20 \text{ kHz}$.
*   **Justificación:** Un click de Dirac puro o un click filtrado linealmente tiene una fase lineal, lo que significa que el retardo de grupo es una constante (varianza cero; todas las frecuencias llegan sincronizadas). Los transitorios acústicos de la voz sufren dispersión de fase severa debido a las resonancias del tracto vocal, mostrando una fase altamente no lineal ($\sigma^2_{GD}$ elevada). Al evaluar el **Modelo 4** (que incluye filtros todo-paso para dispersar la fase), mediremos si la fase del click se mimetiza con la de la voz, destruyendo este criterio clásico de discriminación.

---

## 6. Métricas de Separabilidad Estadística

Para cuantificar rigurosamente la separabilidad entre la clase de voz ($p$) y las clases de clicks ($q$), se utilizarán dos métricas estadísticas:

### 1. Distancia de Bhattacharyya ($D_B$)
Asumiendo que las distribuciones de los descriptores dentro de cada clase aproximan una distribución normal $\mathcal{N}(\mu, \sigma^2)$, la distancia unidimensional se define como:
$$D_B = \frac{1}{4}\frac{(\mu_p - \mu_q)^2}{\sigma_p^2 + \sigma_q^2} + \frac{1}{2}\ln\left(\frac{\sigma_p^2 + \sigma_q^2}{2\sigma_p\sigma_q}\right)$$
*   **Interpretación:** Una distancia $D_B > 2.0$ denota una excelente separabilidad lineal (mínimo solapamiento). Una distancia $D_B < 0.5$ indica un solapamiento estadístico severo, donde es matemáticamente imposible trazar una frontera de decisión lineal sin incurrir en altas tasas de error.

### 2. Solapamiento de Histogramas (Histogram Overlap - $SO$)
Para mitigar la asunción de gaussianidad, se calculará el porcentaje de intersección directa entre las densidades de probabilidad discretas estimadas mediante histogramas normalizados:
$$SO = \sum_{k=1}^{K} \min(H_p[k], H_q[k]) \times 100\%$$
Donde $H_p$ y $H_q$ son los histogramas normalizados con $K$ bins comunes. Un valor de $0\%$ representa separabilidad absoluta, y un $100\%$ representa distribuciones idénticas de los descriptores.

---

## 7. Amenazas a la Validez y Limitaciones Esperadas

### Amenazas a la Validez Interna
*   **Ruido de Fondo Pre-existente (*Noise Floor*):** La pista `vozenoff.wav` contiene ruido térmico de cinta y siseo de fondo. Al inyectar clicks sintéticos limpios, el ruido de fondo circundante puede dispersar de forma natural la fase del click o distorsionar su pendiente espectral, haciendo que se comporten como el Modelo 4 antes de tiempo. Para mitigar esto, las ventanas de inyección se seleccionarán en zonas de alto rango dinámico.
*   **Precisión de Alineación de Ventana (*Jitter* de Onset):** Pequeños desfases (del orden de muestras) al centrar el transitorio de voz en la ventana de 1024 muestras alterarán la pendiente de la fase y los valores de retardo de grupo. El protocolo mitigará esto implementando un centrado estricto basado en la muestra del pico absoluto de energía de sub-banda.

### Amenazas a la Validez Externa (Generalización)
*   **Sesgo de Portadora Única:** La utilización exclusiva de la voz de un único locutor en `vozenoff.wav` limita la validez general de las fronteras de decisión respecto a señales complejas de instrumentos musicales (por ejemplo, transitorios rápidos de un *clavicémbalo* o *pizzicato* de violín). Los resultados deben interpretarse estrictamente dentro del dominio de la restauración de voz histórica.

### Limitaciones del Diseño
*   **Límite de Nyquist:** El análisis espectral está limitado físicamente a $22.05 \text{ kHz}$ por la frecuencia de muestreo de $44.1 \text{ kHz}$. Transitorios físicos reales que posean dinámicas de amortiguación o resonancia por encima de este límite no podrán ser caracterizados.
*   **Asunción de Normalidad de Bhattacharyya:** Si los descriptores muestran distribuciones bimodales, la métrica de Bhattacharyya paramétrica perderá precisión teórica, debiendo ser complementada de forma obligatoria por la métrica de solapamiento de histogramas ($SO$).

---

## 8. Criterios de Éxito del Experimento

El experimento se considerará exitoso si logra cumplir con los siguientes tres hitos científicos:

1.  **Cuantificación del Sesgo Histórico:** Validar con significancia estadística si el benchmark clásico de Dirac ($M_1$) sobreestima artificialmente la facilidad de detección de anomalías ($D_B(M_1) \gg D_B(M_4)$).
2.  **Identificación de Atributos de Confusión:** Isolar con precisión cuál de los fenómenos físicos del click (atenuación, resonancia o desfase) provoca la mayor degradación en el coeficiente de separabilidad respecto a la voz humana.
3.  **Evaluación de Resiliencia de Descriptores:** Identificar qué descriptor de las tres dimensiones mantiene la mayor separabilidad relativa incluso ante la inyección del Modelo 4, sentando las bases teóricas de la robustez del futuro pipeline experimental del laboratorio.
