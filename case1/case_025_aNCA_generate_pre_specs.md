# Case 025: pharmaverse/aNCA/generate_pre_specs

## Metadata
- Task ID: `pharmaverse/aNCA/generate_pre_specs`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `comparison_error`

## Pass/Error
```text
The generated code failed at `comparison_error`.

Main signal:
result.csv: Comparison error: No columns to parse from file.
```

## Prompt
```text
Write an R script to perform generate pre specs using the aNCA clinical task contract.

Input: cdisc_data.tsv, datasets.tsv
Output: result.csv


Required columns for result.csv: source, variable, storage_type, n_nonmissing, n_unique
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `cdisc_data.tsv`


```text
USUBJID	AVAL	PARAMCD
101	10.5	CMAX
102	20.3	AUC
103	30.7	CMAX
```

#### `datasets.tsv`


```text
datasets
ADPC
ADPP
```

## Code
### Ground Truth Code

```r
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

read_tsv <- function(name) {
  read.delim(file.path("inputs", name), check.names = FALSE, stringsAsFactors = FALSE)
}

summarize_specs <- function(df, source) {
  if (ncol(df) == 0) {
    return(data.frame(
      source = character(),
      variable = character(),
      storage_type = character(),
      n_nonmissing = integer(),
      n_unique = integer(),
      stringsAsFactors = FALSE
    ))
  }
  data.frame(
    source = source,
    variable = names(df),
    storage_type = vapply(df, function(x) class(x)[1], character(1)),
    n_nonmissing = vapply(df, function(x) sum(!is.na(x) & nzchar(as.character(x))), integer(1)),
    n_unique = vapply(df, function(x) length(unique(x[!is.na(x) & nzchar(as.character(x))])), integer(1)),
    stringsAsFactors = FALSE
  )
}

cdisc_data <- read_tsv("cdisc_data.tsv")
datasets <- read_tsv("datasets.tsv")
result <- rbind(
  summarize_specs(cdisc_data, "cdisc_data"),
  summarize_specs(datasets, "datasets")
)

write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(purrr)
  library(fs)
})

# Paths
cdisc_path <- file.path("inputs", "cdisc_data.tsv")
datasets_path <- file.path("inputs", "datasets.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

# Read inputs
cdisc_data <- read_tsv(cdisc_path, col_types = cols(.default = col_character()))
datasets <- read_tsv(datasets_path, col_types = cols(.default = col_character()))$datasets

# Determine storage type
get_storage_type <- function(x) {
  suppressWarnings(num <- as.numeric(x))
  if (all(is.na(x) | !is.na(num))) {
    "numeric"
  } else {
    "character"
  }
}

# Generate result
result <- map_dfr(datasets, function(ds) {
  if (!("PARAMCD" %in% names(cdisc_data))) return(tibble())
  ds_rows <- cdisc_data %>% filter(PARAMCD == ds)
  if (nrow(ds_rows) == 0) return(tibble())
  ds_rows %>%
    select(-PARAMCD) %>%
    imap_dfr(function(col, nm) {
      nonmissing <- sum(!is.na(col) & col != "")
      uniq <- n_distinct(col[!is.na(col) & col != ""])
      tibble(
        source = ds,
        variable = nm,
        storage_type = get_storage_type(col),
        n_nonmissing = nonmissing,
        n_unique = uniq
      )
    })
})

# Ensure output directory
if (!dir_exists(output_dir)) dir_create(output_dir, recurse = TRUE)

# Write result
write_csv(result, output_path)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"source","variable","storage_type","n_nonmissing","n_unique"
"cdisc_data","USUBJID","integer",3,3
"cdisc_data","AVAL","numeric",3,3
"cdisc_data","PARAMCD","character",3,2
"datasets","datasets","character",2,2
```

### LLM Output
#### `result.csv`


```csv

```
