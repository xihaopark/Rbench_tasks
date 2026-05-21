# Case 065: pharmaverse/admiral/compute_map

## Metadata
- Task ID: `pharmaverse/admiral/compute_map`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the ground truth.

Main signal:
result.csv: Numeric mismatch in column: result
```

## Prompt
```text
Write an R script for an R package task: perform compute map using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: Call admiral::compute_map with diabp, sysbp, and hr from the aligned input vectors; include diabp, sysbp, hr, and result in result.csv.


Required columns for result.csv: diabp, sysbp, hr, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `diabp.tsv`

```text
diabp
80
85
70
95
```

#### `hr.tsv`

```text
hr
60
75
90
110
```

#### `sysbp.tsv`

```text
sysbp
120
135
110
160
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

diabp <- as.numeric(read.delim(file.path("inputs", "diabp.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$diabp)
sysbp <- as.numeric(read.delim(file.path("inputs", "sysbp.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$sysbp)
hr <- as.numeric(read.delim(file.path("inputs", "hr.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$hr)
result <- admiral::compute_map(diabp = diabp, sysbp = sysbp, hr = hr)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(diabp = diabp, sysbp = sysbp, hr = hr, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(admiral)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input files
diabp_df <- readr::read_tsv("inputs/diabp.tsv", col_types = cols())
hr_df    <- readr::read_tsv("inputs/hr.tsv",    col_types = cols())
sysbp_df <- readr::read_tsv("inputs/sysbp.tsv", col_types = cols())

# Align by row number (assuming equal length and correct order)
data <- dplyr::bind_cols(
  diabp = diabp_df$diabp,
  sysbp = sysbp_df$sysbp,
  hr    = hr_df$hr
)

# Compute MAP using admiral::compute_map
data <- data %>%
  mutate(result = admiral::compute_map(sysbp = sysbp, diabp = diabp))

# Write output
readr::write_csv(
  data %>% select(diabp, sysbp, hr, result),
  "outputs/result.csv",
  na = ""
)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"diabp","sysbp","hr","result"
80,120,60,92.7395237619899
85,135,75,103.240653638352
70,110,90,85.9752811815739
95,160,110,123.186790537475
```

### LLM Output
#### `result.csv`

```csv
diabp,sysbp,hr,result
80,120,60,93.33333333333333
85,135,75,101.66666666666667
70,110,90,83.33333333333333
95,160,110,116.66666666666667
```
