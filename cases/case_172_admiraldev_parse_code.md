# Case 172: pharmaverse/admiraldev/parse_code

## Metadata
- Task ID: `pharmaverse/admiraldev/parse_code`
- Package: `admiraldev`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: srcref.
```

## Prompt
```text
Write an R script to perform parse code using the admiraldev clinical task contract.

Input: code.tsv
Output: result.csv


Required columns for result.csv: item, expr, srcref, eval
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `code.tsv`


```text
code
"x <- 1"
"y <- x + 2"
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiraldev))

code <- read.delim(
  file.path("inputs", "code.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)

code_text <- paste(code[[1]], collapse = "\n")
parsed <- admiraldev::parse_code(code_text)

result_df <- do.call(
  rbind,
  lapply(seq_along(parsed), function(i) {
    item <- parsed[[i]]
    data.frame(
      item = i,
      expr = if (is.null(item$expr)) NA_character_ else paste(deparse(item$expr), collapse = " "),
      srcref = paste(item$srcref, collapse = "\n"),
      eval = isTRUE(item$eval),
      stringsAsFactors = FALSE
    )
  })
)

if (length(parsed) == 0) {
  result_df <- data.frame(
    item = integer(),
    expr = character(),
    srcref = character(),
    eval = logical()
  )
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(purrr)
  library(rlang)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
code_df <- readr::read_tsv(
  file = file.path("inputs", "code.tsv"),
  col_types = cols(.default = col_character())
)

# Ensure column name is 'code'
if (!"code" %in% names(code_df)) {
  stop("Input file must contain a 'code' column.")
}

# Helper to safely parse an expression and get srcref
parse_with_srcref <- function(code_str) {
  # Remove surrounding quotes if present
  code_clean <- str_replace_all(code_str, "^['\"]|['\"]$", "")

  con <- textConnection(code_clean)
  on.exit(close(con), add = TRUE)

  exprs <- tryCatch(
    parse(con, keep.source = TRUE),
    error = function(e) NULL
  )

  if (is.null(exprs) || length(exprs) == 0) {
    return(list(expr = NA_character_, srcref = NA_character_))
  }

  expr <- exprs[[1]]

  # Get srcref if available
  sr <- attr(expr, "srcref")
  sr_txt <- if (!is.null(sr)) {
    paste(as.character(sr), collapse = "")
  } else {
    NA_character_
  }

  list(
    expr   = rlang::expr_text(expr),
    srcref = sr_txt
  )
}

# Helper to safely evaluate an expression
eval_expr <- function(code_str, env) {
  code_clean <- str_replace_all(code_str, "^['\"]|['\"]$", "")

  exprs <- tryCatch(
    parse(text = code_clean),
    error = function(e) return(list(error = e))
  )

  if (is.list(exprs) && !is.null(exprs$error)) {
    return(paste0("ERROR: ", conditionMessage(exprs$error)))
  }

  if (length(exprs) == 0) {
    return(NA_character_)
  }

  expr <- exprs[[1]]

  val <- tryCatch(
    eval(expr, envir = env),
    error = function(e) paste0("ERROR: ", conditionMessage(e))
  )

  if (inherits(val, "error") || is.character(val) && str_starts(val, "ERROR: ")) {
    return(as.character(val))
  }

  # Capture evaluation result as deparsed text
  paste(deparse(val), collapse = " ")
}

# Environment for sequential evaluation
eval_env <- new.env(parent = baseenv())

# Process all rows
result_df <- code_df %>%
  mutate(
    item = row_number()
  ) %>%
  rowwise() %>%
  mutate(
    parsed = list(parse_with_srcref(code)),
    expr   = parsed$expr,
    srcref = parsed$srcref,
    eval   = eval_expr(code, eval_env)
  ) %>%
  ungroup() %>%
  select(item, expr, srcref, eval)

# Write output
write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"item","expr","srcref","eval"
1,"x <- 1","x <- 1",TRUE
2,"y <- x + 2","y <- x + 2",TRUE
```

### LLM Output
#### `result.csv`


```csv
item,expr,srcref,eval
1,x <- 1,,1
2,y <- x + 2,,3
```
