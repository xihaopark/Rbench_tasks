# Case 182: pharmaverse/logrx/log_init

## Metadata
- Task ID: `pharmaverse/logrx/log_init`
- Package: `logrx`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(3, 2) vs llm=(2, 2).
```

## Prompt
```text
Write an R script to perform log init using the logrx clinical task contract.

Input: logrx.tsv
Output: result.csv


Required columns for result.csv: setting, value
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `logrx.tsv`


```text
setting	value
enabled	TRUE
level	INFO
```

## Code
### Ground Truth Code

```r
settings <- read.delim(file.path("inputs", "logrx.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result <- rbind(data.frame(setting = "initialized", value = "TRUE", stringsAsFactors = FALSE), settings)
dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
logrx_path <- file.path("inputs", "logrx.tsv")
logrx <- read_tsv(logrx_path, show_col_types = FALSE)

# Ensure required columns and write output
result <- logrx %>%
  select(setting, value)

write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"setting","value"
"initialized","TRUE"
"enabled","TRUE"
"level","INFO"
```

### LLM Output
#### `result.csv`


```csv
setting,value
enabled,TRUE
level,INFO
```
