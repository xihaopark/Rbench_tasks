# Case 131: pharmaverse/admiral/extend_source_datasets

## Metadata
- Task ID: `pharmaverse/admiral/extend_source_datasets`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(2, 4) vs llm=(3, 4).
```

## Prompt
```text
Write an R script to perform extend source datasets using the admiral clinical task contract.

Input: adlb.tsv, adsl.tsv, by_vars.tsv
Output: result.csv


Required columns for result.csv: dataset, n_rows, n_cols, columns
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `adlb.tsv`


```text
USUBJID	PARAMCD	AVAL
01	ALT	35
02	ALT	28
```

#### `adsl.tsv`


```text
USUBJID	TRT01P
01	Drug A
02	Placebo
```

#### `by_vars.tsv`


```text
var
USUBJID
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(rlang))

adsl <- read.delim(file.path("inputs", "adsl.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
adlb <- read.delim(file.path("inputs", "adlb.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
vars <- read.delim(file.path("inputs", "by_vars.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$var
by_vars <- as.list(lapply(vars, sym))
source_datasets <- list(adsl = adsl, adlb = adlb)
extended <- admiral:::extend_source_datasets(source_datasets, by_vars = by_vars)
result <- data.frame(
  dataset = names(extended),
  n_rows = vapply(extended, nrow, integer(1)),
  n_cols = vapply(extended, ncol, integer(1)),
  columns = vapply(extended, function(x) paste(names(x), collapse = ","), character(1)),
  stringsAsFactors = FALSE
)
dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(stringr)
  library(purrr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
adlb <- read_tsv(
  file = file.path("inputs", "adlb.tsv"),
  col_types = cols(.default = col_character())
)

adsl <- read_tsv(
  file = file.path("inputs", "adsl.tsv"),
  col_types = cols(.default = col_character())
)

by_vars <- read_tsv(
  file = file.path("inputs", "by_vars.tsv"),
  col_types = cols(.default = col_character())
)

by_vec <- by_vars$var

# Function to summarize a dataset
summarize_dataset <- function(df, name) {
  tibble(
    dataset = name,
    n_rows = nrow(df),
    n_cols = ncol(df),
    columns = paste(names(df), collapse = ",")
  )
}

# For this task, "extend source datasets using the admiral clinical task contract"
# is interpreted as: ensure that BY variables exist in source datasets by
# performing left joins where appropriate (a common ADaM extension step).

# Ensure BY vars present: join ADLB with ADSL on BY vars (if all BY vars exist)
if (all(by_vec %in% names(adlb)) && all(by_vec %in% names(adsl))) {
  adlb_extended <- adlb %>%
    left_join(adsl, by = by_vec)
} else {
  adlb_extended <- adlb
}

# Summarize datasets (original and extended)
result <- bind_rows(
  summarize_dataset(adlb, "ADLB"),
  summarize_dataset(adsl, "ADSL"),
  summarize_dataset(adlb_extended, "ADLB_EXTENDED")
)

# Write output
write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dataset","n_rows","n_cols","columns"
"adsl",2,2,"USUBJID,TRT01P"
"adlb",2,3,"USUBJID,PARAMCD,AVAL"
```

### LLM Output
#### `result.csv`


```csv
dataset,n_rows,n_cols,columns
ADLB,2,3,"USUBJID,PARAMCD,AVAL"
ADSL,2,2,"USUBJID,TRT01P"
ADLB_EXTENDED,2,4,"USUBJID,PARAMCD,AVAL,TRT01P"
```
