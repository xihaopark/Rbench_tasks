# Case2 12: pharmaverse/admiral/derive_param_qtc

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_qtc`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script for an R package task: perform derive param qtc using the admiral package. The function(s) you use to solve the task must come from the admiral package.

Output: result.csv

Computation: Derive only QTCBR rows from paired QT and RR records using QT / sqrt(RR), preserving the expected identifier columns.


Required columns for result.csv: USUBJID, PARAMCD, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVAL
01	QT	400
01	RR	1000
02	QT	360
02	RR	800
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
wide <- reshape(dataset, idvar = "USUBJID", timevar = "PARAMCD", direction = "wide")
result_df <- data.frame(
  USUBJID = wide$USUBJID,
  PARAMCD = "QTCBR",
  AVAL = as.numeric(wide$AVAL.QT) / sqrt(as.numeric(wide$AVAL.RR) / 1000),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(readr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
adat <- read_tsv(
  file = "inputs/dataset.tsv",
  col_types = cols(
    USUBJID = col_character(),
    PARAMCD = col_character(),
    AVAL = col_double()
  )
)

# Derive QTCBR from paired QT and RR using QT / sqrt(RR)
res <- adat %>%
  derive_param_qtcbr(
    by_vars = exprs(USUBJID),
    logbase = 10
  ) %>%
  filter(PARAMCD == "QTCBR") %>%
  select(USUBJID, PARAMCD, AVAL)

# Write result
write_csv(res, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL"
1,"QTCBR",400
2,"QTCBR",402.492235949962
```

### LLM Output
No LLM output artifact was produced.
