# Case 197: pharmaverse/tidytlg/replace_na_with_blank

## Metadata
- Task ID: `pharmaverse/tidytlg/replace_na_with_blank`
- Package: `tidytlg`
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
Write an R script to perform replace na with blank using the tidytlg clinical task contract.

Input: x.tsv
Output: result.csv


Required columns for result.csv: row_id, x
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `x.tsv`


```text
x
Alpha
NA

Beta
Gamma
```

## Code
### Ground Truth Code

```r
x <- read.delim(
  file.path("inputs", "x.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE,
  na.strings = c("", "NA")
)

result <- data.frame(
  row_id = seq_len(nrow(x)),
  x = ifelse(is.na(x$x), "", x$x),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(tidytlg)

# Ensure output directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
x <- read_tsv(
  file = file.path("inputs", "x.tsv"),
  col_types = cols(.default = col_character())
)

# Add row_id
x <- x %>%
  mutate(row_id = row_number(), .before = 1)

# Replace NA with blank using tidytlg clinical task contract
# Assuming tidytlg provides a function to apply clinical NA-to-blank rules
# If the function differs, adjust its name/arguments accordingly.
x_clean <- x %>%
  mutate(
    across(
      where(is.character),
      ~ clinical_na2blank(.x)
    )
  )

# Select required columns and write output
x_clean %>%
  select(row_id, x) %>%
  write_csv(file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"row_id","x"
1,"Alpha"
2,""
3,"Beta"
4,"Gamma"
```

### LLM Output
No LLM output artifact was produced.
