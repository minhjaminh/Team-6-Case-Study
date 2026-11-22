# SBA loan risk — data cleaning and preparation

This is the cleaning part of our SBA loan case study. I used the original `SBAnational.csv` to check for missing data, fix inconsistent formatting, and prepare files for the rest of the team. The idea is to do the cleaning once, keep a record of what changed, and give everyone the same starting point.

The project looks at whether a loan was **paid in full** (`PIF`) or **charged off** (`CHGOFF`). In the modeling file, charged off is `1` and paid in full is `0`. If a loan has no recorded status, I leave it out of the model instead of guessing what happened.

## Running the notebook

1. Add the notebook and support files to our **existing GitHub repo**. Keep the folder names as they are.
2. Put `SBAnational.csv` in `data/raw/` on your computer. Leave the original file unchanged. It's too large to upload to GitHub using normal Git, so share it separately with the team.
3. Open the repo in VS Code or Jupyter and install the packages with `python -m pip install -r requirements.txt`.
4. Open `notebooks/01_SBA_Data_Cleaning_and_Preparation.ipynb` and run all cells. If your CSV is somewhere else, change `RAW_CSV_OVERRIDE` near the top of the notebook.
5. Check the output files and `outputs/cleaning_audit/` for the counts, warnings, and cleaning notes.

## What the notebook produces

| File | What it's for |
| --- | --- |
| `data/processed/sba_eda_ready.csv` | The EDA/financial-analysis file. It keeps all original loan records and includes historical information such as disbursement and outcomes. **Don't use it directly to train the approval-time model.** |
| `data/processed/sba_model_candidates.csv` | The modeling file. It includes only loans with known outcomes and a selected set of possible predictors available around approval time. |
| `outputs/cleaning_audit/01_original_missingness.csv` | Missing-value counts for every original column. |
| `outputs/cleaning_audit/02_codes_*.csv` | Values found in category columns, including unusual codes worth checking. |
| `outputs/cleaning_audit/03_quality_checks.csv` | Counts of questionable records, such as zero loan terms and unknown industry codes. |
| `outputs/cleaning_audit/04_status_counts.csv` and `05_reconciliation.csv` | Outcome counts and checks that the row totals add up. |
| `outputs/cleaning_audit/06_model_feature_contract.json` | The exact list of columns the modeling teammate should use, plus fields to avoid. |
| `outputs/cleaning_audit/07_data_dictionary.csv` and `08_handoff_summary.json` | Brief descriptions of the cleaned columns and a summary of the outputs. |

The two processed CSVs are big, so I kept them out of Git using `.gitignore`. Share those with the teammates who need them. The notebook, README, and smaller audit files can go in the repo.

## Main cleaning decisions

I tried not to throw out data without a reason. A missing value or unusual number doesn't automatically mean the whole loan record is bad.

- **Missing loan status:** These loans stay in the EDA file but are excluded from the model, since we don't know if they repaid or defaulted.
- **Dollar amounts and dates:** Converted to consistent formats so we can calculate and compare them properly. The notebook also checks for odd dates and approval-year labels.
- **Unknown codes:** `NAICS = 0` is treated as an unknown industry. Unclear `LowDoc` and `RevLineCr` codes are kept as unknown rather than guessed to be yes/no.
- **Outliers:** Flagged for review instead of deleting every large or unusual value.
- **Information the bank wouldn't know yet:** Charge-off amounts, charge-off dates, and disbursement information are not included as predictors in the approval-time modeling file. Using them would give the model an unfair hint about the outcome. This is called **data leakage**.

One thing for the modeling teammate to double-check: `GrAppv`, `SBA_Appv`, and the guarantee share are included only if we're predicting risk **after the approved loan terms are known**. If the bank is deciding on an application before setting those terms, take these features out too.

## Passing the data to the rest of the team

**For EDA:** Use `sba_eda_ready.csv` for charts and comparisons. When calculating default percentages, count only loans with known repayment outcomes and show how many loans are in each group.

**For logistic regression:** Use `sba_model_candidates.csv`. The target is `Default` (`1` = charged off, `0` = paid in full). The feature list is saved in `06_model_feature_contract.json`; the ID and approval year are there for tracking and checks, not as regular model inputs.

Here's the quick starting point:

```python
import json
from pathlib import Path
import pandas as pd

model = pd.read_csv('data/processed/sba_model_candidates.csv')
contract = json.loads(Path('outputs/cleaning_audit/06_model_feature_contract.json').read_text())
X = model[contract['features']]
y = model[contract['target']]

# Split into train and test sets before fitting missing-value
# replacements, scaling, or one-hot encoding.
```

**For financial analysis:** Use the EDA file for the historical loan information and apply the case study's simplified assumptions separately. The cleaning notebook doesn't calculate or claim actual bank profit.

This dataset only contains historical approved SBA-backed loans, so even a good model can't tell us exactly what would have happened to applications the bank denied.
