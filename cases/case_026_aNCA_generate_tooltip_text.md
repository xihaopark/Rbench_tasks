# Case 026: pharmaverse/aNCA/generate_tooltip_text

## Metadata
- Task ID: `pharmaverse/aNCA/generate_tooltip_text`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: tooltip_text.
```

## Prompt
```text
Write an R script to perform generate tooltip text using the aNCA clinical task contract.

Input: data.tsv, labels_df.tsv, tooltip_vars.tsv, type.tsv
Output: result.csv


Required columns for result.csv: row_id, tooltip_text
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `data.tsv`


```text
id	value	group	category
1	10.5	A	Type1
2	20.3	B	Type2
3	30.7	A	Type1
4	40.2	B	Type2
5	50.9	A	Type1
```

#### `labels_df.tsv`


```text
Dataset	Variable	Label
ADPC	id	Subject ID
ADPC	value	Concentration
ADPC	group	Treatment Group
```

#### `tooltip_vars.tsv`


```text
tooltip_vars
id
value
group
```

#### `type.tsv`


```text
type
ADPC
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages({
  has_anca <- requireNamespace("aNCA", quietly = TRUE)
})

data <- read.delim(
  file.path("inputs", "data.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)
labels_df <- read.delim(
  file.path("inputs", "labels_df.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)
tooltip_vars_df <- read.delim(
  file.path("inputs", "tooltip_vars.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)
tooltip_vars <- tooltip_vars_df$tooltip_vars
type_df <- read.delim(
  file.path("inputs", "type.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)
type <- type_df[[1]][1]

fallback_tooltips <- function(data, labels_df, tooltip_vars, type) {
  if (nrow(data) == 0) {
    return(character())
  }

  tooltip_vars <- tooltip_vars[tooltip_vars %in% names(data)]

  if (length(tooltip_vars) == 0) {
    return(rep("", nrow(data)))
  }

  lookup_label <- function(var_name) {
    required_cols <- c("Dataset", "Variable", "Label")
    if (all(required_cols %in% names(labels_df))) {
      matches <- labels_df$Dataset == type & labels_df$Variable == var_name
      if (any(matches, na.rm = TRUE)) {
        return(labels_df$Label[which(matches)[1]])
      }
    }
    var_name
  }

  tooltip_components <- lapply(tooltip_vars, function(var_name) {
    label <- lookup_label(var_name)
    paste0("<b>", label, "</b>: ", data[[var_name]])
  })

  do.call(paste, c(tooltip_components, sep = "<br>"))
}

if (has_anca && "generate_tooltip_text" %in% getNamespaceExports("aNCA")) {
  tooltips <- aNCA::generate_tooltip_text(data, labels_df, tooltip_vars, type)
} else {
  tooltips <- fallback_tooltips(data, labels_df, tooltip_vars, type)
}

result <- data.frame(
  row_id = seq_along(tooltips),
  tooltip_text = tooltips,
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
unlink(file.path("outputs", c("summary.csv", "result.rds")))
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
data_df       <- readr::read_tsv("inputs/data.tsv", col_types = cols(.default = col_character()))
labels_df     <- readr::read_tsv("inputs/labels_df.tsv", col_types = cols(.default = col_character()))
tooltip_vars  <- readr::read_tsv("inputs/tooltip_vars.tsv", col_types = cols(.default = col_character()))
type_df       <- readr::read_tsv("inputs/type.tsv", col_types = cols(.default = col_character()))

dataset_type <- type_df$type[1]

# Filter labels for selected dataset and tooltip vars
label_map <- labels_df %>%
  filter(Dataset == dataset_type,
         Variable %in% tooltip_vars$tooltip_vars) %>%
  select(Variable, Label)

# Ensure all tooltip_vars are present in data
vars <- intersect(tooltip_vars$tooltip_vars, colnames(data_df))

# Build named vector of labels (fallback to variable name if no label)
label_lookup <- setNames(label_map$Label, label_map$Variable)

format_var_value <- function(var, value) {
  label <- if (!is.na(label_lookup[[var]])) label_lookup[[var]] else var
  paste0(label, ": ", value)
}

# Generate tooltip_text
result <- data_df %>%
  mutate(
    row_id = row_number(),
    tooltip_text = purrr::pmap_chr(
      as.list(.),
      function(...) {
        row <- list(...)
        txts <- vapply(
          vars,
          function(v) format_var_value(v, row[[v]]),
          character(1)
        )
        paste(txts, collapse = "\n")
      }
    )
  ) %>%
  select(row_id, tooltip_text)

# Write output
readr::write_csv(result, "outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"row_id","tooltip_text"
1,"<b>Subject ID</b>: 1<br><b>Concentration</b>: 10.5<br><b>Treatment Group</b>: A"
2,"<b>Subject ID</b>: 2<br><b>Concentration</b>: 20.3<br><b>Treatment Group</b>: B"
3,"<b>Subject ID</b>: 3<br><b>Concentration</b>: 30.7<br><b>Treatment Group</b>: A"
4,"<b>Subject ID</b>: 4<br><b>Concentration</b>: 40.2<br><b>Treatment Group</b>: B"
5,"<b>Subject ID</b>: 5<br><b>Concentration</b>: 50.9<br><b>Treatment Group</b>: A"
```

### LLM Output
#### `result.csv`


```csv
row_id,tooltip_text
1,"Subject ID: 1
Concentration: 10.5
Treatment Group: A"
2,"Subject ID: 2
Concentration: 20.3
Treatment Group: B"
3,"Subject ID: 3
Concentration: 30.7
Treatment Group: A"
4,"Subject ID: 4
Concentration: 40.2
Treatment Group: B"
5,"Subject ID: 5
Concentration: 50.9
Treatment Group: A"
```
