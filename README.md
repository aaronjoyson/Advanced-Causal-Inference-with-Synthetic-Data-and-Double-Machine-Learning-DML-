Advanced Causal Inference with Synthetic Data and Double Machine Learning (DML)
Project Overview
This project implements and evaluates the Double Machine Learning (DML) framework for estimating heterogeneous treatment effects (HTE) in observational studies. We programmatically generated a complex synthetic dataset with known ground truth to rigorously test the DML methodology. The analysis compares the performance of DML using various machine learning models (Random Forest, Linear Models, and XGBoost) for nuisance estimation, providing a critical assessment of bias, variance, and the ability to capture heterogeneity.

Table of Contents
Problem Statement
Synthetic Data Generation
Double Machine Learning (DML) Implementation
Results and Critical Assessment
Conclusion and Next Steps
1. Problem Statement
The goal of this project is to implement and evaluate the DML framework to estimate Conditional Average Treatment Effects (CATE) and Average Treatment Effects (ATE). We aim to demonstrate how DML can address confounding in complex, non-linear data environments and critically assess its performance against known ground truth from a synthetic dataset, especially when varying the complexity of nuisance models.

2. Synthetic Data Generation
A synthetic dataset (N=5000) was programmatically generated to simulate a causal inference scenario. The dataset includes:

Confounding Covariates (X): 20 continuous (Xc*), 5 binary (Xb*), and 5 categorical (Xcat*) variables, combined into a single feature matrix X.
True CATE (true_tau): This crucial column represents the true individual treatment effect for each observation, generated with a non-linear and heterogeneous relationship: true_tau = 1 + 0.5 * Xc1 + sin(Xc2) + 0.3 * Xb3 + 0.7 * (Xcat1 == 2) The true average ATE (mean of true_tau) was approximately 1.399.
Propensity Scores: Calculated from a linear combination of some X variables, transformed using a sigmoid function, and clipped (0.01 to 0.99).
Treatment Variable (T): A binary variable (0 or 1) assigned based on the calculated propensity scores.
Base Outcome (Y0): The outcome if T=0, generated with non-linear dependencies on X variables and added noise.
Observed Outcome (Y): Generated as Y = Y0 + T * true_tau + noise, incorporating the true heterogeneous treatment effect.
The true_tau column serves as the ground truth for evaluating how well the DML models capture individual treatment effects.

3. Double Machine Learning (DML) Implementation
All DML implementations followed a consistent 5-fold cross-fitting procedure. For each fold, nuisance models were trained on training data to predict residuals for the outcome (Y) and treatment (T) on the test data. A final-stage LinearRegression model then regressed outcome residuals on treatment residuals and their interactions with all confounders (X) to estimate CATEs. The ATE was calculated as the mean of the individual CATE estimates.

Nuisance Model Choices:
Random Forest Nuisance Models (Baseline):

RandomForestRegressor for E[Y|X] (outcome model)
RandomForestClassifier for E[T|X] (treatment model)
Linear Nuisance Models (Sensitivity Analysis):

LinearRegression for E[Y|X]
LogisticRegression for E[T|X]
Gradient Boosting (XGBoost) Nuisance Models (Sensitivity Analysis):

XGBoostRegressor for E[Y|X]
XGBoostClassifier for E[T|X]
4. Results and Critical Assessment
Aggregate ATE Estimates:
True Average ATE: ~1.399
Random Forest DML ATE: 0.0122
Linear DML ATE: 0.0005
XGBoost DML ATE: 0.0210
Observation: All DML implementations significantly underestimated the true average ATE, consistently yielding values close to zero.

CATE Comparison Metrics (vs. true_tau):
Model Type	MAE (Estimated CATE vs. True CATE)	Pearson Correlation (Estimated CATE vs. True CATE)
Random Forest	1.3375	0.0327
Linear Models	1.3520	0.0116
XGBoost	1.3338	0.0166
Observation: The Mean Absolute Error (MAE) values were relatively similar across all model types. Crucially, the Pearson Correlation coefficients were extremely low, indicating a severe inability to capture the individual-level heterogeneity of the true_tau function.

Insights on Bias, Variance, and Heterogeneity
Bias: The consistent underestimation of ATE and the poor correlation with true CATEs suggest a significant bias introduced by the final-stage LinearRegression model. Since the true CATE function was non-linear, fitting a linear model to residuals_y on residuals_t and residuals_t * X fundamentally misspecifies the true causal relationship. This linear final stage effectively averages out the complex heterogeneous effects, leading to biased estimates, even if nuisance models perfectly estimate E[Y|X] and E[T|X].

Variance: While CATE estimates showed spread, indicating variance, the poor correlation with true_tau implies that this spread largely represents noise rather than meaningful capture of individual heterogeneity.

Observed Heterogeneity: The synthetic data was designed with explicit non-linear heterogeneity. However, all DML approaches, regardless of nuisance model complexity, failed to accurately capture this individual-level variation, as evidenced by the consistently low Pearson correlations.

5. Conclusion and Next Steps
This DML analysis highlights a critical limitation: while DML effectively addresses confounding bias through nuisance functions and cross-fitting, the choice of the final-stage estimator for CATE can introduce significant bias if it misspecifies the true CATE function. Even powerful non-linear nuisance models like Random Forests and XGBoost did not compensate for a simplistic linear final-stage model when the true CATE was highly non-linear.

Key Takeaways:

The true_tau in synthetic data is invaluable for rigorous evaluation.
The choice of nuisance models had minimal impact on CATE accuracy when the final stage was misspecified.
The final-stage model's flexibility to capture complex CATEs is paramount.
Next Steps:

To improve CATE estimation for truly non-linear heterogeneous effects, future work should focus on exploring more flexible final-stage models, such as:

Non-linear regression techniques.
Tree-based models (e.g., Causal Forests).
Specialized causal inference estimators designed specifically for heterogeneous effects.
This would allow for a more accurate representation of the underlying causal mechanisms and potentially yield CATE and ATE estimates closer to the true values.
