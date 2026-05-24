# Case 055: pharmaverse/admiral/call_derivation

## Metadata
- Task ID: `pharmaverse/admiral/call_derivation`
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
Write an R script to perform call derivation using the admiral clinical task contract.

Input: dataset.tsv, variable_params.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVAL, GE20FL, GE40FL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVAL
01	ALT	12
02	ALT	45
03	AST	30
04	AST	8
```

#### `variable_params.tsv`


```text
threshold	flag_var
20	GE20FL
40	GE40FL
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(rlang))

dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
variable_params_df <- read.delim(file.path("inputs", "variable_params.tsv"), check.names = FALSE, stringsAsFactors = FALSE)

dataset$AVAL <- as.numeric(dataset$AVAL)
variable_params_df$threshold <- as.numeric(variable_params_df$threshold)
variable_params_df$flag_var <- as.character(variable_params_df$flag_var)

derive_threshold_flag <- function(dataset, threshold, flag_var, value_col) {
  dataset[[flag_var]] <- ifelse(dataset[[value_col]] >= threshold, "Y", "N")
  dataset
}

variable_params <- lapply(seq_len(nrow(variable_params_df)), function(i) {
  structure(
    list(
      threshold = variable_params_df$threshold[i],
      flag_var = variable_params_df$flag_var[i]
    ),
    class = c("params", "source", "list"),
    env = rlang::caller_env()
  )
})

result <- admiral::call_derivation(
  dataset = dataset,
  derivation = derive_threshold_flag,
  variable_params = variable_params,
  value_col = "AVAL"
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(tidyr)
library(purrr)
library(stringr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
dataset <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(.default = col_character())
) %>%
  mutate(AVAL = as.numeric(AVAL))

var_params <- read_tsv(
  file = file.path("inputs", "variable_params.tsv"),
  col_types = cols(
    threshold = col_double(),
    flag_var  = col_character()
  )
)

# Derive flags per admiral-style variable parameterization
result <- dataset

for (i in seq_len(nrow(var_params))) {
  thr <- var_params$threshold[i]
  flag_var <- var_params$flag_var[i]

  result <- result %>%
    mutate(!!flag_var := if_else(AVAL >= thr, "Y", "N"))
}

# Keep required columns and write output
result %>%
  select(USUBJID, PARAMCD, AVAL, GE20FL, GE40FL) %>%
  write_csv(file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVAL","GE20FL","GE40FL"
1,"ALT",12,"N","N"
2,"ALT",45,"Y","Y"
3,"AST",30,"Y","N"
4,"AST",8,"N","N"
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,AVAL,GE20FL,GE40FL
01,ALT,12,N,N
02,ALT,45,Y,Y
03,AST,30,Y,N
04,AST,8,N,N
```
