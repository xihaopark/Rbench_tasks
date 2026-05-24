# Case 042: pharmaverse/aNCA/prepare_plot_data

## Metadata
- Task ID: `pharmaverse/aNCA/prepare_plot_data`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform prepare plot data using the aNCA clinical task contract.

Input: colour_var.tsv, data_conc.tsv, data_dose.tsv, grouping_vars.tsv, labels_df.tsv, plot_conc_data.tsv, shape_var.tsv, tooltip_vars.tsv
Output: result.csv


Required columns for result.csv: USUBJID, ARM, TIME, CONC, DOSE, PARAM, ROUTE, TSLD, BLQFL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `colour_var.tsv`


```text
colour_var
ARM
```

#### `data_conc.tsv`


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

#### `data_dose.tsv`


```text
USUBJID	TIME	DOSE	DOSEU	ROUTE	ARM
101	0	100	mg	oral	100 mg
102	0	100	mg	oral	100 mg
```

#### `grouping_vars.tsv`


```text
grouping_vars
USUBJID
ARM
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

#### `plot_conc_data.tsv`


```text
plot_conc_data
TRUE
```

#### `shape_var.tsv`


```text
shape_var
PARAM
```

#### `tooltip_vars.tsv`


```text
tooltip_vars
USUBJID
TIME
CONC
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)

# 1. Read input data
data_conc_path <- file.path("inputs", "data_conc.tsv")
if (!file.exists(data_conc_path)) {
  stop("data_conc.tsv is required input")
}
data_conc <- read.delim(data_conc_path, check.names = FALSE, stringsAsFactors = FALSE)
data_dose_path <- file.path("inputs", "data_dose.tsv")
if (!file.exists(data_dose_path)) {
  stop("data_dose.tsv is required input")
}
data_dose <- read.delim(data_dose_path, check.names = FALSE, stringsAsFactors = FALSE)
shape_var_path <- file.path("inputs", "shape_var.tsv")
if (!file.exists(shape_var_path)) {
  stop("shape_var.tsv is required input")
}
shape_var_df <- read.delim(shape_var_path, check.names = FALSE, stringsAsFactors = FALSE)
shape_var <- shape_var_df$shape_var
colour_var_path <- file.path("inputs", "colour_var.tsv")
if (!file.exists(colour_var_path)) {
  stop("colour_var.tsv is required input")
}
colour_var_df <- read.delim(colour_var_path, check.names = FALSE, stringsAsFactors = FALSE)
colour_var <- colour_var_df$colour_var
grouping_vars_path <- file.path("inputs", "grouping_vars.tsv")
if (!file.exists(grouping_vars_path)) {
  stop("grouping_vars.tsv is required input")
}
grouping_vars_df <- read.delim(grouping_vars_path, check.names = FALSE, stringsAsFactors = FALSE)
grouping_vars <- grouping_vars_df$grouping_vars
labels_df_path <- file.path("inputs", "labels_df.tsv")
if (!file.exists(labels_df_path)) {
  stop("labels_df.tsv is required input")
}
labels_df <- read.delim(labels_df_path, check.names = FALSE, stringsAsFactors = FALSE)
tooltip_vars_path <- file.path("inputs", "tooltip_vars.tsv")
if (!file.exists(tooltip_vars_path)) {
  stop("tooltip_vars.tsv is required input")
}
tooltip_vars_df <- read.delim(tooltip_vars_path, check.names = FALSE, stringsAsFactors = FALSE)
tooltip_vars <- tooltip_vars_df$tooltip_vars
plot_conc_data_path <- file.path("inputs", "plot_conc_data.tsv")
if (!file.exists(plot_conc_data_path)) {
  stop("plot_conc_data.tsv is required input")
}
plot_conc_data_df <- read.delim(plot_conc_data_path, check.names = FALSE, stringsAsFactors = FALSE)
plot_conc_data <- as.logical(plot_conc_data_df[[1]][1])


# 2. Validate data
# Check the basic data frame structure

# 2. Validate data
if (nrow(data_conc) == 0 || nrow(data_dose) == 0) stop("data_conc or data_dose is empty")

# 3. Execute function implementation
result <- tryCatch({
  if (!has_aNCA) stop("aNCA unavailable")
  aNCA::prepare_plot_data(
    data_conc = data_conc,
    data_dose = data_dose,
    shape_var = if (nrow(shape_var_df) > 0) shape_var_df$shape_var[1] else NULL,
    colour_var = if (nrow(colour_var_df) > 0) colour_var_df$colour_var[1] else NULL,
    grouping_vars = if (nrow(grouping_vars_df) > 0) grouping_vars_df$grouping_vars else NULL,
    labels_df = labels_df,
    tooltip_vars = if (nrow(tooltip_vars_df) > 0) tooltip_vars_df$tooltip_vars else NULL,
    plot_conc_data = plot_conc_data
  )
}, error = function(e) data_conc)
if (is.null(result) || (is.list(result) && !is.data.frame(result))) result <- data_conc


# 4. Create result dataframe
# Combine function results with input data when possible
if (is.vector(result) || is.numeric(result) || is.character(result)) {
  result_df <- data.frame(
    shape_var = shape_var, colour_var = colour_var,
    result = result,
    stringsAsFactors = FALSE)
  } else if (is.data.frame(result)) {
  result_df <- result
} else {
  result_df <- data.frame(result = result, stringsAsFactors = FALSE)
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
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input concentration data
data_conc <- read_tsv(
  file = file.path("inputs", "data_conc.tsv"),
  col_types = cols()
)

# Ensure required columns exist
required_cols <- c("USUBJID", "ARM", "TIME", "CONC", "DOSE", "PARAM", "ROUTE", "TSLD", "BLQFL")
missing_cols <- setdiff(required_cols, names(data_conc))
if (length(missing_cols) > 0) {
  stop("Missing required columns in data_conc.tsv: ", paste(missing_cols, collapse = ", "))
}

# Select and order required columns
result <- data_conc %>%
  select(all_of(required_cols))

# Write result
write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","ARM","TIME","CONC","DOSE","PARAM","ROUTE","TSLD","BLQFL"
101,"100 mg",0,0,100,"AUC","oral",0,"Y"
101,"100 mg",1,34.2,100,"AUC","oral",1,"N"
101,"100 mg",2,61.5,100,"AUC","oral",2,"N"
101,"100 mg",4,38.1,100,"AUC","oral",4,"N"
102,"100 mg",0,0,100,"AUC","oral",0,"Y"
102,"100 mg",1,29.8,100,"AUC","oral",1,"N"
102,"100 mg",2,55.4,100,"AUC","oral",2,"N"
102,"100 mg",4,33.6,100,"AUC","oral",4,"N"
```

### LLM Output
#### `result.csv`


```csv
USUBJID,ARM,TIME,CONC,DOSE,PARAM,ROUTE,TSLD,BLQFL
101,100 mg,0,0,100,AUC,oral,0,Y
101,100 mg,1,34.2,100,AUC,oral,1,N
101,100 mg,2,61.5,100,AUC,oral,2,N
101,100 mg,4,38.1,100,AUC,oral,4,N
102,100 mg,0,0,100,AUC,oral,0,Y
102,100 mg,1,29.8,100,AUC,oral,1,N
102,100 mg,2,55.4,100,AUC,oral,2,N
102,100 mg,4,33.6,100,AUC,oral,4,N
```
