# 💧 S.I.G. Riego Pro (Sistema Integral de Gestión de Riego)

**Versión 1.0 (Estable)**

Una herramienta web avanzada de ingeniería agronómica diseñada para el cálculo, planificación y gestión eficiente de recursos hídricos en agricultura. Transforma datos climáticos históricos en planes de riego operativos, aplicando normativas internacionales (USDA, FAO) y lógica de balance hídrico neto.

---

## 🚀 Funcionalidades Clave

### 📍 1. Geolocalización y Climatología
* **Búsqueda Geoespacial:** Algoritmo que identifica automáticamente la estación meteorológica (AEMET) más cercana a las coordenadas exactas de la parcela (Lat/Lon).
* **Procesamiento de Datos:** Ingesta de archivos JSON (formato **AEMET OpenData**) con capacidad de procesar series históricas completas (medias aritméticas de todos los años disponibles) para obtener valores robustos de **ET<sub>0</sub>** (Evapotranspiración de Referencia) y **P** (Precipitación).
* > **⚠️ Nota Técnica sobre Proyección a Futuro:**
> Dado que la herramienta permite planificar campañas de cultivo en fechas futuras, el sistema genera un **modelo climático predictivo**.
> Para ello, calcula la **media aritmética mensual** de los datos presentes en el archivo JSON (utilizando la serie histórica disponible, típicamente los últimos 3 años). De esta forma, se proyecta un comportamiento climático estadísticamente representativo para los meses venideros, suavizando las anomalías puntuales de un año específico.

### 🥇 2. Balance Hídrico Mensual (Agronómico)
El núcleo del sistema se basa en la metodología del **Riego Neto**:
* **Cálculo de ET<sub>c</sub>:** Transformación de la ET<sub>0</sub> climática mediante Coeficientes de Cultivo (**K<sub>c</sub>**) dinámicos para obtener la demanda bruta.
* **Precipitación Efectiva (P<sub>e</sub>):** Implementación del **Método USDA (SCS)** modificado para calcular la lluvia útil aprovechable por el cultivo:
    * *Si P < 70mm:* `P<sub>e</sub> = 0.6 · P - 10`
    * *Si P > 70mm:* `P<sub>e</sub> = 0.8 · P - 24`
* **Necesidad Neta (NH<sub>n</sub>):** Cálculo del déficit real del cultivo (`ET<sub>c</sub> - P<sub>e</sub>`).
* **Gestión de Recursos:** Algoritmo de reparto proporcional (Estrategia de Riego Deficitario Controlado) que ajusta la dotación final si el volumen disponible es inferior a la demanda ideal.

### 📅 3. Planificación Operativa Semanal
* **Flujo Continuo:** Conversión de la planificación mensual a semanas naturales del año (ISO 8601).
* **Distribución Diaria:** Lógica de interpolación diaria que evita los "escalones" o cortes artificiales entre meses, generando una curva de riego suave, continua y agronómicamente viable.

### 📊 4. Visualización y Reporting
* **Dashboard Interactivo:** Gráficos profesionales (Chart.js) con diseño optimizado:
    * **Azul Cielo (`#38bdf8`):** Precipitación Efectiva (P<sub>e</sub>).
    * **Azul Real (`#2563eb`):** Necesidad Neta (NH<sub>n</sub>).
    * **Oro/Ámbar (`#d97706`):** Riego Asignado (Recurso Humano).
* **Exportación de Datos:** Generación automática de informes en Excel (`.xlsx`) con tablas detalladas (Balance Mensual y Plan Semanal) para el cuaderno de campo.

---

## 📐 Lógica Matemática del Balance

El sistema basa sus decisiones en el siguiente flujo de cálculo secuencial:

1.  **Demanda del Cultivo ($ET_c$):**
    $$ET_c = ET_0 \times K_c$$

2.  **Lluvia Útil ($P_e$ - Método USDA S.C.S.):**
    Se implementa el algoritmo empírico del *Soil Conservation Service* para estimar la fracción de lluvia que realmente se almacena en la zona radicular, descartando escorrentía superficial y percolación profunda. Se discrimina según la intensidad de la precipitación mensual ($P_{mes}$):

    * **Para precipitaciones bajas/medias ($P_{mes} < 70 \text{ mm}$):**
        $$P_e = (P_{mes} \times 0.6) - 10$$
        *(Se asume mayor pérdida proporcional por evaporación superficial)*

    * **Para precipitaciones altas ($P_{mes} \ge 70 \text{ mm}$):**
        $$P_e = (P_{mes} \times 0.8) - 24$$
        *(Se asume mayor eficiencia de infiltración, pero mayor pérdida por escorrentía)*

    *> Nota: El sistema aplica un suelo de $0$ ($P_e \ge 0$) para evitar valores negativos en meses muy secos.*

3.  **Necesidad Hídrica Neta ($NH_n$):**
    $$NH_n = Max(0, ET_c - P_e)$$

4.  **Factor de Déficit ($K_s$):**
    $$K_s = \frac{Volumen\ Disponible}{\sum NH_n}$$

5.  **Riego Final Asignado:**
    $$Riego = NH_n \times K_s$$
---

## 🛠️ Tecnologías y Diseño

* **Frontend:** HTML5, CSS3 (Diseño "Clean Card"), Vanilla JavaScript (ES6+).
* **Motor Gráfico:** Chart.js + Plugin DataLabels (Estilo personalizado con tooltips modernos).
* **Motor de Datos:** SheetJS (XLSX) para la generación de hojas de cálculo.
* **UX:** Interfaz reactiva con feedback visual inmediato y validación de datos de entrada.

---

> **Nota:** Este proyecto ha sido desarrollado siguiendo estrictos criterios agronómicos para ofrecer una herramienta de precisión a técnicos y gestores de fincas.
