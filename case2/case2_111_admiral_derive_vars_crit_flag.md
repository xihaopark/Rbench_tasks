# Case 111: pharmaverse/admiral/derive_vars_crit_flag

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_crit_flag`
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
Write an R script for an R package task: perform derive vars crit flag using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: Set CRIT1FL to Y when AVAL > 3 * ULN and to N otherwise.


Required columns for result.csv: USUBJID, PARAMCD, AVAL, ULN, CRIT1, CRIT1FL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVAL	ULN
01	ALT	45	20
02	ALT	75	20
03	AST	30	20
```

#### `description.tsv`

```text
description
ALT greater than three times ULN
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
description <- as.character(read.delim(file.path("inputs", "description.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$description[1])
result_df$CRIT1 <- description
result_df$CRIT1FL <- ifelse(result_df$PARAMCD == "ALT" & result_df$AVAL > 3 * result_df$ULN, "Y", "N")

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(admiral)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
dataset <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols()
)

description <- read_tsv(
  file = file.path("inputs", "description.tsv"),
  col_types = cols()
)

crit1_text <- description$description[1]

# Derive CRIT1 and CRIT1FL using admiral
result <- dataset %>%
  mutate(
    CRIT1 = crit1_text
  ) %>%
  derive_var_crit_flag(
    new_var = CRIT1FL,
    condition = AVAL > 3 * ULN
  ) %>%
  mutate(
    CRIT1FL = if_else(is.na(CRIT1FL), "N", CRIT1FL)
  ) %>%
  select(USUBJID, PARAMCD, AVAL, ULN, CRIT1, CRIT1FL)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL","ULN","CRIT1","CRIT1FL"
1,"ALT",45,20,"ALT greater than three times ULN","N"
2,"ALT",75,20,"ALT greater than three times ULN","Y"
3,"AST",30,20,"ALT greater than three times ULN","N"
```

### LLM Output
No LLM output artifact was produced.
