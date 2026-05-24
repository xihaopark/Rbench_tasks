# Case 128: pharmaverse/admiral/event

## Metadata
- Task ID: `pharmaverse/admiral/event`
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
Write an R script to perform event using the admiral clinical task contract.

Input: condition.tsv, dataset_name.tsv, description.tsv, keep_source_vars.tsv, mode.tsv, order.tsv, set_values_to.tsv
Output: result.csv


Required columns for result.csv: class, description, dataset_name, condition, mode, order, set_values_to, keep_source_vars
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `condition.tsv`


```text
condition
AETOXGR >= 2
```

#### `dataset_name.tsv`


```text
dataset_name
adae
```

#### `description.tsv`


```text
description
First grade 2 or higher adverse event
```

#### `keep_source_vars.tsv`


```text
keep_source_vars
ADT;AETERM;AETOXGR
```

#### `mode.tsv`


```text
mode
first
```

#### `order.tsv`


```text
order
ADT;AESEQ
```

#### `set_values_to.tsv`


```text
variable	value
PARAMCD	G2AE
PARAM	First grade 2 or higher adverse event
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
condition_text <- read_scalar(file.path("inputs", "condition.tsv"), "condition")
mode <- read_scalar(file.path("inputs", "mode.tsv"), "mode")
order <- parse_expr_list(read_scalar(file.path("inputs", "order.tsv"), "order"))
set_values <- read_key_values(file.path("inputs", "set_values_to.tsv"))
keep_source_vars <- parse_expr_list(read_scalar(file.path("inputs", "keep_source_vars.tsv"), "keep_source_vars"))
description <- read_scalar(file.path("inputs", "description.tsv"), "description")

event_def <- admiral::event(
  dataset_name = dataset_name,
  condition = !!rlang::parse_expr(condition_text),
  mode = mode,
  order = order,
  set_values_to = rlang::exprs(!!!set_values),
  keep_source_vars = keep_source_vars,
  description = description
)

result <- data.frame(
  class = paste(class(event_def), collapse = "|"),
  description = event_def$description,
  dataset_name = event_def$dataset_name,
  condition = expr_label(event_def$condition),
  mode = event_def$mode,
  order = exprs_label(event_def$order),
  set_values_to = exprs_label(event_def$set_values_to),
  keep_source_vars = exprs_label(event_def$keep_source_vars),
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
  library(tidyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Helper to read simple single-column TSVs
read_single_col_tsv <- function(path) {
  read_tsv(
    path,
    col_types = cols(.default = col_character())
  )
}

# Read inputs
condition_df      <- read_single_col_tsv("inputs/condition.tsv")
dataset_name_df   <- read_single_col_tsv("inputs/dataset_name.tsv")
description_df    <- read_single_col_tsv("inputs/description.tsv")
keep_source_df    <- read_single_col_tsv("inputs/keep_source_vars.tsv")
mode_df           <- read_single_col_tsv("inputs/mode.tsv")
order_df          <- read_single_col_tsv("inputs/order.tsv")
set_values_to_df  <- read_tsv("inputs/set_values_to.tsv",
                              col_types = cols(.default = col_character()))

# Extract scalar values
condition      <- condition_df$condition[1]
dataset_name   <- dataset_name_df$dataset_name[1]
description    <- description_df$description[1]
mode_val       <- mode_df$mode[1]
order_val      <- order_df$order[1]
keep_source    <- keep_source_df$keep_source_vars[1]

# Collapse set_values_to key/value pairs into a single string "VAR1=VAL1;VAR2=VAL2"
set_values_to <- set_values_to_df %>%
  mutate(pair = paste0(variable, "=", value)) %>%
  pull(pair) %>%
  paste(collapse = ";")

# Build result row
result <- tibble(
  class           = "event",
  description     = description,
  dataset_name    = dataset_name,
  condition       = condition,
  mode            = mode_val,
  order           = order_val,
  set_values_to   = set_values_to,
  keep_source_vars = keep_source
)

# Write output
write_csv(result, "outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"class","description","dataset_name","condition","mode","order","set_values_to","keep_source_vars"
"event|event_def|source|list","First grade 2 or higher adverse event","adae","AETOXGR >= 2","first","ADT;AESEQ","PARAMCD=""G2AE"";PARAM=""First grade 2 or higher adverse event""","ADT;AETERM;AETOXGR"
```

### LLM Output
#### `result.csv`


```csv
class,description,dataset_name,condition,mode,order,set_values_to,keep_source_vars
event,First grade 2 or higher adverse event,adae,AETOXGR >= 2,first,ADT;AESEQ,PARAMCD=G2AE;PARAM=First grade 2 or higher adverse event,ADT;AETERM;AETOXGR
```
