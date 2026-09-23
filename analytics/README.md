# Analytics Module

This module follows the required Titanic workflow. The raw dataset was loaded once with `sns.load_dataset('titanic')`, saved to `titanic.csv`, and all downstream work uses that CSV instead of reloading the dataset.

## Part A — profiling, cleaning, and EDA

The missing-value percentages were checked before cleaning and the decisions followed the required rule: under 5% means drop rows, from 5%-30% means impute, and above 30% means drop or encode as a category. The notebook records those exact percentages and the chosen strategy for each affected column.

The IQR rule was applied to both age and fare, and the outlier counts are reported in the notebook. The fare distribution is right-skewed because mean > median > mode.

The bivariate and multivariate analyses show the survival story clearly: women and first-class passengers had much higher survival rates, and the correlation heatmap highlights the strongest links among class, fare, and family-size variables.

A z-score standardization check for age and fare confirms the transformed columns have mean near 0 and standard deviation near 1.

## Part B — predictive modeling

The train/test split is stratified and occurs before preprocessing, which preserves the class balance. All preprocessing is fit only on the training split and applied to the test split in transform-only mode. The notebook trains Logistic Regression, Decision Tree, and Random Forest on the same split and compares accuracy, precision, recall, F1 score, and ROC-AUC.

The imbalance handling comparison includes baseline, class-weighted, and SMOTE-only-on-training variants. The Random Forest tuning step uses `GridSearchCV` with `RandomForestClassifier(oob_score=True, ...)`, and the OOB score is reported. The regression side-task predicts fare via linear regression and reports MAE, RMSE, R², and adjusted R², along with a residual plot.

The final saved artifact is `best_titanic_pipeline.joblib`, a complete fitted pipeline containing preprocessing and classifier together, which can be reloaded with `joblib.load(...)` and used on raw new data.
