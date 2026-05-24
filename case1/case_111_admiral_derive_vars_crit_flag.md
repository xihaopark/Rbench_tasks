# Case 111: pharmaverse/admiral/derive_vars_crit_flag

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_crit_flag`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: CRIT1FL.
```

## Prompt
```text
Write an R script to perform derive vars crit flag using the admiral clinical task contract.

Input: dataset.tsv, description.tsv
Output: result.csv


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
library(stringr)

# Ensure outputs directory exists
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

# Get CRIT1 from description
crit1_text <- description$description[1]

# Derive CRIT1FL based on ALT greater than three times ULN
result <- dataset %>%
  mutate(
    CRIT1 = crit1_text,
    CRIT1FL = case_when(
      PARAMCD == "ALT" & AVAL > 3 * ULN ~ "Y",
      TRUE ~ NA_character_
    )
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
#### `result.csv`


```csv
USUBJID,PARAMCD,AVAL,ULN,CRIT1,CRIT1FL
01,ALT,45,20,ALT greater than three times ULN,
02,ALT,75,20,ALT greater than three times ULN,Y
03,AST,30,20,ALT greater than three times ULN,
```
