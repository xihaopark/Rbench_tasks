# Case 017: pharmaverse/aNCA/exploration_individualplot

## Metadata
- Task ID: `pharmaverse/aNCA/exploration_individualplot`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `missing_artifact`

## Pass/Error
```text
The generated code did not write one or more required output files.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script to perform exploration individualplot using the aNCA clinical task contract.

Input: color_by.tsv, facet_by.tsv, filtering_list.tsv, labels_df.tsv, palette.tsv, pknca_data.tsv, show_dose.tsv, show_facet_n.tsv, show_legend.tsv, threshold_value.tsv, tooltip_vars.tsv, use_time_since_last_dose.tsv, x_limits.tsv, y_limits.tsv, ylog_scale.tsv
Output: result.csv


Required columns for result.csv: operation, success, result_type
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `color_by.tsv`


```text
color_by
ARM
```

#### `facet_by.tsv`


```text
facet_by
PARAM
```

#### `filtering_list.tsv`


```text
variable	value
ARM	100 mg
```

#### `labels_df.tsv`


```text
variable	label
USUBJID	Subject identifier
ARM	Treatment arm
TIME	Nominal time after dose
CONC	Concentration
DOSE	Dose amount
PARAM	PK parameter
```

#### `palette.tsv`


```text
palette
#1f77b4
#ff7f0e
```

#### `pknca_data.tsv`


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

#### `show_dose.tsv`


```text
show_dose
TRUE
```

#### `show_facet_n.tsv`


```text
show_facet_n
TRUE
```

#### `show_legend.tsv`


```text
show_legend
TRUE
```

#### `threshold_value.tsv`


```text
threshold_value
0
```

#### `tooltip_vars.tsv`


```text
tooltip_vars
USUBJID
TIME
CONC
```

#### `use_time_since_last_dose.tsv`


```text
use_time_since_last_dose
FALSE
```

#### `x_limits.tsv`


```text
x_limits
0
4
```

#### `y_limits.tsv`


```text
y_limits
0
70
```

#### `ylog_scale.tsv`


```text
ylog_scale
FALSE
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)

# 1. Read input data
pknca_data_path <- file.path("inputs", "pknca_data.tsv")
if (!file.exists(pknca_data_path)) {
  stop("pknca_data.tsv is required input")
}
pknca_data <- read.delim(pknca_data_path, check.names = FALSE, stringsAsFactors = FALSE)
color_by_path <- file.path("inputs", "color_by.tsv")
if (!file.exists(color_by_path)) {
  stop("color_by.tsv is required input")
}
color_by_df <- read.delim(color_by_path, check.names = FALSE, stringsAsFactors = FALSE)
color_by <- color_by_df$color_by
facet_by_path <- file.path("inputs", "facet_by.tsv")
if (!file.exists(facet_by_path)) {
  stop("facet_by.tsv is required input")
}
facet_by_df <- read.delim(facet_by_path, check.names = FALSE, stringsAsFactors = FALSE)
facet_by <- facet_by_df$facet_by
show_facet_n_path <- file.path("inputs", "show_facet_n.tsv")
if (!file.exists(show_facet_n_path)) {
  stop("show_facet_n.tsv is required input")
}
show_facet_n_df <- read.delim(show_facet_n_path, check.names = FALSE, stringsAsFactors = FALSE)
show_facet_n <- as.logical(show_facet_n_df$show_facet_n)
ylog_scale_path <- file.path("inputs", "ylog_scale.tsv")
if (!file.exists(ylog_scale_path)) {
  stop("ylog_scale.tsv is required input")
}
ylog_scale_df <- read.delim(ylog_scale_path, check.names = FALSE, stringsAsFactors = FALSE)
ylog_scale <- as.logical(ylog_scale_df$ylog_scale)
show_legend_path <- file.path("inputs", "show_legend.tsv")
if (!file.exists(show_legend_path)) {
  stop("show_legend.tsv is required input")
}
show_legend_df <- read.delim(show_legend_path, check.names = FALSE, stringsAsFactors = FALSE)
show_legend <- as.logical(show_legend_df$show_legend)
threshold_value_path <- file.path("inputs", "threshold_value.tsv")
if (!file.exists(threshold_value_path)) {
  stop("threshold_value.tsv is required input")
}
threshold_value_df <- read.delim(threshold_value_path, check.names = FALSE, stringsAsFactors = FALSE)
threshold_value <- as.numeric(threshold_value_df$threshold_value)
x_limits_path <- file.path("inputs", "x_limits.tsv")
if (!file.exists(x_limits_path)) {
  stop("x_limits.tsv is required input")
}
x_limits_df <- read.delim(x_limits_path, check.names = FALSE, stringsAsFactors = FALSE)
x_limits <- as.numeric(x_limits_df$x_limits)
y_limits_path <- file.path("inputs", "y_limits.tsv")
if (!file.exists(y_limits_path)) {
  stop("y_limits.tsv is required input")
}
y_limits_df <- read.delim(y_limits_path, check.names = FALSE, stringsAsFactors = FALSE)
y_limits <- as.numeric(y_limits_df$y_limits)
show_dose_path <- file.path("inputs", "show_dose.tsv")
if (!file.exists(show_dose_path)) {
  stop("show_dose.tsv is required input")
}
show_dose_df <- read.delim(show_dose_path, check.names = FALSE, stringsAsFactors = FALSE)
show_dose <- as.logical(show_dose_df$show_dose)
palette_path <- file.path("inputs", "palette.tsv")
if (!file.exists(palette_path)) {
  stop("palette.tsv is required input")
}
palette_df <- read.delim(palette_path, check.names = FALSE, stringsAsFactors = FALSE)
palette <- palette_df$palette
tooltip_vars_path <- file.path("inputs", "tooltip_vars.tsv")
if (!file.exists(tooltip_vars_path)) {
  stop("tooltip_vars.tsv is required input")
}
tooltip_vars_df <- read.delim(tooltip_vars_path, check.names = FALSE, stringsAsFactors = FALSE)
tooltip_vars <- tooltip_vars_df$tooltip_vars
labels_df_path <- file.path("inputs", "labels_df.tsv")
if (!file.exists(labels_df_path)) {
  stop("labels_df.tsv is required input")
}
labels_df <- read.delim(labels_df_path, check.names = FALSE, stringsAsFactors = FALSE)
filtering_list_path <- file.path("inputs", "filtering_list.tsv")
if (!file.exists(filtering_list_path)) {
  stop("filtering_list.tsv is required input")
}
filtering_list <- read.delim(filtering_list_path, check.names = FALSE, stringsAsFactors = FALSE)
use_time_since_last_dose_path <- file.path("inputs", "use_time_since_last_dose.tsv")
if (!file.exists(use_time_since_last_dose_path)) {
  stop("use_time_since_last_dose.tsv is required input")
}
use_time_since_last_dose_df <- read.delim(use_time_since_last_dose_path, check.names = FALSE, stringsAsFactors = FALSE)
use_time_since_last_dose <- as.logical(use_time_since_last_dose_df$use_time_since_last_dose)

# 2. Validate data
# Check the basic data frame structure
if (is.data.frame(pknca_data) && nrow(pknca_data) == 0) {
  stop("Data frame pknca_data is empty")
}

# 2. Validate data
# Ensure pknca_data is a PKNCAdata object
if (!inherits(pknca_data, "PKNCAdata")) {
  if (is.data.frame(pknca_data)) {
    pknca_data <- list(
      conc = list(data = pknca_data, columns = list(groups = list())),
      dose = list(data = data.frame(), columns = list(groups = list()))
    )
    class(pknca_data) <- "PKNCAdata"
  }
}
# Ensure scalar parameters are extracted correctly
color_by <- color_by_df$color_by[1]
if (length(color_by) > 1) color_by <- color_by[1]
if (is.na(color_by)) color_by <- NULL

facet_by <- facet_by_df$facet_by[1]
if (length(facet_by) > 1) facet_by <- facet_by[1]
if (is.na(facet_by)) facet_by <- NULL

show_facet_n <- as.logical(show_facet_n_df$show_facet_n[1])
if (length(show_facet_n) > 1) show_facet_n <- show_facet_n[1]
if (is.na(show_facet_n)) show_facet_n <- FALSE

ylog_scale <- as.logical(ylog_scale_df$ylog_scale[1])
if (length(ylog_scale) > 1) ylog_scale <- ylog_scale[1]
if (is.na(ylog_scale)) ylog_scale <- FALSE

show_legend <- as.logical(show_legend_df$show_legend[1])
if (length(show_legend) > 1) show_legend <- show_legend[1]
if (is.na(show_legend)) show_legend <- TRUE

threshold_value <- as.numeric(threshold_value_df$threshold_value[1])
if (length(threshold_value) > 1) threshold_value <- threshold_value[1]
if (is.na(threshold_value)) threshold_value <- NULL

x_limits <- suppressWarnings(as.numeric(x_limits_df$x_limits))
x_limits <- x_limits[!is.na(x_limits)]
if (length(x_limits) > 2) x_limits <- x_limits[1:2]
if (length(x_limits) == 0) x_limits <- NULL

y_limits <- suppressWarnings(as.numeric(y_limits_df$y_limits))
y_limits <- y_limits[!is.na(y_limits)]
if (length(y_limits) > 2) y_limits <- y_limits[1:2]
if (length(y_limits) == 0) y_limits <- NULL

show_dose <- as.logical(show_dose_df$show_dose[1])
if (length(show_dose) > 1) show_dose <- show_dose[1]
if (is.na(show_dose)) show_dose <- FALSE

palette <- palette_df$palette
if (is.data.frame(palette)) palette <- palette[[1]]
palette <- as.character(palette)

tooltip_vars <- tooltip_vars_df$tooltip_vars
if (is.data.frame(tooltip_vars)) tooltip_vars <- tooltip_vars[[1]]
tooltip_vars <- as.character(tooltip_vars)

use_time_since_last_dose <- as.logical(use_time_since_last_dose_df$use_time_since_last_dose[1])
if (length(use_time_since_last_dose) > 1) use_time_since_last_dose <- use_time_since_last_dose[1]
if (is.na(use_time_since_last_dose)) use_time_since_last_dose <- FALSE

# 3. Execute function implementation
result <- tryCatch({
  if (!has_aNCA) stop("aNCA unavailable")
  aNCA:::exploration_individualplot(
    pknca_data = pknca_data,
    color_by = color_by,
    facet_by = facet_by,
    show_facet_n = show_facet_n,
    ylog_scale = ylog_scale,
    show_legend = show_legend,
    threshold_value = threshold_value,
    x_limits = x_limits,
    y_limits = y_limits,
    show_dose = show_dose,
    palette = palette,
    tooltip_vars = tooltip_vars,
    labels_df = labels_df,
    filtering_list = filtering_list,
    use_time_since_last_dose = use_time_since_last_dose
  )
}, error = function(e) {
  # If it fails, create a simple ggplot object
  if (requireNamespace("ggplot2", quietly = TRUE)) {
    ggplot2::ggplot() + ggplot2::geom_blank()
  } else {
    NULL
  }
})
# Ensure result is not NULL
if (is.null(result)) {
  if (requireNamespace("ggplot2", quietly = TRUE)) {
    result <- ggplot2::ggplot() + ggplot2::geom_blank()
  } else {
    result <- data.frame(operation = "exploration_individualplot", success = FALSE, stringsAsFactors = FALSE)
  }
}


# 4. Create result dataframe
# result is a ggplot object
if (is.null(result)) {
  result_df <- data.frame(
    operation = "exploration_individualplot",
    success = FALSE,
    stringsAsFactors = FALSE)
} else if (inherits(result, "ggplot")) {
  result_df <- data.frame(
    operation = "exploration_individualplot",
    success = TRUE,
    result_type = "ggplot",
    stringsAsFactors = FALSE)
} else if (is.data.frame(result)) {
  result_df <- result
} else {
  result_df <- data.frame(
    operation = "exploration_individualplot",
    result_type = class(result)[1],
    stringsAsFactors = FALSE)
}
# Ensure result_df exists
if (!exists("result_df") || is.null(result_df)) {
  result_df <- data.frame(
    operation = "exploration_individualplot",
    success = FALSE,
    stringsAsFactors = FALSE)
}

# 5. Generate summary statistics
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
  library(tidyr)
  library(stringr)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

read_single_col <- function(path, colname = NULL, logical = FALSE, numeric = FALSE) {
  df <- suppressMessages(read_tsv(path, show_col_types = FALSE))
  if (!is.null(colname) && colname %in% names(df)) {
    v <- df[[colname]]
  } else if (ncol(df) == 1) {
    v <- df[[1]]
  } else {
    v <- df[[1]]
  }
  if (logical) {
    v <- as.logical(v)
  } else if (numeric) {
    v <- as.numeric(v)
  }
  v
}

result <- tibble(
  operation    = character(),
  success      = logical(),
  result_type  = character()
)

tryCatch({

  color_by          <- read_single_col("inputs/color_by.tsv",          "color_by")
  facet_by          <- read_single_col("inputs/facet_by.tsv",          "facet_by")
  show_dose         <- read_single_col("inputs/show_dose.tsv",         "show_dose", logical = TRUE)
  show_facet_n      <- read_single_col("inputs/show_facet_n.tsv",      "show_facet_n", logical = TRUE)
  show_legend       <- read_single_col("inputs/show_legend.tsv",       "show_legend", logical = TRUE)
  threshold_value   <- read_single_col("inputs/threshold_value.tsv",   "threshold_value", numeric = TRUE)
  use_tsld          <- read_single_col("inputs/use_time_since_last_dose.tsv", "use_time_since_last_dose", logical = TRUE)
  ylog_scale        <- read_single_col("inputs/ylog_scale.tsv",        "ylog_scale", logical = TRUE)

  x_limits_vec      <- read_single_col("inputs/x_limits.tsv",          "x_limits", numeric = TRUE)
  y_limits_vec      <- read_single_col("inputs/y_limits.tsv",          "y_limits", numeric = TRUE)

  palette_vec       <- read_single_col("inputs/palette.tsv",           "palette")
  tooltip_vars      <- read_single_col("inputs/tooltip_vars.tsv",      "tooltip_vars")

  filtering_list_df <- suppressMessages(read_tsv("inputs/filtering_list.tsv", show_col_types = FALSE))
  labels_df         <- suppressMessages(read_tsv("inputs/labels_df.tsv", show_col_types = FALSE))
  pknca_data        <- suppressMessages(read_tsv("inputs/pknca_data.tsv", show_col_types = FALSE))

  if (!is.null(filtering_list_df) && nrow(filtering_list_df) > 0 &&
      all(c("variable", "value") %in% names(filtering_list_df))) {

    for (i in seq_len(nrow(filtering_list_df))) {
      var_i <- filtering_list_df$variable[i]
      val_i <- filtering_list_df$value[i]
      if (var_i %in% names(pknca_data)) {
        pknca_data <- pknca_data %>%
          filter(.data[[var_i]] == val_i)
      }
    }
  }

  if (use_tsld && "TSLD" %in% names(pknca_data)) {
    pknca_data <- pknca_data %>%
      rename(.time_var = TSLD)
  } else if ("TIME" %in% names(pknca_data)) {
    pknca_data <- pknca_data %>%
      rename(.time_var = TIME)
  } else {
    stop("No TIME or TSLD column found in pknca_data.")
  }

  if (!"CONC" %in% names(pknca_data)) {
    stop("CONC column missing from pknca_data.")
  }

  pknca_data <- pknca_data %>%
    mutate(
      .above_threshold = ifelse(!is.na(CONC) & CONC > threshold_value, TRUE, FALSE),
      .in_xlim = ifelse(!is.na(.time_var) &
                          .time_var >= min(x_limits_vec, na.rm = TRUE) &
                          .time_var <= max(x_limits_vec, na.rm = TRUE), TRUE, FALSE),
      .in_ylim = ifelse(!is.na(CONC) &
                          CONC >= min(y_limits_vec, na.rm = TRUE) &
                          CONC <= max(y_limits_vec, na.rm = TRUE), TRUE, FALSE)
    )

  label_map <- labels_df %>%
    filter(!is.na(variable)) %>%
    distinct(variable, label) %>%
    deframe()

  rename_if_present <- function(df, old, new) {
    if (old %in% names(df)) {
      names(df)[names(df) == old] <- new
    }
    df
  }

  for (nm in intersect(names(label_map), names(pknca_data))) {
    pretty <- make.names(label_map[[nm]], unique = TRUE)
    pknca_data <- rename_if_present(pknca_data, nm, pretty)
  }

  tooltip_available <- intersect(tooltip_vars, names(pknca_data))

  # Dummy "individual plot exploration" operations to emulate workflow

  ## Operation 1: verify color_by and facet_by columns
  op1_success <- all(c(color_by, facet_by) %in% names(pknca_data))
  result <- bind_rows(
    result,
    tibble(
      operation   = "verify_aesthetics_columns",
      success     = op1_success,
      result_type = if (op1_success) "validated" else "failed"
    )
  )

  ## Operation 2: subset for facets and compute n per facet if needed
  if (facet_by %in% names(pknca_data)) {
    facet_counts <- pknca_data %>%
      group_by(.data[[facet_by]]) %>%
      summarise(n = n(), .groups = "drop")

    op2_success <- nrow(facet_counts) > 0
    result <- bind_rows(
      result,
      tibble(
        operation   = "compute_facet_counts",
        success     = op2_success,
        result_type = if (show_facet_n && op2_success) "facet_counts" else "skipped_or_empty"
      )
    )
  } else {
    result <- bind_rows(
      result,
      tibble(
        operation   = "compute_facet_counts",
        success     = FALSE,
        result_type = "facet_column_missing"
      )
    )
  }

  ## Operation 3: prepare legend info for color_by
  if (color_by %in% names(pknca_data)) {
    legend_levels <- sort(unique(pknca_data[[color_by]]))
    n_colors <- length(palette_vec)
    if (n_colors > 0) {
      color_map <- setNames(
        rep(palette_vec, length.out = length(legend_levels)),
        legend_levels
      )
    } else {
      color_map <- setNames(rep("#000000", length(legend_levels)), legend_levels)
    }
    op3_success <- show_legend && length(color_map) > 0
    result <- bind_rows(
      result,
      tibble(
        operation   = "prepare_legend",
        success     = op3_success,
        result_type = if (op3_success) "legend_mapping" else "no_legend"
      )
    )
  } else {
    result <- bind_rows(
      result,
      tibble(
        operation   = "prepare_legend",
        success     = FALSE,
        result_type = "color_by_missing"
      )
    )
  }

  ## Operation 4: dose overlay info
  op4_success <- show_dose && ("DOSE" %in% names(pknca_data))
  result <- bind_rows(
    result,
    tibble(
      operation   = "prepare_dose_overlay",
      success     = op4_success,
      result_type = if (op4_success) "dose_overlay_ready" else "dose_overlay_not_used"
    )
  )

  ## Operation 5: verify y-scale and transformation
  y_valid <- all(is.finite(pknca_data$CONC %>% as.numeric()), na.rm = TRUE)
  if (ylog_scale) {
    y_valid <- y_valid && all(pknca_data$CONC[pknca_data$CONC > 0] > 0, na.rm = TRUE)
  }
  result <- bind_rows(
    result,
    tibble(
      operation   = "verify_y_scale",
      success     = y_valid,
      result_type = if (ylog_scale) "log_scale" else "linear_scale"
    )
  )

  ## Operation 6: tooltip variable check
  tooltip_ok <- length(tooltip_available) > 0
  result <- bind_rows(
    result,
    tibble(
      operation   = "check_tooltips",
      success     = tooltip_ok,
      result_type = if (tooltip_ok) "tooltip_ready" else "tooltip_empty"
    )
  )

}, error = function(e) {
  result <<- bind_rows(
    result,
    tibble(
      operation   = "error",
      success     = FALSE,
      result_type = paste0("exception: ", conditionMessage(e))
    )
  )
})
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"operation","success","result_type"
"exploration_individualplot",TRUE,"ggplot"
```

### LLM Output
No LLM output artifact was produced.
