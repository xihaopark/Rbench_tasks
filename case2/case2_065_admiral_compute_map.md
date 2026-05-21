# Case 065: pharmaverse/admiral/compute_map

## Metadata
- Task ID: `pharmaverse/admiral/compute_map`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
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

# Load required package
library(admiral)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input files
diabp_df <- read.table("inputs/diabp.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
hr_df    <- read.table("inputs/hr.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
sysbp_df <- read.table("inputs/sysbp.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)

# Align vectors by row (assumes equal length and order)
diabp <- diabp_df$diabp
hr    <- hr_df$hr
sysbp <- sysbp_df$sysbp

# Compute MAP using admiral
result <- admiral::compute_map(diabp = diabp, sysbp = sysbp, hr = hr)

# Create result data frame
out_df <- data.frame(
  diabp  = diabp,
  sysbp  = sysbp,
  hr     = hr,
  result = result
)

# Write output
write.csv(out_df, file = "outputs/result.csv", row.names = FALSE, quote = TRUE)
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
"diabp","sysbp","hr","result"
80,120,60,92.7395237619899
85,135,75,103.240653638352
70,110,90,85.9752811815739
95,160,110,123.186790537475
```
