# 1_attempt.md — Diagnosis before AI

> Complete/personalize this section before submitting. This is an evidence-based draft prepared from an initial inspection of the supplied CSV, not a claim that the student independently wrote it before using AI. If your instructor requires a genuine timestamped attempt-before-AI, replace this note with your own original notes.

## Dataset description
The file contains 97 rows and 6 columns: `date`, `item`, `city`, `price`, `unit`, and `data_label`. The columns were initially read as text because the `price` column contains a missing value and a comma-formatted value (`1,250`). The data covers weekly dates from January to March 2026, four teaching items, and two cities.

## Problems observed and planned decisions

| Evidence | Diagnosis | Planned decision | Reason |
|---|---|---|---|
| `price` is blank for Pulse C, City 2 on 2026-01-01 | Missing value | Retain row; flag missing price | No source evidence supports an imputed value |
| Price appears as `1,250` | Comma-formatted numeric text | Remove comma for numeric parsing; retain value 1250 and flag | Formatting can be fixed, but the magnitude still needs investigation |
| Price `9999` for Sweetener D, City 1 on 2026-01-22 | Extreme value relative to other observations | Retain and flag | It may be a typo, but the correct value cannot be inferred safely |
| Item is `oil b` on 2026-02-05 | Inconsistent capitalization | Standardize to `Oil B`; preserve original in audit output if needed | Case difference appears to refer to the same item |
| The 2026-01-08 Staple A, City 1 row occurs twice | Exact duplicate | Remove one exact duplicate | All six fields match; keeping both would double-count the same record |
| Dates, item names, city labels and unit strings | Need type/consistency checks | Parse date; trim text; inspect categories and units | Helps identify malformed values without changing meaning |

## Initial evidence
- Original row count: **97**
- Original columns: **6**
- Exact duplicate rows (excluding first occurrence): **1**
- Missing `price` values: **1**
- Price values parseable after removing commas: **96**
- Numeric price minimum: **99.5**
- Numeric price median: **200.0**
- Numeric price maximum: **9999**
- Values requiring special review: **1250** and **9999**

## Planned cleaning sequence
1. Load a separate working copy; never overwrite the source.
2. Check shape, data types, missing values, duplicates, categories and numeric values.
3. Remove only the exact duplicate row, documenting the count.
4. Trim text and standardize the single `oil b` spelling/capitalization.
5. Convert `price` to numeric after removing thousands separators.
6. Keep missing and suspicious prices; add flags rather than silently deleting or imputing them.
7. Compare before-and-after counts and inspect the resulting rows.
