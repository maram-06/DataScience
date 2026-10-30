# Generalizability of Food Inspection Risk Models (Chicago)

COM0411 Fundamentals of Data Science – Group project (Gaod & Maram), Phase 1: data preparation and statistical analysis.

**Research question:** Does a food-inspection model trained on data from before Chicago's July 2018 food code change still perform well on inspections after the change?

## How to use this repository

The raw and processed CSV files are **not** in the repository (they are too large, see `.gitignore`). To rebuild the final dataset:

1. Download the dataset **Food Inspections** from the City of Chicago Data Portal (Export → CSV) and save it in this folder as `Food_Inspections.csv`.
2. Run the notebooks **in this order** (Run All in each):

| Order | Notebook | What it does | Output |
| --- | --- | --- | --- |
| 1 | `Task1_First_Look.ipynb` | First look at the raw data (size, columns, results per year, periods, violations) | `violations_titles_before.csv`, `violations_titles_after.csv` (already in the repo) |
| 2 | `Task2_Cleaning.ipynb` | Cleaning rules, Business_ID, target and period columns | `food_clean.csv` |
| 3 | `Task2_Joud_Violations.ipynb` | Violation features: counts, severity levels, topics, previous visit | `joud_features.csv` |
| 4 | `Task2_Maram_History.ipynb` | History and calendar features | `maram_features.csv` |
| 5 | `Extra.ipynb` | Extra analysis: risk level and location (optional) | – |
| 6 | `Task3_Merge_and_Tests.ipynb` | Merge, quality checks, statistical tests | `food_final.csv` |

`food_final.csv` is the final dataset: **one row per real inspection** (Pass, Pass w/ Conditions or Fail), **2015–2022**, 126,787 rows.

## Cleaning rules (`Task2_Cleaning.ipynb`)

| Rule | What | Rows after |
| --- | --- | --- |
| Start | Raw file | 317,175 |
| 1 | Keep only Pass, Pass w/ Conditions, Fail | 272,286 |
| 2 | Remove exact duplicate rows (none found) | 272,286 |
| 3 | Remove the same license inspected twice on the same date | 271,222 |
| 4 | Remove rows without a License # or with License # = 0 | 270,583 |
| 5 | Missing Facility Type / Risk → "Unknown"; Risk "All" → "Unknown" | 270,583 |
| 6 | `Business_ID` = License # + business name (1,134 of 43,891 licenses had more than one name) | 270,583 |
| 7 | Keep 2015–2022 | 126,787 |
| – | One missing Inspection Type filled as "License Re-Inspection" (same business, two days after a License inspection) | 126,787 |

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
| Inspection Type | Canvass, License, Complaint, Re-Inspection, … | ✅ (after grouping / dummies) |
| Results | Pass, Pass w/ Conditions or Fail | 📊 |
| Violations | Full violation text of the inspection | 📊 |
| First_Seen_Date | Date of the first inspection of the business in the whole dataset (from 2010) | ➖ (see Days_Since_First_Seen) |
| Year | Year of the inspection | ➖ |
| **Fail** | **1 = Fail, 0 = Pass or Pass w/ Conditions** | 🎯 |
| Period | Pre (before 1 Jul 2018), Grace (90 days), Late transition (to 30 Jun 2019), Post | ➖ (used to split train/test) |

### Violation features (`Task2_Joud_Violations.ipynb`)

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

### History and calendar features (`Task2_Maram_History.ipynb`)

| Column | Meaning | Use in model |
| --- | --- | --- |
| Prev_Fail | 1 if the business failed its previous inspection | ✅ |
| First_Inspection | 1 if this is the first inspection of the business in 2015–2022 | ✅ |
| Days_Since_Last | Days since the previous inspection of the business (first inspections: median, 236) | ✅ |
| Past_Fail_Rate | Share of earlier inspections of the business that failed | ✅ |
| Num_Past_Inspections | Number of earlier inspections of the business | ✅ |
| Days_Since_First_Seen | Days since the first inspection of the business since 2010 (approximate age) | ✅ |
| Is_Reinspection | 1 if the inspection type is a re-inspection | ✅ |
| Month, Season | Month (1–12) and season of the inspection | ✅ |
| Facility_Group | 8 most common facility types, the rest "Other" | ✅ (dummies) |
| Risk_Num | Risk as a number (1 High, 2 Medium, 3 Low, 0 Unknown) | ✅ |
| Placebo_Period | Before / after 1 January 2017 (fake change date, for H3) | ➖ (used to split train/test) |

## Limitations

- The old → new crosswalk is an approximation made by matching titles; the lists are not one-to-one.
- The new Priority Foundation level covers many more problems than the old Serious level, so severity columns do not mean exactly the same thing before and after 2018.
- History features are computed from 2015 (Days_Since_First_Seen from 2010), so businesses older than that look newer than they are.
- Violations are allegations that may be dismissed later; the data does not show this.
- 2020 is affected by COVID (fewer inspections).

