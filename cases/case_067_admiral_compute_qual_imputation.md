# Case 067: pharmaverse/admiral/compute_qual_imputation

## Metadata
- Task ID: `pharmaverse/admiral/compute_qual_imputation`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform compute qual imputation using the admiral clinical task contract.

Input: character_value.tsv, factor.tsv, imputation_type.tsv
Output: result.csv


Required columns for result.csv: character_value, factor, imputation_type, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `character_value.tsv`


```text
character_value
<5.0
>10.0
=7.5
12.0
BLQ
```

#### `factor.tsv`


```text
factor
0.5
```

#### `imputation_type.tsv`


```text
imputation_type
half_min
```

## Code
### Ground Truth Code

```r
character_value <- as.character(read.delim(file.path("inputs", "character_value.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$character_value)
factor <- as.numeric(read.delim(file.path("inputs", "factor.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$factor[1])
imputation_type <- as.character(read.delim(file.path("inputs", "imputation_type.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$imputation_type[1])
clean <- gsub("^[<>=]", "", character_value)
numeric_value <- suppressWarnings(as.numeric(clean))
result <- numeric_value
below <- grepl("^<", character_value)
result[below] <- numeric_value[below] * factor
result[toupper(character_value) == "BLQ"] <- NA_real_
result_df <- data.frame(character_value = character_value, factor = factor, imputation_type = imputation_type, result = result, stringsAsFactors = FALSE)

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

# Read inputs
character_df <- read_tsv(
  file = file.path("inputs", "character_value.tsv"),
  col_types = cols(.default = col_character())
)

factor_df <- read_tsv(
  file = file.path("inputs", "factor.tsv"),
  col_types = cols(.default = col_double())
)

imputation_df <- read_tsv(
  file = file.path("inputs", "imputation_type.tsv"),
  col_types = cols(.default = col_character())
)

char_vals <- character_df$character_value
factor_val <- factor_df$factor[1]
imputation_type <- imputation_df$imputation_type[1]

# Helper to extract numeric part
extract_numeric <- function(x) {
  x_clean <- str_trim(x)
  x_num <- str_replace_all(x_clean, "^[<>=]+", "")
  suppressWarnings(as.numeric(x_num))
}

# Compute result based on imputation_type
compute_imputation <- function(value, imp_type, factor_val) {
  v_trim <- str_trim(value)
  num_val <- extract_numeric(v_trim)

  if (is.na(num_val)) {
    return(NA_real_)
  }

  if (imp_type == "half_min") {
    if (str_detect(v_trim, "^<")) {
      return(num_val * factor_val)
    } else {
      return(num_val)
    }
  } else {
    return(NA_real_)
  }
}

results <- vapply(char_vals, compute_imputation, numeric(1), imp_type = imputation_type, factor_val = factor_val)

output_df <- tibble(
  character_value = char_vals,
  factor = factor_val,
  imputation_type = imputation_type,
  result = results
)

write_csv(output_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"character_value","factor","imputation_type","result"
"<5.0",0.5,"half_min",2.5
">10.0",0.5,"half_min",10
"=7.5",0.5,"half_min",7.5
"12.0",0.5,"half_min",12
"BLQ",0.5,"half_min",NA
```

### LLM Output
#### `result.csv`


```csv
character_value,factor,imputation_type,result
<5.0,0.5,half_min,2.5
>10.0,0.5,half_min,10
=7.5,0.5,half_min,7.5
12.0,0.5,half_min,12
BLQ,0.5,half_min,
```
