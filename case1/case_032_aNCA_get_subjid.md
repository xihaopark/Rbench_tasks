# Case 032: pharmaverse/aNCA/get_subjid

## Metadata
- Task ID: `pharmaverse/aNCA/get_subjid`
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
Write an R script to perform get subjid using the aNCA clinical task contract.

Input: data.tsv
Output: result.csv


Required columns for result.csv: id, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `data.tsv`


```text
SUBJID	USUBJID	STUDYID
1	1	1
2	2	2
3	3	3
4	4	4
5	5	5
```

## Code
### Ground Truth Code

```r
read_tsv <- function(name) {
  path <- file.path("inputs", name)
  if (!file.exists(path)) {
    stop(name, " is required")
  }
  read.delim(path, check.names = FALSE, stringsAsFactors = FALSE)
}

first_column <- function(df) {
  if (ncol(df) == 0) {
    return(character())
  }
  df[[1]]
}

scalar_numeric <- function(df) {
  values <- suppressWarnings(as.numeric(first_column(df)))
  values <- values[!is.na(values)]
  if (length(values) == 0) NA_real_ else values[[1]]
}

write_result <- function(result) {
  dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
  write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "NA")
}

data <- read_tsv("data.tsv")
if ("SUBJID" %in% names(data)) {
  subject <- data$SUBJID
} else if (all(c("USUBJID", "STUDYID") %in% names(data))) {
  subject <- sub(paste0("^", data$STUDYID, "[-_ ]?"), "", as.character(data$USUBJID))
} else if ("USUBJID" %in% names(data)) {
  subject <- data$USUBJID
} else {
  subject <- rep(NA_character_, nrow(data))
}
result <- data.frame(id = seq_along(subject), result = subject)
write_result(result)
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

# Read input
data <- read_tsv(
  file = file.path("inputs", "data.tsv"),
  col_types = cols(.default = col_character())
)

# Derive result: get SUBJID as subjid
result <- data %>%
  transmute(
    id = SUBJID,
    result = SUBJID
  )

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","result"
1,1
2,2
3,3
4,4
5,5
```

### LLM Output
#### `result.csv`


```csv
id,result
1,1
2,2
3,3
4,4
5,5
```
