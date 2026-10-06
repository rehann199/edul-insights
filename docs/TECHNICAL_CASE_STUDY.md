# Technical Case Study

## Problem Statement

Schools collect rich data on student performance and attendance but lack automated tools to surface at-risk students and prescribe interventions. Educators spend time manually analyzing spreadsheets instead of acting on insights. The goal was to build an end-to-end analytics platform that distills raw data into actionable intelligence.

## Solution Approach

Edul-Insights chains synthetic data generation, feature engineering and supervised learning into an interactive dashboard.

### Data Generation

Rather than rely on real student records (privacy risk), a Faker-based generator creates 500 realistic student profiles with 25 teachers, 148,000 performance records and 119,000 attendance logs. This allows demonstration without sensitive data.

### Feature Engineering

Per-student features are aggregated from the database: average performance, attendance rate, grade, gender, family income, motivation and study hours. These feed the RandomForest models.

### Machine Learning

The at-risk classifier learns from historical data using a **hybrid rule-ML approach:**

1. **Definition:** A student is at-risk if average performance < 60% OR attendance < 80% (business rule).
2. **Training:** RandomForest classifier trained on engineered features to predict this rule.
3. **Inference:** The model outputs a probability (0–1). The dashboard maps:
   - P ≥ 0.85 → High risk
   - P ≥ 0.6 → Medium risk
   - P < 0.6 → Watch

Because the model learns the same rule that defines the label, its probability is best read as a ranking signal rather than independent prediction.

### Dashboard Interactivity

Dash callbacks respond to the grade and subject filters and re-render the Overview charts without a page reload. The DataTables are sortable and filterable.

## Technical Highlights

- **Single-page app:** Sidebar-driven navigation between five views.
- **Callback-driven filtering:** Grade and subject filters update the Overview charts without reloading the page.
- **Plotly Visualizations:** Interactive charts with hover tooltips.
- **Pre-trained models:** Models are trained once, pickled and loaded at startup.
- **Synthetic Data:** The full lifecycle from generation to prediction runs locally.

## Choices and Trade-offs

**Rule-Based at-Risk Label**  
The at-risk definition is a hard rule (avg < 60% OR attendance < 80%). This keeps the flagged list transparent. The cost is that the model has little to add beyond the rule.

**RandomForest over Neural Networks**  
RandomForest is a simple, well-understood choice for tabular data of this size.

**Synthetic Data**  
Using Faker allows unencumbered demonstration and realistic complexity without privacy liability. Trade-off: model performance is validated on synthetic patterns, not real distributions. Real-world deployment would require retraining on actual school data.

**Version Pinning in requirements.txt**  
requirements.txt pins older versions; the demo ran on newer ones and loaded the pickles with a version warning.

## Results

The dashboard successfully displays:
- KPI cards (total students, average performance, attendance, at-risk count)
- Grade distribution histogram
- Grade bar chart sorted 9th to 12th
- 131 at-risk students with High / Medium / Watch tiers from model probabilities
- 500-student profile table with column filtering and sorting
- 138 rule-based intervention recommendations
- Monthly performance trend line
