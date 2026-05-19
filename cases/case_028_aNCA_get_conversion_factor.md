# Case 028: pharmaverse/aNCA/get_conversion_factor

## Metadata
- Task ID: `pharmaverse/aNCA/get_conversion_factor`
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
Write an R script to perform get conversion factor using the aNCA clinical task contract.

Input: initial_unit.tsv, target_unit.tsv
Output: result.csv


Required columns for result.csv: initial_unit, target_unit, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `initial_unit.tsv`


```text
initial_unit
mg
mg
L
mL
hour
```

#### `target_unit.tsv`


```text
target_unit
mg
g
mL
L
hour
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

initial_unit <- as.character(first_column(read_tsv("initial_unit.tsv")))
target_unit <- as.character(first_column(read_tsv("target_unit.tsv")))
n <- max(length(initial_unit), length(target_unit))
length(initial_unit) <- n
length(target_unit) <- n
factor <- rep(NA_real_, n)
same <- !is.na(initial_unit) & !is.na(target_unit) & initial_unit == target_unit
factor[same] <- 1
result <- data.frame(initial_unit = initial_unit, target_unit = target_unit, result = factor)
write_result(result)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
  library(units)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
initial_df <- read_tsv("inputs/initial_unit.tsv", show_col_types = FALSE)
target_df  <- read_tsv("inputs/target_unit.tsv",  show_col_types = FALSE)

# Basic check: same number of rows
if (nrow(initial_df) != nrow(target_df)) {
  stop("initial_unit.tsv and target_unit.tsv must have the same number of rows")
}

# Map common clinical units to 'units' package syntax
normalize_unit <- function(u) {
  u <- str_trim(u)
  u <- tolower(u)
  # basic mapping
  if (u == "mg")  return("mg")
  if (u == "g")   return("g")
  if (u == "kg")  return("kg")
  if (u == "µg" || u == "mcg") return("ug")
  if (u == "l")   return("L")
  if (u == "ml")  return("mL")
  if (u == "ul")  return("uL")
  if (u == "hour" || u == "hr" || u == "h") return("h")
  if (u == "min" || u == "minute") return("min")
  if (u == "sec" || u == "s" || u == "second") return("s")
  u
}

# Function to get conversion factor aNCA-style
get_conversion_factor <- function(init_u, targ_u) {
  nu_init <- normalize_unit(init_u)
  nu_targ <- normalize_unit(targ_u)

  # If units are identical after normalization, factor is 1
  if (!is.na(nu_init) && !is.na(nu_targ) && nu_init == nu_targ) {
    return(1)
  }

  # Special handling for volume vs volume (e.g., mL <-> L)
  # and mass vs mass (mg <-> g) etc. using units package
  v <- tryCatch({
    x <- set_units(1, nu_init, mode = "standard")
    as.numeric(set_units(x, nu_targ))
  }, error = function(e) NA_real_)

  # If conversion fails, return NA (aNCA contracts often leave invalid as NA)
  v
}

result_vec <- mapply(
  get_conversion_factor,
  initial_df$initial_unit,
  target_df$target_unit,
  USE.NAMES = FALSE
)

result_df <- tibble(
  initial_unit = initial_df$initial_unit,
  target_unit  = target_df$target_unit,
  result       = result_vec
)

write_csv(result_df, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"initial_unit","target_unit","result"
"mg","mg",1
"mg","g",NA
"L","mL",NA
"mL","L",NA
"hour","hour",1
```

### LLM Output
#### `result.csv`


```csv
initial_unit,target_unit,result
mg,mg,1
mg,g,
L,mL,
mL,L,
hour,hour,1
```
