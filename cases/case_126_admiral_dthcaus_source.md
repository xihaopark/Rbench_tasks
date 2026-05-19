# Case 126: pharmaverse/admiral/dthcaus_source

## Metadata
- Task ID: `pharmaverse/admiral/dthcaus_source`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(2, 8) vs llm=(3, 8).
```

## Prompt
```text
Write an R script to perform dthcaus source using the admiral clinical task contract.

Input: ds.tsv
Output: result.csv


Required columns for result.csv: dataset_name, mode, filter_expr, date_expr, dthcaus_expr, USUBJID, DSSTDTC, DSTERM
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `ds.tsv`


```text
USUBJID	DSDECOD	DSTERM	DSSTDTC
01	DEATH	Disease progression	2022-02-01
02	COMPLETED	Completed study	2022-01-15
03	DEATH	Adverse event	2022-03-03
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

ds <- read.delim(file.path("inputs", "ds.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
src <- suppressWarnings(admiral::dthcaus_source(
  dataset_name = "ds",
  filter = DSDECOD == "DEATH",
  date = admiral::convert_dtc_to_dt(DSSTDTC),
  mode = "first",
  dthcaus = DSTERM
))

death_rows <- ds[ds$DSDECOD == "DEATH", , drop = FALSE]
result <- data.frame(
  dataset_name = src$dataset_name,
  mode = src$mode,
  filter_expr = paste(deparse(src$filter), collapse = " "),
  date_expr = paste(deparse(src$date), collapse = " "),
  dthcaus_expr = paste(deparse(src$dthcaus), collapse = " "),
  USUBJID = death_rows$USUBJID,
  DSSTDTC = death_rows$DSSTDTC,
  DSTERM = death_rows$DSTERM,
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
ds <- read_tsv(
  file = file.path("inputs", "ds.tsv"),
  col_types = cols(.default = col_character())
)

# Prepare result
result <- ds %>%
  transmute(
    dataset_name = "DS",
    mode         = "source",
    filter_expr  = "DSDECOD == 'DEATH'",
    date_expr    = "DSSTDTC",
    dthcaus_expr = "DSTERM",
    USUBJID,
    DSSTDTC,
    DSTERM
  )

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dataset_name","mode","filter_expr","date_expr","dthcaus_expr","USUBJID","DSSTDTC","DSTERM"
"ds","first","DSDECOD == ""DEATH""","admiral::convert_dtc_to_dt(DSSTDTC)","DSTERM",1,"2022-02-01","Disease progression"
"ds","first","DSDECOD == ""DEATH""","admiral::convert_dtc_to_dt(DSSTDTC)","DSTERM",3,"2022-03-03","Adverse event"
```

### LLM Output
#### `result.csv`


```csv
dataset_name,mode,filter_expr,date_expr,dthcaus_expr,USUBJID,DSSTDTC,DSTERM
DS,source,DSDECOD == 'DEATH',DSSTDTC,DSTERM,01,2022-02-01,Disease progression
DS,source,DSDECOD == 'DEATH',DSSTDTC,DSTERM,02,2022-01-15,Completed study
DS,source,DSDECOD == 'DEATH',DSSTDTC,DSTERM,03,2022-03-03,Adverse event
```
