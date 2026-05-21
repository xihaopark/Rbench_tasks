# Case 101: pharmaverse/admiral/derive_var_atoxgr_dir

## Metadata
- Task ID: `pharmaverse/admiral/derive_var_atoxgr_dir`
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
Write an R script for an R package task: perform derive var atoxgr dir using the admiral package. The function(s) you use to solve the task must come from the admiral package.

Output: result.csv

Computation: Set ATOXGRDIR to L when AVAL < ANRLO, H when AVAL > ANRHI, and N otherwise.


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

library(admiral)
library(dplyr)
library(readr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
adslab <- readr::read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    PARAMCD = col_character(),
    AVAL    = col_double(),
    ANRLO   = col_double(),
    ANRHI   = col_double()
  )
)

# Derive ATOXGRDIR using admiral
adslab <- adslab %>%
  admiral::derive_var_atoxgr_dir(
    new_var = ATOXGRDIR,
    aval = AVAL,
    anrlo = ANRLO,
    anrhi = ANRHI
  )

# Select required columns and write output
adslab %>%
  select(USUBJID, PARAMCD, AVAL, ANRLO, ANRHI, ATOXGRDIR) %>%
  readr::write_csv(
    file = file.path("outputs", "result.csv"),
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
No LLM output artifact was produced.
