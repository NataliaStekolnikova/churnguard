# ChurnGuard v7 — Guía de Usuario Completa

## Tabla de Contenidos

1. [Introducción](#introducción)
2. [Primeros Pasos](#primeros-pasos)
3. [Módulo 1: Upload Data](#módulo-1-upload-data)
4. [Módulo 2: Overview](#módulo-2-overview)
5. [Módulo 3: At-Risk List](#módulo-3-at-risk-list)
6. [Módulo 4: Risk Predictor](#módulo-4-risk-predictor)
7. [Módulo 5: Model Details](#módulo-5-model-details)
8. [Casos de Uso](#casos-de-uso)
9. [Mejores Prácticas](#mejores-prácticas)
10. [FAQ](#faq)

---

## Introducción

ChurnGuard es una herramienta de inteligencia predictiva diseñada para identificar clientes en riesgo de abandono (churn) y proporcionar recomendaciones accionables para retenerlos.

### ¿Para quién es ChurnGuard?

- **Equipos CRM** — Identificar clientes en riesgo antes de que se vayan
- **Equipos de Ventas** — Priorizar llamadas de retención por nivel de riesgo
- **Marketing** — Segmentar campañas por probabilidad de churn
- **Dirección** — Visualizar KPIs de retención y capital en riesgo
- **Analistas** — Explorar patrones de churn y factores de impacto

---

## Primeros Pasos

### Paso 1: Abrir ChurnGuard

1. Descarga `ChurnGuard_v7.html` en tu ordenador
2. Haz doble clic en el archivo (se abrirá en tu navegador predeterminado)
3. **Alternativa:** Arrastra el archivo a una ventana del navegador

> 💡 **Tip:** ChurnGuard funciona completamente offline. No necesitas conexión a internet después de cargarlo.

### Paso 2: Preparar tus datos

ChurnGuard requiere un archivo CSV con datos de clientes. El formato mínimo es:

```csv
customerID,tenure,MonthlyCharges,Contract
CUST001,12,45.50,Month-to-month
CUST002,36,75.20,One year
CUST003,2,89.95,Month-to-month
```

**Columnas obligatorias:**
- `tenure` — Antigüedad del cliente en meses
- `MonthlyCharges` — Cargo mensual en euros
- `Contract` — Tipo de contrato

**Columnas opcionales (mejoran la precisión):**
- Todas las demás variables del modelo (ver README.md)

### Paso 3: Cargar datos

1. En la pestaña **📂 Upload Data**, arrastra tu CSV o haz clic para seleccionarlo
2. ChurnGuard detectará automáticamente las columnas
3. Si es necesario, mapea manualmente las columnas con los desplegables
4. Haz clic en **▶ Run Analysis**
5. Espera a que se complete el análisis (barra de progreso)

---

## Módulo 1: Upload Data

### Funcionalidad

Este módulo gestiona la carga y validación de datos de clientes.

### Pasos Detallados

#### 1. Seleccionar archivo

**Opción A: Drag & Drop**
- Arrastra tu archivo CSV directamente sobre la zona de carga
- Verás un indicador visual cuando el archivo esté sobre la zona

**Opción B: Click to Browse**
- Haz clic en la zona de carga
- Selecciona tu archivo CSV desde el explorador de archivos

#### 2. Detección automática

ChurnGuard automáticamente:
- ✅ Detecta el delimitador (coma `,` o punto y coma `;`)
- ✅ Lee los nombres de las columnas
- ✅ Mapea columnas conocidas
- ✅ Maneja campos entre comillas con delimitadores embebidos

#### 3. Mapeo de columnas (si es necesario)

Si ChurnGuard no puede mapear automáticamente las columnas:

1. Aparecerá una sección **Column Mapping**
2. Para cada variable del modelo, selecciona la columna correspondiente de tu CSV
3. Las columnas marcadas con **⚠** son obligatorias
4. Las columnas opcionales mejorarán la precisión pero no son requeridas

**Ejemplo de mapeo:**
```
tenure        → [Seleccionar columna]   ⚠ Obligatorio
MonthlyCharges → [Seleccionar columna]   ⚠ Obligatorio
Contract       → [Seleccionar columna]   ⚠ Obligatorio
gender         → [Seleccionar columna]   Opcional
```

#### 4. Ejecutar análisis

1. Haz clic en **▶ Run Analysis**
2. ChurnGuard procesará todos los registros:
   - Limpia y valida los datos
   - Calcula la probabilidad de churn para cada cliente
   - Asigna nivel de riesgo (Low/Moderate/High/Critical)
   - Genera visualizaciones y estadísticas
3. Las pestañas **Overview**, **At-Risk List**, **Predictor** y **Model Details** se habilitarán automáticamente

### Formatos Soportados

| Formato | Soportado | Notas |
|---------|-----------|-------|
| CSV (UTF-8) | ✅ | Recomendado |
| CSV (ISO-8859-1) | ✅ | Automático |
| Excel (.xlsx) | ❌ | Guardar como CSV primero |
| TSV (tabs) | ⚠️ | Convertir a CSV con comas |

### Límites de Tamaño

- **Recomendado:** < 5,000 registros
- **Máximo:** ~10,000 registros
- **Tiempo de procesamiento:** ~0.5 segundos por 1,000 registros

---

## Módulo 2: Overview

### Funcionalidad

Dashboard ejecutivo con KPIs, gráficos y distribución de riesgo.

### KPIs Principales

#### 1. Churn Rate
```
26.5%
1,869 of 7,043 customers
```
- **Qué muestra:** Porcentaje de clientes que han abandonado (si la columna `Churn` está presente)
- **Interpretación:** 
  - < 15%: Tasa de churn baja (buena retención)
  - 15-25%: Tasa de churn media (mejoras necesarias)
  - > 25%: Tasa de churn alta (acción urgente)

#### 2. Capital at Risk
```
€1.2M
2,120 customers with P ≥ 50%
```
- **Qué muestra:** Ingresos anuales en riesgo de clientes con ≥50% probabilidad de churn
- **Cálculo:** Suma de (MonthlyCharges × 12) para clientes High + Critical
- **Uso:** Dimensionar el presupuesto de campañas de retención

#### 3. Customers Loaded
```
7,043
23 features detected
```
- **Qué muestra:** Total de registros procesados
- **Features:** Número de variables usadas por el modelo

#### 4. Model AUC-ROC
```
0.8416
Recall 78.3%
```
- **Qué muestra:** Precisión del modelo (0.5 = aleatorio, 1.0 = perfecto)
- **Interpretación:**
  - 0.5-0.7: Modelo débil
  - 0.7-0.8: Modelo aceptable
  - 0.8-0.9: Modelo bueno ✅
  - 0.9-1.0: Modelo excelente

### Gráficos Interactivos

#### Churn Rate by Contract Type
```
Month-to-month  ████████████ 42.7%
One year        ███ 11.3%
Two year        █ 2.8%
```
- **Insight clave:** Los contratos mensuales tienen 15× más riesgo que contratos de 2 años
- **Acción:** Priorizar conversión de month-to-month a contratos anuales

#### Churn Rate by Tenure Group
```
0-12 months   ████████████ 47.4%
13-24 months  ████ 15.2%
25-48 months  ██ 8.1%
49+ months    █ 3.4%
```
- **Insight clave:** El primer año es crítico
- **Acción:** Programa de onboarding intensivo para nuevos clientes

#### Churn Rate by Internet Service
```
Fiber optic   ████████ 41.9%
DSL           ███ 18.9%
No internet   █ 7.4%
```
- **Insight clave:** Fiber optic tiene mayor abandono (gap precio/valor)
- **Acción:** Bundling de servicios para justificar precio premium

#### Risk Tier Distribution
```
🟢 Low       2,963 (42.1%)
🟡 Moderate  1,160 (16.5%)
🟠 High      1,143 (16.2%)
🔴 Critical  1,777 (25.2%)
```
- **Uso:** Cuantificar la magnitud del problema de churn
- **Acción:** Asignar recursos proporcionalmente a cada tier

### Acciones en Overview

#### Imprimir / Guardar PDF
- Haz clic en **🖨 Print / Save PDF** (esquina superior derecha)
- El navegador mostrará el diálogo de impresión
- Selecciona "Guardar como PDF" como destino
- **Uso:** Reportes ejecutivos, presentaciones

#### Navegación a Detalles
- Haz clic en cualquier tarjeta de tier (🔴 Critical, 🟠 High, 🟡 Moderate, 🟢 Low)
- Serás redirigido automáticamente a la pestaña **At-Risk List** con el filtro aplicado

---

## Módulo 3: At-Risk List

### Funcionalidad

Lista completa de clientes con probabilidad de churn calculada, ordenados por riesgo.

### Tabla de Clientes

#### Columnas

| Columna | Descripción | Ejemplo |
|---------|-------------|---------|
| **#** | Índice secuencial | 1, 2, 3... |
| **Customer ID** | Identificador único | 7590-VHVEG |
| **Churn Prob** | Probabilidad de churn | 85.2% |
| **Risk Tier** | Nivel de riesgo | 🔴 Critical |
| **Tenure** | Antigüedad en meses | 12 mo |
| **Monthly €** | Cargo mensual | €75.50 |
| **Contract** | Tipo de contrato | Month-to-month |
| **Internet** | Servicio de internet | Fiber optic |
| **Action** | Recomendación | Premium call |

#### Orden Predeterminado

Por defecto, los clientes están ordenados de **mayor a menor probabilidad de churn** (Critical primero).

### Filtros Avanzados

#### 1. Búsqueda por Customer ID
```
🔍 Search by Customer ID...
```
- Escribe el ID del cliente (completo o parcial)
- Los resultados se filtran en tiempo real
- **Uso:** Buscar un cliente específico rápidamente

#### 2. Filtro por Risk Tier
```
[All tiers ▼]
```
Opciones:
- All tiers (mostrar todos)
- 🔴 Critical
- 🟠 High
- 🟡 Moderate
- 🟢 Low

**Uso:** Segmentar clientes por nivel de riesgo para campañas diferenciadas

#### 3. Filtro por Contrato
```
[All contracts ▼]
```
Opciones:
- All contracts
- Month-to-month
- One year
- Two year

**Uso:** Identificar oportunidades de conversión contractual

#### 4. Filtro por Internet Service
```
[All internet ▼]
```
Opciones:
- All internet
- Fiber optic
- DSL
- No internet

**Uso:** Segmentar por tipo de servicio (ej: solo clientes Fiber)

#### 5. Filtro por Rango de Probabilidad
```
Min risk %: [━━━○━━━━━━] 35%
Max risk %: [━━━━━━━○━━] 85%
```
- Arrastra los sliders para definir el rango
- **Uso:** Crear segmentos personalizados (ej: 50-70% para campaña específica)

#### 6. Reset Filters
```
[✕ Reset]
```
- Elimina todos los filtros aplicados
- Restaura la vista completa

### Ordenación

Haz clic en cualquier encabezado de columna para ordenar:

- **Customer ID** — Orden alfabético
- **Churn Prob** — Probabilidad (descendente/ascendente)
- **Risk Tier** — Critical → High → Moderate → Low
- **Tenure** — Antigüedad en meses
- **Monthly €** — Cargo mensual
- **Contract** — Tipo de contrato

**Indicador:** ▲ (ascendente) o ▼ (descendente)

### Paginación

```
Showing 1-50 of 7,043 customers
[◀] [1] [2] [3] ... [141] [▶]
```

- **Registros por página:** 50 (fijo)
- **Navegación:** Usa los botones numéricos o flechas
- **Página activa:** Resaltada en color accent

### Exportación a CSV

```
[⬇ Export CSV]
```

#### Pasos:
1. Aplica filtros si solo quieres exportar un subconjunto
2. Haz clic en **⬇ Export CSV**
3. El navegador descargará un archivo `at-risk-customers.csv`

#### Contenido del Export:
El CSV exportado incluye **todos los registros filtrados** (no solo la página actual) con las siguientes columnas:

```csv
CustomerID,ChurnProb,RiskTier,Tenure,MonthlyCharges,Contract,InternetService,Action
7590-VHVEG,0.852,Critical,1,29.85,Month-to-month,DSL,Premium call
...
```

#### Uso del Export:
- **Importar a CRM** — Salesforce, HubSpot, Dynamics
- **Campañas de email** — Mailchimp, SendGrid
- **Call center** — Priorización de llamadas
- **Análisis adicional** — Excel, Power BI, Tableau

---

## Módulo 4: Risk Predictor

### Funcionalidad

Predictor individual en tiempo real. Introduce los datos de un cliente y obtiene instantáneamente su probabilidad de churn y recomendación de retención.

### Formulario de Entrada

El formulario está organizado en **3 tiers** según el impacto en la predicción:

#### ⭐ Tier 1: Critical Factors
**Top-5 features en los 3 modelos (LR, RF, GB)**

1. **Contract**
   ```
   [Month-to-month ▼]
   ```
   - Month-to-month (mayor riesgo)
   - One year
   - Two year (menor riesgo)

2. **MonthlyCharges**
   ```
   [━━━━━○━━━━] €75.50
   ```
   - Rango: €18 - €120
   - Valor predeterminado: €65

3. **tenure**
   ```
   [━○━━━━━━━━] 12 months
   ```
   - Rango: 0 - 72 meses
   - Valor predeterminado: 12 meses

#### ⭐ Tier 2: High Impact Factors

4. **TotalCharges**
   ```
   [Automatic calculation ✓]
   ```
   - Calculado automáticamente como: `MonthlyCharges × tenure`
   - Campo de solo lectura

5. **InternetService_Fiber optic**
   ```
   [Yes ▼]
   ```
   - Yes (mayor riesgo)
   - No

#### ⭐ Tier 3: Moderate Impact

- **OnlineSecurity** — Yes/No
- **PaymentMethod** — Electronic check / Mailed check / Bank transfer / Credit card
- **PaperlessBilling** — Yes/No
- **SeniorCitizen** — 0/1
- **Partner** — Yes/No
- **Dependents** — Yes/No
- **PhoneService** — Yes/No
- **MultipleLines** — Yes/No
- **OnlineBackup** — Yes/No
- **DeviceProtection** — Yes/No
- **TechSupport** — Yes/No
- **StreamingTV** — Yes/No
- **StreamingMovies** — Yes/No

### Ejecutar Predicción

1. Completa al menos los **Tier 1 factors** (obligatorios)
2. Opcionalmente, completa Tier 2 y 3 para mayor precisión
3. Haz clic en **🎯 PREDICT CHURN RISK**
4. Los resultados aparecen instantáneamente en el panel derecho

### Panel de Resultados

#### 1. Probabilidad de Churn
```
    85.2%
```
- Número grande y destacado
- Color dinámico según nivel de riesgo

#### 2. Risk Label
```
[🔴 CRITICAL RISK]
```
- Etiqueta codificada por colores
- 4 posibles valores: Low / Moderate / High / Critical

#### 3. Gauge Visual
```
[████████▓▓] 85.2%
```
- Barra de progreso que representa visualmente el riesgo
- Color matching con el nivel de riesgo

#### 4. Recommended Action
```
⚡ RETENTION ACTION
Call within 24 hours. Offer annual contract upgrade.
If Fiber Optic: free OnlineSecurity 6 months.
Switch to auto-payment.
```
- Recomendación específica basada en el nivel de riesgo
- Accionable inmediatamente por el equipo CRM

#### 5. Impact Factors
```
🔴 Contract (Month-to-month)     +HIGH
🔴 tenure (12 mo)                +HIGH
🟡 MonthlyCharges (€75.50)       +MED
🟢 Partner (Yes)                 -LOW
```
- Lista de factores que influyen en la predicción
- 🔴 = Aumenta el riesgo
- 🟢 = Reduce el riesgo
- Ordenados por impacto (mayor a menor)

### Lógica de Recomendaciones

| P(churn) | Risk Tier | Acción Recomendada |
|----------|-----------|-------------------|
| ≥ 70% | 🔴 Critical | **Llamada urgente (< 24h)** — Ofrecer upgrade a contrato anual con descuento. Si tiene Fiber: gratis OnlineSecurity 6 meses. Cambiar a autopago. |
| 50-69% | 🟠 High | **Llamada proactiva** — Revisar contrato y método de pago. Proponer upgrade o soporte de onboarding. |
| 30-49% | 🟡 Moderate | **Campaña nurture** — Email de satisfacción. Resaltar servicios no usados para aumentar switching cost. |
| < 30% | 🟢 Low | **Cliente estable** — Considerar upselling. Agregar servicios aumenta lealtad y switching cost. |

---

## Módulo 5: Model Details

### Funcionalidad

Información técnica del modelo de Machine Learning, métricas de rendimiento y detalles del dataset.

### Sección 1: Model Performance Metrics

#### Tabla de Métricas

| Métrica | Valor | Interpretación |
|---------|-------|----------------|
| **Accuracy** | 73.9% | % de predicciones correctas |
| **Precision** | 50.5% | % de alarmas que son reales |
| **Recall** | 78.3% | % de churners detectados |
| **F1-Score** | 61.4% | Media armónica Precision-Recall |
| **AUC-ROC** | 0.8416 | Separabilidad general |

#### Interpretación de Métricas

**Recall (78.3%)**  
"De cada 100 clientes que realmente abandonan, el modelo detecta 78"
- **Crítico:** Alto Recall minimiza clientes perdidos (falsos negativos)
- Trade-off: Mayor Recall → Menor Precision (más falsas alarmas)

**Precision (50.5%)**  
"De cada 100 clientes que el modelo predice que abandonarán, 50 realmente lo harán"
- Moderada Precision aceptable bajo imbalance de clases 2.77:1
- Trade-off: Mayor Precision → Menor Recall (más clientes perdidos)

**AUC-ROC (0.8416)**  
Probabilidad de que el modelo asigne mayor score a un churner que a un cliente retenido
- 0.5 = Aleatorio
- 0.8 = Bueno ✅
- 0.9+ = Excelente

### Sección 2: Top Coefficients

Muestra los factores con mayor impacto en la predicción (coeficientes del modelo Logistic Regression):

```
Contract (Month-to-month)     ████████████ +2.531
tenure                        ████████     -1.842
InternetService_Fiber optic   ██████       +1.204
MonthlyCharges                ████         +0.892
PaymentMethod_Electronic check ███         +0.673
```

**Interpretación:**
- **Positivo (+)** — Aumenta la probabilidad de churn
- **Negativo (-)** — Reduce la probabilidad de churn
- **Magnitud** — Importancia relativa del factor

### Sección 3: Dataset Statistics

```
Total Customers:    7,043
Features:           23
Churn Rate:         26.5% (1,869 churned)
Average Tenure:     32.4 months
Average Monthly:    €64.76
```

- **Uso:** Contextualizar las predicciones
- **Benchmark:** Comparar con datasets futuros

### Sección 4: Export Python Script

```
[🐍 Export Python Scoring Script]
```

#### Funcionalidad:
Genera un script Python standalone que replica el modelo de ChurnGuard para uso en otros entornos.

#### Contenido del Script:
- Coeficientes del modelo (intercept + weights)
- Función `predict_churn(customer_data)`
- Función `score_dataframe(df)`
- Ejemplo de uso

#### Uso:
```python
import pandas as pd
from churnguard_model import predict_churn

customer = {
    'tenure': 12,
    'MonthlyCharges': 75.50,
    'Contract': 'Month-to-month',
    # ... otros features
}

churn_prob, risk_tier = predict_churn(customer)
print(f"Probability: {churn_prob:.1%}, Tier: {risk_tier}")
# Output: Probability: 85.2%, Tier: Critical
```

---

## Casos de Uso

### Caso 1: Campaña de Retención Mensual

**Objetivo:** Identificar clientes de alto riesgo para contacto proactivo

**Pasos:**
1. Carga el CSV con datos actualizados del mes
2. Ve a **At-Risk List**
3. Filtra por **Risk Tier = Critical** o **High**
4. Exporta la lista a CSV
5. Importa el CSV a tu sistema de call center
6. Prioriza llamadas por probabilidad de churn (mayor primero)

**Resultado:** Lista priorizada de 2,920 clientes (Critical + High) para contactar

### Caso 2: Evaluación Individual Pre-Llamada

**Objetivo:** Preparar argumentos de retención personalizados antes de llamar a un cliente

**Pasos:**
1. Busca el cliente en **At-Risk List** por Customer ID
2. Anota su probabilidad de churn y Risk Tier
3. Ve a **Risk Predictor**
4. Introduce los datos del cliente (copiar de CRM)
5. Revisa los **Impact Factors** y la **Recommended Action**
6. Usa esta información para personalizar la llamada

**Resultado:** Argumentos de retención específicos basados en los factores de riesgo del cliente

### Caso 3: Análisis de Cohortes

**Objetivo:** Comparar tasas de churn entre diferentes segmentos

**Pasos:**
1. Carga todos los clientes en **Overview**
2. Analiza los gráficos de churn rate por:
   - Contract type
   - Tenure group
   - Internet service
3. Identifica el segmento con mayor riesgo
4. Ve a **At-Risk List** y filtra por ese segmento
5. Exporta y analiza los clientes específicos

**Resultado:** Identificación de segmento de mayor riesgo (ej: Month-to-month + Fiber optic en primer año)

### Caso 4: Dashboard Ejecutivo

**Objetivo:** Reportar KPIs de churn a Dirección mensualmente

**Pasos:**
1. Carga datos del mes actual
2. Ve a **Overview**
3. Captura pantalla o exporta PDF con:
   - Churn Rate actual
   - Capital at Risk
   - Distribución de Risk Tiers
   - Gráficos de análisis
4. Compara con meses anteriores

**Resultado:** Reporte visual ejecutivo listo para presentación

### Caso 5: Validación de Modelo

**Objetivo:** Verificar la precisión del modelo con datos históricos

**Pasos:**
1. Carga un CSV histórico que incluya la columna `Churn` (Yes/No real)
2. Ve a **Model Details**
3. Revisa las métricas de rendimiento
4. Compara predicciones con resultados reales
5. Analiza la matriz de confusión (si disponible)

**Resultado:** Validación de que el modelo mantiene > 75% de precisión

---

## Mejores Prácticas

### Preparación de Datos

✅ **DO:**
- Usa encoding UTF-8 para el CSV
- Incluye todas las columnas disponibles (mejor precisión)
- Verifica que `tenure` está en meses (no años)
- Asegura que `MonthlyCharges` está en la misma moneda (€)
- Elimina filas con valores faltantes en columnas obligatorias

❌ **DON'T:**
- No uses Excel directamente (guarda como CSV primero)
- No mezcles unidades (meses y años en tenure)
- No dejes columnas obligatorias vacías
- No uses caracteres especiales en Customer IDs

### Frecuencia de Actualización

| Frecuencia | Uso Recomendado |
|------------|-----------------|
| **Diaria** | Call centers con alta rotación |
| **Semanal** | Equipos CRM proactivos |
| **Mensual** | Reportes ejecutivos y campañas programadas |
| **Trimestral** | Análisis de tendencias y validación de modelo |

### Segmentación de Campañas

**Por Risk Tier:**
```
🔴 Critical (P > 70%)  → Llamada urgente + descuento agresivo
🟠 High (50-70%)       → Llamada proactiva + upgrade
🟡 Moderate (30-50%)   → Email personalizado + oferta
🟢 Low (< 30%)         → No acción / upselling
```

**Por Valor del Cliente:**
```
Alto valor + Critical  → Máxima prioridad, gestor dedicado
Alto valor + High      → Prioridad alta, oferta premium
Bajo valor + Critical  → Email automatizado
```

### Integración con CRM

**Salesforce:**
1. Exporta CSV desde At-Risk List
2. Importa como "Leads" o actualiza registros existentes
3. Crea campo personalizado "Churn_Risk_Score"
4. Configura alertas automáticas para Critical tier

**HubSpot:**
1. Exporta CSV
2. Usa "Import" → "Contacts"
3. Mapea "ChurnProb" a propiedad personalizada
4. Crea flujo de trabajo (workflow) basado en score

**Dynamics 365:**
1. Exporta CSV
2. Importa vía Data Management
3. Asigna clientes Critical a equipos de retención
4. Automatiza creación de tareas de seguimiento

---

## FAQ

### ¿Qué navegadores son compatibles?

ChurnGuard funciona en todos los navegadores modernos:
- Chrome 90+ ✅
- Firefox 88+ ✅
- Safari 14+ ✅
- Edge 90+ ✅
- Opera 76+ ✅

### ¿Necesito conexión a internet?

**No.** ChurnGuard funciona completamente offline una vez cargado. Todos los cálculos se realizan localmente en tu navegador.

### ¿Dónde se almacenan mis datos?

**Localmente en tu navegador.** ChurnGuard no envía datos a ningún servidor. Todo el procesamiento es local.

**Para persistir datos entre sesiones:**
- Los datos se almacenan en `localStorage` del navegador
- Se mantienen hasta que cierres la pestaña o recargues la página
- Para guardar permanentemente, exporta a CSV

### ¿Cuántos registros puede procesar?

**Límite recomendado:** 5,000 - 10,000 registros

**Rendimiento:**
- 1,000 registros: < 1 segundo
- 5,000 registros: ~3 segundos
- 10,000 registros: ~8 segundos

**Para datasets más grandes:**
- Divide el archivo en múltiples CSVs
- Procesa por lotes (ej: por región, por mes)

### ¿Puedo usar ChurnGuard en móvil?

**Sí**, pero con limitaciones:
- ✅ Visualizar dashboards
- ✅ Buscar clientes específicos
- ✅ Usar Risk Predictor
- ⚠️ Carga de archivos puede ser lenta en datasets grandes
- ⚠️ Gráficos pueden verse pequeños en pantallas < 6"

**Recomendación:** Usa desktop para análisis completo, móvil para consultas rápidas.

### ¿Cómo actualizo el modelo con nuevos datos?

El modelo está embebido en el archivo HTML. Para reentrenar:
1. Ejecuta los notebooks del proyecto (04b_ML_Training)
2. Exporta los nuevos coeficientes
3. Reemplaza los valores en ChurnGuard_v7.html (sección JavaScript)
4. Guarda y redistribuye

**Alternativa:** Contacta con el desarrollador para generar una nueva versión.

### ¿El modelo funciona para otros sectores?

ChurnGuard está **optimizado para telecomunicaciones**, pero puede adaptarse:
- **Retail / eCommerce:** Requiere reentrenamiento con features de compra
- **SaaS / Software:** Requiere features de uso del producto
- **Banca / Seguros:** Requiere features de productos financieros

**Para adaptar:** Necesitas:
1. Dataset del sector objetivo
2. Reentrenar modelo con features relevantes
3. Actualizar coeficientes en ChurnGuard

### ¿Qué hago si las predicciones parecen incorrectas?

**Diagnóstico:**
1. Verifica que los datos de entrada están en las unidades correctas
2. Comprueba que el modelo fue entrenado con datos similares
3. Revisa la columna `Churn` real (si disponible) vs predicciones
4. Analiza métricas en **Model Details**

**Si el problema persiste:**
- El modelo puede estar desactualizado (reentrenar con datos recientes)
- Los datos pueden tener drift (cambio en distribución)
- Features importantes pueden estar faltando

### ¿Puedo integrar ChurnGuard con mi sistema?

**Opciones:**

1. **Export CSV** — Método más simple, ideal para CRMs
2. **Python Script** — Exporta el modelo como script standalone
3. **API REST** — Requiere envolver ChurnGuard en un backend (Flask, FastAPI)
4. **Power BI / Tableau** — Importa CSVs exportados como fuente de datos

**Para integración avanzada:** Contacta con el desarrollador.

### ¿Cómo interpreto un Recall de 78.3%?

**Recall = 78.3%** significa:
- De cada 100 clientes que **realmente abandonan**, el modelo **detecta 78**
- **21 clientes** que abandonan **no son detectados** (falsos negativos)

**¿Es bueno?**
- Sí, para un modelo binario bajo imbalance 2.77:1
- El modelo multi-clase (v7.0) alcanza **96.9% Critical Recall**

**Trade-off:**
- Aumentar Recall → Más falsos positivos (más llamadas innecesarias)
- Reducir Recall → Menos falsos positivos pero más clientes perdidos

### ¿Qué significa "class imbalance 2.77:1"?

En el dataset original:
- **5,174 clientes retenidos** (73.5%)
- **1,869 clientes churn** (26.5%)
- **Ratio:** 5,174 / 1,869 = 2.77:1

**Impacto:**
- Modelos simples pueden alcanzar 73.5% accuracy prediciendo siempre "No churn"
- Por eso usamos métricas como Recall, Precision, F1 y AUC-ROC
- ChurnGuard usa `class_weight='balanced'` para compensar el imbalance

---

## Soporte Técnico

### Reportar Bugs

Si encuentras un error:
1. Anota el navegador y versión
2. Captura pantalla del error (si aplica)
3. Describe los pasos para reproducirlo
4. Envía a: support@email.com

### Solicitar Features

Para sugerir nuevas funcionalidades:
- Describe el caso de uso
- Explica el beneficio esperado
- Envía a: support@email.com

### Documentación Adicional

- [README.md](./README.md) — Resumen general
- [TECHNICAL_DOCS.md](./TECHNICAL_DOCS.md) — Arquitectura y modelo

---

**ChurnGuard v7 — Predecir. Retener. Crecer.**

*Última actualización: Abril 2026*
