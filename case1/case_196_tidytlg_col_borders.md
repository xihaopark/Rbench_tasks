# Case 196: pharmaverse/tidytlg/col_borders

## Metadata
- Task ID: `pharmaverse/tidytlg/col_borders`
- Package: `tidytlg`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(1, 2) vs llm=(3, 2).
```

## Prompt
```text
Write an R script to perform col borders using the tidytlg clinical task contract.

Input: col.tsv, rows.tsv
Output: result.csv


Required columns for result.csv: col, rows
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `col.tsv`


```text
col
2
```

#### `rows.tsv`


```text
rows
0
2
4
```

## Code
### Ground Truth Code

```r
col <- as.integer(read.delim(file.path("inputs", "col.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$col[1])
rows <- as.integer(read.delim(file.path("inputs", "rows.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$rows)
result <- data.frame(col = col, rows = paste(rows + 1L, collapse = ","), stringsAsFactors = FALSE)
dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(tidyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
col_df  <- readr::read_tsv("inputs/col.tsv", show_col_types = FALSE)
rows_df <- readr::read_tsv("inputs/rows.tsv", show_col_types = FALSE)

# Perform col borders: repeat col value(s) for each row
result <- tidyr::crossing(col_df, rows_df) %>%
  select(col, rows)

# Write output
readr::write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"col","rows"
2,"1,3,5"
```

### LLM Output
#### `result.csv`


```csv
col,rows
2,0
2,2
2,4
```
