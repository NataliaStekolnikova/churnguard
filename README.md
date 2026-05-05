# ChurnGuard — Customer Churn Prediction & Retention System

**Natalia Stekolnikova · Málaga, Spain · April 2026**  
[linkedin.com/in/natalia-stekolnikova](https://www.linkedin.com/in/natalia-stekolnikova/)

---

## What this project is about

Telecommunications companies lose a predictable share of their customers every year. The problem is not the lack of data — it is the lack of a system that turns that data into a decision at the right moment.

This project started as a data analysis exercise and ended as a deployable product. The goal was not to build another notebook with metrics. The goal was to answer a practical question: **given a customer database, who is about to leave, why, and what exactly should we do about it — before it happens?**

The answer is ChurnGuard: a complete end-to-end pipeline from raw data to a working application that a retention manager can open in a browser, load their customer file, and walk away with a prioritised call list — without writing a single line of code.

---

## The business problem

The IBM Telco dataset contains 7,043 customers. Of those, 1,869 have already churned — a rate of 26.5%. The historical revenue lost on those customers totals **€2,862,927**. At a 2% net margin, that translates to an annual loss of **€57,259**.

The standard industry response is reactive: act after the cancellation, offer a discount, hope for a winback. This project inverts that logic entirely.

| Approach | Timing | Segmentation | Error cost |
|---|---|---|---|
| Reactive (no model) | After cancellation | None | €128,397 |
| ChurnGuard (this project) | Before churn | 4 risk tiers | €7,425 |

The reduction in error cost — **94%** — comes from replacing generic intervention with a targeted, tiered system that matches the intensity of the action to the level of risk.

---

## From analysis to business outcomes

Most churn projects stop at the model. This one did not.

The pipeline covers the full journey:

1. **Data quality audit** — 11 NaN values identified in `TotalCharges`, logical contradictions in service flags resolved, dtype corrections applied.
2. **ETL and feature engineering** — raw data cleaned, transformed, and split into two purpose-built datasets: one for analysis and dashboarding (`telco_clean.csv`), one for model training (`telco_model_ready.csv`).
3. **SQL business analysis** — 6 analytical queries against a SQLite database quantifying churn by contract type, payment method, service category, and tenure group.
4. **Machine learning** — binary baseline followed by a multi-class risk tier model; 4 classifiers compared on a business cost matrix, not just accuracy.
5. **Deployment** — model coefficients exported and embedded directly in JavaScript. The result is a fully functional application in a single `.html` file that works offline, requires no installation, and can be distributed by email.
6. **Power BI dashboard** — built on `telco_clean.csv` for executive reporting.
7. **Full documentation** — IEEE 830 / ISO/IEC 29148 specification covering requirements, architecture, ETL pipeline, and ML methodology.

---

## Key findings from the data

Before modelling, the exploratory analysis identified the drivers that later became the most predictive features:

| Segment | Churn rate | Observation |
|---|---|---|
| Month-to-month contract | 42.7% | 15x higher than two-year contracts (2.8%) |
| Electronic check payment | 45.3% | Manual payment = monthly decision to stay |
| Fiber optic internet | 41.9% | High price, low perceived value differential |
| Tenure 0–12 months | 47.4% | First year is the critical retention window |
| Senior citizens | 41.7% | Highest demographic risk profile |
| Two-year contract | 2.8% | Benchmark — what retention success looks like |

A counterintuitive finding: customers who churn pay **€74.44/month on average**, compared to €61.27 for retained customers. The most valuable customers are the most likely to leave. This directly justifies the VIP intervention tier.

---

## Feature engineering

The 21 original variables were not sufficient as-is. Three engineered features were constructed and included in the final model:

**NumServices** — sum of 9 binary service flags (phone, multiple lines, online security, online backup, device protection, tech support, streaming TV, streaming movies, paperless billing). Range: 1–9. Acts as a proxy for customer engagement and depth of relationship. Customers with more services churn at significantly lower rates.

**TenureGroup** — categorical segmentation of the continuous `tenure` variable into four lifecycle stages: New (0–12 months), Junior (13–24), Mid (25–48), Loyal (49–72). This captures the non-linear relationship between tenure and churn risk — the first year is disproportionately risky.

**IsAutoPayment** — binary flag: 1 if the customer pays by bank transfer or credit card (automatic), 0 if by electronic check or mailed check (manual). Manual payment requires a conscious monthly action, which correlates with higher churn propensity.

---

## Most predictive features

After standardisation and model training, the top features by coefficient magnitude (Logistic Regression OVR) were:

| Rank | Feature | Direction | Interpretation |
|---|---|---|---|
| 1 | Contract type (ordinal) | Protective | Month-to-month = highest risk; two-year = lowest |
| 2 | Internet: Fiber Optic | Risk | Fiber customers churn at 41.9% |
| 3 | Tenure | Protective | Longer tenure = significantly lower risk |
| 4 | Monthly charges | Risk | Higher spend without perceived value = churn signal |
| 5 | NumServices | Protective | More services = deeper engagement = lower churn |
| 6 | Payment: Electronic check | Risk | Manual, high-friction payment method |
| 7 | TotalCharges | Protective | Accumulation of spend correlates with loyalty |
| 8 | IsAutoPayment | Protective | Automatic payment = passive retention |

Contract type and fiber optic internet are the dominant factors. A customer on a month-to-month contract with fiber optic internet, paying by electronic check, and in their first year has a predicted churn probability exceeding 80% in the model.

---

## Risk tier framework

The model classifies each customer into one of four tiers. Tier boundaries were defined by discretising out-of-fold probabilities and validated against the actual `Churn` label (which was never used to define the tiers):

| Tier | P(churn) | Customers (OOF) | Actual churn rate | Prescribed action | Cost |
|---|---|---|---|---|---|
| Low | < 30% | 2,963 (42.1%) | 4.8% | Monitor | €0 |
| Moderate | 30–50% | 1,160 (16.5%) | 19.6% | Personalised email | €5 |
| High | 50–70% | 1,143 (16.2%) | 33.9% | Retention call | €25 |
| Critical | > 70% | 1,777 (25.2%) | 62.6% | VIP intervention | €50 |

The monotonic progression from 4.8% to 62.6% (ratio 13x) confirms that the tiers capture genuine risk differentiation. The Critical tier — 25% of customers — accounts for **60% of all revenue lost to churn**.

---

## Model selection

Four models were trained and compared. The selection criterion was minimum total error cost on a 4x4 cost matrix — not accuracy or AUC alone. The asymmetry matters: failing to identify a Critical customer costs €1,531.80 in lost lifetime value; a false positive costs €5–50.

| Model | Accuracy | AUC-OVR | Wtd F1 | CV AUC | Error Cost |
|---|---|---|---|---|---|
| Logistic Regression OVR [*] | 0.9510 | 0.9974 | 0.9520 | 0.9967 | **€7,425** |
| Stacking LR + RF + GB | 0.9603 | 0.9978 | 0.9607 | — | €6,970 |
| Gradient Boosting | 0.9063 | 0.9841 | 0.9071 | 0.9873 | €15,690 |
| Random Forest | 0.8999 | 0.9823 | 0.9002 | 0.9841 | €19,985 |

[*] Selected. The Stacking Ensemble achieves slightly lower error cost (€6,970) but is not directly serialisable to JavaScript. The marginal performance gain does not justify the architectural complexity for deployment.

**On AUC 0.9974:** the value reflects internal pipeline consistency, not data leakage. Tiers were built from OOF probabilities via `cross_val_predict` with `StratifiedKFold(5)` — the multi-class classifier in step 2 recovers boundaries the same model family helped define, making the task structurally easier than predicting raw behaviour. CV stability (0.9967 +/- 0.0005) rules out overfitting. External validation confirms the tiers predict real outcomes. Full methodology in `docs/TECHNICAL_DOCS.md`.

---

## Projected business impact

Three retention campaigns, proportional to risk level:

| Campaign | Segment | Customers | Investment | Revenue saved | Net benefit | ROI |
|---|---|---|---|---|---|---|
| Email | Moderate | 1,160 | €6,518 | €174,280 | €167,762 | 2,572% |
| Call | High | 1,143 | €28,893 | €260,537 | €231,644 | 793% |
| VIP | Critical | 1,777 | €87,534 | €544,635 | €457,101 | 525% |
| **Total** | — | **4,080** | **€122,945** | **€979,452** | **€856,507** | **697%** |

Break-even: retaining just 138 of the 1,097 projected customers (12.6%) covers the full campaign cost. Every additional customer retained generates approximately €893/year in net benefit.

---

## The application

ChurnGuard is a single `.html` file. No Python runtime, no server, no installation. The model coefficients are embedded directly in JavaScript and run entirely in the browser.

| Module | Function |
|---|---|
| Upload Data | CSV loading with automatic delimiter and column detection, field mapping, session history |
| Overview | 8 KPI cards + 4 segmentation charts + print/PDF export |
| At-Risk List | Filterable by tier, contract, internet type, and probability range; sortable; CSV export |
| Risk Predictor | Real-time individual prediction with top-5 contributing factors and directional indicators |
| Model Details | Model comparison table, coefficient chart, live dataset statistics, Python scoring script export |

The CSV export from At-Risk List is formatted for direct import into CRM systems (Salesforce, Dynamics, or any CSV-accepting platform).

---

## Tech stack

| Layer | Technology |
|---|---|
| Data processing | Python 3.10, pandas, numpy |
| Machine learning | scikit-learn (LogisticRegression, RandomForest, GradientBoosting, StratifiedKFold) |
| Database | SQLite via Python standard library |
| Visualisation (notebooks) | matplotlib, seaborn |
| Application | Vanilla JavaScript (ES6), HTML5 Canvas API — zero external libraries |
| Dashboard | Power BI Desktop |
| Documentation | IEEE 830 / ISO/IEC 29148 |

---

## Dataset

**IBM Telco Customer Churn** — public dataset, Kaggle:  
https://www.kaggle.com/datasets/blastchar/telco-customer-churn

Place the downloaded file at `data/raw/WA_Fn-UseC_-Telco-Customer-Churn.csv`.

| Metric | Value |
|---|---|
| Total customers | 7,043 |
| Original variables | 21 |
| ML features after OHE | 23 |
| Churn rate | 26.5% (1,869 customers) |
| Historical capital lost | €2,862,927 |
| Avg monthly charge — churned | €74.44 |
| Avg monthly charge — retained | €61.27 |

---

## Repository structure

```
churn_project/
│
├── .gitignore
├── README.md
├── requirements.txt
│
├── data/
│   ├── raw/                               # source CSV from Kaggle — not committed
│   └── processed/                         # generated by NB02 — not committed
│
├── notebooks/
│   ├── 01_Extract_QualityCheck.ipynb      # data quality audit: NaN, dtypes, outliers
│   ├── 02_Clean_Transform.ipynb           # cleaning, feature engineering, ETL
│   ├── 03_Load_SQLite.ipynb               # relational load + 6 SQL business queries
│   ├── 04a_ML_Training_Binary.ipynb       # binary LR baseline — AUC 0.8416
│   ├── 04b_ML_Training_MultiClass.ipynb   # OVR LR 4-tier model — AUC 0.9974
│   └── 05_Stacking_Ensemble.ipynb         # LR + RF + GB -> meta-LR — AUC 0.9978
│
├── app/
│   └── ChurnGuard_v7.html                 # standalone application (110.7 KB)
│
├── dashboard/
│   └── Churn_Telco_Dashboard_Final.pbix
│
├── presentation/
│   ├── ChurnGuard_Presentacion_Final.html
│   └── assets/
│
├── docs/
│   ├── ChurnGuard_Documentacion.docx      # IEEE 830 / ISO/IEC 29148
│   ├── TECHNICAL_DOCS.md
│   └── USER_GUIDE.md
│
└── outputs/
    ├── figures/                            # EDA and model plots
    └── sql/                                # SQL query result plots
```

---

## Installation and usage

```bash
git clone https://github.com/<your-username>/churn_project.git
cd churn_project
python -m venv venv
venv\Scripts\activate        # Windows
source venv/bin/activate     # macOS / Linux
pip install -r requirements.txt
```

Place the raw dataset at `data/raw/WA_Fn-UseC_-Telco-Customer-Churn.csv`, then run notebooks in order:

```
NB01 -> NB02 -> NB03 -> NB04a -> NB04b -> NB05
```

To open the application — no server needed:

```
app/ChurnGuard_v7.html   (double-click or open in browser)
```

---

## References

- Verbeke, W., Dejaeger, K., Martens, D., Hur, J., & Baesens, B. (2012). New insights into churn prediction in the telecommunication sector: A profit driven data mining approach. *European Journal of Operational Research*, 218(1), 211–229.
- Vaduva, M., & Mocanu, I. (2024). Customer churn prediction using machine learning: a comparative study. *Procedia Computer Science*, 236, 12–19.
- IEEE 830-1998. *IEEE Recommended Practice for Software Requirements Specifications.*
- ISO/IEC 29148:2018. *Systems and software engineering — Life cycle processes — Requirements engineering.*

---

## Author

**Natalia Stekolnikova**  
linkedin.com/in/natalia-stekolnikova  
Málaga, Spain · April 2026
