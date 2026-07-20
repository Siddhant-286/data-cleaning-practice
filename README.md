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

# 📝 Pandas Text Data Cleaning — Project 2

A focused data cleaning project targeting **text/string-specific problems** in pandas — names, emails, phone numbers, addresses, and free-text feedback — separate from numeric or mixed data issues.

## 📌 Why this project

Text data breaks in fundamentally different ways than numeric data — inconsistent casing, formatting conventions, obfuscation, and free-text noise don't respond to the same fixes as missing numbers or out-of-range values. This project isolates those problems to practice the specific tools pandas offers for string cleaning: `.str` accessor methods, `.replace()`, and `.apply()`.

## 📊 Dataset

A synthetic dataset with **146 rows and 15 columns**, built entirely around text-column problems:

| Column | Issues introduced |
|---|---|
| `full_name` | Inconsistent casing, extra whitespace, mixed name order (`"Last, First"` vs `"First Last"`) |
| `email` | Obfuscated formats (`nameATgmailDOTcom`), invalid formats, missing `@domain`, whitespace |
| `phone_number` | Multiple formats (plain digits, `+91-XXXXX-XXXXX`, `(XXX) XXX-XXXX`, dotted), leading zeros |
| `gender` | Inconsistent casing/abbreviations (`Male`, `male`, `M`, `MALE`) |
| `country` | Inconsistent spellings (`India`, `india`, `IN`, `INDIA`) |
| `city` | Inconsistent casing, abbreviations (`Nyc` vs `New York`) |
| `address` | Whitespace, inconsistent casing, abbreviations (`MG Road`) |
| `job_title` | Inconsistent casing, spacing issues |
| `department` | Inconsistent casing/abbreviations (`Hr` vs `Human Resources`) |
| `company_name` | Inconsistent punctuation (`Inc.` vs `Inc`), `&` vs `and` |
| `product_category` | Inconsistent spacing/punctuation (`Home & Kitchen` vs `HomeAndKitchen`) |
| `feedback_text` | Extra whitespace, inconsistent casing, HTML entities (`&amp;`), punctuation noise |
| `status` | Inconsistent casing/abbreviations (`Active`, `active`, `A`) |
| `notes` | Inconsistent placeholder text (`N/A`, `-`, `null`, `none`, empty) |
| — | Exact duplicate rows |

## 🛠️ Cleaning process

1. **Explore** — `.unique()` on every text column to see the real spread of formats before touching anything
2. **Remove duplicates** — exact row duplicates
3. **Normalize casing & whitespace** — `.str.strip()`, `.str.lower()` / `.str.title()` applied consistently
4. **Fix structural signals before removing them** — e.g. using the comma in `"Last, First"` to detect name order *before* stripping it out
5. **Standardize categories** — complete mapping dictionaries for `country`, `department`, `city`, `product_category` (verified with `.unique()` after, not assumed)
6. **Fix data types** — keeping `phone_number` as text/string, never numeric, to preserve leading zeros
7. **Handle missing/placeholder text** — normalizing every placeholder spelling (`N/A`, `null`, `none`, `-`) to one consistent value before filling

### Key techniques used
- `.str.replace()` with regex for pattern-based cleaning inside free text (HTML entities, extra spaces, symbols)
- `.replace()` with mapping dictionaries for whole-value category standardization
- `.apply()` with a custom function for conditional logic that `.replace()`/`.str` methods can't express (name order detection and correction)
- Explicit type control — refusing to let `phone_number` or any identifier-like column silently become numeric

## 📈 Before vs After

| Metric | Before | After |
|---|---|---|
| Rows | 146 | 140 (duplicates removed) |
| Malformed email domains | 14 | 0 |
| Phone numbers as float (`.0` artifacts) | Present | 0 — kept as text |
| Department spelling variants | `Hr`, `Human Resources`, `HR` | 1 clean value |
| Product category spelling variants | `Home & Kitchen`, `HomeAndKitchen`, `home and kitchen` | 1 clean value |
| Missing values (all text columns) | Scattered, multiple placeholder spellings | Standardized to `"Not Provided"` |

## 📂 Project structure

```
pandas-text-data-cleaning/
├── README.md
├── data/
│   ├── raw/messy_text_data.csv
│   └── cleaned/clean_text_data.csv
└── notebooks/
    └── text_data_cleaning.ipynb
```

## 🔧 Tools & Libraries

- Python 3
- pandas
- numpy
- re (regex, via `.str.replace(..., regex=True)`)
- Jupyter Notebook

## 🚀 How to run

```bash
git clone https://github.com/<your-username>/pandas-text-data-cleaning.git
cd pandas-text-data-cleaning
pip install -r requirements.txt
jupyter notebook notebooks/text_data_cleaning.ipynb
```

## 🎯 Key learnings

- `.replace()` and `.str.replace()` are not interchangeable — the former matches whole cell values, the latter matches substrings/patterns within text
- Any transformation step that reads a formatting signal (like a comma indicating name order) must run *before* any step that removes that signal — order of operations is not optional
- `.str.title()` breaks known abbreviations (`"MG Road"` → `"Mg Road"`) and needs a follow-up correction pass
- Category standardization is only complete once verified — a partial mapping dictionary leaves duplicate categories that silently split `groupby()` results
- Identifier-like text (phone numbers, IDs, zip codes) should never be allowed to convert to a numeric dtype — it silently destroys leading zeros
- `.map()` converts any unmatched value to `NaN`, while `.replace()` leaves unmatched values untouched — an important distinction when the mapping might be incomplete

# 🔢 Pandas Numeric Data Cleaning — Project 3

A focused data cleaning project targeting **numeric-specific problems** in pandas — currency formats, scientific notation, unit mismatches, and out-of-range values — separate from text or mixed data issues.

## 📌 Why this project

Numeric data doesn't just go missing or duplicate — it goes wrong in ways that are easy to miss because the result still *looks* like a valid number. A negative age converted with `abs()` becomes a plausible-looking positive number. A height recorded in meters instead of centimeters sits quietly inside a column of otherwise-correct values. This project was built specifically to hide invalid data inside ranges that look reasonable at a glance, to practice catching it properly.

## 📊 Dataset

A synthetic dataset with **147 rows and 15 columns**, built entirely around numeric-column problems:

| Column | Issues introduced |
|---|---|
| `age` | Missing values, negative values, impossible values (200+), non-numeric text |
| `salary` | Mixed currency formats (`$85,000`, `"45k"`), negative values, missing values |
| `height_cm` | **Unit mismatch** — some values recorded in meters (e.g. `1.75`) instead of cm |
| `weight_kg` | Negative values, zero as an invalid placeholder |
| `bmi` | Negative values, impossible extremes |
| `temperature_c` | **Unit mismatch** — some values in Fahrenheit mixed into a Celsius column |
| `blood_pressure` | Compound values (`"120/80"`) mixed with plain numeric rows |
| `heart_rate` | Negative values, impossible extremes |
| `test_score` | Out-of-range values (>100), non-numeric text (`"absent"`) |
| `monthly_expense` | Mixed currency formats (`Rs.`), negative values |
| `account_balance` | Scientific notation as text (`"1.5e5"`), text placeholders (`"closed"`) |
| `discount_percent` | `%` sign attached to numbers, out-of-range values |
| `quantity_ordered` | Zero/negative as invalid placeholders, text values (`"few"`) |
| `rating` | Out-of-range values (>5), negative values, text values (`"good"`, `"5 stars"`) |
| — | Exact duplicate rows |

## 🛠️ Cleaning process

1. **Explore** — `.describe()` and `.dtypes` per column to see real min/max before assuming anything is clean
2. **Remove duplicates** — exact duplicate rows and duplicate IDs
3. **Parse formats** — strip currency symbols, `%` signs, and commas; convert scientific-notation strings and text placeholders to proper numeric types
4. **Fix unit mismatches** — detect and correct values recorded in the wrong unit (meters→cm, Fahrenheit→Celsius) based on realistic thresholds, *before* range-validating
5. **Validate ranges** — flag values outside what's physically/logically possible for each column as `NaN`, rather than transforming them in place
6. **Impute** — fill flagged values using median or realistic random sampling from valid existing values, never a single constant applied blindly

### Key techniques used
- `pd.to_numeric(..., errors='coerce')` combined with pre-cleaning of symbols (`$`, `,`, `%`) so valid numbers aren't lost as `NaN`
- Threshold-based unit correction (e.g. `df.loc[df['height_cm'] < 10, 'height_cm'] *= 100`) instead of assuming every row uses the same unit
- Masking invalid values to `NaN` *before* imputing — explicitly avoiding `.abs()` on negative values, since it converts bad data into fake-valid data without any indication that a fix occurred
- Random-sample imputation (`np.random.choice` from valid existing values) instead of a single mean/median fill, to preserve a realistic distribution across large numbers of invalid rows

## 📈 Before vs After

| Metric | Before | After |
|---|---|---|
| Rows | 147 | 140 (duplicates removed) |
| Invalid `rating` values (outside 1–5) | 17 | 0 |
| Invalid `test_score` values (>100) | 25 | 0 |
| `height_cm` unit mismatches (meters vs cm) | 58 rows | Corrected |
| `temperature_c` unit mismatches (F vs C) | 84 rows | Corrected |
| Negative values across numeric columns | Present in 8 columns | Flagged and resolved |

## 📂 Project structure

```
pandas-numeric-data-cleaning/
├── README.md
├── data/
│   ├── raw/messy_numeric_data.csv
│   └── cleaned/clean_numeric_data.csv
└── notebooks/
    └── numeric_data_cleaning.ipynb
```

## 🔧 Tools & Libraries

- Python 3
- pandas
- numpy
- Jupyter Notebook

## 🚀 How to run

```bash
git clone https://github.com/<your-username>/pandas-numeric-data-cleaning.git
cd pandas-numeric-data-cleaning
pip install -r requirements.txt
jupyter notebook notebooks/numeric_data_cleaning.ipynb
```

## 🎯 Key learnings

- A number that *looks* valid isn't the same as a number that *is* valid — `abs(-5)` on a 1–5 rating scale produces `5`, which is indistinguishable from a genuinely correct value once transformed
- The right fix order is: mask invalid values to `NaN` first, impute second — never transform bad data in place, since it removes any trace that a correction happened
- Unit mismatches don't throw errors and don't show up as missing data — they hide inside otherwise-normal-looking ranges and are only caught by checking `.describe()` distributions carefully, not just `.isnull().sum()`
- Range validity depends on the dataset's real-world context (e.g. `18–100` makes sense for an employee age column, but not for a general-population dataset that includes children) — there's no universal "valid range," only physically impossible values (negative ages, 200+ years) that are invalid in any context
- A single mean/constant fill across a large number of invalid values destroys the column's real distribution — sampling from existing valid values preserves more realistic variation

---
*This is a practice project for learning numeric-specific data cleaning workflows in pandas.*
---
*This is a practice project for learning text-specific data cleaning workflows in pandas.*
