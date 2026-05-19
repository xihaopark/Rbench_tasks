# Case 095: pharmaverse/admiral/derive_param_bmi

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_bmi`
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
Write an R script to perform derive param bmi using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVAL
01	HEIGHT	170
01	WEIGHT	70
02	HEIGHT	180
02	WEIGHT	80
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
wide <- reshape(dataset, idvar = "USUBJID", timevar = "PARAMCD", direction = "wide")
result_df <- data.frame(
  USUBJID = wide$USUBJID,
  PARAMCD = "BMI",
  AVAL = as.numeric(wide$AVAL.WEIGHT) / (as.numeric(wide$AVAL.HEIGHT) / 100)^2,
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(tidyr)
library(stringr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input dataset
adsl <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols()
)

# Pivot to get HEIGHT and WEIGHT per subject
wide <- adsl %>%
  select(USUBJID, PARAMCD, AVAL) %>%
  mutate(PARAMCD = toupper(PARAMCD)) %>%
  filter(PARAMCD %in% c("HEIGHT", "WEIGHT")) %>%
  pivot_wider(
    id_cols = USUBJID,
    names_from = PARAMCD,
    values_from = AVAL
  )

# Derive BMI (assuming HEIGHT in cm, WEIGHT in kg)
bmi <- wide %>%
  mutate(
    HEIGHT_M = HEIGHT / 100,
    BMI = WEIGHT / (HEIGHT_M ^ 2)
  ) %>%
  transmute(
    USUBJID,
    PARAMCD = "BMI",
    AVAL = BMI
  )

# Write result
write_csv(
  bmi,
  file = file.path("outputs", "result.csv"),
  na = ""
)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVAL"
1,"BMI",24.2214532871972
2,"BMI",24.6913580246914
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,AVAL
01,BMI,24.221453287197235
02,BMI,24.691358024691358
```
