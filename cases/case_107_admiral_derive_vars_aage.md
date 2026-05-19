# Case 107: pharmaverse/admiral/derive_vars_aage

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_aage`
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
Write an R script to perform derive vars aage using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, BRTHDT, RFSTDTC, AAGE
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	BRTHDT	RFSTDTC
01	1980-06-15	2020-06-14
02	1980-06-15	2020-06-15
03	1975-01-01	2020-12-31
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
birth <- as.Date(dataset$BRTHDT)
ref <- as.Date(dataset$RFSTDTC)
age <- as.integer(format(ref, "%Y")) - as.integer(format(birth, "%Y"))
had_birthday <- format(ref, "%m-%d") >= format(birth, "%m-%d")
aage <- age - ifelse(had_birthday, 0L, 1L)
result_df <- data.frame(USUBJID = dataset$USUBJID, BRTHDT = birth, RFSTDTC = ref, AAGE = aage, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(admiral)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
adat <- readr::read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    BRTHDT  = col_date(format = ""),
    RFSTDTC = col_date(format = "")
  )
)

# Derive AAGE per admiral clinical conventions
adat <- adat %>%
  admiral::derive_vars_aage(
    dob     = BRTHDT,
    refdate = RFSTDTC,
    new_var = AAGE
  )

# Select required columns and write output
adat %>%
  select(USUBJID, BRTHDT, RFSTDTC, AAGE) %>%
  write_csv(
    file = file.path("outputs", "result.csv"),
    na = ""
  )
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","BRTHDT","RFSTDTC","AAGE"
1,1980-06-15,2020-06-14,39
2,1980-06-15,2020-06-15,40
3,1975-01-01,2020-12-31,45
```

### LLM Output
No LLM output artifact was produced.
