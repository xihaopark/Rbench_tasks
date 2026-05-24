# Case 117: pharmaverse/admiral/derive_vars_dy

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_dy`
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
Write an R script to perform derive vars dy using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, ADT, TRTSDT, ADY
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	ADT	TRTSDT
01	2020-01-01	2020-01-01
02	2020-01-05	2020-01-01
03	2019-12-31	2020-01-01
04	NA	2020-01-01
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
adt <- as.Date(ifelse(toupper(dataset$ADT) == "NA", NA_character_, dataset$ADT))
trtsdt <- as.Date(dataset$TRTSDT)
ady <- ifelse(is.na(adt) | is.na(trtsdt), NA_integer_, ifelse(adt >= trtsdt, as.integer(adt - trtsdt) + 1L, as.integer(adt - trtsdt)))
result_df <- data.frame(USUBJID = dataset$USUBJID, ADT = adt, TRTSDT = trtsdt, ADY = ady, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(lubridate)
library(admiral)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
df <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    ADT = col_character(),
    TRTSDT = col_character()
  )
)

# Convert dates
df <- df %>%
  mutate(
    ADT = ymd(ADT),
    TRTSDT = ymd(TRTSDT)
  )

# Derive ADY using admiral
df <- df %>%
  derive_vars_dy(ref_date = TRTSDT, date = ADT, new_var = ADY)

# Select required columns and write output
df %>%
  select(USUBJID, ADT, TRTSDT, ADY) %>%
  write_csv(file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","ADT","TRTSDT","ADY"
1,2020-01-01,2020-01-01,1
2,2020-01-05,2020-01-01,5
3,2019-12-31,2020-01-01,-1
4,NA,2020-01-01,NA
```

### LLM Output
No LLM output artifact was produced.
