# Case 098: pharmaverse/admiral/derive_param_qtc

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_qtc`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`
- Failure pattern: `package_api_hallucination`
- Failure pattern note: could not find function "derivation_add_merged"

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script for an R package task: perform derive param qtc using the admiral package. Use functions from the admiral package when suitable.

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

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
input_df <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    PARAMCD = col_character(),
    AVAL = col_double()
  )
)

# Split QT and RR
qt_df <- input_df %>%
  filter(PARAMCD == "QT") %>%
  rename(AVAL_QT = AVAL)

rr_df <- input_df %>%
  filter(PARAMCD == "RR") %>%
  rename(AVAL_RR = AVAL)

# Join QT and RR by subject
qt_rr <- derivation_add_merged(
  dataset_add = qt_df,
  dataset_ref = rr_df,
  by_vars = exprs(USUBJID),
  new_vars = exprs(AVAL_RR)
)

# Derive QTCBR
qtcbr_df <- qt_rr %>%
  mutate(
    PARAMCD = "QTCBR",
    AVAL = AVAL_QT / sqrt(AVAL_RR)
  ) %>%
  select(USUBJID, PARAMCD, AVAL)

# Write output
write_csv(qtcbr_df, file.path("outputs", "result.csv"))
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
