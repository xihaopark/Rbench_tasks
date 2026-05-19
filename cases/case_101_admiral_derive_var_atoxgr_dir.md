# Case 101: pharmaverse/admiral/derive_var_atoxgr_dir

## Metadata
- Task ID: `pharmaverse/admiral/derive_var_atoxgr_dir`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: ATOXGRDIR.
```

## Prompt
```text
Write an R script to perform derive var atoxgr dir using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVAL, ANRLO, ANRHI, ATOXGRDIR
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVAL	ANRLO	ANRHI
01	ALT	15	20	40
01	AST	55	10	35
02	BILI	0.8	0.2	1.2
03	CREAT	1.5	0.5	1.1
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df$ATOXGRDIR <- ifelse(
  result_df$AVAL < result_df$ANRLO,
  "L",
  ifelse(result_df$AVAL > result_df$ANRHI, "H", "N")
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

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input
adat <- read.delim(
  file = file.path("inputs", "dataset.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE
)

# Helper: derive directionality per CDISC oncology convention
derive_atoxgr_dir <- function(paramcd, aval, anrlo, anrhi) {
  dir <- dplyr::case_when(
    is.na(aval) | is.na(anrlo) | is.na(anrhi) ~ NA_character_,
    aval < anrlo ~ "L",
    aval > anrhi ~ "H",
    TRUE ~ ""
  )
  dir
}

result <- adat %>%
  mutate(
    ATOXGRDIR = derive_atoxgr_dir(PARAMCD, AVAL, ANRLO, ANRHI)
  ) %>%
  select(USUBJID, PARAMCD, AVAL, ANRLO, ANRHI, ATOXGRDIR)

# Write output
write.csv(
  result,
  file = file.path("outputs", "result.csv"),
  row.names = FALSE,
  na = ""
)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVAL","ANRLO","ANRHI","ATOXGRDIR"
1,"ALT",15,20,40,"L"
1,"AST",55,10,35,"H"
2,"BILI",0.8,0.2,1.2,"N"
3,"CREAT",1.5,0.5,1.1,"H"
```

### LLM Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVAL","ANRLO","ANRHI","ATOXGRDIR"
1,"ALT",15,20,40,"L"
1,"AST",55,10,35,"H"
2,"BILI",0.8,0.2,1.2,""
3,"CREAT",1.5,0.5,1.1,"H"
```
