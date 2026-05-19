# Case 041: pharmaverse/aNCA/pk.calc.volpk

## Metadata
- Task ID: `pharmaverse/aNCA/pk.calc.volpk`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(1, 2) vs llm=(3, 2).
```

## Prompt
```text
Write an R script to perform PK calc volpk using the aNCA clinical task contract.

Input: volume.tsv
Output: result.csv


Required columns for result.csv: id, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `volume.tsv`


```text
volume
0.25
1.5
10.0
```

## Code
### Ground Truth Code

```r
read_tsv <- function(name) {
  path <- file.path("inputs", name)
  if (!file.exists(path)) {
    stop(name, " is required")
  }
  read.delim(path, check.names = FALSE, stringsAsFactors = FALSE)
}

first_column <- function(df) {
  if (ncol(df) == 0) {
    return(character())
  }
  df[[1]]
}

scalar_numeric <- function(df) {
  values <- suppressWarnings(as.numeric(first_column(df)))
  values <- values[!is.na(values)]
  if (length(values) == 0) NA_real_ else values[[1]]
}

write_result <- function(result) {
  dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
  write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "NA")
}

volume <- suppressWarnings(as.numeric(first_column(read_tsv("volume.tsv"))))
value <- if (length(volume) == 0 || is.na(volume[[1]])) NA_real_ else volume[[1]]
result <- data.frame(id = 1L, result = value)
write_result(result)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required packages
suppressPackageStartupMessages({
  if (!requireNamespace("dplyr", quietly = TRUE)) {
    install.packages("dplyr", repos = "https://cloud.r-project.org")
  }
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
vol_path <- file.path("inputs", "volume.tsv")
vol_df <- read.delim(vol_path, header = TRUE, sep = "\t", stringsAsFactors = FALSE)

# Ensure required column exists
if (!"volume" %in% names(vol_df)) {
  stop("Input file must contain a 'volume' column.")
}

# PK calculation: volpk (here assumed identity of volume as result)
result_df <- vol_df %>%
  mutate(
    id = seq_len(n()),
    result = volume
  ) %>%
  select(id, result)

# Write output
out_path <- file.path("outputs", "result.csv")
write.csv(result_df, out_path, row.names = FALSE, quote = TRUE)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","result"
1,0.25
```

### LLM Output
#### `result.csv`


```csv
"id","result"
1,0.25
2,1.5
3,10
```
