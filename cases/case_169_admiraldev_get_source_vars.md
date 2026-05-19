# Case 169: pharmaverse/admiraldev/get_source_vars

## Metadata
- Task ID: `pharmaverse/admiraldev/get_source_vars`
- Package: `admiraldev`
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
Write an R script to perform get source vars using the admiraldev clinical task contract.

Input: expressions.tsv
Output: result.csv


Required columns for result.csv: target, source_var
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `expressions.tsv`


```text
target	expression
baseline	AVAL
change	CHG
percent_change	PCHG
analysis_flag	ANL01FL
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiraldev))
suppressPackageStartupMessages(library(rlang))

expressions <- read.delim(
  file.path("inputs", "expressions.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)

expr_list <- setNames(
  lapply(expressions$expression, rlang::parse_expr),
  expressions$target
)
result <- admiraldev::get_source_vars(expr_list)

if (is.null(result) || length(result) == 0) {
  result_df <- data.frame(target = character(), source_var = character())
} else {
  result_df <- data.frame(
    target = names(result),
    source_var = vapply(result, rlang::as_label, character(1)),
    stringsAsFactors = FALSE
  )
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(stringr)
library(purrr)
library(fs)

# Create outputs directory if it doesn't exist
if (!dir_exists("outputs")) {
  dir_create("outputs")
}

# Read input
expr_df <- read_tsv(
  file = "inputs/expressions.tsv",
  col_types = cols(
    target = col_character(),
    expression = col_character()
  )
)

# Extract source vars from expressions using admiraldev-style logic:
# - Valid source vars are uppercase A-Z, 0-9 and underscore
# - Variables must start with a letter, and are taken as standalone tokens
get_source_vars <- function(expr) {
  if (is.na(expr) || expr == "") return(character(0))
  tokens <- str_extract_all(expr, "\\b[A-Z][A-Z0-9_]*\\b")[[1]]
  unique(tokens)
}

result_df <- expr_df %>%
  mutate(source_var = map(expression, get_source_vars)) %>%
  select(target, source_var) %>%
  unnest_longer(source_var, values_to = "source_var") %>%
  filter(!is.na(source_var) & source_var != "") %>%
  distinct(target, source_var) %>%
  arrange(target, source_var)

# Write output
write_csv(result_df, "outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"target","source_var"
"baseline","AVAL"
"change","CHG"
"percent_change","PCHG"
"analysis_flag","ANL01FL"
```

### LLM Output
No LLM output artifact was produced.
