# Case 009: pharmaverse/aNCA/apply_mapping

## Metadata
- Task ID: `pharmaverse/aNCA/apply_mapping`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: conc.
```

## Prompt
```text
Write an R script to perform apply mapping using the aNCA clinical task contract.

Input: dataset.tsv, desired_order.tsv, mapping.tsv, req_mappings.tsv, silent.tsv
Output: result.csv


Required columns for result.csv: id, conc, treatment, analyte
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
id	value	group	category
1	10.5	A	Type1
2	20.3	B	Type2
3	30.7	A	Type1
4	40.2	B	Type2
5	50.9	A	Type1
```

#### `desired_order.tsv`


```text
desired_order
"id"
"conc"
"treatment"
"analyte"
```

#### `mapping.tsv`


```text
from	to
value	conc
group	treatment
category	analyte
```

#### `req_mappings.tsv`


```text
req_mappings
"value"
"group"
"category"
```

#### `silent.tsv`


```text
silent
TRUE
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

dataset <- read_tsv("dataset.tsv")
mapping <- read_tsv("mapping.tsv")
result <- dataset
if (ncol(mapping) >= 2) {
  from <- as.character(mapping[[1]])
  to <- as.character(mapping[[2]])
  valid <- from %in% names(result) & nzchar(to)
  names(result)[match(from[valid], names(result))] <- to[valid]
}
write_result(result)
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

# Read inputs
dataset <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols()
)

desired_order <- read_tsv(
  file = file.path("inputs", "desired_order.tsv"),
  col_types = cols()
) %>%
  pull(desired_order) %>%
  gsub('"', '', .)

mapping <- read_tsv(
  file = file.path("inputs", "mapping.tsv"),
  col_types = cols()
)

req_mappings <- read_tsv(
  file = file.path("inputs", "req_mappings.tsv"),
  col_types = cols()
) %>%
  pull(req_mappings) %>%
  gsub('"', '', .)

silent_flag <- read_tsv(
  file = file.path("inputs", "silent.tsv"),
  col_types = cols()
) %>%
  pull(silent) %>%
  as.logical()

# Optionally suppress messages (basic handling)
if (isTRUE(silent_flag)) {
  options(warn = -1)
}

# Check required mappings
available_from <- mapping$from
missing_required <- setdiff(req_mappings, available_from)
if (length(missing_required) > 0 && !isTRUE(silent_flag)) {
  stop("Missing required mappings for: ", paste(missing_required, collapse = ", "))
}

# Rename columns according to mapping
rename_vec <- setNames(mapping$to, mapping$from)
dataset_mapped <- dataset %>%
  rename(any_of(rename_vec))

# Ensure all required output columns exist
required_cols <- c("id", "conc", "treatment", "analyte")
missing_cols <- setdiff(required_cols, colnames(dataset_mapped))
if (length(missing_cols) > 0) {
  # Add missing columns as NA
  for (mc in missing_cols) {
    dataset_mapped[[mc]] <- NA
  }
}

# Reorder columns according to desired_order but keep only required ones
final_order <- intersect(desired_order, required_cols)
result <- dataset_mapped %>%
  select(all_of(final_order))

# Ensure final has exactly required columns in desired order
missing_in_desired <- setdiff(required_cols, final_order)
if (length(missing_in_desired) > 0) {
  # Append missing required columns at the end
  result <- result %>%
    bind_cols(dataset_mapped[, missing_in_desired, drop = FALSE])
}

# Write output
write_csv(
  result,
  file = file.path("outputs", "result.csv"),
  na = ""
)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","conc","treatment","analyte"
1,10.5,"A","Type1"
2,20.3,"B","Type2"
3,30.7,"A","Type1"
4,40.2,"B","Type2"
5,50.9,"A","Type1"
```

### LLM Output
#### `result.csv`


```csv
id,conc,treatment,analyte
1,,,
2,,,
3,,,
4,,,
5,,,
```
