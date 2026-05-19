# Case 188: pharmaverse/metatools/remove_labels

## Metadata
- Task ID: `pharmaverse/metatools/remove_labels`
- Package: `metatools`
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
Write an R script to perform remove labels using the metatools clinical task contract.

Input: data.tsv, labels.tsv
Output: result.csv


Required columns for result.csv: column, label_present
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
unlabeled <- metatools::remove_labels(labeled)
result <- data.frame(
  column = names(unlabeled),
  label_present = vapply(unlabeled, function(x) !is.null(attr(x, "label", exact = TRUE)), logical(1)),
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

# Read input files
data_path <- file.path("inputs", "data.tsv")
labels_path <- file.path("inputs", "labels.tsv")

dat <- read_tsv(data_path, col_types = cols(.default = "c"))
lbl <- read_tsv(labels_path, col_types = cols(.default = "c"))

# Prepare labels list named by column
label_list <- lbl$label
names(label_list) <- lbl$column

# Apply labels to data using metatools
dat_labeled <- add_labels(dat, labels = label_list)

# Remove labels: metatools uses attributes, so drop them
dat_unlabeled <- dat_labeled
for (nm in names(dat_unlabeled)) {
  attr(dat_unlabeled[[nm]], "label") <- NULL
}

# Prepare result: for each column whether label was present in labels.tsv
result <- tibble(
  column = names(dat),
  label_present = names(dat) %in% lbl$column
)

# Write result
write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"column","label_present"
"USUBJID",FALSE
"AVAL",FALSE
```

### LLM Output
No LLM output artifact was produced.
