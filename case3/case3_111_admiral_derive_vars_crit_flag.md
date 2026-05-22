# Case 111: pharmaverse/admiral/derive_vars_crit_flag

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_crit_flag`
- Package: `admiral`
- Model: `openai/gpt-5.5`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the ground truth.

Main signal:
result.csv: Value mismatch in column: CRIT1
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
library(admiral)
library(dplyr)

dir.create("outputs", recursive = TRUE, showWarnings = FALSE)

dataset <- read.delim(
  file = "inputs/dataset.tsv",
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE
)

description <- read.delim(
  file = "inputs/description.tsv",
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE
)

crit1_desc <- description$description[1]

result <- dataset %>%
  derive_vars_crit_flag(
    crit_nr = 1,
    condition = AVAL > 3 * ULN,
    description = crit1_desc
  ) %>%
  select(USUBJID, PARAMCD, AVAL, ULN, CRIT1, CRIT1FL)

write.csv(
  result,
  file = "outputs/result.csv",
  row.names = FALSE
)
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
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL","ULN","CRIT1","CRIT1FL"
1,"ALT",45,20,NA,NA
2,"ALT",75,20,"ALT greater than three times ULN","Y"
3,"AST",30,20,NA,NA
```
