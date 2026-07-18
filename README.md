# 🧹 Pandas Data Cleaning Practice
## project -01

A hands-on project where I practiced real-world data cleaning techniques in pandas by intentionally working with a messy, synthetic HR/employee dataset — covering missing values, invalid ranges, inconsistent categories, mixed date formats, and duplicate records.

## 📌 Why this project

Data cleaning is one of the most underrated but critical skills in data analysis — most real-world datasets aren't analysis-ready out of the box. This project was built to practice diagnosing and fixing common data quality issues systematically, using a dataset that mimics the kind of messiness you actually encounter in real HR/employee records.

## 📊 Dataset

A synthetic employee dataset with **178 rows and 12 columns**, deliberately seeded with common real-world data problems:

| Column | Issues introduced |
|---|---|
| `full_name` | Inconsistent casing, leading/trailing whitespace |
| `age` | Missing values, non-numeric text (`"twenty five"`), out-of-range values (`-5`, `200`) |
| `gender` | Inconsistent casing/abbreviations (`Male`, `male`, `M`) |
| `country` | Inconsistent spellings (`US`, `USA`, `United States`) |
| `salary` | Mixed formats (`$85,000`, `"45k"`, plain numbers), negative values, missing values |
| `join_date` | 5 different date formats mixed in one column, plus text placeholders |
| `department` | Inconsistent casing and abbreviations (`IT`, `I.T.`, `it`) |
| `email` | Invalid formats, missing `@domain`, whitespace |
| `performance_rating` | Out-of-range values (`11`, `-2`), non-numeric text (`"five"`) |
| `years_experience` | Negative values, missing values |
| `is_active` | Inconsistent boolean representations (`Yes`/`Y`/`TRUE`/`1`) |
| — | Exact duplicate rows and duplicate IDs with reformatted names |

## 🛠️ Cleaning process

The cleaning follows a structured 6-stage pipeline:

1. **Explore** — `.info()`, `.describe()`, `.isnull().sum()` to understand what's broken
2. **Fix structure** — standardize column names, drop redundant columns
3. **Handle missing values** — distinguish real missingness from placeholder text before imputing
4. **Remove duplicates** — exact row duplicates and duplicate IDs
5. **Fix types & formatting** — convert strings to numeric/datetime, standardize categorical text
6. **Validate** — enforce logical ranges (e.g. age 18–100, rating 1–5) after types are correct, not before

### Key techniques used
- Multi-format date parsing with an explicit fallback loop (`pd.to_datetime` per format, applied only to still-unparsed rows) instead of relying on a single ambiguous auto-parse
- Category standardization via mapping dictionaries (`country`, `department`) instead of ad-hoc `.replace()` calls
- Random-sample imputation for invalid/missing numeric values (`np.random.choice` from valid existing values) instead of a single mean/median fill, to preserve realistic distribution
- Careful ordering: invalid values are converted to `NaN` first, then imputed — never transformed in place (e.g. avoiding `abs()` on negative values, which silently turns invalid data into fake-valid data)

## 📈 Before vs After

| Metric | Before | After |
|---|---|---|
| Rows | 178 | 165 (duplicates removed) |
| Missing values (`age`) | 22 | 0 |
| Invalid ages (<18 or >100) | 96 | 0 |
| Duplicate rows | 9 | 0 |
| Date formats in `join_date` | 5 mixed formats + text | Single standardized `datetime64` |
| Country spelling variants | 9+ | 6 clean values |

## 📂 Project structure

```
pandas-data-cleaning-practice/
├── README.md
├── requirements.txt
├── data/
│   ├── raw/messy_employee_data.csv
│   └── cleaned/clean_employee_data.csv
└── notebooks/
    └── data_cleaning.ipynb
```

## 🔧 Tools & Libraries

- Python 3
- pandas
- numpy
- Jupyter Notebook

## 🚀 How to run

```bash
git clone https://github.com/<your-username>/pandas-data-cleaning-practice.git
cd pandas-data-cleaning-practice
pip install -r requirements.txt
jupyter notebook notebooks/data_cleaning.ipynb
```

## 🎯 Key learnings

- Missing-value handling isn't one-size-fits-all — the right strategy depends on how much of a column is affected and what the data will be used for
- Transforming invalid values (e.g. `abs()` on negatives) can silently create fake-valid data that's harder to catch than an obvious `NaN`
- Multi-format date columns need explicit, ordered format matching — a single auto-parse call is a common source of silent data loss
- Category standardization needs a complete mapping, not partial fixes, or downstream `groupby()`/aggregation will silently split the same category into multiple buckets

---
*This is a practice project for learning data cleaning workflows in pandas.*
