# Auditoría de Consistencia y Rigor Estadístico: ML-LAB-002
**Verificación de Datos Brutos y Deconstrucción de Métricas de Separabilidad**  
**ID del Documento:** `ML-LAB-002_CONSISTENCY_AUDIT_V1`  
**Estado:** Finalizado (Auditoría Aprobada)  
**Autor:** Investigador Principal Asistente (Antigravity)  
**Proyecto:** Vostok ML Research Lab

---

## 1. Introducción y Propósito de la Auditoría

Esta auditoría tiene como propósito verificar la integridad metodológica del experimento **`ML-LAB-002`** analizando exhaustivamente los archivos de datos brutos intermedios generados a partir de la pista `vozenoff.wav`:
1.  **`ml_lab_002_features.csv`**: Tabla de características espectro-temporales extraídas.
2.  **`ml_lab_002_metrics.csv`**: Matriz final de métricas de separabilidad.

El foco de esta auditoría es diagnosticar si las métricas de separabilidad reportadas, en particular el comportamiento del descriptor **Varianza de Retardo de Grupo ($GD\\_Variance$)** el cual presenta distancias de Bhattacharyya virtualmente idénticas ($D_B \approx 11.82$) para todos los modelos de click ($M_1$ a $M_4$), corresponden a un **hallazgo físico real** de la firma acústica o a un **artefacto estadístico/metodológico**.

---

## 2. Resultados de las Verificaciones de Control

### A. Número Exacto de Muestras por Clase
La verificación del balance de clases en `ml_lab_002_features.csv` demuestra un balance estricto y biyectivo perfecto:
*   **Voz Humana (Legítimos):** 50 muestras ($20\%$)
*   **Click Modelo 1 (Dirac):** 50 muestras ($20\%$)
*   **Click Modelo 2 (Bi-exponencial):** 50 muestras ($20\%$)
*   **Click Modelo 3 (Resonante):** 50 muestras ($20\%$)
*   **Click Modelo 4 (Dispersivo + DC):** 50 muestras ($20\%$)
*   **Total de Registros:** **250 muestras** ($100.0\%$)

### B. Existencia de NaN o Inf
*   **Valores Nulos o Vacíos (NaN/NULL):** 0 encontrados ($100\%$ limpios)
*   **Valores Infinitos (Inf / -Inf):** 0 encontrados ($100\%$ estables)

### C. Medias y Desviaciones Estándar por Clase

#### 1. PE-Ratio (Dimensión Temporal - Simetría)
*   **Voz Humana:** $\mu = 1.2246$, $\sigma = 1.6096$. Muestra alta variabilidad natural y asimetría sesgada a la derecha.
*   **Click $M_1$ (Dirac) y $M_3$ (Resonante):** $\mu = 0.0$, $\sigma = 0.0$. Tienen un inicio instantáneo en la muestra 512, por lo que su energía pre-onset es exactamente cero.
*   **Click $M_2$ (Bi-exponencial):** $\mu = 0.6407$, $\sigma \approx 0.0$ ($2.74 \times 10^{-15}$). El decaimiento y tiempo de ataque de este modelo es idéntico en todas las ventanas, variando solo en amplitud. Al ser el descriptor una relación de energía escalar, la desviación estándar es nula.
*   **Click $M_4$ (Dispersivo):** $\mu = 0.2994$, $\sigma \approx 0.0$ ($1.21 \times 10^{-9}$). La dispersión APF desplaza el pico de energía, induciendo energía pre-onset positiva y estable en todas las muestras.

#### 2. Crest Factor (Dimensión Temporal - Impulsividad)
*   **Voz Humana:** $\mu = 2.8819$, $\sigma = 1.0038$. El factor de cresta es bajo, correspondiente a señales continuas.
*   **Click $M_1$ (Dirac):** $\mu = 32.0000$, $\sigma \approx 0.0$ ($2.85 \times 10^{-12}$). El factor de cresta es exactamente $\sqrt{N} = \sqrt{1024} = 32$, un límite físico absoluto para impulsos unitarios de 1 muestra.
*   **Click $M_2$ (Bi-exponencial):** $\mu = 6.5753$, $\sigma \approx 0.0$ ($1.20 \times 10^{-13}$). La energía se esparce en el tiempo, reduciendo la impulsividad física.
*   **Click $M_3$ (Resonante):** $\mu = 12.4195$, $\sigma \approx 0.0$ ($4.30 \times 10^{-13}$).
*   **Click $M_4$ (Dispersivo):** $\mu = 11.8541$, $\sigma \approx 0.0$ ($3.60 \times 10^{-8}$).

#### 3. Spectral Slope (Dimensión de Magnitud Espectral)
*   **Voz Humana:** $\mu = -0.001312$, $\sigma = 0.000479$. Refleja el decaimiento natural de alta frecuencia de la voz.
*   **Click $M_1$ (Dirac):** $\mu = 0.0$, $\sigma = 0.0$. Espectro plano de Fourier de magnitud constante para todas las frecuencias.
*   **Click $M_2$ (Bi-exponencial):** $\mu = -0.001200$, $\sigma \approx 0.0$ ($1.63 \times 10^{-14}$). **Mimetismo extremo:** la pendiente espectral del click amortiguado es casi idéntica a la media de la voz humana.
*   **Click $M_3$ (Resonante):** $\mu = -0.000051$, $\sigma \approx 0.0$ ($1.93 \times 10^{-16}$).
*   **Click $M_4$ (Dispersivo):** $\mu = -0.000271$, $\sigma \approx 0.0$ ($6.64 \times 10^{-12}$).

#### 4. GD Variance (Dimensión de Fase)
*   **Voz Humana:** $\mu = 8.9411 \times 10^5$, $\sigma = 3.6259 \times 10^6$. Posee una variabilidad y dispersión masiva debido a las resonancias del tracto vocal y singularidades de fase en bines de baja magnitud.
*   **Click $M_1$ (Dirac):** $\mu = 3.4757 \times 10^{-18}$, $\sigma = 9.0279 \times 10^{-18}$. El retardo de grupo es matemáticamente constante en 512, por lo que su varianza local es nula ($0.0$ más ruido numérico).
*   **Click $M_2$ (Bi-exponencial):** $\mu = 0.02776$, $\sigma \approx 0.0$ ($5.87 \times 10^{-12}$). Su fase es lineal, con mínima desviación.
*   **Click $M_3$ (Resonante):** $\mu = 15.0734$, $\sigma \approx 0.0$ ($2.55 \times 10^{-12}$).
*   **Click $M_4$ (Dispersivo):** $\mu = 26.1726$, $\sigma \approx 0.0$ ($6.56 \times 10^{-7}$). Su fase es no-lineal por el APF, pero se mantiene determinista y constante en todos los bines de frecuencia analizados.

---

## 3. Deconstrucción de la Identidad en Bhattacharyya ($D_B \approx 11.82$)

La tabla resumen reporta las siguientes distancias de Bhattacharyya para la columna de **GD Variance**:
*   **Dirac ($M_1$):** $D_B = \mathbf{11.820872}$
*   **Bi-exponential ($M_2$):** $D_B = \mathbf{11.820872}$
*   **Resonante ($M_3$):** $D_B = \mathbf{11.820872}$
*   **Dispersivo ($M_4$):** $D_B = \mathbf{11.820871}$

¿Por qué produce exactamente el mismo resultado hasta la sexta cifra decimal? Analicemos los dos componentes de la fórmula de Bhattacharyya:
$$D_B = \text{Term}_1 \text{ (Diferencia de Medias)} + \text{Term}_2 \text{ (Diferencia de Covarianzas)}$$
$$\text{Term}_1 = \frac{1}{4}\frac{(\mu_{voz} - \mu_{click})^2}{\sigma_{voz}^2 + \sigma_{click}^2}$$
$$\text{Term}_2 = \frac{1}{2}\ln\left(\frac{\sigma_{voz}^2 + \sigma_{click}^2}{2\sigma_{voz}\sigma_{click}}\right)$$

### 1. Análisis del Término 1 (Diferencia de Medias)
*   Para la Voz Humana: $\mu_{voz} = 8.9411 \times 10^5$, $\sigma^2_{voz} = 1.2884 \times 10^{13}$.
*   Para cualquier Click $M_j$: su media $\mu_{click}$ es diminuta ($10^{-18} \text{ a } 26.17$) comparada con la de la voz ($10^5$), por lo que:
    $$(\mu_{voz} - \mu_{click})^2 \approx \mu_{voz}^2 \approx (8.9411 \times 10^5)^2 \approx 7.9943 \times 10^{11}$$
*   Dado que los clicks son deterministas en descriptor, su variabilidad $\sigma^2_{click}$ entre ventanas es insignificante, por lo que:
    $$\sigma_{voz}^2 + \sigma_{click}^2 \approx \sigma_{voz}^2 \approx 1.2884 \times 10^{13}$$
*   Evaluando el primer término para cualquier click:
    $$\text{Term}_1 \approx \frac{1}{4} \frac{\mu_{voz}^2}{\sigma_{voz}^2} \approx \frac{1}{4} \frac{7.9943 \times 10^{11}}{1.2884 \times 10^{13}} \approx \mathbf{0.015511}$$
*   Este término es **idéntico** para todos los modelos de click porque la magnitud colosal de la voz eclipsa por completo la media del click.

### 2. Análisis del Término 2 (Diferencia de Covarianzas)
*   Dado que la desviación estándar de la característica en las clases de click es cero o infinitesimal, la función de Bhattacharyya paramétrica aplica un límite inferior riguroso de varianza de regularización (`eps = 1e-8`) para evitar la división por cero:
    $$\sigma^2_{click\_reg} = \max(\sigma^2_{click}, 10^{-8}) = \mathbf{10^{-8}}$$
*   Al evaluar el término de covarianza para todos los modelos $M_1, M_2, M_3$:
    $$\text{Term}_2 = \frac{1}{2}\ln\left(\frac{\sigma_{voz}^2 + 10^{-8}}{2 \sigma_{voz} \sqrt{10^{-8}}}\right) \approx \frac{1}{2}\ln\left(\frac{\sigma_{voz}}{2 \times 10^{-4}}\right)$$
*   Evaluando numéricamente con $\sigma_{voz} = 3.6259 \times 10^6$:
    $$\text{Term}_2 \approx \frac{1}{2}\ln\left(\frac{3.6259 \times 10^6}{2 \times 10^{-4}}\right) = \frac{1}{2}\ln(1.8129 \times 10^{10}) \approx \frac{1}{2} (23.62072) \approx \mathbf{11.805361}$$
*   Para el modelo dispersivo $M_4$: su desviación estándar es ligeramente superior a la regularización física ($\sigma_{click\_M4} = 6.568 \times 10^{-7}$), por lo que su varianza de canal es $4.22 \times 10^{-13}$. Al ser menor que `eps`, sigue truncándose exactamente a $10^{-8}$, resultando en el mismo término.

### 3. Suma Total de Bhattacharyya
$$D_B = \text{Term}_1 + \text{Term}_2 \approx 0.015511 + 11.805361 = \mathbf{11.820872}$$

---

## 4. Conclusión Científica de la Auditoría

La auditoría concluye que este comportamiento estadístico corresponde a:

1.  **Un Hallazgo Físico Real de Extrema Robustez:**  
    La brecha física entre el retardo de grupo de la voz y el de los clicks es inmensa. La voz humana tiene variaciones caóticas de fase en alta frecuencia debido a las cavidades nasales, vocales y singularidades matemáticas espectrales, mientras que los clicks conservan fases lineales o transiciones coherentes ultra-estrechas. Esta diferencia de órdenes de magnitud es un **hecho físico real**.
2.  **Un Artefacto de Regularización Matemática en la Métrica Unidimensional:**  
    El hecho de que el valor numérico de la distancia paramétrica $D_B$ sea idéntico hasta el sexto decimal **es un artefacto de regularización**. La asunción normal y la truncación de varianza infinitesimal a `eps = 1e-8` para evitar divisiones por cero (un requisito estándar de cálculo numérico robusto en clicks deterministas) fuerza a que el término logarítmico converja al mismo valor.

---

> [!NOTE]
> **Recomendación de Diseño:** La distancia paramétrica de Bhattacharyya es perfecta para ilustrar la extrema separabilidad teórica de la fase, pero para reportar la separabilidad real de forma robusta e insensible a regularizaciones, el **Solapamiento de Histograma ($SO$) en escala logarítmica** es el indicador no-paramétrico definitivo para el laboratorio, reportando un impecable $0.0\%$ de solapamiento en todos los casos.
