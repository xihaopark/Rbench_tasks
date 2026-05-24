# Case 098: pharmaverse/admiral/derive_param_qtc

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_qtc`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: PARAMCD.
```

## Prompt
```text
Write an R script to perform derive param qtc using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


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

library(dplyr)
library(readr)
library(stringr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input
adsl <- read.delim(
  file = file.path("inputs", "dataset.tsv"),
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE
)

# Ensure correct column types
adsl <- adsl %>%
  mutate(
    USUBJID = as.character(USUBJID),
    PARAMCD = as.character(PARAMCD),
    AVAL = as.numeric(AVAL)
  )

# Pivot-like reshape to get QT and RR per subject
qt_rr <- adsl %>%
  filter(PARAMCD %in% c("QT", "RR")) %>%
  select(USUBJID, PARAMCD, AVAL) %>%
  tidyr::pivot_wider(
    names_from = PARAMCD,
    values_from = AVAL
  )

# Derive QTc (Fridericia) as example: QTcF = QT / (RR/1000)^(1/3)
# RR is in ms; convert to seconds in formula
result <- qt_rr %>%
  mutate(
    AVAL = ifelse(!is.na(QT) & !is.na(RR),
                  QT / ( (RR / 1000)^(1/3) ),
                  NA_real_)
  ) %>%
  transmute(
    USUBJID,
    PARAMCD = "QTCF",
    AVAL
  )

# Write output
write_csv(
  result,
  file.path("outputs", "result.csv"),
  na = ""
)
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
#### `result.csv`


```csv
USUBJID,PARAMCD,AVAL
1,QTCF,400
2,QTCF,387.798244205739
```
