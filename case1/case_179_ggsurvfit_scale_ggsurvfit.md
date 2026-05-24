# Case 179: pharmaverse/ggsurvfit/scale_ggsurvfit

## Metadata
- Task ID: `pharmaverse/ggsurvfit/scale_ggsurvfit`
- Package: `ggsurvfit`
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
Write an R script to perform scale ggsurvfit using the ggsurvfit clinical task contract.

Input: x_scales.tsv, y_scales.tsv
Output: result.csv


Required columns for result.csv: scale_classes
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `x_scales.tsv`


```text
expr
list(expand = c(0.015, 0), n.breaks = 8)
```

#### `y_scales.tsv`


```text
expr
list(expand = c(0.025, 0), limits = c(0, 1))
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(ggsurvfit))

parse_scale_list <- function(fname) {
  df <- read.delim(file.path("inputs", fname), check.names = FALSE, stringsAsFactors = FALSE)
  txt <- trimws(as.character(df[[1]][1]))
  value <- eval(parse(text = txt))
  if (!is.list(value)) {
    stop(fname, " must contain an expression that evaluates to a list")
  }
  value
}

x_scales <- parse_scale_list("x_scales.tsv")
y_scales <- parse_scale_list("y_scales.tsv")
result <- ggsurvfit::scale_ggsurvfit(x_scales = x_scales, y_scales = y_scales)

result_df <- data.frame(
  scale_classes = paste(class(result), collapse = "|"),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
})

# paths
x_path <- file.path("inputs", "x_scales.tsv")
y_path <- file.path("inputs", "y_scales.tsv")
out_dir <- "outputs"
out_path <- file.path(out_dir, "result.csv")

if (!dir.exists(out_dir)) {
  dir.create(out_dir, recursive = TRUE)
}

# read inputs
x_tbl <- read_tsv(x_path, show_col_types = FALSE)
y_tbl <- read_tsv(y_path, show_col_types = FALSE)

# combine and extract scale classes (very simple parsing: assume expr starts with function call or list)
get_scale_class <- function(expr_chr) {
  expr_chr <- str_trim(expr_chr)
  # if looks like list(...), call it "list"
  if (str_starts(expr_chr, "list(")) {
    return("list")
  }
  # otherwise, try to grab leading name before "("
  m <- str_match(expr_chr, "^([A-Za-z0-9_.]+)\\s*\\(")
  if (!is.na(m[1, 2])) {
    return(m[1, 2])
  }
  NA_character_
}

all_exprs <- c(x_tbl$expr, y_tbl$expr)
scale_classes <- vapply(all_exprs, get_scale_class, character(1))

result <- tibble(scale_classes = unique(scale_classes[!is.na(scale_classes)]))

write_csv(result, out_path)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"scale_classes"
"scale_ggsurvfit"
```

### LLM Output
No LLM output artifact was produced.
