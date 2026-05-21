# Case 094: pharmaverse/admiral/derive_locf_records

## Metadata
- Task ID: `pharmaverse/admiral/derive_locf_records`
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
Write an R script for an R package task: perform derive locf records using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: Carry the last observed value forward by USUBJID and PARAMCD ordered by AVISITN, creating only carried rows with DTYPE set to LOCF.


Required columns for result.csv: USUBJID, PARAMCD, AVISITN, AVAL, DTYPE
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVISITN	AVAL
01	ALT	1	20
01	ALT	3	30
02	ALT	1	15
02	ALT	4	24
```

#### `visits.tsv`

```text
USUBJID	PARAMCD	AVISITN
01	ALT	1
01	ALT	2
01	ALT	3
02	ALT	1
02	ALT	2
02	ALT	3
02	ALT	4
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
visits <- read.delim(file.path("inputs", "visits.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
visits <- visits[order(visits$USUBJID, visits$PARAMCD, visits$AVISITN), , drop = FALSE]
dataset <- dataset[order(dataset$USUBJID, dataset$PARAMCD, dataset$AVISITN), , drop = FALSE]
result_df <- visits
result_df$AVAL <- NA_real_
result_df$DTYPE <- ""
for (i in seq_len(nrow(result_df))) {
  prior <- dataset[
    dataset$USUBJID == result_df$USUBJID[i] &
      dataset$PARAMCD == result_df$PARAMCD[i] &
      dataset$AVISITN <= result_df$AVISITN[i],
    ,
    drop = FALSE
  ]
  if (nrow(prior) > 0) {
    prior <- prior[order(prior$AVISITN), , drop = FALSE]
    last <- prior[nrow(prior), , drop = FALSE]
    result_df$AVAL[i] <- last$AVAL
    result_df$DTYPE[i] <- ifelse(last$AVISITN == result_df$AVISITN[i], "", "LOCF")
  }
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
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

# Read input data
dataset <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    PARAMCD = col_character(),
    AVISITN = col_double(),
    AVAL    = col_double()
  )
)

visits <- read_tsv(
  file = file.path("inputs", "visits.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    PARAMCD = col_character(),
    AVISITN = col_double()
  )
)

# Ensure only needed columns
dataset <- dataset %>%
  select(USUBJID, PARAMCD, AVISITN, AVAL)

visits <- visits %>%
  select(USUBJID, PARAMCD, AVISITN)

# Derive LOCF records using admiral
# Merge visits with existing data, then apply LOCF within USUBJID/PARAMCD ordered by AVISITN
adslike <- visits %>%
  left_join(dataset, by = c("USUBJID", "PARAMCD", "AVISITN")) %>%
  arrange(USUBJID, PARAMCD, AVISITN)

locf_derived <- adslike %>%
  derive_var_extreme_flag(
    by_vars     = exprs(USUBJID, PARAMCD),
    order       = exprs(AVISITN),
    new_var     = AVAL,
    mode        = "locf",
    inclusive   = TRUE
  )

# Keep only rows that were imputed (no original AVAL but have LOCF value)
# In this context, we identify carried-forward records as those where original AVAL was NA
# but after LOCF we have a non-NA AVAL.
result <- locf_derived %>%
  mutate(
    AVAL_LOCF = AVAL,
    AVAL_ORIG = dataset$AVAL[match(
      paste(USUBJID, PARAMCD, AVISITN),
      paste(dataset$USUBJID, dataset$PARAMCD, dataset$AVISITN)
    )]
  ) %>%
  filter(is.na(AVAL_ORIG) & !is.na(AVAL_LOCF)) %>%
  transmute(
    USUBJID,
    PARAMCD,
    AVISITN,
    AVAL = AVAL_LOCF,
    DTYPE = "LOCF"
  ) %>%
  arrange(USUBJID, PARAMCD, AVISITN)

# Write result to CSV
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVISITN","AVAL","DTYPE"
1,"ALT",1,20,""
1,"ALT",2,20,"LOCF"
1,"ALT",3,30,""
2,"ALT",1,15,""
2,"ALT",2,15,"LOCF"
2,"ALT",3,15,"LOCF"
2,"ALT",4,24,""
```

### LLM Output
No LLM output artifact was produced.
