# Case 121: pharmaverse/admiral/derive_vars_merged_summary

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_merged_summary`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform derive vars merged summary using the admiral clinical task contract.

Input: dataset.tsv, summary.tsv
Output: result.csv


Required columns for result.csv: USUBJID, AVISIT, AVAL, MEAN_BASE
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	AVISIT	AVAL
01	Week 1	10
01	Week 2	15
02	Week 1	20
02	Week 2	18
```

#### `summary.tsv`


```text
USUBJID	MEAN_BASE
01	12.5
02	19.0
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
summary <- read.delim(file.path("inputs", "summary.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df <- merge(dataset, summary, by = "USUBJID", all.x = TRUE, sort = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Create outputs directory if it does not exist
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

summary_df <- read.delim(
  file = "inputs/summary.tsv",
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE
)

# Derive vars merged summary (merge MEAN_BASE onto dataset by USUBJID)
result <- dataset %>%
  left_join(summary_df, by = "USUBJID") %>%
  select(USUBJID, AVISIT, AVAL, MEAN_BASE)

# Write output as CSV
write.csv(
  result,
  file = "outputs/result.csv",
  row.names = FALSE,
  na = ""
)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","AVISIT","AVAL","MEAN_BASE"
1,"Week 1",10,12.5
1,"Week 2",15,12.5
2,"Week 1",20,19
2,"Week 2",18,19
```

### LLM Output
#### `result.csv`


```csv
"USUBJID","AVISIT","AVAL","MEAN_BASE"
1,"Week 1",10,12.5
1,"Week 2",15,12.5
2,"Week 1",20,19
2,"Week 2",18,19
```
