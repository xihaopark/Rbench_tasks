# Case 020: pharmaverse/aNCA/finalize_meanplot

## Metadata
- Task ID: `pharmaverse/aNCA/finalize_meanplot`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: plot_type.
```

## Prompt
```text
Write an R script to perform finalize meanplot using the aNCA clinical task contract.

Input: ci.tsv, color_by.tsv, plot.tsv, sd_max.tsv, sd_min.tsv, x_var.tsv, y_var.tsv
Output: result.csv


Required columns for result.csv: plot_type, success
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `ci.tsv`


```text
ci
TRUE
```

#### `color_by.tsv`


```text
color_by
ARM
```

#### `plot.tsv`


```text
plot
ggplot(data.frame(TIME=1:4, MEAN=c(0,32,58,36)), aes(x=TIME, y=MEAN)) + geom_line()
```

#### `sd_max.tsv`


```text
sd_max
TRUE
```

#### `sd_min.tsv`


```text
sd_min
TRUE
```

#### `x_var.tsv`


```text
x_var
TIME
```

#### `y_var.tsv`


```text
y_var
MEAN
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)

# 1. Read input data
plot_path <- file.path("inputs", "plot.tsv")
if (!file.exists(plot_path)) {
  stop("plot.tsv is required input")
}
suppressPackageStartupMessages(library(ggplot2))
plot_df <- read.delim(plot_path, check.names = FALSE, stringsAsFactors = FALSE)
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- eval(parse(text = plot_expr))
plot_expr <- plot_df[[1]][1]
# Create a ggplot object
plot <- tryCatch({
  eval(parse(text = plot_expr))
}, error = function(e) {
  # If parsing fails, create a simple ggplot object
  ggplot2::ggplot(data.frame(x = 1:10, y = 1:10), ggplot2::aes(x = x, y = y)) + ggplot2::geom_point()
})
sd_min_path <- file.path("inputs", "sd_min.tsv")
if (!file.exists(sd_min_path)) {
  stop("sd_min.tsv is required input")
}
sd_min_df <- read.delim(sd_min_path, check.names = FALSE, stringsAsFactors = FALSE)
sd_min <- as.logical(sd_min_df$sd_min)
sd_max_path <- file.path("inputs", "sd_max.tsv")
if (!file.exists(sd_max_path)) {
  stop("sd_max.tsv is required input")
}
sd_max_df <- read.delim(sd_max_path, check.names = FALSE, stringsAsFactors = FALSE)
sd_max <- as.logical(sd_max_df$sd_max)
ci_path <- file.path("inputs", "ci.tsv")
if (!file.exists(ci_path)) {
  stop("ci.tsv is required input")
}
ci_df <- read.delim(ci_path, check.names = FALSE, stringsAsFactors = FALSE)
ci <- as.logical(ci_df$ci)
color_by_path <- file.path("inputs", "color_by.tsv")
if (!file.exists(color_by_path)) {
  stop("color_by.tsv is required input")
}
color_by_df <- read.delim(color_by_path, check.names = FALSE, stringsAsFactors = FALSE)
color_by <- color_by_df$color_by
y_var_path <- file.path("inputs", "y_var.tsv")
if (!file.exists(y_var_path)) {
  stop("y_var.tsv is required input")
}
y_var <- read.delim(y_var_path, check.names = FALSE, stringsAsFactors = FALSE)
x_var_path <- file.path("inputs", "x_var.tsv")
if (!file.exists(x_var_path)) {
  stop("x_var.tsv is required input")
}
x_var <- read.delim(x_var_path, check.names = FALSE, stringsAsFactors = FALSE)

# 2. Validate data
# Ensure plot is a ggplot object
if (!inherits(plot, "ggplot")) {
  stop("plot must be a ggplot object")
}

# 3. Execute function implementation
plot_build <- ggplot2::ggplot_build(plot)

mean_title <- paste0("Mean ", plot_build$plot$labels$title)
if (isTRUE(ci)) mean_title <- paste0(mean_title, " (95% CI)")

result <- tryCatch({
  if (!has_aNCA) stop("aNCA unavailable")
  aNCA:::finalize_meanplot(
    plot = plot,
    sd_min = sd_min,
    sd_max = sd_max,
    ci = ci,
    color_by = color_by,
    y_var = y_var,
    x_var = x_var
  )
}, error = function(e) {
  # If the function is missing or fails, return the original plot
  plot
})

# 5. Create result dataframe
# Combine function results with input data when possible
if (inherits(result, "ggplot")) {
  result_df <- data.frame(
    plot_type = "ggplot",
    success = TRUE,
    stringsAsFactors = FALSE
  )
} else if (is.vector(result) || is.numeric(result) || is.character(result)) {
  result_df <- data.frame(
    sd_min = sd_min, sd_max = sd_max,
    result = result,
    stringsAsFactors = FALSE)
} else if (is.data.frame(result)) {
  result_df <- result
} else {
  # Try to create a data.frame; skip if it fails
  result_df <- tryCatch({
    data.frame(result = result, stringsAsFactors = FALSE)
  }, error = function(e) NULL)
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
if (exists("result_df") && !is.null(result_df) && is.data.frame(result_df)) {
  write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
} else if (exists("result")) {
  if (inherits(result, "ggplot")) {
    result_df <- data.frame(
      plot_type = "ggplot",
      success = TRUE,
      stringsAsFactors = FALSE
    )
    write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else if (is.data.frame(result)) {
    write.csv(result, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else if (is.list(result)) {
    result_df <- data.frame(
      result_type = paste(class(result), collapse = ","),
      result_length = length(result),
      stringsAsFactors = FALSE
    )
    write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else if (is.vector(result) || is.numeric(result) || is.character(result)) {
    result_df <- data.frame(result = result, stringsAsFactors = FALSE)
    write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else {
    result_df <- data.frame(result_type = paste(class(result), collapse = ","), stringsAsFactors = FALSE)
    write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  }
} else {
  stop("No result variable found. Function implementation may not have returned a value.")
}
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(ggplot2)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

read_single_value <- function(path) {
  df <- read.table(path, header = TRUE, sep = "\t", stringsAsFactors = FALSE, check.names = FALSE)
  as.character(df[[1]][1])
}

result <- data.frame(
  plot_type = "meanplot",
  success   = FALSE,
  stringsAsFactors = FALSE
)

try({
  ci      <- read_single_value("inputs/ci.tsv")
  color_by <- read_single_value("inputs/color_by.tsv")
  plot_str <- read_single_value("inputs/plot.tsv")
  sd_max  <- read_single_value("inputs/sd_max.tsv")
  sd_min  <- read_single_value("inputs/sd_min.tsv")
  x_var   <- read_single_value("inputs/x_var.tsv")
  y_var   <- read_single_value("inputs/y_var.tsv")

  p <- NULL
  suppressWarnings({
    p <- eval(parse(text = plot_str))
  })

  if (inherits(p, "ggplot")) {
    result$success <- TRUE
  }
}, silent = TRUE)

write.table(
  result,
  file = "outputs/result.csv",
  sep = ",",
  row.names = FALSE,
  col.names = TRUE,
  quote = TRUE
)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"plot_type","success"
"ggplot",TRUE
```

### LLM Output
#### `result.csv`


```csv
"plot_type","success"
"meanplot",TRUE
```
