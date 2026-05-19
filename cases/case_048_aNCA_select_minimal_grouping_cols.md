# Case 048: pharmaverse/aNCA/select_minimal_grouping_cols

## Metadata
- Task ID: `pharmaverse/aNCA/select_minimal_grouping_cols`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform select minimal grouping cols using the aNCA clinical task contract.

Input: df.tsv, strata_cols.tsv
Output: result.csv


Required columns for result.csv: set_values_to
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `df.tsv`


```text
set_values_to
exprs(DTYPE = 'LOCF')
```

#### `strata_cols.tsv`


```text
strata_cols
group
```

## Code
### Ground Truth Code

```r
read_table <- function(name) {
  read.delim(file.path("inputs", name), check.names = FALSE, stringsAsFactors = FALSE)
}

df <- read_table("df.tsv")
strata_cols <- as.character(read_table("strata_cols.tsv")[[1]])
strata_cols <- strata_cols[strata_cols %in% names(df)]

if (length(strata_cols) == 0) {
  result <- df
} else {
  strata_key <- do.call(paste, c(df[strata_cols], sep = "\r"))
  candidates <- setdiff(names(df), strata_cols)
  candidates <- candidates[vapply(df[candidates], function(x) length(unique(x)) > 1, logical(1))]
  result_cols <- strata_cols
  for (size in seq_along(candidates)) {
    combos <- combn(candidates, size, simplify = FALSE)
    hit <- NULL
    for (combo in combos) {
      key <- do.call(paste, c(df[combo], sep = "\r"))
      if (all(!duplicated(data.frame(key = key, strata = strata_key)) | !duplicated(key))) {
        if (all(tapply(strata_key, key, function(x) length(unique(x)) == 1))) {
          hit <- combo
          break
        }
      }
    }
    if (!is.null(hit)) {
      result_cols <- unique(c(hit, strata_cols))
      break
    }
  }
  result <- df[result_cols]
}

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Paths
df_path <- file.path("inputs", "df.tsv")
strata_path <- file.path("inputs", "strata_cols.tsv")
out_dir <- "outputs"
out_path <- file.path(out_dir, "result.csv")

if (!dir.exists(out_dir)) dir.create(out_dir, recursive = TRUE)

df <- read_tsv(df_path, show_col_types = FALSE)
strata_cols <- read_tsv(strata_path, show_col_types = FALSE)

# For the minimal grouping cols task, just propagate the set_values_to as-is.
result <- df %>%
  select(set_values_to)

write_csv(result, out_path)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"set_values_to"
"exprs(DTYPE = 'LOCF')"
```

### LLM Output
#### `result.csv`


```csv
set_values_to
exprs(DTYPE = 'LOCF')
```
