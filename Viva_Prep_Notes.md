# Viva Prep: Data Preprocessing Assessment

Practise saying each answer **out loud in your own words**. The numbers in brackets come from your notebook, so you can point to them.

---

## The 6 official viva questions

**1. Why is median imputation often preferred for skewed data?**
The mean is pulled toward extreme values; the median is the middle value and is not. In a skewed column the mean sits away from where most values are, so filling gaps with it gives a typical-looking value that is not typical. *In our data* skew is mild (salary 0.28), so mean (83,952) and median (80,536) are close. The median is still the safe default because of the salary floor (22,000) and cap (180,000).

**2. Can MCAR, MAR, MNAR always be identified from a dataset alone?**
No. We can test whether missingness is related to *observed* columns (that can rule out MCAR). We cannot test whether it depends on the *missing value itself* (MNAR), because that value is exactly what we don't have. *Our example:* salary is missing for older/more experienced staff (mean age 53 vs 40, p near 0). That rules out MCAR, but it fits both MAR (senior records kept elsewhere) and MNAR (high earners hide pay). Only domain knowledge can decide.

**3. What changes when n_neighbors changes in KNNImputer?**
Small k = fill-in follows only the few closest rows: flexible but noisy (high variance). Large k = averages more rows: smoother and more stable, but blurs local detail and drifts toward the overall average (more bias). k = 1 copies one neighbour; k = all rows becomes mean imputation. *Our validation:* salary error 23.5k (k=3), 21.3k (k=5), 19.5k (k=10); mean/median were about 45-48k.

**4. Why consider scaling before distance-based imputation?**
KNN uses distances. Salary ranges over about 22,000-180,000 and satisfaction over 1-5, so salary would decide who the "nearest neighbours" are and the other columns would be ignored. StandardScaler puts them on the same scale. *Evidence:* scaling changed 44 of 45 imputed cells, and unscaled KNN had higher error (0.91 vs 0.84 standardised RMSE).

**5. Why fit imputation and scaling on training data only?**
If you fit on the full dataset, the means/medians/neighbours used to fill training rows include test rows, so test information leaks into training and the score looks better than it truly is. Correct way: split first, `fit` on train, only `transform` test. A scikit-learn `Pipeline` does this automatically, including inside each cross-validation fold.

**6. How would you assess whether imputation distorted the original data?**
Compare before vs. after: mean, std, histograms, KS test, and correlations. If truth is known: mask known values, impute, compute RMSE/MAE. *Our results:* mean/median imputation dropped the Experience-Salary correlation from 0.959 to 0.878; KNN kept it at 0.958. Std also shrank under mean/median imputation.

---

## Other questions an examiner could ask

- **Why did you drop duplicates, and how did you know they were duplicates?** E008 and E023 appeared twice with identical values in every column (152 to 150 rows). Duplicates would double-weight records.
- **Why not delete rows with missing values?** 52 of 150 rows (35%) have at least one gap. Dropping them would lose a third of the data and could bias the sample, especially because salary gaps are concentrated among older staff.
- **Why not drop a column?** Highest missingness is 11.3% (Satisfaction). Dropping is only considered at very high missingness (roughly 40-50%+) and low usefulness.
- **Why a missingness indicator?** Being missing can be informative (e.g., seniority). Imputing hides that; a 0/1 flag keeps it. It must be created *before* imputing.
- **Why "Unknown" for Department but mode for Performance_Rating?** Missing Department rows are mostly younger staff, so mode (IT) would invent information. For Performance_Rating there was no clear pattern and the mode is a simple, transparent baseline (with the drawback that it inflates "Medium").
- **Why does KNN do worse on Satisfaction and Projects?** They are almost unrelated to other columns, so neighbours have nothing useful to say about them. KNN only helps where columns carry information about each other (Age, Experience, Salary: correlation about 0.9+).
- **Why did mean salary rise after KNN?** Missing salaries belong to senior staff, so KNN fills them with higher values. That is expected, not a flaw.
- **Why round Satisfaction after KNN?** KNN returns averages like 3.4, which is not a valid 1-5 rating.
- **What is nan-euclidean distance?** Euclidean distance computed only over the features both rows have observed, scaled up to account for the missing ones.
- **What is the limitation of single imputation?** It treats imputed values as if they were real, so uncertainty is understated. Multiple imputation addresses this.
- **Why is the model accuracy low?** The rating is weakly related to the other columns in this synthetic data. The goal of that cell was demonstrating leak-free procedure, not model quality.
- **Why do the salary 22,000 and 180,000 repeats matter?** They look like a floor and a cap (censoring), so true values beyond them are unknown and no imputer can recover them.

---

## Traps to avoid in the viva
- Do **not** say "salary is MAR" or "satisfaction is MCAR" as fact. Say "consistent with / hypothesis / cannot be proven from this data".
- Do **not** say a heatmap proves a mechanism.
- Do **not** claim KNN is always better. It was better only for the correlated columns.
- Be ready to **run any cell and explain the output** line by line.
