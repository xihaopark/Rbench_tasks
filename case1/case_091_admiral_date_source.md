# Case 091: pharmaverse/admiral/date_source

## Metadata
- Task ID: `pharmaverse/admiral/date_source`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: class.
```

## Prompt
```text
Write an R script to perform date source using the admiral clinical task contract.

Input: dataset_name.tsv, date.tsv, filter.tsv, set_values_to.tsv
Output: result.csv


Required columns for result.csv: class, dataset_name, filter, date, set_values_to
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset_name.tsv`


```text
dataset_name
adsl
```

#### `date.tsv`


```text
date
TRTSDT
```

#### `filter.tsv`


```text
filter
SAFFL == "Y"
```

#### `set_values_to.tsv`


```text
variable	value
PARAMCD	TRTSTART
PARAM	Treatment start date
EVNTDESC	Treatment start
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(rlang))

read_scalar <- function(path, column, default = "") {
  if (!file.exists(path)) return(default)
  data <- read.delim(path, check.names = FALSE, stringsAsFactors = FALSE)
  if (!nrow(data)) return(default)
  if (column %in% names(data)) return(as.character(data[[column]][1]))
  as.character(data[[1]][1])
}

read_key_values <- function(path) {
  data <- read.delim(path, check.names = FALSE, stringsAsFactors = FALSE)
  values <- as.list(as.character(data$value))
  names(values) <- as.character(data$variable)
  values
}

parse_expr_list <- function(text) {
  parts <- trimws(strsplit(text, ";", fixed = TRUE)[[1]])
  parts <- parts[nzchar(parts)]
  rlang::parse_exprs(parts)
}

expr_label <- function(x) {
  if (is.null(x)) return("")
  rlang::as_label(x)
}

exprs_label <- function(x) {
  if (is.null(x) || length(x) == 0) return("")
  labels <- vapply(x, rlang::as_label, character(1))
  names_x <- names(x)
  if (is.null(names_x)) names_x <- rep("", length(labels))
  labels <- ifelse(nzchar(names_x), paste0(names_x, "=", labels), labels)
  paste(labels, collapse = ";")
}

dataset_name <- read_scalar(file.path("inputs", "dataset_name.tsv"), "dataset_name")
date_var <- read_scalar(file.path("inputs", "date.tsv"), "date")
filter_text <- read_scalar(file.path("inputs", "filter.tsv"), "filter")
set_values <- read_key_values(file.path("inputs", "set_values_to.tsv"))

source <- suppressWarnings(admiral::date_source(
  dataset_name = dataset_name,
  filter = !!rlang::parse_expr(filter_text),
  date = !!rlang::sym(date_var),
  set_values_to = rlang::exprs(!!!set_values)
))

result <- data.frame(
  class = paste(class(source), collapse = "|"),
  dataset_name = source$dataset_name,
  filter = expr_label(source$filter),
  date = expr_label(source$date),
  set_values_to = exprs_label(source$set_values_to),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
dataset_name_df <- read_tsv(
  file.path("inputs", "dataset_name.tsv"),
  show_col_types = FALSE
)

date_df <- read_tsv(
  file.path("inputs", "date.tsv"),
  show_col_types = FALSE
)

filter_df <- read_tsv(
  file.path("inputs", "filter.tsv"),
  show_col_types = FALSE
)

set_values_to_df <- read_tsv(
  file.path("inputs", "set_values_to.tsv"),
  show_col_types = FALSE
)

# Prepare components
dataset_name_val <- dataset_name_df$dataset_name[1]
date_val         <- date_df$date[1]
filter_val       <- filter_df$filter[1]

# Collapse set_values_to into a single string of key=value pairs
set_values_to_str <- set_values_to_df %>%
  mutate(pair = paste0(variable, "=", value)) %>%
  pull(pair) %>%
  paste(collapse = ";")

# Build result
result <- tibble(
  class        = "admiral::derive_var_adt",
  dataset_name = dataset_name_val,
  filter       = filter_val,
  date         = date_val,
  set_values_to = set_values_to_str
)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"class","dataset_name","filter","date","set_values_to"
"date_source|source|list","adsl","SAFFL == Y","TRTSDT","PARAMCD=""TRTSTART"";PARAM=""Treatment start date"";EVNTDESC=""Treatment start"""
```

### LLM Output
#### `result.csv`


```csv
class,dataset_name,filter,date,set_values_to
admiral::derive_var_adt,adsl,"SAFFL == ""Y""",TRTSDT,PARAMCD=TRTSTART;PARAM=Treatment start date;EVNTDESC=Treatment start
```
