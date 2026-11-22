# SBA loan case study — cleaning and prep

This folder has the cleaning work for our SBA loan project. I used the original `SBAnational.csv` to check for missing values, clean up dates and dollar amounts, and make two files the rest of the team can work with.

Nothing gets changed in the original CSV. The notebook reads it, cleans a copy, and saves the results.

## Where everything goes

```text
notebooks/
  01_SBA_Data_Cleaning_and_Preparation.ipynb
data/
  raw/
    SBAnational.csv                   # download separately
  processed/
    sba_eda_ready.csv                # created by the notebook
    sba_model_candidates.csv         # created by the notebook
outputs/
  cleaning_audit/                     # small checks and notes
README_CLEANING.md
requirements.txt
.gitignore
```

The large CSVs aren't included in GitHub by default. They're ignored on purpose so we don't run into the upload limits. We'll need to share the raw file and processed files through our team Drive or Git LFS.

## How to run it

1. Clone or download our repo and open the folder in **VS Code** or **Jupyter**.
2. Put `SBAnational.csv` inside `data/raw/`. If you already have it elsewhere, you can change `RAW_CSV_OVERRIDE` near the top of the notebook instead.
3. Open a terminal in the repo folder and run:

   ```bash
   python -m pip install -r requirements.txt
   ```

4. Open `notebooks/01_SBA_Data_Cleaning_and_Preparation.ipynb`, select your Python kernel, and **Run All**.
5. The notebook should save the cleaned files in `data/processed/` and the checks in `outputs/cleaning_audit/`.

If you get a file-not-found error, double-check the CSV location first. You don't need to edit any other file paths if you're using the folder layout above.

## Which file should you use?

| File | Who it's for | What it includes |
| --- | --- | --- |
| `sba_eda_ready.csv` | EDA and financial analysis | All the original loan rows with cleaned values and historical loan details |
| `sba_model_candidates.csv` | Logistic regression | Loans with a known repayment outcome and a smaller set of possible approval-time inputs |
| `06_model_feature_contract.json` | Modeling teammate | List of columns to use as features, plus columns to leave out |
| `01_original_missingness.csv` | Anyone checking the data | Missing-value counts by column |
| `03_quality_checks.csv` | Anyone checking the data | Counts of unusual values we flagged |
| `05_reconciliation.csv` | Everyone | Confirms how the raw, EDA, and model row counts match up |

There are a few other small files in `outputs/cleaning_audit/`, including category counts and a basic data dictionary. They're just there so we have a record of what the cleaning did.

## What I cleaned

- **Missing outcomes:** Loans without `PIF` or `CHGOFF` stay in the EDA file, but they don't go into model training. We can't label them without knowing what happened.
- **Dollar amounts and dates:** Converted into consistent formats. I also checked approval dates against the federal fiscal year, since October–December dates can roll into the next fiscal year.
- **Industry and other codes:** A zero or missing `NAICS` becomes **Unknown**. Unclear `LowDoc` and `RevLineCr` values also stay **Unknown** instead of being guessed.
- **Unusual values:** Zero loan terms and other questionable entries are flagged in the audit files. I didn't delete all of them automatically.
- **Model inputs:** Charge-off dates, charge-off amounts, and other information from after approval aren't used as predictors.

From the original file, I counted **899,164 loans**. Of those, **897,167 have known outcomes** and **1,997 have missing outcomes**. The notebook checks these totals again every time it runs.

## A couple notes for the next steps

**For EDA:** When you compare default rates, only use loans with a known outcome in the percentage. Show the group sizes too so a small group doesn't look more important than it is.

**For the model:** `Default` is the target, where `1` means charged off and `0` means paid in full. Use the feature list in `06_model_feature_contract.json`, not every column in the EDA file. Fit missing-value handling, encoding, and scaling **after** the train/test split.

The model file includes `GrAppv`, `SBA_Appv`, and the guarantee share under the assumption that we're predicting after the loan terms have been approved. If we're trying to predict **before** those amounts are set, the modeling teammate should remove those inputs.

**For the financial analysis:** The case study uses **+5% of the disbursed amount** for loans repaid and **−25%** for loans charged off. Those are assignment assumptions, not real profit margins.

Also, the data only includes historically approved SBA-backed loans. We don't know what would have happened to applicants who were denied.

## Sharing this in our repo

You can commit the notebook, this README, and the small audit files. The two processed CSVs and the original CSV are too big for a normal GitHub web upload. Share them separately through the team Drive or set up Git LFS if we decide we want the data stored with the repo.

If you change the cleaning code, rerun the notebook and check the audit totals before passing new files to the group. That way we're all using the same version.
