# Case 096: pharmaverse/admiral/derive_param_bsa

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_bsa`
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
Write an R script to perform derive param BSA using the admiral clinical task contract.

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
  PARAMCD = "BSA",
  AVAL = sqrt(as.numeric(wide$AVAL.HEIGHT) * as.numeric(wide$AVAL.WEIGHT) / 3600),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required packages
library(dplyr)
library(readr)
library(stringr)
library(fs)

# Create outputs directory if it doesn't exist
if (!dir_exists("outputs")) {
  dir_create("outputs")
}

# Read input dataset
input_df <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  show_col_types = FALSE
)

# Pivot HEIGHT and WEIGHT, then derive BSA using Mosteller formula:
# BSA (m^2) = sqrt( (height(cm) * weight(kg)) / 3600 )
bsa_df <- input_df %>%
  filter(PARAMCD %in% c("HEIGHT", "WEIGHT")) %>%
  select(USUBJID, PARAMCD, AVAL) %>%
  tidyr::pivot_wider(
    id_cols = USUBJID,
    names_from = PARAMCD,
    values_from = AVAL
  ) %>%
  mutate(
    AVAL = sqrt((HEIGHT * WEIGHT) / 3600),
    PARAMCD = "BSA"
  ) %>%
  select(USUBJID, PARAMCD, AVAL)

# Write result
write_csv(bsa_df, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVAL"
1,"BSA",1.81811868577262
2,"BSA",2
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,AVAL
01,BSA,1.818118685772619
02,BSA,2
```
