# Case 059: pharmaverse/admiral/compute_bmi

## Metadata
- Task ID: `pharmaverse/admiral/compute_bmi`
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
Write an R script to perform compute bmi using the admiral clinical task contract.

Input: height.tsv, weight.tsv
Output: result.csv


Required columns for result.csv: height, weight, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `height.tsv`


```text
height
170
180
165
```

#### `weight.tsv`


```text
weight
70
80
60
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

height <- as.numeric(read.delim(file.path("inputs", "height.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
weight <- as.numeric(read.delim(file.path("inputs", "weight.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
result <- admiral::compute_bmi(height = height, weight = weight)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(height = height, weight = weight, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
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
height_df <- read_tsv(
  file = file.path("inputs", "height.tsv"),
  col_types = cols(),
  show_col_types = FALSE
)

weight_df <- read_tsv(
  file = file.path("inputs", "weight.tsv"),
  col_types = cols(),
  show_col_types = FALSE
)

# Join and compute BMI (height in cm -> m)
result_df <- bind_cols(height_df, weight_df) %>%
  mutate(
    result = weight / ( (height / 100) ^ 2 )
  ) %>%
  select(height, weight, result)

# Write output
write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"height","weight","result"
170,70,24.2214532871972
180,80,24.6913580246914
165,60,22.0385674931129
```

### LLM Output
#### `result.csv`


```csv
height,weight,result
170,70,24.221453287197235
180,80,24.691358024691358
165,60,22.03856749311295
```
