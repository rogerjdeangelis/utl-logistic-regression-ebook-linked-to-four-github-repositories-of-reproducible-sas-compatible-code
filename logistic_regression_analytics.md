---
title: "Logistic Regression"
pdf-metadata: false
header-includes: |
  \renewcommand{\familydefault}{\ttdefault}
  \usepackage{geometry}
  \geometry{left=0.5in, right=0.5in, top=0.5in, bottom=0.5in}
  \geometry{landscape}
  \usepackage{xcolor}
  \definecolor{linkblue}{HTML}{00008B}
  \usepackage{hyperref}
  \hypersetup{
    colorlinks=true,
    linkcolor=linkblue,
    filecolor=linkblue,
    urlcolor=linkblue,
    citecolor=linkblue,
    pdftitle={Logistic Regression Using Jenner Analytics},
    pdfcreator={LaTeX via pandoc}
  }
---

## Table of Contents

[Preface](#preface)

[**CHAPTER I: OPTIMUM BINNING IN PREPARATION FOR LOGISTIC REGRESSION**](#chapter-i)

- [I. Analysis Overview](#i-analysis-overview)
- [II. Campaign Strategy](#ii-campaign-strategy)
- [III. Key Output -- Sample of Binning One Covariate](#iii-key-output--sample-of-binning-one-covariate)
- [IV. Interpreting Binned Data](#iv-interpreting-binned-data)
- [V. Input](#v-input)
- [VI. Output -- Five Tables](#vi-output--five-tables)
- [VII. Theory](#vii-theory)
- [VIII. Method -- Program Steps](#viii-method--program-steps)
- [IX. Contents of Future Chapters](#ix-contents-of-future-chapters)

[**CHAPTER II: IDENTIFYING THE BEST FIVE LOGISTIC MODELS**](#chapter-ii)

- [I. Preparation](#i-preparation)
- [II. Contents](#ii-contents)
- [III. Inputs](#iii-inputs)
- [IV. Outputs](#iv-outputs)
- [V. Key Notes](#v-key-notes)

[**CHAPTER III: VERIFYING TRAINING LOGISTIC USING HOLDOUT**](#chapter-iii)

- [I. Best Model from Chapter II](#i-best-model-from-chapter-ii)
- [II. Training Logistic](#ii-training-logistic)
- [III. Create SAS Code to Bin Variables](#iii-create-sas-code-to-bin-variables)
- [IV. Map Character Variables](#iv-map-character-variables)
- [V. Map Numeric Variables](#v-map-numeric-variables)
- [VI. Apply IF-THEN-ELSE to Holdout](#vi-apply-if-then-else-to-holdout)
- [VII. Run Logistic on Holdout Sample](#vii-run-logistic-on-holdout-sample)
- [VIII. Rank Probabilities](#viii-rank-probabilities)
- [IX. Verify Training Logistic Using Holdout](#ix-verify-training-logistic-using-holdout)
- [X. Compare Lift Plots -- Training and Holdout](#x-compare-lift-plots--training-and-holdout)
- [XI. ASCII Plot Verification](#xi-ascii-plot-verification)
- [XII. Output Datasets](#xii-output-datasets)

[**CHAPTER IV: KEY OUTPUTS FROM THE FINAL LOGISTIC MODEL**](#chapter-iv)

- [I. Key Outputs Overview](#i-key-outputs-overview)
- [II. Gains Lift Table](#ii-gains-lift-table)
- [III. ROC Curves](#iii-roc-curves)
- [IV. Perfect Relative Contributions to Max Probability](#iv-perfect-relative-contributions-to-max-probability)
- [V. Full Logistic Analysis Tables](#v-full-logistic-analysis-tables)
- [VI. Inputs](#vi-inputs)
- [VII. Outputs](#vii-outputs)

[Index](#index)

---

# Preface {#preface}

This ebook documents a complete logistic regression workflow using the **Jenner Analytics** methodology, as implemented across four GitHub repositories by Roger J. DeAngelis. The series demonstrates a systematic approach to:

1. **Optimum binning** of both character and numeric covariates in preparation for logistic regression (Chapter I)
2. **Identifying the best five logistic models** from all possible combinations of predictors (Chapter II)
3. **Verifying the training logistic model** using a holdout sample (Chapter III)
4. **Extracting and presenting key outputs** from the final logistic model (Chapter IV)

The methodology emphasizes interpretability, validation, and practical deployment. All code and examples are provided in SAS, with extensive use of macros and reproducible workflows.

The repository text throughout this ebook is presented in a fixed font for clarity.

I suggest readers use this ebook to acquaint themselves with the process, but spend more time
implementing workflow using the four repositories below.

**GitHub:** [https://github.com/rogerjdeangelis/utl-altair-slc-chapter-I-optimum-binning-in-preparation-for-logistic-regression](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-I-optimum-binning-in-preparation-for-logistic-regression)

**GitHub:** [https://github.com/rogerjdeangelis/utl-altair-slc-chapter-II-identifying-the-best-five-logistic-models](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-II-identifying-the-best-five-logistic-models)

**GitHub:** [https://github.com/rogerjdeangelis/utl-altair-slc-chapter-III-verifying-training-logistic-using-holdout](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-III-verifying-training-logistic-using-holdout)

**GitHub:** [https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model)

---

# CHAPTER I: OPTIMUM BINNING IN PREPARATION FOR LOGISTIC REGRESSION {#chapter-i}

**Repository:** `utl-altair-slc-chapter-I-optimum-binning-in-preparation-for-logistic-regression`

**GitHub:** [https://github.com/rogerjdeangelis/utl-altair-slc-chapter-I-optimum-binning-in-preparation-for-logistic-regression](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-I-optimum-binning-in-preparation-for-logistic-regression)

### I. Analysis Overview {#i-analysis-overview}

The first chapter focuses on the critical preprocessing step of **optimum binning** -- grouping continuous and categorical variables into bins that exhibit monotonic relationships with the binary response variable. This step is essential for:

- Ensuring that the logistic model satisfies the **linearity assumption** for continuous predictors
- Creating **interpretable** categorical predictors with meaningful odds ratios
- Reducing **noise** and overfitting by collapsing sparse categories
- Improving **model stability** and performance

The binning process uses **Mantel--Haenszel chi-square statistics** to determine optimal cut points that maximize the association between each predictor and the response.

### II. Campaign Strategy {#ii-campaign-strategy}

The campaign strategy involves:

1. **Data validation and verification** of raw input
2. **Handling missing values** -- converting all missing formats to a consistent `'?'` representation
3. **Dropping low-cardinality variables** (one-to-one relationships)
4. **Optimizing variable length** to the longest observed value in the data
5. **Creating holdout and training tables** (70,000 training / 30,000 holdout)

### III. Key Output -- Sample of Binning One Covariate {#iii-key-output--sample-of-binning-one-covariate}

The binning process produces two key Excel reports:

+---------------------------+--------------------------------------------------------+------------------+
| Report                    | Description                                            | Link             |
+---------------------------+--------------------------------------------------------+------------------+
| Character Binning Report  | Binned character covariates with chi-square & MH stats | lgs_MgmChrCut.xl |
| Numeric Binning Report    | Binned numeric covariates with chi-square & MH stats   | lgs_MgmNumCut.xl |
+---------------------------+--------------------------------------------------------+------------------+

Each report contains:

- **Bin boundaries** for each covariate
- **Odds ratios** per bin (compared to a reference bin)
- **Chi-square statistics** and p-values
- **Mantel--Haenszel trend tests** for monotonic association

### IV. Interpreting Binned Data {#iv-interpreting-binned-data}

The binned data is represented in two forms:

1. **Normalized (long and skinny) format** -- each observation is expanded to multiple rows (one per bin variable), with just 5 key variables:
   - `_ID` (identifier)
   - `_NAME` (variable name)
   - `_VALUE` (bin value)
   - `_RESPONSE` (binary outcome)
   - `_WEIGHT` (optional)

2. **Denormalized (wide) format** -- each original observation has one row with 30--38 binned indicator variables

Character variables:  14 original -> 40 bin indicators (denormalized)

Numeric variables:    13 original -> 46 bin indicators (denormalized)

### V. Input {#v-input}

The raw input data is provided as:

- **`lgs_raw.zip`** -- containing the raw training and holdout datasets

[https://github.com/rogerjdeangelis/utl-altair-slc-chapter-I-optimum-binning-in-preparation-for-logistic-regression/blob/main/lgs_raw.zip](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-I-optimum-binning-in-preparation-for-logistic-regression/blob/main/lgs_raw.zip)

### VI. Output -- Five Tables {#vi-output--five-tables}

The chapter produces five key output tables:

+-------------------+-----------------------------------------------------+-------------+
| Table             | Description                                         | Format      |
+-------------------+-----------------------------------------------------+-------------+
| lgs_mgmAllChrNum  | Final logistic-ready binned training data (wide)    | SAS dataset |
| lgs_mgmNrmChr     | Normalized character binned data (980k rows)        | SAS dataset |
| lgs_mgmNrmNumSrt  | Normalized numeric binned data (910k rows)         | SAS dataset |
| lgs_mgmChrCutRpt  | Character binning summary (40 obs, 14 vars)        | Excel       |
| lgs_mgmNumCut     | Numeric binning summary (46 obs, 13 vars)           | Excel       |
+-------------------+-----------------------------------------------------+-------------+

### VII. Theory {#vii-theory}

The binning algorithm is based on the principle of **maximum association**:

- For each predictor, the algorithm searches for cut points that maximize the **chi-square statistic** for the 2xk contingency table (response x bins)
- **Mantel--Haenszel chi-square** is used to test for **linear trend** across ordered bins
- The optimal number of bins is determined by balancing:
  - **Statistical significance** (maximizing chi-square)
  - **Interpretability** (minimizing the number of bins)
  - **Monotonicity** (ensuring odds ratios increase or decrease consistently)

### VIII. Method -- Program Steps {#viii-method--program-steps}

The complete program consists of ten major steps:

1.  Contents of raw input data
2.  Validate and verify raw data; fix issues with raw data
3.  One missing value -- Drop one-to-one variables; very low cardinality variables. Convert the many missing variable formats to just '?'
4.  Optimize variable length to the longest in the data
5.  Create Holdout and Training Tables
6.  Bin the character variables in groups with common odds ratios
    - Create normalized (long and skinny binned data with just 5 variables)
    - Create denormalized (wide binned table with 30 variables)
7.  Create Excel character data summary bin report with chi-square and Mantel--Haenszel stats
8.  Bin the numeric variables in groups with common odds ratios
    - Create normalized (long and skinny binned table with just 5 variables)
    - Create denormalized (wide binned table with 38 variables)
9.  Create Excel numeric data summary bin report with chi-square and Mantel--Haenszel stats
10. Join raw training table, numeric binned data with character binned data for analysis

**Macros Required** (place in autocall library):

[https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories](https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories)

  - utlfkil
  - slc_optimalbin.sas (also in this repository)

[https://github.com/rogerjdeangelis/utl_optlen](https://github.com/rogerjdeangelis/utl_optlen)

  - utloptlen.sas

[https://github.com/rogerjdeangelis/voodoo](https://github.com/rogerjdeangelis/voodoo)

  - slc_voodoo20251126.sas (documentation included)

### IX. Contents of Future Chapters {#ix-contents-of-future-chapters}

The subsequent chapters in this series cover:

+----------+-------------------------------------------+
| Chapter  | Topic                                     |
+----------+-------------------------------------------+
| II       | Identifying the best five logistic models |
| III      | Verifying training logistic using holdout |
| IV       | Key outputs from the final logistic model |
+----------+-------------------------------------------+

**Key outputs from the full series include:**

- `lgs_mgmFinalLogisticDiag` -- Logistic model diagnostics
- `lgs_MgmGainsChart` -- Gains chart
- `lgs_mgmTopChiValues` -- Most influential variables
- `lgs_mgmTopIndexValues` -- Highest response variables
- `lgs_MgmTopTen` -- List of top 12 scores
- `lgs_MgmVenn` -- Comparison of covariate contribution to top percentile
- Final PDF presentation (slide deck)

**Final logistic-ready output:**

[https://github.com/rogerjdeangelis/utl-altair-slc-chapter-I-optimum-binning-in-preparation-for-logistic-regression/blob/main/lgs_mgmallchrnum](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-I-optimum-binning-in-preparation-for-logistic-regression/blob/main/lgs_mgmallchrnum)

---

# CHAPTER II: IDENTIFYING THE BEST FIVE LOGISTIC MODELS {#chapter-ii}

**Repository:** `utl-altair-slc-chapter-II-identifying-the-best-five-logistic-models`

**GitHub:** [https://github.com/rogerjdeangelis/utl-altair-slc-chapter-II-identifying-the-best-five-logistic-models](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-II-identifying-the-best-five-logistic-models)


### I. Preparation {#i-preparation}

Before running the analysis, two preparation steps are required:

1. **Library statement** -- Add this to your autoexec:
   `libname workx "d:/wpswrkx";`

2. **Macro** -- Copy `utl_mdlgetpos.sas` to your autocall library

### II. Contents {#ii-contents}

All Excel files referenced in this chapter are available in the repository:

+---------------------------+----------------------------------+
| File                      | Description                      |
+---------------------------+----------------------------------+
| lgs_mgmtopchivalues.xlsx  | Top chi-square values            |
| lgs_mgmtopoddsvalues.xlsx | Top odds ratios                  |
| lgs_stepselect.xlsx       | Stepwise logistic results        |
+---------------------------+----------------------------------+

### III. Inputs {#iii-inputs}

The chapter uses outputs from Chapter I:

\begin{minipage}{\linewidth}
\small
\begin{verbatim}
+-----------------------+-------------------------------+--------+-----------------------------+
| Table                 | Description                   | Obs    | Location                    |
+-----------------------+-------------------------------+--------+-----------------------------+
| RAW TRAINING          | Raw training data (no binning)| 70,000 | d:/lgs/lgs_rawTrain.sas7bdat|
| RAW HOLDOUT           | Raw holdout data (no binning) | 30,000 | d:/lgs/lgs_rawHold.sas7bdat |
| LOGISTIC INPUT BINNED | Binned training data          | 70,000 | d:/lgs/lgs_MgmAllChrNum.sas.|
| NORMALIZED CHAR BINNED| Character bins (long format)  | 980,000| d:/lgs/lgs_mgmNrmChr.sas7bd.|
| NORMALIZED NUM BINNED | Numeric bins (long format)    | 910,000| d:/lgs/lgs_mgmNrmNumSrt.sas.|
| CHR EXCEL REPORT      | Character binning report      | 40     | d:/lgs/lgs_mgmChrCutRpt     |
| NUM EXCEL REPORT      | Numeric binning report        | 46     | d:/lgs/lgs_mgmNumCut        |
+-----------------------+-------------------------------+--------+-----------------------------+
\end{verbatim}
\end{minipage}

### IV. Outputs {#iv-outputs}

The primary output is the **top 100 models** for each model size (4--7 predictors):

+----------------+------------------+
| Model Size     | Number of Models |
+----------------+------------------+
| 4 predictors   |              100 |
| 5 predictors   |              100 |
| 6 predictors   |              100 |
| 7 predictors   |              100 |
| **Total**      |        **400**   |
+----------------+------------------+

Output location: `d:/lgs/LGS_POSMGMMODNRM`

### V. Key Notes {#v-key-notes}

1. **This analysis works best with hundreds of independent variables.** The methodology scales well with high-dimensional data.

2. **It also works better if increasing values of the independent variables align with increasing response.** Monotonic relationships improve model interpretability and performance.

3. **Positive coefficients requirement.** By default, all model predictors are required to have **positive coefficients** (intercept excluded). This provides:
   - More insight into the direction of effects
   - Easier interpretation of odds ratios
   - Models that "make sense" clinically or logically

   The macro also supports identifying models with **mixed sign coefficients** to check for potentially better fits. Negative predictors often work against other predictors and may indicate multicollinearity or reverse coding.

4. **All possible models with 4--7 independent variables are examined.** The number of models tested is substantial:

   ```
   n_choose_4 = comb(20, 4) = 4,845
   n_choose_5 = comb(20, 5) = 38,760
   n_choose_6 = comb(20, 6) = 38,760
   n_choose_7 = comb(20, 7) = 77,520
   Total = 159,885 models
   ```

   **Note:** SLC does not support all-possible `proc logistic` models, so `proc reg` is used instead.

5. **Stepwise logistic regression** is performed to validate the best models identified by the all-possible search.

**Additional Analysis:**

- **VIF (Variance Inflation Factor)** -- Variables with high VIF are removed and the model is rechecked
- **Top Chi-Square Values** -- Identifies the most influential individual predictors
- **Top Odds Ratios** -- Identifies predictors with the strongest effect sizes

---

# CHAPTER III: VERIFYING TRAINING LOGISTIC USING HOLDOUT {#chapter-iii}

**Repository:** `utl-altair-slc-chapter-III-verifying-training-logistic-using-holdout`

**GitHub:** [https://github.com/rogerjdeangelis/utl-altair-slc-chapter-III-verifying-training-logistic-using-holdout](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-III-verifying-training-logistic-using-holdout)


### I. Best Model from Chapter II {#i-best-model-from-chapter-ii}

The best model identified in Chapter II uses the following predictors:

- **Response:** `_MARRIED`
- **Predictors:**
  - `_ACSGENDER`
  - `_DIVISIONCDE`
  - `_INCOME`
  - `_LOANHOME`

### II. Training Logistic {#ii-training-logistic}

A logistic regression model is fitted on the **training data** (70,000 observations) using the five predictors identified above.

### III. Create SAS Code to Bin Variables {#iii-create-sas-code-to-bin-variables}

The binning scheme from Chapter I is converted into SAS `IF-THEN-ELSE` code. This code is generated automatically using the binning cut points determined by the `optimalbin` macro on the training data.

### IV. Map Character Variables {#iv-map-character-variables}

SAS `IF-THEN-ELSE` code is created to map **character variables** to their binned categories:

```sas
/* Example: Mapping character variable to bins */
IF _ACSGENDER IN ('M','Male') THEN _ACSGENDER_BIN = 1;
ELSE IF _ACSGENDER IN ('F','Female') THEN _ACSGENDER_BIN = 2;
/* ... etc. */
```

### V. Map Numeric Variables {#v-map-numeric-variables}

Similarly, SAS `IF-THEN-ELSE` code is created to map **numeric variables** to their binned categories:

```sas
/* Example: Mapping numeric variable to bins */
IF _INCOME < 25000 THEN _INCOME_BIN = 1;
ELSE IF 25000 <= _INCOME < 50000 THEN _INCOME_BIN = 2;
ELSE IF 50000 <= _INCOME < 75000 THEN _INCOME_BIN = 3;
ELSE _INCOME_BIN = 4;
```

### VI. Apply IF-THEN-ELSE to Holdout {#vi-apply-if-then-else-to-holdout}

The generated `IF-THEN-ELSE` code is applied to the **holdout sample** (30,000 observations) to bin the data using the same cut points as the training data.

### VII. Run Logistic on Holdout Sample {#vii-run-logistic-on-holdout-sample}

The logistic model (with the same coefficients estimated from the training data) is applied to the holdout sample to generate predicted probabilities.

### VIII. Rank Probabilities {#viii-rank-probabilities}

Predicted probabilities are ranked into **deciles** (10 groups) for both:

- **Training sample** predicted probabilities
- **Holdout sample** predicted probabilities

### IX. Verify Training Logistic Using Holdout {#ix-verify-training-logistic-using-holdout}

The model is verified by comparing the **lift** (ratio of response rate in each decile to the overall response rate) between training and holdout samples.

### X. Compare Lift Plots -- Training and Holdout {#x-compare-lift-plots--training-and-holdout}

A lift plot comparison is generated:

[https://github.com/rogerjdeangelis/utl-altair-slc-chapter-III-verifying-training-logistic-using-holdout/blob/main/liftplots.png](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-III-verifying-training-logistic-using-holdout/blob/main/liftplots.png)

### XI. ASCII Plot Verification {#xi-ascii-plot-verification}

The differences between training and holdout probabilities are small (most < 0.01), indicating that the model generalizes well to new data.

### XII. Output Datasets {#xii-output-datasets}

The chapter produces the following output datasets:

+---------------------------+-------------------------------------------------------+
| Dataset                   | Description                                           |
+---------------------------+-------------------------------------------------------+
| Training probabilities    | Predicted probabilities for training sample, by decile|
| Holdout probabilities     | Predicted probabilities for holdout sample, by decile |
| Lift comparison           | Side-by-side lift values for training and holdout     |
+---------------------------+-------------------------------------------------------+

**Key Finding:** The holdout validation confirms that the logistic model developed on the training data performs consistently on unseen data, with minimal degradation in predictive performance.

---

# CHAPTER IV: KEY OUTPUTS FROM THE FINAL LOGISTIC MODEL {#chapter-iv}

**Repository:** `utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model`

**GitHub:** [https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model)


### I. Key Outputs Overview {#i-key-outputs-overview}

This chapter presents the **final outputs** from the logistic regression model, including:

- Gains and lift tables
- ROC curves
- Relative contribution charts
- Comprehensive logistic analysis tables (14 Excel tabs)
- Publishable PDF reports

### II. Gains Lift Table {#ii-gains-lift-table}

The gains lift table provides a comprehensive view of model performance:

**File:** `lgs_ganrptfin.xlsx`

[https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model/blob/main/lgs_ganrptfin.xlsx](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model/blob/main/lgs_ganrptfin.xlsx)

The table includes:

- **Decile** (1--10, from highest to lowest predicted probability)
- **Number of observations** per decile
- **Number of responders** per decile
- **Response rate** per decile
- **Cumulative response rate**
- **Lift** (response rate in decile / overall response rate)
- **Cumulative lift**

### III. ROC Curves {#iii-roc-curves}

ROC (Receiver Operating Characteristic) curves are provided for:

- The **overall model** (overlay of all predictors)
- **Individual predictors:**
  - `MARRIED` ROC curve
  - `ACSGENDER` ROC curve
  - `DIVISIONCDE` ROC curve
  - `INCOME` ROC curve
  - `LOANHOME` ROC curve

**File:** `ROCOverlay.png`

[https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model/blob/main/ROCOverlay.png](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model/blob/main/ROCOverlay.png)

### IV. Perfect Relative Contributions to Max Probability {#iv-perfect-relative-contributions-to-max-probability}

A pie chart showing the **relative contribution of each predictor** to the maximum probability score:

**File:** `Relative_Contributions.png` (also `SGPie.png`)

[https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model/blob/main/SGPie.png](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model/blob/main/SGPie.png)

This visualization helps identify which variables are driving the highest predicted probabilities.

### V. Full Logistic Analysis Tables {#v-full-logistic-analysis-tables}

Comprehensive logistic analysis is provided in multiple formats:

+----------+-------------------------+--------------------------------------------------+
| Format   | File                    | Description                                      |
+----------+-------------------------+--------------------------------------------------+
| Excel    | logistic_tables.xlsx    | 14 Excel tabs with detailed logistic analysis    |
| HTML     | logistic_tables.html    | Web-compatible version                           |
| PDF      | logistic_tables.pdf     | Publishable version                              |
| RTF      | logistic_tables.rtf     | Microsoft Word compatible                        |
+----------+-------------------------+--------------------------------------------------+

[https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model/blob/main/logistic_tables.xlsx](https://github.com/rogerjdeangelis/utl-altair-slc-chapter-IV-key-outputs-from-the-final-logistic-model/blob/main/logistic_tables.xlsx)

### VI. Inputs {#vi-inputs}

Only **one input** is required for this repository -- the outputs from Chapters I and II are used as inputs.

### VII. Outputs {#vii-outputs}

All outputs are in the repository:

```
d:/lgs
+---htm
|   logistic_tables.html
+---pdf
|   logistic_tables.pdf
+---sas
|   slc_gain.sas          (Gains and Lift table code, Word-compatible)
|   slc_pdflan100.sas     (Proc template for PDF output)
|   slc_rtflan100.sas     (Proc template for RTF output)
+---rtf
|   logistic_tables.rtf
+---png
|   Relative_Contributions.png
|   ROCOverlay.png
|   ROCCurve1.png
|   SGPie.png
\---xls
    lgs_5best.xlsx        (Best model from 136,789 models examined)
    lgs_ganrptfin.xlsx    (Gains Lift Table)
    lgs_MgmChrCut.xlsx    (Best Character Bins)
    lgs_MgmNumCut.xlsx    (Best Numeric Bins)
    lgs_mgmtopchivalues.xlsx  (Chi-Square analysis -- Mantel-Haenszel linear test)
    lgs_mgmtopoddsvalues.xlsx (Odds Ratio analysis)
    lgs_stepselect.xlsx   (Stepwise logistic -- validates best models)
```

**Summary of Key Files:**

+---------------------------+----------------------------------------------------+
| File                      | Description                                        |
+---------------------------+----------------------------------------------------+
| lgs_5best.xlsx            | Best model from 136,789 models examined            |
| lgs_ganrptfin.xlsx        | Gains and Lift Table                               |
| lgs_MgmChrCut.xlsx        | Best Character Bins                                |
| lgs_MgmNumCut.xlsx        | Best Numeric Bins                                  |
| lgs_mgmtopchivalues.xlsx  | Chi-Square analysis (Mantel-Haenszel linear test)  |
| lgs_mgmtopoddsvalues.xlsx | Odds Ratio analysis                                |
| lgs_stepselect.xlsx       | Stepwise logistic results                          |
+---------------------------+----------------------------------------------------+

---

## Index {#index}

- All-possible models -- II
- ASCII plot -- III
- Autocall library -- I, II
- Best model -- II, III, IV
- Bin boundaries -- I
- Binning -- I
- Campaign strategy -- I
- Character variables -- I, III
- Chi-square -- I, II, IV
- Coefficients (positive) -- II
- Covariates -- I
- Decile -- III, IV
- Denormalized -- I
- Excel reports -- I, II, IV
- Gains chart -- I, IV
- Holdout -- I, III
- IF-THEN-ELSE -- III
- Lift -- III, IV
- Logistic regression -- II, III, IV
- Macros -- I, II
- Mantel--Haenszel -- I, IV
- Missing values -- I
- Monotonic relationship -- I, II
- Normalized data -- I
- Numeric variables -- I, III
- Odds ratios -- I, II, IV
- Optimum binning -- I
- Output datasets -- I, III, IV
- Positive coefficients -- II
- Proc logistic -- II
- Proc reg -- II
- ROC curves -- IV
- Stepwise logistic -- II, IV
- Training data -- I, III
- VIF (Variance Inflation Factor) -- II

---

*End of ebook*
