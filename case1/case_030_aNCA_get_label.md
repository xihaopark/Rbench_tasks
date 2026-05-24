# Case 030: pharmaverse/aNCA/get_label

## Metadata
- Task ID: `pharmaverse/aNCA/get_label`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: id.
```

## Prompt
```text
Write an R script to perform get label using the aNCA clinical task contract.

Input: labels_df.tsv, type.tsv, variable.tsv
Output: result.csv


Required columns for result.csv: id, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `labels_df.tsv`


```text
Dataset	Variable	Label
ADPC	AVAL	Analysis Value
ADPC	PARAMCD	Parameter Code
ADPC	USUBJID	Subject ID
```

#### `type.tsv`


```text
type
ADPC
```

#### `variable.tsv`


```text
variable
AVAL
PARAMCD
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)

# 1. Read input data
variable_path <- file.path("inputs", "variable.tsv")
if (!file.exists(variable_path)) {
  stop("variable.tsv is required input")
}
variable <- read.delim(variable_path, check.names = FALSE, stringsAsFactors = FALSE)
type_path <- file.path("inputs", "type.tsv")
if (!file.exists(type_path)) {
  stop("type.tsv is required input")
}
type <- read.delim(type_path, check.names = FALSE, stringsAsFactors = FALSE)
labels_df_path <- file.path("inputs", "labels_df.tsv")
if (!file.exists(labels_df_path)) {
  stop("labels_df.tsv is required input")
}
labels_df <- read.delim(labels_df_path, check.names = FALSE, stringsAsFactors = FALSE)

# 2. Validate data
# Check the basic data frame structure

# 2. Validate data
# Check the basic data frame structure
for (df_name in c("variable", "labels_df")) {
  df <- get(df_name)
  if (is.data.frame(df)) {
    if (nrow(df) == 0) {
      stop(paste("Data frame", df_name, "is empty"))
    }
    if (ncol(df) == 0) {
      stop(paste("Data frame", df_name, "has no columns"))
    }
  }
}

# Extract the type parameter
if (is.data.frame(type) && ncol(type) > 0 && nrow(type) > 0) {
  type <- type[[1]][1]
} else if (is.character(type)) {
  type <- type[1]
}

# Extract the variable parameter
if (is.data.frame(variable) && ncol(variable) > 0 && nrow(variable) > 0) {
  variable <- variable[[1]]
} else if (is.character(variable) || is.factor(variable)) {
  variable <- as.character(variable)
}

# 3. Execute function implementation
suppressPackageStartupMessages(library(dplyr))
if (is.null(labels_df) || (is.data.frame(labels_df) && nrow(labels_df) == 0)) {
  result <- variable
} else {
  result <- tryCatch({
    if (!has_aNCA) stop("aNCA unavailable")
    aNCA:::get_label(variable = variable, type = type, labels_df = labels_df)
  }, error = function(e) {
    # If the function is unavailable, use translate_terms
    if (exists("translate_terms") && is.data.frame(labels_df) && "Dataset" %in% names(labels_df)) {
      filtered_labels <- dplyr::filter(labels_df, Dataset == type)
      if (nrow(filtered_labels) > 0 && "Variable" %in% names(filtered_labels) && "Label" %in% names(filtered_labels)) {
        translate_terms(
          variable,
          "Variable",
          "Label",
          metadata = filtered_labels
        )
      } else {
        variable
      }
    } else {
      variable
    }
  })
}

# 4. Create result dataframe
# Combine function results with input data when possible
if (is.vector(result) || is.numeric(result) || is.character(result)) {
  result_df <- data.frame(
    id = seq_along(result),
    result = result,
    stringsAsFactors = FALSE)
  } else if (is.data.frame(result)) {
  result_df <- result
} else {
  result_df <- data.frame(result = result, stringsAsFactors = FALSE)
}

# 6. Generate summary statistics
if (is.data.frame(result_df)) {
  summary_stats <- data.frame(
    n_rows = nrow(result_df),
    n_cols = ncol(result_df),
    col_names = paste(colnames(result_df), collapse = ", "),
    stringsAsFactors = FALSE)
  } else if (is.vector(result) || is.numeric(result)) {
  summary_stats <- data.frame(
    n_values = length(result),
    mean_value = mean(result, na.rm = TRUE),
    min_value = min(result, na.rm = TRUE),
    max_value = max(result, na.rm = TRUE),
    median_value = median(result, na.rm = TRUE),
    stringsAsFactors = FALSE)
  } else {
  summary_stats <- data.frame(
    result_type = class(result)[1],
    stringsAsFactors = FALSE
  )
}

# 7. Save outputs
outputs_dir <- "outputs"
dir.create(outputs_dir, showWarnings = FALSE)
unlink(file.path(outputs_dir, c("result.rds", "summary.csv")))
if (exists("result_df") && is.data.frame(result_df)) {
  write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else if (exists("result")) {
  if (is.data.frame(result)) {
    write.csv(result, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else if (is.list(result)) {
    saveRDS(result, file.path(outputs_dir, "result.rds"))
  } else if (is.vector(result) || is.numeric(result) || is.character(result)) {
    result_df <- data.frame(result = result, stringsAsFactors = FALSE)
    write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else {
    saveRDS(result, file.path(outputs_dir, "result.rds"))
  }
} else {
  stop("No result variable found. Function implementation may not have returned a value.")
}
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
labels_df <- read_tsv(
  file = file.path("inputs", "labels_df.tsv"),
  show_col_types = FALSE
)

type_df <- read_tsv(
  file = file.path("inputs", "type.tsv"),
  show_col_types = FALSE
)

variable_df <- read_tsv(
  file = file.path("inputs", "variable.tsv"),
  show_col_types = FALSE
)

# Extract type and variable values
types <- unique(type_df$type)
variables <- unique(variable_df$variable)

# Filter labels_df by requested type(s) and variable(s)
result_df <- labels_df %>%
  filter(Dataset %in% types,
         Variable %in% variables) %>%
  transmute(
    id = Variable,
    result = Label
  )

# Write output
write_csv(result_df, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","result"
1,"Analysis Value"
2,"Parameter Code"
```

### LLM Output
#### `result.csv`


```csv
id,result
AVAL,Analysis Value
PARAMCD,Parameter Code
```
