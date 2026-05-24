# Case 072: pharmaverse/admiral/convert_blanks_to_na

## Metadata
- Task ID: `pharmaverse/admiral/convert_blanks_to_na`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: x.
```

## Prompt
```text
Write an R script to perform convert blanks to na using the admiral clinical task contract.

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
""
" "
NA
B
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

x <- as.character(read.delim(file.path("inputs", "x.tsv"), check.names = FALSE, stringsAsFactors = FALSE, blank.lines.skip = FALSE)$x)
result <- admiral::convert_blanks_to_na(x)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(id = seq_along(x), x = x, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(stringr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input
x <- read_tsv(
  file = "inputs/x.tsv",
  col_types = cols(.default = col_character())
)

# Process according to admiral-style convert_blanks_to_na:
# - Trim whitespace
# - Convert "" or strings that become "" after trimming to NA
x_clean <- x %>%
  mutate(
    x = str_trim(x),
    x = if_else(x == "" | x == "NA", NA_character_, x)
  )

# Create result data frame
result <- x_clean %>%
  mutate(
    id = row_number(),
    result = x
  ) %>%
  select(id, x, result)

# Write output
write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","x","result"
1,"A","A"
2,"",NA
3," "," "
4,NA,NA
5,"B","B"
```

### LLM Output
#### `result.csv`


```csv
id,x,result
1,A,A
2,,
3,,
4,,
5,B,B
```
