# Generalizability of Food Inspection Risk Models (Chicago)

COM0411 Fundamentals of Data Science – Group project. Phase 1: data preparation and statistical analysis, Phase 2: location feature, model data and classification models.

**Research question:** Does a food-inspection model trained on data from before Chicago's July 2018 food code change still perform well on inspections after the change, and is any drop larger than the normal change over time (placebo date 1 January 2017)?

## How to use this repository

The raw and processed CSV files are **not** in the repository (they are too large, see `.gitignore`). To rebuild the final dataset:

1. Download the dataset **Food Inspections** from the City of Chicago Data Portal (Export → CSV) and save it in this folder as `Food_Inspections.csv`.
2. Run the notebooks **in this order** (Run All in each):

| Order | Notebook | What it does | Output |
| --- | --- | --- | --- |
| 1 | `01_First_Look.ipynb` | First look at the raw data (size, columns, results per year, periods, violations) | `violations_titles_before.csv`, `violations_titles_after.csv` (already in the repo) |
| 2 | `02_Cleaning.ipynb` | Cleaning rules, Business_ID, target and period columns (keeps 2010–2022 for the history) | `food_clean.csv` |
| 3 | `03_Violations.ipynb` | Violation features: counts, severity levels, topics, previous visit | `violation_features.csv` |
| 4 | `04_History.ipynb` | History and calendar features | `history_features.csv` |
| 5 | `05_Risk_Location.ipynb` | Extra checks: risk level and location (optional) | – |
| 6 | `06_Merge.ipynb` | Merge and quality checks (incl. outliers and distribution shape) | `food_final.csv` |
| 7 | `07_Tests.ipynb` | Statistical tests (H1a, H1b, H1c, H4, H5) | – |
| 8 | `08_Location_Check.ipynb` | Location feature: Zip groups vs. k-means areas (chi-square, Cramér's V); Zip groups chosen | `location_features.csv` |
| 9 | `09_Model_Data.ipynb` | Model features (no leakage), dummy variables, periods for H2/H3 | `model_data.csv` |
| 10 | `10_Models.ipynb` | Logistic regression, decision tree, KNN; cross-validation, H2, H3, random split, ROC, confusion matrix, cutoffs | – |

`food_final.csv` is the final dataset: **one row per real inspection** (Pass, Pass w/ Conditions or Fail), **2015–2022**, 126,787 rows.

## Cleaning rules (`02_Cleaning.ipynb`)

| Rule | What | Rows after |
| --- | --- | --- |
| Start | Raw file | 317,175 |
| 1 | Keep only Pass, Pass w/ Conditions, Fail | 272,286 |
| 2 | Remove exact duplicate rows (none found) | 272,286 |
| 3 | Remove the same license inspected twice on the same date | 271,222 |
| 4 | Remove rows without a License # or with License # = 0 | 270,583 |
| 5 | Missing Facility Type / Risk → "Unknown"; Risk "All" → "Unknown" | 270,583 |
| 6 | `Business_ID` = License # + business name (1,134 of 43,891 licenses had more than one name) | 270,583 |
| 7 | Keep inspections up to 2022 (2010–2022, so the history columns can use the earlier visits) | 212,463 |
| – | One missing Inspection Type in the study years filled as "License Re-Inspection" (same business, two days after a License inspection) | 212,463 |
| Merge | After the history is calculated, keep the study years 2015–2022 (`06_Merge.ipynb`) | 126,787 |

## Outside information (not in the CSV)

The CSV only contains violation numbers, titles and comments. The following comes from official City of Chicago documents:

- **1 July 2018**: Chicago adopted a new food code (new violation list: 45 → 64 numbers, new numbering and new severity levels).
- **Severity levels:** before the change, violations 1–14 are *Critical*, 15–29 *Serious*, 30–44 and 70 *Minor*; after the change, *Priority*, *Priority Foundation* or *Core*, written in the inspector's comment.
- **Grace period:** for the first inspection after 1 July 2018, no citations and 90 days to comply.
- **Disclaimer:** violations are allegations recorded at the time of the inspection and may be disputed later; the data is not intended to predict future findings (limitation).

Sources: City of Chicago Data Portal – Food Inspections dataset description; Chicago Department of Public Health, *Chicago Food Code Major Changes*; CDPH Food Inspection FAQ (2019); Notice of Increased Food Safety Violation Fines (2026).

## Column dictionary (`food_final.csv`)

**Use in model:**
- ✅ = can be used as a model feature (known before the inspection).
- 🎯 = target.
- 📊 = for statistical tests only. It is known only after the inspection, so using it in the model would be leakage.
- ➖ = ID or description, not used.

### From the original data (`food_clean.csv`)

| Column | Meaning | Use in model |
| --- | --- | --- |
| Inspection ID | Unique ID of the inspection (used to merge the files) | ➖ |
| DBA Name, AKA Name | Legal name / public name of the business | ➖ |
| License # | Business license number | ➖ |
| Business_ID | License # + upper-case business name; one ID per business | ➖ (used for history features) |
| Facility Type | Type of place (Restaurant, Grocery Store, …) | ➖ (see Facility_Group) |
| Risk | Official risk category of the place (Risk 1 High, 2 Medium, 3 Low) | ➖ (see Risk_Num) |
| Address, City, State, Zip, Latitude, Longitude, Location | Location of the business | ➖ (Zip possible later) |
| Inspection Date | Date of the inspection | ➖ |
| Inspection Type | Canvass, License, Complaint, Re-Inspection, … | ➖ (see Inspection_Group) |
| Results | Pass, Pass w/ Conditions or Fail | 📊 |
| Violations | Full violation text of the inspection | 📊 |
| First_Seen_Date | Date of the first inspection of the business in the whole dataset (from 2010) | ➖ (see Days_Since_First_Seen) |
| Year | Year of the inspection | ➖ |
| **Fail** | **1 = Fail, 0 = Pass or Pass w/ Conditions** | 🎯 |
| Period | Pre (before 1 Jul 2018), Grace (90 days), Late transition (to 30 Jun 2019), Post | ➖ (used to split train/test) |

### Violation features (`03_Violations.ipynb`)

| Column | Meaning | Use in model |
| --- | --- | --- |
| Num_Violations | Number of violations in this inspection | 📊 |
| Num_Critical | Critical (old 1–14) / Priority (new) violations in this inspection | 📊 |
| Num_Serious | Serious (old 15–29) / Priority Foundation (new) violations in this inspection | 📊 |
| has_rodents, has_temperature, has_handwashing, has_toxic, has_water_sewage, has_dirty_surfaces, has_thermometers, has_food_handler, has_smoking, has_summary_report | 1 if the inspection has a violation of this topic (old/new numbers matched by the crosswalk) | 📊 |
| Prev_Critical | 1 if the previous visit of the business had a critical violation | ✅ |
| Prev_Serious | 1 if the previous visit had a serious violation | ✅ |
| Prev_Num_Violations | Number of violations at the previous visit | ✅ |
| Past_Critical_Rate | Share of all earlier visits with a critical violation | ✅ |
| Past_Serious_Rate | Share of all earlier visits with a serious violation | ✅ |

**Crosswalk (old → new violation numbers):**

| Topic | Old | New |
| --- | --- | --- |
| Rodents / insects | 13, 18 | 38 |
| Temperature | 2, 3 | 18–22, 33 |
| Hand washing | 6, 12 | 8, 9, 10 |
| Toxic items | 25 | 28 |
| Water / sewage | 9, 10 | 50, 52 |
| Dirty surfaces | 33, 34, 35 | 49, 55 |
| Thermometers | 40 | 36 |
| Food handler training | 45 | 57 |
| Smoking | 70 | 62 |
| Summary report displayed | 28 | 61 |

### History and calendar features (`04_History.ipynb`)

| Column | Meaning | Use in model |
| --- | --- | --- |
| Prev_Fail | 1 if the business failed its previous inspection | ✅ |
| First_Inspection | 1 if this is the first inspection of the business in the whole dataset (since 2010); same information as Num_Past_Inspections = 0 and Days_Since_First_Seen = 0 | ➖ (repeats other columns, left out of the model) |
| Days_Since_Last | Days since the previous inspection of the business (first inspections have no previous visit, so they are filled with the median, 231 days, not 0) | ✅ |
| Past_Fail_Rate | Share of earlier inspections of the business that failed | ✅ |
| Num_Past_Inspections | Number of earlier inspections of the business | ✅ |
| Days_Since_First_Seen | Days since the first inspection of the business since 2010 (approximate age) | ✅ |
| Is_Reinspection | 1 if the inspection type is a re-inspection | ✅ |
| Month | Month (1–12) of the inspection | ➖ (Season is used instead) |
| Season | Season of the inspection | ✅ (dummies) |
| Facility_Group | 8 most common facility types, the rest "Other" | ✅ (dummies) |
| Inspection_Group | Inspection type grouped: Canvass, License, Complaint (incl. food poisoning), Other; re-inspections are in Is_Reinspection | ✅ (dummies) |
| Risk_Num | Risk as a number (1 High, 2 Medium, 3 Low, 0 Unknown) | ✅ |
| Placebo_Period | Before / after 1 January 2017 (fake change date, for H3) | ➖ (used to split train/test) |

### Location feature (`08_Location_Check.ipynb`)

| Column | Meaning | Use in model |
| --- | --- | --- |
| Zip_Group | Chicago Zip code of the business (starts with 606); Zip codes outside Chicago, missing or with fewer than 100 inspections are "Other" (58 groups) | ✅ (dummies) |
| Location_Cluster | Area from k-means clustering of the coordinates (k = 5, chosen with the elbow method) | ➖ (weaker relation with Fail than Zip: Cramér's V 0.061 vs 0.102) |

### Model data (`model_data.csv`)

126,787 rows; **83 features**: 12 numeric (Prev_Fail, Past_Fail_Rate, Prev_Critical, Prev_Serious, Prev_Num_Violations, Past_Critical_Rate, Past_Serious_Rate, Days_Since_Last, Num_Past_Inspections, Days_Since_First_Seen, Is_Reinspection, Risk_Num) and 71 dummies (8 facility, 3 inspection type, 3 season, 57 Zip; one category dropped per column). Class balance: 22.4% Fail.

## Main results (`10_Models.ipynb`)

Hyperparameters chosen with 5-fold cross-validation: decision tree max_depth = 5, KNN k = 25. Features standardized (scaler fitted on the training data only). Cutoff = fail rate of the training data (0.216 in H2, 0.208 in H3, 0.225 in the random split).

| Model | AUC H2: Pre → Post | AUC H3: before → after 2017 | AUC random split |
| --- | --- | --- | --- |
| Logistic regression | 0.690 → 0.643 (−0.048) | 0.705 → 0.648 (−0.057) | 0.675 |
| Decision tree | 0.674 → 0.647 (−0.027) | 0.681 → 0.671 (−0.010) | 0.671 |
| KNN | 0.664 → 0.607 (−0.057) | 0.683 → 0.610 (−0.073) | 0.653 |

**H2 supported** (all models are worse after the change), **H3 not supported** (the drop at the placebo date is similar or larger): the loss is mainly ordinary change over time, not an effect of the new food code, so the models should be retrained regularly.

## Limitations

- The old → new crosswalk is an approximation made by matching titles, the lists are not one-to-one.
- The new Priority Foundation level covers many more problems than the old Serious level, so severity columns do not mean exactly the same thing before and after 2018.
- History features are computed from 2010, the start of the dataset, so businesses inspected before 2010 still look newer than they are (their first visit in the data is not their real first visit).
- Violations are allegations that may be dismissed later; the data does not show this.
- 2020 is affected by COVID (fewer inspections).
- Outliers (very long violation lists, long gaps between inspections) were checked with the IQR method and **kept**, because they are real inspections and the long violation lists are part of the change we study (see `06_Merge.ipynb`, quality check 5).
- The numeric columns are right-skewed (see `06_Merge.ipynb`, quality check 6), so we compare them with the median and the Mann–Whitney test instead of the mean and the t-test.
- The same business is inspected many times, so the inspections are not fully independent. This can make the p-values smaller than they should be, so in `07_Tests.ipynb` we focus on the effect sizes (Cramér's V, rank-biserial).
- The grace period applied to each business's first inspection after 1 July 2018, not to a fixed date range, so our 90-day "Grace" period is an approximation.
- The dummy variables are also standardized, so in KNN a rare Zip code gets a large distance and can count more than the business's history.
- We use one random split and one placebo date; the Post period includes COVID-19 and is longer than the placebo test period.
- The models reach only moderate AUC values (0.61–0.71), so failing inspections are hard to predict even without a rule change.

