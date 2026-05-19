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
Write an R script to perform derive locf records using the admiral clinical task contract.

Input: dataset.tsv, visits.tsv
Output: result.csv


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

library(dplyr)
library(readr)
library(tidyr)
library(admiral)

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input data
dataset <- read.delim(
  file = "inputs/dataset.tsv",
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE
)

visits <- read.delim(
  file = "inputs/visits.tsv",
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE
)

# Ensure correct types
dataset <- dataset %>%
  mutate(
    USUBJID = as.character(USUBJID),
    PARAMCD = as.character(PARAMCD),
    AVISITN = as.integer(AVISITN),
    AVAL = as.numeric(AVAL)
  )

visits <- visits %>%
  mutate(
    USUBJID = as.character(USUBJID),
    PARAMCD = as.character(PARAMCD),
    AVISITN = as.integer(AVISITN)
  )

# Merge observed with planned visits
adslike <- visits %>%
  left_join(dataset, by = c("USUBJID", "PARAMCD", "AVISITN"))

# Derive LOCF records
result <- derive_locf_records(
  adslike,
  by_vars = exprs(USUBJID, PARAMCD),
  order = exprs(AVISITN),
  analysis_var = AVAL,
  filter_source = is.na(AVAL),
  filter_source_records = TRUE
) %>%
  mutate(DTYPE = "LOCF") %>%
  select(USUBJID, PARAMCD, AVISITN, AVAL, DTYPE)

# Write output
write_csv(result, "outputs/result.csv")
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
