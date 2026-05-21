# Case 142: pharmaverse/admiral/get_flagged_records

## Metadata
- Task ID: `pharmaverse/admiral/get_flagged_records`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output files were produced, but the schema or output shape differed from the ground truth.

Main signal:
result.csv: Shape mismatch: ref=(1, 5) vs llm=(2, 5)
```

## Prompt
```text
Write an R script for an R package task: perform get flagged records using the admiral package. Use functions from the admiral package when suitable.

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

library(readr)
library(dplyr)
library(admiral)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
dataset   <- read_tsv("inputs/dataset.tsv", col_types = cols())
new_var   <- read_tsv("inputs/new_var.tsv", col_types = cols())
threshold <- read_tsv("inputs/threshold.tsv", col_types = cols())

flag_var <- new_var$new_var[1]
thresh   <- threshold$threshold[1]

# Flag using admiral::derive_var_merged_exist_flag then filter
result <- dataset %>%
  derive_var_merged_exist_flag(
    dataset_add = dataset,
    by_vars = exprs(USUBJID, PARAMCD, AVISIT),
    new_var = !!sym(flag_var),
    condition = AVAL > thresh,
    true_value = "Y",
    false_value = NA_character_
  ) %>%
  filter(AVAL > thresh) %>%
  select(USUBJID, PARAMCD, AVAL, AVISIT, !!sym(flag_var)) %>%
  rename(HIGHFL = !!sym(flag_var))

# Write output
write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL","AVISIT","HIGHFL"
1,"ALT",45,"WEEK 1","Y"
```

### LLM Output
#### `result.csv`

```csv
USUBJID,PARAMCD,AVAL,AVISIT,HIGHFL
01,ALT,45,WEEK 1,Y
02,AST,52,WEEK 1,Y
```
