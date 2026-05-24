# Case 046: pharmaverse/aNCA/remove_pp_not_requested

## Metadata
- Task ID: `pharmaverse/aNCA/remove_pp_not_requested`
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
Write an R script to perform remove pp not requested using the aNCA clinical task contract.

Input: pknca_res.tsv
Output: result.csv


Required columns for result.csv: subject, parameter, value, requested
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `pknca_res.tsv`


```text
subject	parameter	value	requested
101	CMAX	10.5	TRUE
101	AUC	125.0	FALSE
102	CMAX	12.1	TRUE
```

## Code
### Ground Truth Code

```r
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

read_tsv <- function(name) {
  read.delim(file.path("inputs", name), check.names = FALSE, stringsAsFactors = FALSE)
}

is_false_like <- function(x) {
  value <- trimws(tolower(as.character(x)))
  value %in% c("false", "f", "no", "n", "0")
}

pknca_res <- read_tsv("pknca_res.tsv")
result <- pknca_res
flag_cols <- intersect(c("requested", "is_requested", "include", "analyze"), names(result))
if (length(flag_cols) > 0) {
  keep <- rep(TRUE, nrow(result))
  for (col in flag_cols) {
    keep <- keep & !is_false_like(result[[col]])
  }
  result <- result[keep, , drop = FALSE]
}

write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
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

# Read input
pknca_res <- read_tsv(
  file = file.path("inputs", "pknca_res.tsv"),
  col_types = cols(
    subject   = col_character(),
    parameter = col_character(),
    value     = col_double(),
    requested = col_logical()
  )
)

# Filter to only requested parameters
result <- pknca_res %>%
  filter(requested) %>%
  select(subject, parameter, value, requested)

# Write output
write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"subject","parameter","value","requested"
101,"CMAX",10.5,TRUE
102,"CMAX",12.1,TRUE
```

### LLM Output
#### `result.csv`


```csv
subject,parameter,value,requested
101,CMAX,10.5,TRUE
102,CMAX,12.1,TRUE
```
