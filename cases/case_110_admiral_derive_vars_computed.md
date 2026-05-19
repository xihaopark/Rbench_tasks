# Case 110: pharmaverse/admiral/derive_vars_computed

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_computed`
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
Write an R script to perform derive vars computed using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, HEIGHTCM, WEIGHTKG, BMI
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	HEIGHTCM	WEIGHTKG
01	170	70
02	165	80
03	180	90
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df$BMI <- result_df$WEIGHTKG / (result_df$HEIGHTCM / 100)^2

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required packages
suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input data
input_path <- file.path("inputs", "dataset.tsv")
dat <- read_tsv(input_path, show_col_types = FALSE)

# Derive BMI
result <- dat %>%
  mutate(
    BMI = WEIGHTKG / ( (HEIGHTCM / 100) ^ 2 )
  ) %>%
  select(USUBJID, HEIGHTCM, WEIGHTKG, BMI)

# Write output
output_path <- file.path("outputs", "result.csv")
write_csv(result, output_path)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","HEIGHTCM","WEIGHTKG","BMI"
1,170,70,24.2214532871972
2,165,80,29.3847566574839
3,180,90,27.7777777777778
```

### LLM Output
#### `result.csv`


```csv
USUBJID,HEIGHTCM,WEIGHTKG,BMI
01,170,70,24.221453287197235
02,165,80,29.384756657483933
03,180,90,27.777777777777775
```
