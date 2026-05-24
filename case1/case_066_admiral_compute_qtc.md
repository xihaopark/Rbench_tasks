# Case 066: pharmaverse/admiral/compute_qtc

## Metadata
- Task ID: `pharmaverse/admiral/compute_qtc`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform compute qtc using the admiral clinical task contract.

Input: method.tsv, qt.tsv, rr.tsv
Output: result.csv


Required columns for result.csv: qt, rr, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `method.tsv`


```text
method
bazett
```

#### `qt.tsv`


```text
qt
380
410
450
```

#### `rr.tsv`


```text
rr
800
1000
1200
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

qt <- as.numeric(read.delim(file.path("inputs", "qt.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
rr <- as.numeric(read.delim(file.path("inputs", "rr.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
method <- tolower(as.character(read.delim(file.path("inputs", "method.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1]))
if (!method %in% c("bazett", "fridericia", "sagie")) method <- "bazett"
result <- switch(method, bazett = qt / sqrt(rr / 1000), fridericia = qt / (rr / 1000)^(1 / 3), sagie = 1000 * (qt / 1000 + 0.154 * (1 - rr / 1000)))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(qt = qt, rr = rr, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
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
method_df <- read_tsv(
  file = file.path("inputs", "method.tsv"),
  col_types = cols(
    method = col_character()
  )
)

qt_df <- read_tsv(
  file = file.path("inputs", "qt.tsv"),
  col_types = cols(
    qt = col_double()
  )
)

rr_df <- read_tsv(
  file = file.path("inputs", "rr.tsv"),
  col_types = cols(
    rr = col_double()
  )
)

# Determine method (first non-NA value)
method <- method_df$method[1]

# Align qt and rr by position (recycle shorter vector if needed)
n <- max(nrow(qt_df), nrow(rr_df))
qt_vec <- rep(qt_df$qt, length.out = n)
rr_vec <- rep(rr_df$rr, length.out = n)

# Compute corrected QT (qtc)
if (tolower(method) == "bazett") {
  result_vec <- qt_vec / sqrt(rr_vec / 1000)
} else {
  stop("Unsupported method: ", method)
}

# Prepare result
result_df <- tibble(
  qt = qt_vec,
  rr = rr_vec,
  result = result_vec
)

# Write output
write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"qt","rr","result"
380,800,424.85291572496
410,1000,410
450,1200,410.791918128875
```

### LLM Output
#### `result.csv`


```csv
qt,rr,result
380,800,424.85291572496004
410,1000,410
450,1200,410.7919181288746
```
