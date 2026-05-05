---
marp: true
theme: default
paginate: true
backgroundColor: #0f172a
color: #e2e8f0
style: |
  section {
    font-family: 'Segoe UI', Arial, sans-serif;
    font-size: 16px;
  }
  h1 {
    color: #38bdf8;
    font-size: 1.8em;
    border-bottom: 3px solid #0ea5e9;
    padding-bottom: 8px;
    margin-bottom: 16px;
  }
  h2 {
    color: #7dd3fc;
    font-size: 1.25em;
    margin-top: 14px;
    margin-bottom: 8px;
  }
  h3 {
    color: #bae6fd;
    font-size: 1.05em;
    margin-top: 10px;
    margin-bottom: 6px;
  }
  table {
    font-size: 0.7em;
    width: 100%;
    border-collapse: collapse;
  }
  th {
    background-color: #2b548a;
    color: #ffffff;
    padding: 8px 10px;
    font-weight: 700;
  }
  td {
    padding: 6px 10px;
    border: 1px solid #1e3a5f;
    color: #e2e8f0;
  }
  tbody tr:nth-child(even) td {
    background-color: #1e293b;
  }
  tbody tr:nth-child(odd) td {
    background-color: #43526d;
  }
  .lead h1 {
    font-size: 2.2em;
    text-align: center;
    border: none;
    color: #38bdf8;
  }
  .lead h2 {
    text-align: center;
    color: #94a3b8;
    font-size: 1.1em;
  }
  .lead p {
    text-align: center;
    color: #64748b;
    font-size: 0.8em;
  }
  blockquote {
    background: #2b548a;
    border-left: 4px solid #38bdf8;
    padding: 10px 16px;
    border-radius: 4px;
    font-style: normal;
    color: #ffffff;
    margin: 10px 0;
    font-size: 0.9em;
  }
  code {
    background: #1e293b;
    color: #7dd3fc;
    padding: 2px 6px;
    border-radius: 3px;
    font-size: 0.9em;
  }
  pre {
    background: #1e293b;
    border: 1px solid #1e3a5f;
    border-radius: 6px;
    padding: 12px;
    font-size: 0.72em;
    line-height: 1.5;
  }
  ul {
    margin: 6px 0;
    padding-left: 20px;
  }
  li {
    margin-bottom: 4px;
  }
  footer {
    color: #475569;
    font-size: 0.65em;
  }
  .warn {
    color: #fb923c;
    font-weight: bold;
  }
  .ok {
    color: #4ade80;
  }
  .cost {
    color: #facc15;
    font-weight: bold;
  }
  .highlight {
    color: #38bdf8;
  }
---

<!-- _class: lead -->

<br>

# Reducción del Churn en<br>Telecomunicaciones<br>mediante IA

**Estrategia de Retención Proactiva con Analítica Predictiva y Machine Learning**
Programación en Inteligencia Artificial y Big Data aplicables en entornos 5G 

<br>

**Natalia Stekolnikova**
Abril de 2026

---

# El Problema — ¿Cuánto cuesta el churn?

**Contexto:** Una empresa de telecomunicaciones pierde clientes de forma silenciosa cada mes.
![w:1000](slide_problem.png)

| Indicador | Valor |
|---|---|
| **Tasa de churn anual** | <span class="red">26.5%</span> — 1 de cada 4 clientes |
| **Capital histórico perdido** | <span class="red">€2.86M</span> |
| **Pérdida neta anual (margen 2%)** | <span class="orange">€57.3K</span> |

<br>

> 💡 Un cliente retenido vale **1.7×** más que uno perdido.
> El primer año es crítico — **47.4% de los nuevos clientes** se van antes de los 12 meses.

---

# Detalle Técnico — Pipeline del Proyecto (6 notebooks Python + Power BI + Web App)

**Notebook 01 — Extract & Quality Check**
Control de calidad: NaN, dtypes, outliers, contradicciones lógicas

**Notebook 02 — Clean & Transform**
Fix Data · Feature Engineering

**Notebook 03 — SQLite + SQL**
Clientes de máximo riesgo identificados

**Notebook 04a — ML Training: Binary Classification**
Clasificación binaria (churn 0/1) · 3 modelos · AUC-ROC 0.84 · Recall 78.3%

**Notebook 04b — ML Training: Multi-Class (4 niveles de riesgo)**
Out-of-fold sin leakage · **AUC-OVR 0.9974** · Accuracy 95.1% · Critical Recall 96.9% · Modelo desplegado

**Notebook 05 — Stacking Ensemble**
Meta-learner: LR+RF+GB → LR · Critical Recall 98.0%

<br>

> **ChurnGuard.html** — aplicación web standalone con predictor en tiempo real

> **Churn_dashdoard.pbix** — dashboard interactivo Power BI con análisis de churn, KPIs y visualizaciones (versión 3)

---

# ChurnGuard — Valor para el Negocio

**Aplicación web standalone** — sin instalación, sin Python, accesible desde cualquier navegador

<br>

| Módulo | Función | Usuario |
|---|---|---|
| **📂 Upload Data** | Carga y procesamiento de archivos CSV con datos de clientes | Todos los usuarios |
| **📊 Overview** | Dashboard KPI: churn rate, capital en riesgo, segmentación por contrato, tenure y servicio de internet | Dirección, Analytics |
| **👥 At-Risk List** | Lista completa de clientes con scoring ML, ordenación por riesgo, filtros avanzados y exportación a CRM | Equipos CRM y Ventas |
| **🎯 Risk Predictor** | Predicción de riesgo individual en tiempo real mediante Regresión Logística embebida | Equipos CRM y Ventas |
| **🧠 Model Details** | Métricas del modelo, coeficientes estandarizados y estadísticas del dataset | Dirección Técnica |

<br>

> **Impacto directo:** cualquier agente comercial puede evaluar el riesgo de un cliente en segundos y recibir una recomendación de retención concreta — sin conocimientos técnicos.

> **Despliegue cero:** un único archivo `.html` — no requiere servidor, ni base de datos, ni infraestructura adicional.

---

# Detalle Técnico — EDA — Análisis Exploratorio de Datos

![w:1100](slide_eda_categorical.png)

---

# Detalle Técnico — Importancia de las variables

![w:1100](slide_feature_importance.png)

---

# Detalle Técnico — Factores Clave de Churn

<br>

- **Tipo de contrato:** 42.7% vs 2.8% → diferencia **15×**
- **Monthly Charges:** pico en €86-105 → **37.8%**
- **Tenure:** primer año → **47.4%** churn
- **Fiber Optic:** 41.9% vs 7.4% → gap precio/valor
- **Electronic Check:** 45.3% — pago manual = decisión de salir

---

# Detalle Técnico — Comparación de Modelos Base

![bg right:55% contain](slide_roc_curves.png)

**Evaluación de 3 modelos (NB05)**

| Modelo | AUC-OVR | Wtd Recall | Coste de error |
|---|---|---|---|
| Log. Regression | **0.9974** | **0.951** | **€7.425** |
| Gradient Boosting | 0.9841 | 0.907 | €15.690 |
| Random Forest | 0.9823 | 0.900 | €19.985 |

<br>

**Mejor modelo base: Logistic Regression**

> Recall Crítico 96.9% = detectamos **344 de cada 355** clientes de máximo riesgo
> Matriz de coste 4×4 penaliza falsos negativos en Critical (€1.532 LTV perdido)

---

# Detalle Técnico — Stacking Ensemble (Modelo Final)

![bg right:50% contain](slide_stacking_comparison.png)

**Arquitectura del Ensemble (Notebook 05):** Stacking LR+RF+GB → LR

### Valor del Ensemble vs modelo individual

| Métrica | LR Base | Stacking | Mejora |
|---|---|---|---|
| **Critical Recall** | 96.9% | **98.0%** | **+1.1pp** |
| **Coste de error** | €7.425 | **€6.970** | **−6.1%** |
| **Accuracy** | 95.1% | **96.0%** | +0.9pp |
| **Macro AUC-OVR** | 0.9974 | **0.9978** | +0.0004 |

> **348 de 355 clientes críticos detectados** — solo 7 falsos negativos
> Validado en 1.409 clientes de test — mejora consistente en todas las métricas
> Limitación: no embebible en ChurnGuard.html (requiere 3 modelos + meta)

---

# Solución Propuesta — Pipeline de IA + Acciones de Marketing

<br>

**El objetivo:** predecir *quién* va a irse *antes* de que ocurra y actuar con la intervención exacta al nivel de riesgo.

<br>

### Tres campañas escalonadas derivadas directamente del modelo ML

| Acción | Segmento | P(churn) | Intervención |
|---|---|---|---|
| **1. Email segmentado** | 1.160 clientes Riesgo Moderado | 0.30 – 0.50 | Oferta personalizada por email (€5/cliente) |
| **2. Llamada proactiva** | 1.143 clientes Riesgo Alto | 0.50 – 0.70 | Llamada de retención preventiva (€25/cliente) |
| **3. Campaña Premium** | 1.777 clientes Riesgo Crítico | > 0.70 | Descuento VIP + llamada personal (€50/cliente) |

<br>

> La intervención es **proporcional al riesgo**: no se llama a clientes de bajo riesgo y no se envía solo un email a clientes críticos. Cada euro se invierte donde tiene mayor impacto.

---

# Propuesta de Valor — Retorno sobre la Inversión

![bg right:52% contain](slide_roi_v2.png)

**Inversión total de la campaña:** €122.945

<br>

| Acción | Inversión | Ingresos | **Net Benefit** | **ROI** |
|---|---|---|---|---|
| 1. Email Moderado | €6.5K | €174K | €167K | **2.572%** |
| 2. Llamada Alto | €29K | €261K | €232K | **793%** |
| 3. Premium Crítico | €87K | €545K | €458K | **525%** |
| **TOTAL** | **€123K** | **€980K** | **€857K** | **697%** |

<br>

> Por cada **€1 invertido → retorno de €6.97**
> La campaña cubre las pérdidas anuales **15.0 veces**

<span class="tag tag-green">1.097 clientes retenidos</span> &nbsp;
<span class="tag tag-red">sin acción: -€57K/año</span>

---

# Conclusión

<br>

## Lo que hemos demostrado
<br>

✅ **El problema está cuantificado** — €2.86M capital perdido, 26.5% tasa de churn anual

✅ **El modelo predice con fiabilidad** — Stacking Ensemble: AUC-OVR 0.9978 · Recall Crítico 98.0% · Coste €6.970 · validado en 1.409 clientes

✅ **Las acciones están basadas en datos** — cada campaña ataca exactamente el nivel de riesgo predicho por el modelo ML

✅ **El ROI es positivo y medible** — €857K beneficio neto · 697% ROI · 1.097 clientes retenidos


---
<!-- _paginate: false -->

<br><br>

# Gracias
<br>

**Natalia Stekolnikova**
<br>

🐙 &nbsp; `https://github.com/NataliaStekolnikova`
💼 &nbsp; `linkedin.com/in/natalia-stekolnikova`