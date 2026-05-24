# Case 187: pharmaverse/metatools/dash_to_eq

## Metadata
- Task ID: `pharmaverse/metatools/dash_to_eq`
- Package: `metatools`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: result.
```

## Prompt
```text
Write an R script to perform dash to eq using the metatools clinical task contract.

Input: id.tsv
Output: result.csv


Required columns for result.csv: id, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `id.tsv`


```text
id
10-20
5-15
0.5-1.5
```

## Code
### Ground Truth Code

```r
id <- as.character(read.delim(file.path("inputs", "id.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$id)
parts <- strsplit(id, "-", fixed = TRUE)
result <- vapply(parts, function(x) paste0("x >= ", x[[1]], " & x <= ", x[[2]]), character(1))
result_df <- data.frame(id = id, result = result, stringsAsFactors = FALSE)
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

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
id_df <- read_tsv(
  file = file.path("inputs", "id.tsv"),
  col_types = cols(
    id = col_character()
  )
)

# Perform dash to eq
result_df <- id_df %>%
  mutate(
    result = str_replace_all(id, "-", "=")
  ) %>%
  select(id, result)

# Write output
write_csv(
  result_df,
  file.path("outputs", "result.csv"),
  na = ""
)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","result"
"10-20","x >= 10 & x <= 20"
"5-15","x >= 5 & x <= 15"
"0.5-1.5","x >= 0.5 & x <= 1.5"
```

### LLM Output
#### `result.csv`


```csv
id,result
10-20,10=20
5-15,5=15
0.5-1.5,0.5=1.5
```
