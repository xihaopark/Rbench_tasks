# Case 049: pharmaverse/aNCA/simplify_unit

## Metadata
- Task ID: `pharmaverse/aNCA/simplify_unit`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: result.
```

## Prompt
```text
Write an R script to perform simplify unit using the aNCA clinical task contract.

Input: as_character.tsv, x.tsv
Output: result.csv


Required columns for result.csv: x, as_character, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `as_character.tsv`


```text
as_character
TRUE
FALSE
```

#### `x.tsv`


```text
x
mg/L
ng/mL
```

## Code
### Ground Truth Code

```r
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
read_tsv <- function(name) read.delim(file.path("inputs", paste0(name, ".tsv")), check.names = FALSE, stringsAsFactors = FALSE)
x <- as.character(read_tsv("x")[[1]])
as_character <- as.character(read_tsv("as_character")[[1]])
n <- max(length(x), length(as_character))
write.csv(data.frame(x = rep_len(x, n), as_character = rep_len(as_character, n), result = rep_len(x, n), stringsAsFactors = FALSE), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
as_character_df <- read_tsv("inputs/as_character.tsv", show_col_types = FALSE)
x_df            <- read_tsv("inputs/x.tsv", show_col_types = FALSE)

# Coerce as_character column to logical
as_character_logical <- as.logical(as_character_df$as_character)

# Simplify unit function (example behavior; adjust as per aNCA spec if needed)
simplify_unit <- function(unit, as_character) {
  u <- trimws(unit)
  if (as_character) {
    # Return simplified / canonical string
    return(u)
  } else {
    # Return numeric multiplier relative to the unit itself (identity)
    return(1)
  }
}

# Apply row-wise for all combinations (recycling like R would)
n <- max(nrow(x_df), length(as_character_logical))
x_vals  <- rep(x_df$x, length.out = n)
ac_vals <- rep(as_character_logical, length.out = n)

result_vals <- mapply(simplify_unit, x_vals, ac_vals, USE.NAMES = FALSE)

# Build result data frame
result_df <- tibble(
  x            = x_vals,
  as_character = ac_vals,
  result       = result_vals
)

# Write CSV
write_csv(result_df, "outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"x","as_character","result"
"mg/L","TRUE","mg/L"
"ng/mL","FALSE","ng/mL"
```

### LLM Output
#### `result.csv`


```csv
x,as_character,result
mg/L,TRUE,mg/L
ng/mL,FALSE,1
```
