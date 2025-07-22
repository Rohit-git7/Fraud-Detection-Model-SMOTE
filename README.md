# Fraud-Detection-Model---SMOTE
Developed a model for predicting fraudulent transactions for a financial company and used insights from the model to develop an actionable plan. Data for the case is in CSV format having 6362620 rows and 10 columns.

## Questions given in the assessment :
1.Data cleaning including missing values, outliers and multi-collinearity.
2.Describe your fraud detection model in elaboration.
3.How did you select variables to be included in the model?
4.Demonstrate the performance of the model by using best set of tools.
5.What are the key factors that predict fraudulent customer?
6.Do these factors make sense? If yes, How? If not, How not?
7.What kind of prevention should be adopted while company update its infrastructure?
8.Assuming these actions have been implemented, how would you determine if they work?

## Answers with respect to our model
1. Data Cleaning: Missing Values, Outliers, and Multi-Collinearity
   Missing Values & Duplicates : I have inspected in the entire dataset for missing values and duplicate rows and 
   no missing values or duplicates were found, ensuring data completeness.

   Outlier Treatment: Examined numerical columns (`amount`, balance differences, error features) for extreme values.
   capped critical numeric features at the 1st and 99th percentiles to control the influence of outliers without 
   removing information-rich anomalies and i have also applied a log-transformation to transaction amount (`log_amount`) 
   to reduce skewness.

   Multi-Collinearity:I have calculated correlation matrix to inspect relationships between features. Retained both raw 
   and engineered features only when they provided unique predictive signal (e.g., kept `orig_error` and `dest_error` with low correlation).
   Removed or consolidated highly correlated (correlation > 0.90) redundant predictors to avoid multicollinearity.

2. Model Description: Architecture and Approach

   Type: Supervised machine learning classification
   Core Algorithm: Random Forest Classifier , chosen for its robustness, interpretability, and efficacy with structured & imbalanced fraud data.
   Key Steps:
    Focused only on transaction types associated with fraud: `TRANSFER` and `CASH_OUT`.
    Applied class balancing via SMOTE oversampling on the training set, ensuring the model learns patterns from both fraud and non-fraud transactions.
    Extensive feature engineering: ratio-based, error-based, and behavioral flags to reveal subtle fraud signals.
    Final threshold for fraud alerting set at 0.91, optimized for highest F1-score and operational fit.
    Evaluation: Used stratified train/test split and untouched holdout set for unbiased validation. No data leakage occurred.

3. Variable (Feature) Selection:

  Focused on features available at transaction scoring time (no information from “future”).
  Included domain-driven engineered variables like:
    Amount and log(amount), Customer and destination zero-balance flags, Difference between pre- and post-transaction balances,
    Error (discrepancy) between expected and actual balance change, Ratios of transaction amount to originating/destination balances,
    Binary flags for round numbers, exact balance transfers, and simultaneous zero balances. Categorical features 
    (`type`, customer/destination role) encoded using numerical label encoding. Kept variables only if they contributed signal after checking for high collinearity.

4. Model Performance Demonstration

 Threshold was set to 0.91 on (Test Set: ~553K) and we got the results as :
 Precision    0.994      
 Recall       0.995      
 F1-score     0.995      
 ROC AUC      0.995     
 Fraud Caught (Recall)       1,635/1,643 (99.5%) 
 False Alerts (FP)  10    
 Missed Frauds (FN)  8    

 Confusion Matrix for our model
                     Predicted Legitimate  Predicted Fraud 

 Actual Legitimate      552,429                10      
 Actual Fraud               8                1,635     

 The gist is nearly all frauds detected by our model and less than one false positive per 160 true fraud flagged.

5. Key Factors Predicting Fraudulent Customers

 Large errors between expected and actual account balance changes (`orig_error`, `dest_error`).
 Zero starting balances on the customer or destination side—typical for accounts created solely to move or receive stolen funds.
 High ratio of transaction amount to pre-transaction balance—indicative of “draining” an account.
 Specific transaction types/codes: most fraud occurs in `TRANSFER` and `CASH_OUT`.
 Suspicious patterns such as exact balance transfers, round-number amounts, and both ends having zero balances.

6. Do These Factors Make Sense? Why/Why Not

 Yes, these factors are logical and validated by both internal data and the financial fraud data:
 In real world too, fraudsters often create new (zero-balance) "burner" accounts to move or cash out stolen funds. Balance errors and 
 discrepancies point to manipulation, hacking, or use of invalid/fake balances. Transactions that drain an account or move funds to 
 newly created destinations are classic money laundering or theft signals. By focusing on `TRANSFER` and `CASH_OUT` aligns with 
 real-world trend & fraud rarely occurs in `CASH_IN`, `PAYMENT`, or `DEBIT`.

 These engineered features directly reflect the criminal behavior the model is meant to detect, and their strong predictive power 
 is both statistically and operationally sensible.

7. Infrastructure Prevention Recommendations

 Enforce strong authentication for initiating `TRANSFER` and `CASH_OUT` actions, especially for new or zero-balance accounts.
 Real-time balance verification: Flag and hold transactions with large discrepancies or that empty accounts.
 Adaptive thresholding: Allow the system to dynamically adjust fraud-alert thresholds based on ongoing feedback and seasonal trends.
 Detailed audit trails: Record all attempted and successful changes to balances immediately before and after each transaction.
 Automatic blocking or pausing: Temporarily freeze accounts if a suspicious pattern is detected until human review.
 Continuous monitoring: Deploy dashboards and schedule weekly/biweekly analytics to spot drift in model performance or emerging fraud techniques.

8. How to Determine Effectiveness of These Actions

 We can use a before-and-after approach with key metrics like:

 Fraud Rate: Monitor total detected fraud as a percentage of all transactions, and the rate of newly emerging fraud types.
 False Positive Rate: Track numbers of legitimate customers flagged and intervene if complaint volumes rise.
 Detection Rate (Recall): Compare the percentage of confirmed fraud caught before vs. after updating controls.
 Alert Precision: Analyze the ratio of true frauds to total alerts investigated post-implementation.
 Loss Reduction: Quantify net savings in fraud-related financial loss (direct and indirect) in the months after changes.
 A/B Testing: Apply new security controls to a subset of traffic, compare fraud and false alert rates to baseline.
 Feedback Loop: Integrate investigator/analyst feedback into ongoing retraining and threshold calibration.

If metrics show an increase in caught fraud, a decrease in false positives, and reduced losses (with no unacceptable rise in legitimate customer friction), 
our interventions are effective.
