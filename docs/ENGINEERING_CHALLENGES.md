# Engineering Challenges

## Challenge 1: Empty Histogram in Performance Distribution

**Problem**  
The Performance Distribution chart rendered an empty histogram, failing to display the grade distribution across the student population.

**Root Cause**  
Plotly Express was called with `bins=20` (Python keyword), but the correct parameter name is `nbins` in `px.histogram()`. The invalid keyword was silently ignored, leaving no binning specification and producing no output.

**Solution**  
Changed `px.histogram(bins=20)` to `px.histogram(nbins=20)`. The chart now correctly renders 20 bins spanning the grade range.

**Learning**  
Plotly's histogram API differs from NumPy conventions. API documentation must be consulted per library, and integration tests should validate chart rendering, not just data loading.

## Challenge 2: Missing Risk Level Classifications

**Problem**  
The At-Risk table displayed 131 flagged students but the "Risk Level" column was empty. Without severity tiers (High/Medium/Watch), educators could not prioritize interventions.

**Root Cause**  
The original feature set lacked the risk-level field. The at-risk flag came from a filtering rule, but no risk probability was computed to support multi-tier classification.

**Solution**  
Integrated RandomForest risk predictions into the dashboard callback:
1. After filtering at-risk students, call `ml_system.predict_risk()` to retrieve prediction probabilities for each.
2. Classify: High (P ≥ 0.85), Medium (0.6–0.85), Watch (< 0.6).
3. Populate the "Risk Level" column with these tiers.

The ML models were already trained; only the dashboard had to invoke and display them. Note that the tier reflects a model that learned the rule, so it is a ranking signal, not independent validation.

**Learning**  
Feature completeness must span data pipeline and UI. A feature is incomplete if the data exists but is not surfaced in the user interface.

## Challenge 3: Grade Bar Chart Sorting

**Problem**  
The bar chart of current grades by academic level displayed grades in arbitrary (likely alphabetical) order rather than logical progression (9th, 10th, 11th, 12th).

**Root Cause**  
Plotly's category_orders parameter was not specified, so Plotly used the order of first appearance in the data or default alphabetical sort.

**Solution**  
Added explicit `category_orders={"grade": ["9th", "10th", "11th", "12th"]}` to the bar chart configuration. The chart now displays grades in academic progression.

**Learning**  
Categorical axes require explicit ordering in interactive charting libraries. Defaults are often alphabetical and may not match domain intent. Always define sort order for ranked dimensions.

## Challenge 4: Synthetic Data Versioning and Compatibility

**Problem**  
The pickled RandomForest models were saved with an older scikit-learn version than the one used to run the demo, causing deserialization warnings.

**Root Cause**  
Pickles are tied to the scikit-learn version. requirements.txt pins older versions, but the demo environment used newer packages.

**Solution**  
Ran the demo in an isolated virtual environment and accepted the version warning on load. Retraining with the target scikit-learn version, or a format-stable export, would be the cleaner fix.

**Learning**  
Pickle-based model persistence is environment-fragile across scikit-learn versions.

## Challenge 5: Incomplete Feature Scope

**Problem**  
The README claimed PDF/Excel export and advanced analytics (cohort comparison, predictive trending) that were not implemented in the dashboard UI.

**Root Cause**  
Design specifications outpaced implementation. The ml_models.py module contained advanced feature engineering but was unused by dashboard.py.

**Solution**  
Documented scope: the dashboard implements the core views (KPIs, at-risk list, trends, profiles, interventions). Export and advanced analytics are not available in the UI; ml_models.py is an unused module.

**Learning**  
MVP scope should be explicit and documented. Unimplemented designs should be clearly marked as "future" to set expectations.

## Challenge 6: Sidebar Navigation State

**Problem**  
The sidebar's active-section highlight did not update as users navigated between pages, confusing the information architecture.

**Root Cause**  
No callback updated the active state of the sidebar items.

**Solution**  
A small callback now tracks which sidebar item was clicked last and sets the matching item active. URL-based routing (dcc.Location) was not added.

**Learning**  
Navigation state must be explicitly managed; URL-driven routing would be more robust than click tracking.
