# Case 142: pharmaverse/admiral/get_flagged_records

## Metadata
- Task ID: `pharmaverse/admiral/get_flagged_records`
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
Write an R script for an R package task: perform get flagged records using the admiral package. The function(s) you use to solve the task must come from the admiral package.

Output: result.csv

Computation: Return only records satisfying AVAL > threshold and set the requested flag variable to Y in those returned rows.


Required columns for result.csv: USUBJID, PARAMCD, AVAL, AVISIT, HIGHFL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVAL	AVISIT
01	ALT	45	WEEK 1
01	ALT	18	WEEK 2
02	AST	52	WEEK 1
02	ALT	38	WEEK 1
```

#### `new_var.tsv`

```text
new_var
HIGHFL
```

#### `threshold.tsv`

```text
threshold
40
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
threshold <- as.numeric(read.delim(file.path("inputs", "threshold.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$threshold[1])
new_var <- as.character(read.delim(file.path("inputs", "new_var.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$new_var[1])
result_df <- dataset[dataset$PARAMCD == "ALT" & dataset$AVAL > threshold, , drop = FALSE]
result_df[[new_var]] <- "Y"

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

# Read inputs
dataset   <- readr::read_tsv("inputs/dataset.tsv", show_col_types = FALSE)
new_var   <- readr::read_tsv("inputs/new_var.tsv", show_col_types = FALSE)
threshold <- readr::read_tsv("inputs/threshold.tsv", show_col_types = FALSE)

flag_var <- new_var$new_var[1]
thr_val  <- threshold$threshold[1]

# Apply admiral flagging and filtering
result <- dataset %>%
  admiral::derive_var_robust_flag(
    new_var     = !!rlang::sym(flag_var),
    condition   = AVAL > thr_val,
    false_value = NA_character_,
    true_value  = "Y"
  ) %>%
  filter(AVAL > thr_val) %>%
  select(USUBJID, PARAMCD, AVAL, AVISIT, HIGHFL = !!rlang::sym(flag_var))

# Write output
readr::write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL","AVISIT","HIGHFL"
1,"ALT",45,"WEEK 1","Y"
```

### LLM Output
No LLM output artifact was produced.
