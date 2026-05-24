# Case 185: pharmaverse/metatools/add_labels

## Metadata
- Task ID: `pharmaverse/metatools/add_labels`
- Package: `metatools`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform add labels using the metatools clinical task contract.

Input: data.tsv, labels.tsv
Output: result.csv


Required columns for result.csv: column, label
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `data.tsv`


```text
USUBJID	AVAL
01	10
02	15
```

#### `labels.tsv`


```text
column	label
USUBJID	Subject ID
AVAL	Analysis Value
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(metatools))

data <- read.delim(file.path("inputs", "data.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
labels <- read.delim(file.path("inputs", "labels.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
args <- as.list(labels$label)
names(args) <- labels$column
labeled <- do.call(metatools::add_labels, c(list(data = data), args))
result <- data.frame(
  column = names(labeled),
  label = vapply(labeled, function(x) {
    value <- attr(x, "label", exact = TRUE)
    if (is.null(value)) NA_character_ else as.character(value)
  }, character(1)),
  stringsAsFactors = FALSE
)
dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(metatools)
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input data
data_path   <- file.path("inputs", "data.tsv")
labels_path <- file.path("inputs", "labels.tsv")

dat <- read_tsv(data_path, show_col_types = FALSE)
lbl <- read_tsv(labels_path, show_col_types = FALSE)

# Ensure required columns
lbl <- lbl %>% select(column, label)

# Apply labels using metatools clinical task contract
# (Assumes 'column' in labels.tsv matches column names in dat)
for (i in seq_len(nrow(lbl))) {
  col_name  <- lbl$column[i]
  col_label <- lbl$label[i]
  if (col_name %in% names(dat)) {
    attr(dat[[col_name]], "label") <- col_label
  }
}

# Prepare result according to required columns
result <- lbl %>% select(column, label)

# Write output
write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"column","label"
"USUBJID","Subject ID"
"AVAL","Analysis Value"
```

### LLM Output
#### `result.csv`


```csv
column,label
USUBJID,Subject ID
AVAL,Analysis Value
```
