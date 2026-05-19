# Case 097: pharmaverse/admiral/derive_param_map

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_map`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(2, 3) vs llm=(6, 3).
```

## Prompt
```text
Write an R script to perform derive param map using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVAL
01	SYSBP	120
01	DIABP	80
02	SYSBP	135
02	DIABP	85
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
wide <- reshape(dataset, idvar = "USUBJID", timevar = "PARAMCD", direction = "wide")
result_df <- data.frame(
  USUBJID = wide$USUBJID,
  PARAMCD = "MAP",
  AVAL = (2 * as.numeric(wide$AVAL.DIABP) + as.numeric(wide$AVAL.SYSBP)) / 3,
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
library(admiral)

# Input / output paths
in_file <- file.path("inputs", "dataset.tsv")
out_dir <- "outputs"
out_file <- file.path(out_dir, "result.csv")

if (!dir.exists(out_dir)) {
  dir.create(out_dir, recursive = TRUE)
}

# Read input
adsl_like <- read_tsv(
  in_file,
  col_types = cols(
    USUBJID = col_character(),
    PARAMCD = col_character(),
    AVAL = col_double()
  )
)

# Example: derive map based on existing PARAMCD/AVAL
# Assume contract to derive mean of SYSBP and DIABP as PARAMCD = "MEANBP"
param_map <- tibble(
  PARAMCD = c("MEANBP"),
  PARAM = c("Mean Blood Pressure")
)

# Derive additional parameter using admiral-style derivation
# For demonstration, we calculate mean of SYSBP and DIABP per USUBJID
sys_diab_wide <- adsl_like %>%
  filter(PARAMCD %in% c("SYSBP", "DIABP")) %>%
  pivot_wider(
    id_cols = USUBJID,
    names_from = PARAMCD,
    values_from = AVAL
  )

derived_meanbp <- sys_diab_wide %>%
  mutate(
    PARAMCD = "MEANBP",
    AVAL = (SYSBP + DIABP) / 2
  ) %>%
  select(USUBJID, PARAMCD, AVAL)

# Combine original and derived data to create final parameter map result
result <- adsl_like %>%
  select(USUBJID, PARAMCD, AVAL) %>%
  bind_rows(derived_meanbp) %>%
  arrange(USUBJID, PARAMCD)

# Write output
write_csv(result, out_file)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVAL"
1,"MAP",93.3333333333333
2,"MAP",101.666666666667
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,AVAL
01,DIABP,80
01,MEANBP,100
01,SYSBP,120
02,DIABP,85
02,MEANBP,110
02,SYSBP,135
```
