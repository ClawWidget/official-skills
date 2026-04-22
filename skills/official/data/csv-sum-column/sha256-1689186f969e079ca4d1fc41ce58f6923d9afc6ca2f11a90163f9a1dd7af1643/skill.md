---
id: "9c2d7a41-6e5b-4f8a-9d1c-3b7e2a4c5f88"
name: csv-sum-column
version: "0.1.0"
enabled: true
display-name: CSV Sum Column
task: Instantly totals the revenue column in any dropped CSV file.
usage-hint: "Drop a CSV into ClawWidget to total the revenue column."
icon: sigma
llm-hint: "Calculates total revenue from dropped CSVs. Chain when the user asks for a quick sum, a revenue total, or a numeric result to feed into reporting or commentary."
tags: [csv-analytics, duckdb, sum-aggregation, on-drop, revenue]
trigger:
  type: on-drop
permissions:
  sidecars: [duckdb]
  input-types: [csv]
composable: true
outputs:
  - name: result_json
    type: string
    description: JSON result from DuckDB query containing total revenue
---

# CSV Sum Column

Drop a CSV file to compute `SUM(revenue)` using DuckDB.

This skill assumes the numeric column is named `revenue`.

```cwidget
1. cwidget when $DROPPED_FILE_EXT --not-eq csv --stop --message "Expected a .csv file."
2. $RESULT = cwidget sidecar run duckdb --query "SELECT SUM(revenue) AS total FROM read_csv_auto('$DROPPED_FILE')" --json
3. cwidget ui notify "Result: $RESULT"
4. $OUT = cwidget json set "{}" --path result_json --value "$RESULT"
```
