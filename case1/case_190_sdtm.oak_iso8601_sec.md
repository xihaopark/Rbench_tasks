# Case 190: pharmaverse/sdtm.oak/iso8601_sec

## Metadata
- Task ID: `pharmaverse/sdtm.oak/iso8601_sec`
- Package: `sdtm.oak`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script to perform ISO 8601 sec using the sdtm.oak clinical task contract.

Input: x.tsv
Output: result.csv


Required columns for result.csv: x, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `x.tsv`


```text
x
PT1H2M3S
PT45S
PT2M
PT1H
```

## Code
### Ground Truth Code

```r
x <- as.character(read.delim(file.path("inputs", "x.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$x)
grab <- function(value, suffix) {
  m <- regexec(paste0("([0-9]+)", suffix), value)
  hit <- regmatches(value, m)[[1]]
  if (length(hit) == 0) 0L else as.integer(hit[[2]])
}
result <- vapply(x, function(value) grab(value, "H") * 3600L + grab(value, "M") * 60L + grab(value, "S"), integer(1))
result_df <- data.frame(x = x, result = result, stringsAsFactors = FALSE)
dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
x_path <- file.path("inputs", "x.tsv")
df <- readr::read_tsv(x_path, show_col_types = FALSE)

# Function to convert ISO 8601 duration PT… to seconds
iso_to_seconds <- function(x) {
  if (is.na(x) | x == "") return(NA_real_)
  # Expecting time-based duration starting with PT
  m <- str_match(x, "^PT(?:(\\d+)H)?(?:(\\d+)M)?(?:(\\d+)S)?$")
  if (is.na(m[1,1])) return(NA_real_)
  h <- as.numeric(m[1,2])
  mnt <- as.numeric(m[1,3])
  s <- as.numeric(m[1,4])
  h[is.na(h)] <- 0
  mnt[is.na(mnt)] <- 0
  s[is.na(s)] <- 0
  h * 3600 + mnt * 60 + s
}

result <- df %>%
  mutate(result = iso_to_seconds(x)) %>%
  select(x, result)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"x","result"
"PT1H2M3S",3723
"PT45S",45
"PT2M",120
"PT1H",3600
```

### LLM Output
No LLM output artifact was produced.
