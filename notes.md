# SQL Notes

## Day 1 — Tue 17 Nov 2026 (Kaggle Intro to SQL, lessons 1–4)

### Basics
- Basic shape: `SELECT columns FROM table WHERE condition`
- Separate columns with commas. `SELECT name city` (no comma) does NOT error: SQL reads it as `name AS city`, so it returns one column with the wrong label. Silent bug.
- **Quotes = a value, no quotes = a column/table name.**
  - `WHERE city = 'Cairo'` compares to the text Cairo
  - `WHERE city = Cairo` looks for a column called Cairo, which usually gives an error
- Backticks in BigQuery are for names: `` `bigquery-public-data.pet_records.pets` ``
- Text comparison is case-sensitive: `'cairo'` ≠ `'Cairo'`. Check stored values first with `SELECT DISTINCT city FROM ...`
- TRUE/FALSE columns (BOOLEAN): write `WHERE deleted = TRUE`, not `'True'`. With quotes it's text, and BigQuery refuses to compare a BOOLEAN to text.
- Python side: variable names can't have hyphens (`test-query` ✗, `test_query` ✓)

### BigQuery cost
- BigQuery charges by **data scanned**, not rows returned.
- `LIMIT 10` does NOT make a query cheaper. It still scans everything you selected.
- Selecting fewer columns IS cheaper (data is stored column by column). Avoid `SELECT *` on big tables.
- `maximum_bytes_billed` cancels a query before it runs if it would scan too much. A failed query costs nothing, so experimenting is safe.
- `10**10` bytes = 10 GB (Kaggle's comment says 1 GB, which is wrong: read the code, not the comment).

### GROUP BY
- `GROUP BY` splits rows into piles (one per distinct value). Aggregates like `COUNT`, `SUM`, `AVG`, `MAX` then run **once per pile**, and each pile becomes one output row.
- Without GROUP BY, the whole table is one pile, so `COUNT(ID)` gives one number.
- The column in SELECT (e.g. `Animal`) only **labels** the piles. GROUP BY does the splitting.
- **Rule:** every column in SELECT must be either in GROUP BY or inside an aggregate.
  - `SELECT Animal, Name, COUNT(ID) ... GROUP BY Animal` is invalid. SQL checks the rule, not the data.
- Row order is not guaranteed unless you use `ORDER BY`.

### WHERE vs HAVING
- **WHERE filters ROWS, before grouping.** It looks at one row at a time.
- **HAVING filters GROUPS, after grouping.** It can use `COUNT`, `AVG`, `SUM`...
- Test: *Can I decide by looking at one row alone?* Yes → WHERE. Need the whole group? → HAVING.
- It is NOT about numbers vs text:
  - `WHERE salary > 50000` ✅ (one employee = one row)
  - `HAVING AVG(salary) > 50000` ✅ (one department = one group)
  - `WHERE AVG(salary) > 50000` ❌ (groups don't exist yet when WHERE runs)
- WHERE can filter on columns that are NOT in SELECT (e.g. `WHERE year = 2016` without selecting year).

### Order: written vs run

| Written | Run by SQL |
|---|---|
| SELECT | FROM |
| FROM | WHERE |
| WHERE | GROUP BY |
| GROUP BY | HAVING |
| HAVING | SELECT |
| ORDER BY | ORDER BY |

- HAVING must be written after GROUP BY (you can't filter groups before making them).

### Example: both filters together
```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE city = 'Cairo'          -- rows
GROUP BY department
HAVING AVG(salary) > 50000    -- groups
ORDER BY avg_salary DESC
```

### Ranges and ORDER BY
- `year BETWEEN 2010 AND 2017` = `year >= 2010 AND year <= 2017` (both ends included).
- A range has two ends; missing one silently lets the wrong rows in.
- `ORDER BY col DESC` = largest first. `ASC` (default) = smallest first.
- `AND` = must meet both conditions. `OR` = needs only one.

### Debugging habits
- **Always look at the output and ask: does this make sense?** Queries can run fine and still be wrong.
- If an aggregate looks wrong, swap it for the raw rows with the same WHERE + `LIMIT 20` and inspect.
- Read the **last line** of an error message first.
- Red box ≠ always an error. A *warning* doesn't stop the code; an *error* does.
- pandas scientific notation: `1.18e+01` = 11.8, `9.32e+00` = 9.32, `1.86e+06` = 1,860,000.
- World Bank "countries" include regions and groups (Arab World, World, Low income). Watch for this when averaging.

---

## Recall check (answer from memory before your next session)
1. Why does `SELECT name city` not throw an error?
2. What does SQL think `Cairo` is without quotes?
3. Does `LIMIT 10` make a BigQuery query cheaper? Why?
4. Write the running order of a query from memory.
5. WHERE or HAVING: "only countries with more than 50 rows"?
6. Write the Cairo department query above from memory.

## Still open
- Exam Q2: What does a LEFT JOIN return when there's no match on the right?
- Exam Q3: What does `df.groupby('country')['salary'].median()` return, and what type is it?
