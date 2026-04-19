---
id: "f7b2a3d1-8e4c-4f9a-9d6b-2c1e5a7f8b90"
name: csv-rolling-average
version: "0.1.0"
enabled: true
display-name: CSV Rolling Average
task: Drop a CSV file and compute a 7-row rolling average of a revenue column locally with DuckDB.
icon: chart-line
llm-hint: "Fires on file drop. Uses DuckDB to compute a 7-row rolling average for a `revenue` column from a dropped CSV via read_csv_auto('$DROPPED_FILE'). Returns {result_json} with revenue and rolling_avg_7 rows in input order. Local-only, no network."
tags: [csv-analytics, duckdb, rolling-average, on-drop, window-function]
trigger:
  type: on-drop
permissions:
  sidecars: [duckdb]
  input-types: [csv]
composable: true
outputs:
  - name: result_json
    type: string
    description: JSON rows with original revenue and rolling average
---

# CSV Rolling Average

Drop a CSV file to compute a 7-row rolling average for `revenue` using DuckDB.

This skill assumes the CSV has a numeric column named `revenue`.

```cwidget
1. cwidget when $DROPPED_FILE_EXT --not-eq csv --stop --message "Expected a .csv file."
2. $RESULT = cwidget sidecar run duckdb --query "WITH rows AS (SELECT *, row_number() OVER () AS rn FROM read_csv_auto('$DROPPED_FILE')) SELECT revenue, AVG(revenue) OVER (ORDER BY rn ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS rolling_avg_7 FROM rows ORDER BY rn" --json
3. cwidget ui notify "Rolling average computed."
4. $OUT = cwidget json set "{}" --path result_json --value "$RESULT"
```
