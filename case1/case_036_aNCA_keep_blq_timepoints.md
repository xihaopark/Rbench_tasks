# Case 036: pharmaverse/aNCA/keep_blq_timepoints

## Metadata
- Task ID: `pharmaverse/aNCA/keep_blq_timepoints`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(4, 2) vs llm=(1, 2).
```

## Prompt
```text
Write an R script to perform keep BLQ timepoints using the aNCA clinical task contract.

Input: mean_group_var.tsv, plot_data.tsv, xvar.tsv
Output: result.csv


Required columns for result.csv: ARM, TIME
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `mean_group_var.tsv`


```text
mean_group_var
ARM
```

#### `plot_data.tsv`


```text
USUBJID	ARM	TIME	CONC	DOSE	PARAM	ROUTE	TSLD	BLQFL
101	100 mg	0	0.0	100	AUC	oral	0	Y
101	100 mg	1	34.2	100	AUC	oral	1	N
101	100 mg	2	61.5	100	AUC	oral	2	N
101	100 mg	4	38.1	100	AUC	oral	4	N
102	100 mg	0	0.0	100	AUC	oral	0	Y
102	100 mg	1	29.8	100	AUC	oral	1	N
102	100 mg	2	55.4	100	AUC	oral	2	N
102	100 mg	4	33.6	100	AUC	oral	4	N
```

#### `xvar.tsv`


```text
xvar
TIME
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)

suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(rlang))

# 1. Read input data
plot_data_path <- file.path("inputs", "plot_data.tsv")
if (!file.exists(plot_data_path)) {
  stop("plot_data.tsv is required input")
}
plot_data <- read.delim(plot_data_path, check.names = FALSE, stringsAsFactors = FALSE)
xvar_path <- file.path("inputs", "xvar.tsv")
if (!file.exists(xvar_path)) {
  stop("xvar.tsv is required input")
}
xvar_df <- read.delim(xvar_path, check.names = FALSE, stringsAsFactors = FALSE)
xvar <- xvar_df$xvar
mean_group_var_path <- file.path("inputs", "mean_group_var.tsv")
if (!file.exists(mean_group_var_path)) {
  stop("mean_group_var.tsv is required input")
}
mean_group_var_df <- read.delim(mean_group_var_path, check.names = FALSE, stringsAsFactors = FALSE)
mean_group_var <- mean_group_var_df$mean_group_var

# 2. Validate data
# Check the basic data frame structure
for (df_name in c("plot_data")) {
  df <- get(df_name)
  if (nrow(df) == 0) {
    stop(paste("Data frame", df_name, "is empty"))
  }
  if (ncol(df) == 0) {
    stop(paste("Data frame", df_name, "has no columns"))
  }
}

# Extract scalar parameters
if (is.data.frame(mean_group_var) && ncol(mean_group_var) > 0 && nrow(mean_group_var) > 0) {
  mean_group_var <- mean_group_var[[1]][1]
} else if (is.character(mean_group_var)) {
  mean_group_var <- mean_group_var[1]
}
if (is.data.frame(xvar) && ncol(xvar) > 0 && nrow(xvar) > 0) {
  xvar <- xvar[[1]][1]
} else if (is.character(xvar)) {
  xvar <- xvar[1]
}

# 3. Execute function implementation
result <- tryCatch({
  if (!has_aNCA) stop("aNCA unavailable")
  aNCA:::keep_blq_timepoints(
    plot_data = plot_data,
    xvar = xvar,
    mean_group_var = mean_group_var
  )
}, error = function(e) {
  # If the function is unavailable, implement the logic manually
  # identify valid BLQ column
  blq_col <- intersect(c("AVALC", "AVALCAT1"), names(plot_data))[1]

  if (!is.na(blq_col) && blq_col %in% names(plot_data)) {
    plot_data <- plot_data %>%
      mutate(is_blq = grepl("BLQ|LTR|<[1-9]|<PCLLOQ", .data[[blq_col]]))
  } else {
    plot_data$is_blq <- FALSE
  }

  if ("USUBJID" %in% names(plot_data)) {
    result_df <- plot_data %>%
      group_by(!!sym(mean_group_var), !!sym(xvar)) %>%
      summarise(
        n_samples = n_distinct(USUBJID),
        n_blq_ratio = sum(is_blq) / n_samples,
        .groups = "drop"
      ) %>%
      filter(n_blq_ratio <= 0.5, n_samples > 1) %>%
      select(all_of(c(mean_group_var, xvar)))
    result_df
  } else {
    data.frame()
  }
})

# 4. Create result dataframe
# Combine function results with input data when possible
if (is.vector(result) || is.numeric(result) || is.character(result)) {
  result_df <- data.frame(
    xvar = xvar, mean_group_var = mean_group_var,
    result = result,
    stringsAsFactors = FALSE)
  } else if (is.data.frame(result)) {
  result_df <- result
} else {
  result_df <- data.frame(result = result, stringsAsFactors = FALSE)
}
# 6. Save outputs
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

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input files
mean_group_var <- read.table(
  file = "inputs/mean_group_var.tsv",
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE,
  check.names = FALSE
)

xvar <- read.table(
  file = "inputs/xvar.tsv",
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE,
  check.names = FALSE
)

plot_data <- read.table(
  file = "inputs/plot_data.tsv",
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE,
  check.names = FALSE
)

# Extract variable names
mgv <- mean_group_var$mean_group_var[1]
xv <- xvar$xvar[1]

# Ensure required columns exist
required_cols <- c("ARM", "TIME", "BLQFL")
missing_cols <- setdiff(required_cols, colnames(plot_data))
if (length(missing_cols) > 0) {
  stop(paste("Missing required columns in plot_data:", paste(missing_cols, collapse = ", ")))
}

# Filter to timepoints where all observations within ARM/TIME are BLQ (BLQFL == "Y")
library(dplyr)

result <- plot_data %>%
  group_by(.data[[mgv]], .data[[xv]]) %>%
  summarise(all_blq = all(BLQFL == "Y"), .groups = "drop") %>%
  filter(all_blq) %>%
  select(ARM = .data[[mgv]], TIME = .data[[xv]]) %>%
  distinct() %>%
  arrange(ARM, TIME)

# Write result
write.csv(result, file = "outputs/result.csv", row.names = FALSE, quote = TRUE)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"ARM","TIME"
"100 mg",0
"100 mg",1
"100 mg",2
"100 mg",4
```

### LLM Output
#### `result.csv`


```csv
"ARM","TIME"
"100 mg",0
```
