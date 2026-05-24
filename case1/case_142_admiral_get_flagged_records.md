# Case 142: pharmaverse/admiral/get_flagged_records

## Metadata
- Task ID: `pharmaverse/admiral/get_flagged_records`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(1, 5) vs llm=(4, 5).
```

## Prompt
```text
Write an R script to perform get flagged records using the admiral clinical task contract.

Input: dataset.tsv, new_var.tsv, threshold.tsv
Output: result.csv


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

# Load necessary packages
suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
dataset   <- readr::read_tsv("inputs/dataset.tsv", show_col_types = FALSE)
new_var   <- readr::read_tsv("inputs/new_var.tsv", show_col_types = FALSE)
threshold <- readr::read_tsv("inputs/threshold.tsv", show_col_types = FALSE)

# Extract variable name and threshold value
flag_var   <- new_var$new_var[1]
cutoff_val <- threshold$threshold[1]

# Apply flag using admiral-style logic (HIGHFL = "Y" if AVAL > threshold, else "")
result <- dataset %>%
  mutate(
    !!flag_var := if_else(AVAL > cutoff_val, "Y", "")
  ) %>%
  select(USUBJID, PARAMCD, AVAL, AVISIT, all_of(flag_var))

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
#### `result.csv`


```csv
USUBJID,PARAMCD,AVAL,AVISIT,HIGHFL
01,ALT,45,WEEK 1,Y
01,ALT,18,WEEK 2,
02,AST,52,WEEK 1,Y
02,ALT,38,WEEK 1,
```
