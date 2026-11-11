# 3_log_reflection.md — Verification log and reflection

## Before/after evidence

| Check | Before cleaning | After cleaning |
|---|---:|---:|
| Rows | 97 | 96 |
| Columns | 6 | 6 source columns plus audit columns in working output |
| Exact duplicate rows | 1 | 0 |
| Missing prices | 1 | 1 |
| Numeric prices | 96 parseable after comma removal | 95 numeric; 1 missing |
| Price minimum (numeric) | 99.5 | 99.5 |
| Price median (numeric) | 200.0 | 200.0 |
| Price maximum (numeric) | 9999 | 9999 |

The row count falls by exactly one because one exact duplicate is removed. The missing price is deliberately retained. The two unusual numeric values (1250 and 9999) are retained and flagged, so the maximum remains 9999. The comma in `1,250` is a formatting issue; parsing it gives 1250, but parsing does not prove the value is correct.

## AI cleaning code: line-by-line court

The following is the proposed AI code under review. It is intentionally conservative: it creates flags and does not automatically delete or replace suspicious values.

```python
import pandas as pd
df = pd.read_csv("teaching_prices_messy.csv")
df["date"] = pd.to_datetime(df["date"], errors="coerce")
df["item_original"] = df["item"]
df["item"] = df["item"].str.strip().str.replace(r"(?i)^oil b$", "Oil B", regex=True)
df["price_original"] = df["price"]
df["price"] = pd.to_numeric(df["price"].astype("string").str.replace(",", "", regex=False), errors="coerce")
df["is_duplicate"] = df.duplicated(keep="first")
df = df.loc[~df["is_duplicate"]].copy()
df["missing_price"] = df["price"].isna()
df["review_price"] = df["price"].gt(500)
```

| Line | Verdict | Evidence / reasoning |
|---|---|---|
| `import pandas as pd` | KEEP | Pandas is used for loading and cleaning tabular data |
| `df = pd.read_csv(...)` | MODIFY | Use the original file path and read a separate working copy; preserve source file |
| Convert date with `errors="coerce"` | KEEP | Makes date a datetime type and exposes invalid dates as missing; inspect any coerced values |
| Save `item_original` | KEEP | Preserves evidence before standardization |
| Trim and standardize `oil b` | KEEP | Only one inconsistent `oil b` spelling is observed; the original is preserved |
| Save `price_original` | KEEP | Maintains the original text representation for audit |
| Remove comma and convert price | KEEP | Converts `1,250` to numeric 1250; invalid strings become missing and must be reviewed |
| Create duplicate flag | KEEP | Makes duplicate detection visible before removal |
| Remove flagged duplicate | MODIFY | Safe here only after confirming the duplicate is identical across all six original fields; document one row removed |
| Create missing-price flag | KEEP | Makes the missing value explicit without inventing a replacement |
| Flag prices greater than 500 | MODIFY | This is a review threshold based on the visible pattern, not an official price rule. Treat as a manual review flag, not proof of error or grounds for deletion |

## Additional verification
- Check that the exact duplicate count is zero after removal.
- Check that the row count is 96.
- Check that one missing price remains.
- Check that `oil b` has been standardized to `Oil B`.
- Check that 1250 and 9999 remain present and are flagged for review.
- Check that no source columns were silently dropped.

## Reflection
My main difference from an aggressive automated cleaning approach is that I would not replace the missing price with a mean or median, and I would not delete 1250 or 9999 just because they look unusual. The dataset does not provide an independent source that proves what those prices should be. I would rather preserve the original observations, attach review flags, and ask the data owner to verify them.

The duplicate is different: every field in the repeated row matches the earlier row, so removing one copy is supported by direct evidence. Standardizing `oil b` to `Oil B` is also supported because it is the only capitalization variation and the original text is retained.

AI helped organize repeatable checks and conversion steps. However, a line of code can be syntactically correct while its cleaning decision is not justified. For example, a threshold such as 500 is only a screening rule here; it cannot prove a value is wrong. Without manual inspection, I might have missed that `1,250` is both a formatting issue and a suspicious magnitude, or I might have mistakenly deleted valid-but-unusual data.

## Limitations
The supplied file is illustrative teaching data, not actual PBS observations. The file alone cannot establish the correct values for the missing price, 1250 or 9999. These remain unresolved pending confirmation from the data owner.
