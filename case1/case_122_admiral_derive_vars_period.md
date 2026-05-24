# Case 122: pharmaverse/admiral/derive_vars_period

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_period`
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
Write an R script to perform derive vars period using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, ADT, APERIOD
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	ADT	P1SDT	P1EDT	P2SDT	P2EDT
01	2020-01-10	2020-01-01	2020-01-31	2020-02-01	2020-02-28
02	2020-02-15	2020-01-01	2020-01-31	2020-02-01	2020-02-28
03	2020-03-01	2020-01-01	2020-01-31	2020-02-01	2020-02-28
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
adt <- as.Date(dataset$ADT)
p1s <- as.Date(dataset$P1SDT); p1e <- as.Date(dataset$P1EDT)
p2s <- as.Date(dataset$P2SDT); p2e <- as.Date(dataset$P2EDT)
aperiod <- ifelse(adt >= p1s & adt <= p1e, 1L, ifelse(adt >= p2s & adt <= p2e, 2L, NA_integer_))
result_df <- data.frame(USUBJID = dataset$USUBJID, ADT = adt, APERIOD = aperiod, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(readr)
library(lubridate)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input
adsl <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    ADT     = col_date(format = "%Y-%m-%d"),
    P1SDT   = col_date(format = "%Y-%m-%d"),
    P1EDT   = col_date(format = "%Y-%m-%d"),
    P2SDT   = col_date(format = "%Y-%m-%d"),
    P2EDT   = col_date(format = "%Y-%m-%d")
  )
)

# Derive period variable using admiral
adsl_period <- adsl %>%
  derive_vars_period(
    new_var = APERIOD,
    reference_date = ADT,
    start_date = exprs(
      P1SDT,
      P2SDT
    ),
    end_date = exprs(
      P1EDT,
      P2EDT
    ),
    period_source = "number"
  )

# Select required columns and write output
adsl_period %>%
  select(USUBJID, ADT, APERIOD) %>%
  mutate(
    ADT = format(ADT, "%Y-%m-%d")
  ) %>%
  write_csv(file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","ADT","APERIOD"
1,2020-01-10,1
2,2020-02-15,2
3,2020-03-01,NA
```

### LLM Output
No LLM output artifact was produced.
