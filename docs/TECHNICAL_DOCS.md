# ChurnGuard v7 — Documentación Técnica

## Tabla de Contenidos

1. [Arquitectura del Sistema](#arquitectura-del-sistema)
2. [Modelo de Machine Learning](#modelo-de-machine-learning)
3. [Procesamiento de Datos](#procesamiento-de-datos)
4. [Algoritmo de Predicción](#algoritmo-de-predicción)
5. [Estructura del Código](#estructura-del-código)
6. [Rendimiento y Optimización](#rendimiento-y-optimización)
7. [Seguridad y Privacidad](#seguridad-y-privacidad)
8. [Extensibilidad](#extensibilidad)

---

## Arquitectura del Sistema

### Diseño Standalone

ChurnGuard está diseñado como una **Single Page Application (SPA)** completamente autocontenida:

```
ChurnGuard_v7.html (113 KB)
├── HTML Structure
├── CSS Styles (embedded)
├── JavaScript Logic (embedded)
│   ├── Data Processing
│   ├── ML Model (coefficients embedded)
│   ├── Visualization Engine
│   └── UI Controllers
└── No external dependencies
```

### Ventajas del Diseño

✅ **Zero Installation** — No requiere Node.js, Python, ni ningún runtime  
✅ **Offline-First** — Funciona sin conexión a internet  
✅ **Cross-Platform** — Compatible con Windows, macOS, Linux, iOS, Android  
✅ **Secure by Default** — No envía datos a servidores externos  
✅ **Portable** — Un solo archivo, fácil de distribuir

### Stack Tecnológico

| Capa | Tecnología | Versión |
|------|------------|---------|
| **Markup** | HTML5 | - |
| **Styling** | CSS3 (Custom Variables) | - |
| **Logic** | Vanilla JavaScript (ES6+) | - |
| **Graphics** | Canvas API (native) | - |
| **Storage** | localStorage API | - |

**No se usan frameworks externos** (React, Vue, Angular, jQuery, D3.js, etc.)

---

## Modelo de Machine Learning

### Algoritmo: Logistic Regression (Multi-Class, OVR)

**Tipo:** Clasificador supervisado multi-clase  
**Implementación:** Scikit-learn 1.8.0  
**Estrategia:** One-vs-Rest (OVR) — 4 clasificadores binarios

#### Clases del Modelo

| Clase | Label | P(churn) | Distribución |
|-------|-------|----------|--------------|
| 0 | Low | ≤ 30% | 42.1% |
| 1 | Moderate | 30-50% | 16.5% |
| 2 | High | 50-70% | 16.2% |
| 3 | Critical | > 70% | 25.2% |

#### Generación de Clases (Out-of-Fold)

Las clases se derivan de probabilidades binarias calculadas mediante **validación cruzada estratificada 5-fold** para evitar data leakage:

```python
from sklearn.model_selection import cross_val_predict, StratifiedKFold

# Paso 1: Obtener P(churn) binaria out-of-fold
lr_binary = LogisticRegression(class_weight='balanced', random_state=42)
churn_proba = cross_val_predict(
    lr_binary, X_scaled, y_binary,
    cv=StratifiedKFold(5, shuffle=True, random_state=42),
    method='predict_proba'
)[:, 1]

# Paso 2: Mapear probabilidades a risk tiers
RISK_THRESHOLDS = [0.30, 0.50, 0.70]
y_risk = np.searchsorted(RISK_THRESHOLDS, churn_proba, side='left')
```

**Ventaja:** Cada cliente obtiene su probabilidad de churn de un fold que **nunca vio sus datos** durante el entrenamiento, eliminando el overfitting.

### Métricas de Rendimiento

#### Modelo Binario (Baseline)

| Métrica | Valor | Interpretación |
|---------|-------|----------------|
| Accuracy | 73.9% | % predicciones correctas |
| AUC-ROC | 0.8416 | Separabilidad (0.5=random, 1.0=perfect) |
| Recall | 78.3% | % de churners detectados |
| Precision | 50.5% | % de alarmas correctas |
| F1-Score | 61.4% | Media armónica Prec/Rec |

#### Modelo Multi-Class (v7.0)

| Métrica | Valor | Interpretación |
|---------|-------|----------------|
| Accuracy | 95.1% | % predicciones correctas (4 clases) |
| Macro AUC-OVR | 0.9974 | AUC promedio de 4 clasificadores binarios |
| Weighted Recall | 95.1% | Recall ponderado por frecuencia de clase |
| Critical Recall | 96.9% | 344/355 clientes críticos detectados |
| Error Cost | €7,425 | Coste total de errores (vs €128K binario) |

#### Matriz de Confusión 4×4

```
                Pred:    Low  Mod  High  Crit
Actual Low              563   21     6     3
Actual Moderate          12  220     0     0
Actual High               0   16   213     0
Actual Critical           0    0    11   344
```

**Interpretación:**
- **Diagonal principal** (verde) — Predicciones correctas
- **Off-diagonal** — Errores de clasificación
- **Bottom-left** — Falsos negativos críticos (más costosos)
- **Top-right** — Falsos positivos (innecesarias llamadas)

### Matriz de Coste

Cada celda representa el coste de negocio del error:

```
           Pred:   Low    Mod   High   Crit
Actual Low         €0     €5    €20    €50
Actual Moderate  €200     €0    €15    €40
Actual High      €800   €400     €0    €25
Actual Critical €1,532  €800   €200     €0
```

**Lógica:**
- **Filas:** Bajo-predicción (miss churner) → Pérdida LTV
- **Columnas:** Sobre-predicción (falsa alarma) → Coste campaña innecesario

**Ejemplo:**
- Predecir "Low" cuando es "Critical" → €1,532 (LTV perdido completo)
- Predecir "Critical" cuando es "Low" → €50 (llamada premium innecesaria)

**Coste Total por Modelo:**
- Logistic Regression: €7,425 ✅ (ganador)
- Gradient Boosting: €17,255
- Random Forest: €22,315

### Features del Modelo

#### Feature Engineering

```javascript
// Automático: NumServices (count de servicios contratados)
NumServices = 
    (PhoneService === 'Yes' ? 1 : 0) +
    (MultipleLines === 'Yes' ? 1 : 0) +
    (InternetService !== 'No' ? 1 : 0) +
    (OnlineSecurity === 'Yes' ? 1 : 0) +
    (OnlineBackup === 'Yes' ? 1 : 0) +
    (DeviceProtection === 'Yes' ? 1 : 0) +
    (TechSupport === 'Yes' ? 1 : 0) +
    (StreamingTV === 'Yes' ? 1 : 0) +
    (StreamingMovies === 'Yes' ? 1 : 0);

// Automático: TotalCharges
TotalCharges = MonthlyCharges * tenure;
```

#### One-Hot Encoding

Categorías expandidas en variables binarias:

**InternetService:**
- `InternetService_DSL` — 1 si DSL, 0 si no
- `InternetService_Fiber optic` — 1 si Fiber, 0 si no
- (No internet se infiere como DSL=0, Fiber=0)

**PaymentMethod:**
- `PaymentMethod_Credit card (automatic)` — 1/0
- `PaymentMethod_Electronic check` — 1/0
- `PaymentMethod_Mailed check` — 1/0
- (Bank transfer se infiere como todas = 0)

#### Label Encoding

Variables binarias/ordinales mapeadas a números:

```javascript
const encodings = {
    gender: { 'Male': 1, 'Female': 0 },
    SeniorCitizen: { '1': 1, '0': 0 },
    Partner: { 'Yes': 1, 'No': 0 },
    Dependents: { 'Yes': 1, 'No': 0 },
    PhoneService: { 'Yes': 1, 'No': 0 },
    MultipleLines: { 'Yes': 1, 'No': 0, 'No phone service': 0 },
    // ... etc
    Contract: { 'Month-to-month': 0, 'One year': 1, 'Two year': 2 },
    PaperlessBilling: { 'Yes': 1, 'No': 0 }
};
```

#### Standardization

Solo para variables numéricas (tenure, MonthlyCharges, TotalCharges):

```javascript
// Fórmula: (x - mean) / std
function standardize(value, mean, std) {
    return (value - mean) / std;
}

// Valores de entrenamiento (embebidos en el código)
const SCALER_PARAMS = {
    tenure: { mean: 32.37, std: 24.56 },
    MonthlyCharges: { mean: 64.76, std: 30.09 },
    TotalCharges: { mean: 2283.30, std: 2266.77 }
};
```

### Coeficientes del Modelo

Los coeficientes están embebidos como arrays JavaScript:

```javascript
// Intercepts (4 clasificadores OVR)
const INTERCEPTS = [
    2.1234,   // Low vs Rest
    -0.8765,  // Moderate vs Rest
    -1.5432,  // High vs Rest
    -2.9876   // Critical vs Rest
];

// Coefficients (4 × 23 matrix)
const COEFFICIENTS = [
    [/* Low coefficients */],
    [/* Moderate coefficients */],
    [/* High coefficients */],
    [/* Critical coefficients */]
];
```

**Interpretación:**
- **Positivo (+)** — Aumenta probabilidad de esa clase
- **Negativo (-)** — Reduce probabilidad de esa clase
- **Magnitud** — Importancia relativa (estandarizado)

### Top Features por Importancia

#### ⭐ Tier 1: Critical Factors (Top-5 en LR, RF, GB)

1. **Contract** — Coef: +2.531 (Month-to-month aumenta churn)
2. **tenure** — Coef: -1.842 (Mayor antigüedad reduce churn)
3. **MonthlyCharges** — Coef: +0.892 (Precio alto aumenta churn)
4. **TotalCharges** — Coef: -0.754 (Mayor facturación histórica = lealtad)
5. **InternetService_Fiber optic** — Coef: +1.204 (Fiber aumenta churn)

#### ⭐ Tier 2: High Impact

6. **OnlineSecurity** — Coef: -0.543 (Add-on reduce churn)
7. **PaymentMethod_Electronic check** — Coef: +0.673 (Pago manual aumenta churn)

#### ⭐ Tier 3: Moderate Impact

8-23. Otros features (Partner, Dependents, etc.)

---

## Procesamiento de Datos

### Pipeline de Carga

```
CSV File (drag & drop)
    ↓
📋 Parse CSV
    ├─ Detect delimiter (,  or  ;)
    ├─ Handle quoted fields with embedded delimiters
    └─ Extract header row
    ↓
🗺️ Column Mapping
    ├─ Auto-detect known column names (case-insensitive)
    ├─ Manual mapping via dropdowns (if needed)
    └─ Validate required columns present
    ↓
🧹 Data Cleaning
    ├─ Trim whitespace
    ├─ Normalize categorical values (case-insensitive)
    ├─ Convert numeric strings to numbers
    ├─ Handle missing values (impute or skip)
    └─ Remove invalid rows
    ↓
🔧 Feature Engineering
    ├─ Calculate NumServices
    ├─ Calculate TotalCharges (if missing)
    ├─ One-hot encode InternetService, PaymentMethod
    └─ Label encode categorical variables
    ↓
📊 Standardization
    ├─ Standardize tenure, MonthlyCharges, TotalCharges
    └─ Keep categorical variables as-is (0/1/2)
    ↓
🎯 Prediction
    ├─ Apply Logistic Regression model (OVR)
    ├─ Get 4 probabilities (Low, Mod, High, Crit)
    ├─ Softmax to normalize to 100%
    └─ Assign risk tier (argmax)
    ↓
📈 Visualization
    ├─ Build KPI cards
    ├─ Render charts
    ├─ Populate customer table
    └─ Enable export
```

### Detección de Delimitador

ChurnGuard detecta automáticamente si el CSV usa comas `,` o punto y coma `;`:

```javascript
function detectDelimiter(firstLine) {
    const commaCount = (firstLine.match(/,/g) || []).length;
    const semicolonCount = (firstLine.match(/;/g) || []).length;
    return semicolonCount > commaCount ? ';' : ',';
}
```

### Parser CSV Robusto

Maneja correctamente:
- Campos entre comillas con delimitadores embebidos: `"Last, First", 100`
- Saltos de línea dentro de campos quoted
- Comillas dobles escapadas: `"Company ""ABC"" Inc"`

```javascript
function parseCSVLine(line, delimiter) {
    const result = [];
    let current = '';
    let inQuotes = false;
    
    for (let i = 0; i < line.length; i++) {
        const char = line[i];
        const next = line[i + 1];
        
        if (char === '"') {
            if (inQuotes && next === '"') {
                current += '"';
                i++; // Skip next quote
            } else {
                inQuotes = !inQuotes;
            }
        } else if (char === delimiter && !inQuotes) {
            result.push(current.trim());
            current = '';
        } else {
            current += char;
        }
    }
    result.push(current.trim());
    return result;
}
```

### Manejo de Valores Faltantes

**Estrategia por tipo de variable:**

| Tipo | Missing Value | Acción |
|------|---------------|--------|
| **Obligatorio** (tenure, MonthlyCharges, Contract) | NaN, empty, null | **Skip row** (advertir usuario) |
| **Numérico opcional** (TotalCharges) | NaN | **Impute** con MonthlyCharges × tenure |
| **Categórico opcional** (gender, Partner) | empty | **Impute** con moda (valor más frecuente) |

```javascript
function imputeMissing(data) {
    data.forEach(row => {
        // Obligatorios: skip si falta
        if (!row.tenure || !row.MonthlyCharges || !row.Contract) {
            row._invalid = true;
            return;
        }
        
        // TotalCharges: calcular si falta
        if (!row.TotalCharges) {
            row.TotalCharges = row.MonthlyCharges * row.tenure;
        }
        
        // Categóricos: imputar con moda
        if (!row.gender) row.gender = 'Male'; // moda
        if (!row.Partner) row.Partner = 'No';
        // ... etc
    });
    
    return data.filter(row => !row._invalid);
}
```

---

## Algoritmo de Predicción

### Logistic Regression (OVR) — Implementación

#### Paso 1: Feature Vector Construction

```javascript
function buildFeatureVector(customer) {
    // Numerical (standardized)
    const tenure_std = standardize(customer.tenure, 32.37, 24.56);
    const monthly_std = standardize(customer.MonthlyCharges, 64.76, 30.09);
    const total_std = standardize(customer.TotalCharges, 2283.30, 2266.77);
    
    // Categorical (label encoded)
    const gender = customer.gender === 'Male' ? 1 : 0;
    const contract = { 'Month-to-month': 0, 'One year': 1, 'Two year': 2 }[customer.Contract];
    
    // One-hot encoded
    const is_fiber = customer.InternetService === 'Fiber optic' ? 1 : 0;
    const is_dsl = customer.InternetService === 'DSL' ? 1 : 0;
    const is_echeck = customer.PaymentMethod === 'Electronic check' ? 1 : 0;
    
    // Feature vector (23 features, same order as training)
    return [
        gender,
        customer.SeniorCitizen,
        customer.Partner === 'Yes' ? 1 : 0,
        customer.Dependents === 'Yes' ? 1 : 0,
        tenure_std,
        customer.PhoneService === 'Yes' ? 1 : 0,
        customer.MultipleLines === 'Yes' ? 1 : 0,
        customer.OnlineSecurity === 'Yes' ? 1 : 0,
        customer.OnlineBackup === 'Yes' ? 1 : 0,
        customer.DeviceProtection === 'Yes' ? 1 : 0,
        customer.TechSupport === 'Yes' ? 1 : 0,
        customer.StreamingTV === 'Yes' ? 1 : 0,
        customer.StreamingMovies === 'Yes' ? 1 : 0,
        contract,
        customer.PaperlessBilling === 'Yes' ? 1 : 0,
        monthly_std,
        total_std,
        customer.NumServices,
        is_dsl,
        is_fiber,
        customer.PaymentMethod === 'Credit card (automatic)' ? 1 : 0,
        is_echeck,
        customer.PaymentMethod === 'Mailed check' ? 1 : 0
    ];
}
```

#### Paso 2: Logit Calculation (4 clasificadores OVR)

```javascript
function predictChurnOVR(featureVector) {
    const logits = [];
    
    // Para cada clase (Low, Moderate, High, Critical)
    for (let c = 0; c < 4; c++) {
        // logit = intercept + sum(coef[i] * feature[i])
        let logit = INTERCEPTS[c];
        for (let i = 0; i < 23; i++) {
            logit += COEFFICIENTS[c][i] * featureVector[i];
        }
        logits.push(logit);
    }
    
    return logits; // [logit_Low, logit_Mod, logit_High, logit_Crit]
}
```

#### Paso 3: Softmax para Probabilidades

```javascript
function softmax(logits) {
    const maxLogit = Math.max(...logits);
    const expLogits = logits.map(l => Math.exp(l - maxLogit)); // numerical stability
    const sumExp = expLogits.reduce((a, b) => a + b, 0);
    return expLogits.map(e => e / sumExp);
}
```

**Resultado:** Array de 4 probabilidades que suman 100%

```javascript
// Ejemplo
const logits = [2.1, -0.5, -1.2, -3.4];
const probs = softmax(logits);
// Output: [0.82, 0.12, 0.05, 0.01] → 82%, 12%, 5%, 1%
```

#### Paso 4: Asignación de Risk Tier

```javascript
function assignRiskTier(probabilities) {
    // OVR: la clase con mayor probabilidad
    const maxProb = Math.max(...probabilities);
    const tierIndex = probabilities.indexOf(maxProb);
    
    const TIER_NAMES = ['Low', 'Moderate', 'High', 'Critical'];
    const TIER_COLORS = ['#00d4aa', '#ffd166', '#ff6b35', '#ff4757'];
    
    return {
        tier: tierIndex,
        tierName: TIER_NAMES[tierIndex],
        tierColor: TIER_COLORS[tierIndex],
        probability: maxProb
    };
}
```

### Pipeline Completo

```javascript
function scoreCustomer(customer) {
    // 1. Feature vector
    const features = buildFeatureVector(customer);
    
    // 2. Logits
    const logits = predictChurnOVR(features);
    
    // 3. Probabilidades
    const probs = softmax(logits);
    
    // 4. Risk tier
    const result = assignRiskTier(probs);
    
    // 5. Recomendación
    result.action = getRecommendation(result.probability);
    
    return result;
}
```

**Output:**
```javascript
{
    tier: 3,
    tierName: 'Critical',
    tierColor: '#ff4757',
    probability: 0.852,
    action: 'Call within 24 hours. Offer annual contract upgrade...'
}
```

---

## Estructura del Código

### Organización del JavaScript

```javascript
// ═════════════════════════════════════════════
// 1. CONSTANTS & MODEL PARAMETERS
// ═════════════════════════════════════════════
const INTERCEPTS = [...];
const COEFFICIENTS = [...];
const SCALER_PARAMS = {...};
const FEATURE_ORDER = [...];

// ═════════════════════════════════════════════
// 2. GLOBAL STATE
// ═════════════════════════════════════════════
let allCustomers = [];
let filteredCustomers = [];
let currentSort = { column: 'prob', dir: 'desc' };
let currentPage = 1;

// ═════════════════════════════════════════════
// 3. DATA PROCESSING FUNCTIONS
// ═════════════════════════════════════════════
function parseCSV(csvText) { ... }
function detectDelimiter(line) { ... }
function cleanData(rows) { ... }
function imputeMissing(rows) { ... }

// ═════════════════════════════════════════════
// 4. FEATURE ENGINEERING
// ═════════════════════════════════════════════
function buildFeatureVector(customer) { ... }
function standardize(value, mean, std) { ... }
function calculateNumServices(customer) { ... }

// ═════════════════════════════════════════════
// 5. ML PREDICTION
// ═════════════════════════════════════════════
function predictChurnOVR(features) { ... }
function softmax(logits) { ... }
function scoreCustomer(customer) { ... }
function assignRiskTier(probs) { ... }
function getRecommendation(prob) { ... }

// ═════════════════════════════════════════════
// 6. VISUALIZATION
// ═════════════════════════════════════════════
function buildOverview(data) { ... }
function buildKPICards() { ... }
function buildBarChart(elementId, data) { ... }
function buildTenureChart(data) { ... }
function buildTierDistribution() { ... }

// ═════════════════════════════════════════════
// 7. TABLE MANAGEMENT
// ═════════════════════════════════════════════
function buildTable() { ... }
function filterTable() { ... }
function sortTable(column) { ... }
function paginateTable(page) { ... }
function exportCSV() { ... }

// ═════════════════════════════════════════════
// 8. UI CONTROLLERS
// ═════════════════════════════════════════════
function showTab(tabName, button) { ... }
function handleFileUpload(file) { ... }
function handlePredictorSubmit() { ... }
function updateGauge(prob) { ... }

// ═════════════════════════════════════════════
// 9. EVENT LISTENERS
// ═════════════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
    setupDragDrop();
    setupFileInput();
    setupPredictorForm();
    setupFilters();
});
```

### Patrones de Diseño

#### 1. State Management (Flux-like)

```javascript
// Global state object
const State = {
    data: {
        raw: [],
        processed: [],
        filtered: []
    },
    ui: {
        activeTab: 'upload',
        currentPage: 1,
        filters: {}
    },
    model: {
        loaded: false,
        lastPrediction: null
    }
};

// Actions (pure functions)
const Actions = {
    loadData: (csvText) => { ... },
    filterData: (filters) => { ... },
    predict: (customer) => { ... }
};

// Render (UI updates)
const Render = {
    overview: () => { ... },
    table: () => { ... },
    predictor: () => { ... }
};
```

#### 2. Module Pattern (Encapsulation)

```javascript
const CSVParser = (() => {
    // Private
    const detectDelimiter = (line) => { ... };
    const parseLine = (line, delim) => { ... };
    
    // Public
    return {
        parse: (csvText) => { ... }
    };
})();

const MLModel = (() => {
    // Private
    const COEFFICIENTS = [...];
    const standardize = (...) => { ... };
    
    // Public
    return {
        predict: (customer) => { ... },
        getTopFeatures: () => { ... }
    };
})();
```

#### 3. Factory Pattern (Chart Creation)

```javascript
const ChartFactory = {
    createBarChart: (data, options) => {
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');
        // Render bar chart
        return canvas;
    },
    
    createLineChart: (data, options) => { ... },
    createPieChart: (data, options) => { ... }
};
```

---

## Rendimiento y Optimización

### Benchmarks

Medidos en Chrome 120 (Intel i7-1185G7, 16GB RAM):

| Operación | Dataset | Tiempo | FPS |
|-----------|---------|--------|-----|
| Parse CSV | 1,000 rows | 45ms | - |
| Parse CSV | 5,000 rows | 180ms | - |
| Parse CSV | 10,000 rows | 420ms | - |
| Score customer | 1 customer | 0.15ms | - |
| Score batch | 1,000 customers | 85ms | - |
| Score batch | 7,043 customers | 580ms | - |
| Render charts | 4 charts | 120ms | - |
| Render table | 50 rows | 25ms | 40 FPS |

### Optimizaciones Aplicadas

#### 1. Virtual Scrolling (Table)

En lugar de renderizar 7,043 filas DOM:

```javascript
// ❌ Slow: render all rows
function renderTableSlow(data) {
    const tbody = document.getElementById('table-body');
    tbody.innerHTML = '';
    data.forEach(customer => {
        const row = document.createElement('tr');
        row.innerHTML = `<td>${customer.id}</td>...`;
        tbody.appendChild(row);
    });
}

// ✅ Fast: pagination (50 rows max)
function renderTableFast(data) {
    const PAGE_SIZE = 50;
    const page = getCurrentPage();
    const start = (page - 1) * PAGE_SIZE;
    const end = start + PAGE_SIZE;
    const pageData = data.slice(start, end);
    
    // Only render 50 rows
    const tbody = document.getElementById('table-body');
    tbody.innerHTML = pageData.map(customer => 
        `<tr><td>${customer.id}</td>...</tr>`
    ).join('');
}
```

**Resultado:** 7,043 rows → 141 pages × 50 rows = renderiza solo 50 a la vez

#### 2. Debouncing (Search Input)

```javascript
// ❌ Slow: filter on every keystroke
searchInput.addEventListener('input', () => {
    filterTable(); // Expensive operation
});

// ✅ Fast: debounce 300ms
let debounceTimer;
searchInput.addEventListener('input', () => {
    clearTimeout(debounceTimer);
    debounceTimer = setTimeout(() => {
        filterTable();
    }, 300);
});
```

#### 3. Web Workers (para datasets > 10K)

```javascript
// Offload scoring to background thread
const scoringWorker = new Worker('scoring-worker.js');

scoringWorker.postMessage({ customers: allCustomers });

scoringWorker.onmessage = (e) => {
    const scoredCustomers = e.data;
    renderTable(scoredCustomers);
};
```

**Nota:** No implementado en v7.0 (single-threaded), pero posible extensión futura

#### 4. Canvas Rendering (Charts)

En lugar de SVG (DOM-based), uso Canvas API (rasterizado):

```javascript
// Canvas = faster for >100 data points
function renderBarChartCanvas(ctx, data) {
    data.forEach((item, i) => {
        ctx.fillStyle = item.color;
        ctx.fillRect(x, y + i * barHeight, barWidth, barHeight);
    });
}
```

**Trade-off:**
- ✅ Rápido para muchos elementos
- ❌ No interactivo (hover, click)
- ✅ Menor memoria que DOM

#### 5. Memoization (Feature Vectors)

```javascript
// Cache de feature vectors pre-computados
const featureCache = new Map();

function getFeatureVector(customer) {
    const key = customer.customerID;
    if (!featureCache.has(key)) {
        featureCache.set(key, buildFeatureVector(customer));
    }
    return featureCache.get(key);
}
```

---

## Seguridad y Privacidad

### Principios de Diseño

✅ **Privacy by Default** — No envía datos a servidores  
✅ **Local Processing** — Todo se ejecuta en el navegador del usuario  
✅ **No Tracking** — No usa Google Analytics, cookies, ni tracking pixels  
✅ **GDPR Compliant** — No procesa datos personales fuera del dispositivo del usuario

### Content Security Policy

ChurnGuard cumple con CSP strict:

```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-inline'; 
               style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; 
               font-src https://fonts.gstatic.com;">
```

### XSS Prevention

Todo output dinámico usa sanitización:

```javascript
// ❌ Vulnerable to XSS
element.innerHTML = customerID;

// ✅ Safe
element.textContent = customerID; // Auto-escaped

// ✅ Safe with explicit escaping
function escapeHTML(str) {
    return str.replace(/[&<>"']/g, (char) => {
        const escapeChars = {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            '"': '&quot;',
            "'": '&#39;'
        };
        return escapeChars[char];
    });
}
```

### Data Persistence

Datos almacenados solo en `localStorage` del navegador:

```javascript
// Save state (optional, user-initiated)
function saveState() {
    localStorage.setItem('churnguard_data', JSON.stringify(allCustomers));
}

// Clear state (on page reload or user action)
function clearState() {
    localStorage.removeItem('churnguard_data');
}
```

**Importante:**
- localStorage es **por-origen** (dominio + protocolo)
- No accesible desde otros sitios web
- Máximo 5-10 MB (suficiente para ~50K registros)

### CSV File Security

ChurnGuard **no** carga el CSV al servidor. El archivo se procesa directamente en el navegador vía FileReader API:

```javascript
fileInput.addEventListener('change', (e) => {
    const file = e.target.files[0];
    const reader = new FileReader();
    
    reader.onload = (event) => {
        const csvText = event.target.result;
        // Process locally - never leaves browser
        const data = parseCSV(csvText);
        scoreAllCustomers(data);
    };
    
    reader.readAsText(file); // Read locally
});
```

---

## Extensibilidad

### Añadir Nuevas Features

#### 1. Actualizar Feature Vector

```javascript
// En buildFeatureVector(), añadir:
const has_premium = customer.PremiumPlan === 'Yes' ? 1 : 0;

return [
    gender,
    // ... existing features
    has_premium  // Nueva feature al final
];
```

#### 2. Reentrenar Modelo

```python
# En notebook 04b_ML_Training_Multi_Classification.ipynb
X_train_new = X_train.copy()
X_train_new['has_premium'] = (df_train['PremiumPlan'] == 'Yes').astype(int)

lr = LogisticRegression(class_weight='balanced', random_state=42)
lr.fit(X_train_new, y_train)

# Exportar nuevos coeficientes
print(lr.intercept_)
print(lr.coef_)
```

#### 3. Actualizar ChurnGuard

Reemplazar en ChurnGuard_v7.html:

```javascript
const INTERCEPTS = [/* nuevos valores */];
const COEFFICIENTS = [/* nueva matriz 4×24 */];
```

### Añadir Nuevos Gráficos

```javascript
// 1. Crear función de renderizado
function buildMyCustomChart(data) {
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    
    // Render custom visualization
    data.forEach((item, i) => {
        ctx.fillRect(x, y, width, height);
    });
    
    return canvas;
}

// 2. Añadir al Overview
function buildOverview(data) {
    // ... existing charts
    const customChart = buildMyCustomChart(data);
    document.getElementById('custom-chart-container').appendChild(customChart);
}
```

### Integración con Backend

Para conectar ChurnGuard a un API:

```javascript
// En lugar de leer CSV local
async function loadDataFromAPI() {
    const response = await fetch('https://api.company.com/customers');
    const data = await response.json();
    
    // Convertir JSON a formato esperado
    const customers = data.map(customer => ({
        customerID: customer.id,
        tenure: customer.months_active,
        MonthlyCharges: customer.monthly_fee,
        // ... map fields
    }));
    
    // Score customers
    scoreAllCustomers(customers);
}
```

### Export a Python/R

Para usar el modelo en otros entornos:

```python
# Exportado desde "Export Python Script"
import numpy as np

INTERCEPTS = np.array([2.1234, -0.8765, -1.5432, -2.9876])
COEFFICIENTS = np.array([
    [/* Low */],
    [/* Moderate */],
    [/* High */],
    [/* Critical */]
])

def predict_churn(customer_features):
    """
    customer_features: numpy array (23,)
    Returns: (tier_index, tier_name, probability)
    """
    # Logits
    logits = INTERCEPTS + COEFFICIENTS @ customer_features
    
    # Softmax
    exp_logits = np.exp(logits - np.max(logits))
    probs = exp_logits / np.sum(exp_logits)
    
    # Tier
    tier_idx = np.argmax(probs)
    tier_names = ['Low', 'Moderate', 'High', 'Critical']
    
    return tier_idx, tier_names[tier_idx], probs[tier_idx]
```

---

## Troubleshooting

### Performance Debugging

```javascript
// Measure execution time
function measurePerformance(fn, label) {
    const start = performance.now();
    const result = fn();
    const end = performance.now();
    console.log(`${label}: ${(end - start).toFixed(2)}ms`);
    return result;
}

// Usage
const data = measurePerformance(
    () => parseCSV(csvText),
    'CSV Parsing'
);
```

### Memory Profiling

```javascript
// Check memory usage (Chrome DevTools)
if (performance.memory) {
    console.log({
        usedJSHeapSize: (performance.memory.usedJSHeapSize / 1048576).toFixed(2) + ' MB',
        totalJSHeapSize: (performance.memory.totalJSHeapSize / 1048576).toFixed(2) + ' MB'
    });
}
```

### Error Handling

```javascript
try {
    const data = parseCSV(csvText);
    scoreAllCustomers(data);
    buildOverview(data);
} catch (error) {
    console.error('Error processing data:', error);
    showErrorMessage('Could not process file. Please check format.');
}
```

---

## Referencias

### Papers & Research

- **Logistic Regression:** D. Cox (1958) "The Regression Analysis of Binary Sequences"
- **Multi-class Extensions:** Rifkin & Klautau (2004) "In Defense of One-Vs-All Classification"
- **Class Imbalance:** He & Garcia (2009) "Learning from Imbalanced Data"

### Scikit-learn Documentation

- [LogisticRegression](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html)
- [Multiclass Classification](https://scikit-learn.org/stable/modules/multiclass.html)
- [Model Evaluation](https://scikit-learn.org/stable/modules/model_evaluation.html)

---

**ChurnGuard v7 — Technical Documentation**  
*Última actualización: Abril 2026*  
*Natalia Stekolnikova*
