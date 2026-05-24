# Case 078: pharmaverse/admiral/convert_na_to_blanks

## Metadata
- Task ID: `pharmaverse/admiral/convert_na_to_blanks`
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
Write an R script to perform convert na to blanks using the admiral clinical task contract.

Input: x.tsv
Output: result.csv


Required columns for result.csv: id, x, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `x.tsv`


```text
x
A
NA
B
NA
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

x <- as.character(read.delim(file.path("inputs", "x.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$x)
result <- admiral::convert_na_to_blanks(x)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(id = seq_along(x), x = x, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
x_df <- read_tsv(
  file = file.path("inputs", "x.tsv"),
  col_types = cols(.default = col_character())
)

# Add row id
x_df <- x_df %>%
  mutate(id = row_number())

# Convert string "NA" and actual NA to blanks
x_df <- x_df %>%
  mutate(
    x = if_else(is.na(x) | x == "NA", "", x),
    result = x
  ) %>%
  select(id, x, result)

# Write output
write_csv(x_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","x","result"
1,"A","A"
2,NA,""
3,"B","B"
4,NA,""
```

### LLM Output
#### `result.csv`


```csv
id,x,result
1,A,A
2,,
3,B,B
4,,
```
