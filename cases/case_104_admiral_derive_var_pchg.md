# Case 104: pharmaverse/admiral/derive_var_pchg

## Metadata
- Task ID: `pharmaverse/admiral/derive_var_pchg`
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
Write an R script to perform derive var pchg using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVISIT, AVAL, BASE, PCHG
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVISIT	AVAL	BASE
01	ALT	Week 1	12	10
01	ALT	Week 2	15	10
02	ALT	Week 1	18	20
03	ALT	Week 1	5	0
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
base <- as.numeric(dataset$BASE)
aval <- as.numeric(dataset$AVAL)
pchg <- ifelse(is.na(base) | base == 0, NA_real_, 100 * (aval - base) / base)
result_df <- data.frame(dataset, PCHG = pchg, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
adsl <- read.delim(
  file = file.path("inputs", "dataset.tsv"),
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE
)

# Derive percent change from baseline
result <- adsl %>%
  mutate(
    PCHG = dplyr::if_else(
      !is.na(AVAL) & !is.na(BASE) & BASE != 0,
      (AVAL - BASE) / BASE * 100,
      NA_real_
    )
  ) %>%
  select(USUBJID, PARAMCD, AVISIT, AVAL, BASE, PCHG)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVISIT","AVAL","BASE","PCHG"
1,"ALT","Week 1",12,10,20
1,"ALT","Week 2",15,10,50
2,"ALT","Week 1",18,20,-10
3,"ALT","Week 1",5,0,NA
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,AVISIT,AVAL,BASE,PCHG
1,ALT,Week 1,12,10,20
1,ALT,Week 2,15,10,50
2,ALT,Week 1,18,20,-10
3,ALT,Week 1,5,0,
```
