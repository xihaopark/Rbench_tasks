# Case 068: pharmaverse/admiral/compute_qual_imputation_dec

## Metadata
- Task ID: `pharmaverse/admiral/compute_qual_imputation_dec`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: result.
```

## Prompt
```text
Write an R script to perform compute qual imputation dec using the admiral clinical task contract.

Input: character_value_decimal.tsv
Output: result.csv


Required columns for result.csv: character_value_decimal, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `character_value_decimal.tsv`


```text
character_value_decimal
5
5.0
5.25
<0.125
12.3000
```

## Code
### Ground Truth Code

```r
character_value_decimal <- as.character(read.delim(file.path("inputs", "character_value_decimal.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$character_value_decimal)
x <- trimws(character_value_decimal)
dot_pos <- regexpr("\\.", x)
result <- ifelse(dot_pos > 0, 1 / (10^(nchar(x) - dot_pos)), 1)
result_df <- data.frame(character_value_decimal = character_value_decimal, result = result, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required package
suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
input_path <- file.path("inputs", "character_value_decimal.tsv")
df <- readr::read_tsv(input_path, col_types = cols(.default = "c"))

# Function to compute qualitative imputation (DEC) per admiral clinical task contract
compute_qual_imputation_dec <- function(x) {
  x_trim <- str_trim(x)

  # Missing or blank
  if (is.na(x_trim) || x_trim == "") {
    return(NA_character_)
  }

  # Detect inequality prefix
  prefix <- ""
  core <- x_trim

  if (str_starts(core, "<=")) {
    prefix <- "<="
    core <- str_sub(core, 3)
  } else if (str_starts(core, ">=")) {
    prefix <- ">="
    core <- str_sub(core, 3)
  } else if (str_starts(core, "<")) {
    prefix <- "<"
    core <- str_sub(core, 2)
  } else if (str_starts(core, ">")) {
    prefix <- ">"
    core <- str_sub(core, 2)
  }

  core <- str_trim(core)

  # If remaining is not numeric, return original
  suppressWarnings(num <- as.numeric(core))
  if (is.na(num)) {
    return(x_trim)
  }

  # Remove trailing zeros and decimal point if not needed
  core_std <- format(num, scientific = FALSE, trim = TRUE)
  core_std <- sub("\\.$", "", core_std)

  paste0(prefix, core_std)
}

# Apply computation
result_df <- df %>%
  mutate(
    result = vapply(
      character_value_decimal,
      compute_qual_imputation_dec,
      FUN.VALUE = character(1)
    )
  ) %>%
  select(character_value_decimal, result)

# Write output
output_path <- file.path("outputs", "result.csv")
readr::write_csv(result_df, output_path, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"character_value_decimal","result"
"5",1
"5.0",0.1
"5.25",0.01
"<0.125",0.001
"12.3000",1e-04
```

### LLM Output
#### `result.csv`


```csv
character_value_decimal,result
5,5
5.0,5
5.25,5.25
<0.125,<0.125
12.3000,12.3
```
