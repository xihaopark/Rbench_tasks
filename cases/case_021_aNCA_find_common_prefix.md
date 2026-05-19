# Case 021: pharmaverse/aNCA/find_common_prefix

## Metadata
- Task ID: `pharmaverse/aNCA/find_common_prefix`
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
Write an R script to perform find common prefix using the aNCA clinical task contract.

Input: strings.tsv
Output: result.csv


Required columns for result.csv: strings, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `strings.tsv`


```text
strings
AVAL
AVISITN
USUBJID
```

## Code
### Ground Truth Code

```r
read_table <- function(name) {
  read.delim(file.path("inputs", name), check.names = FALSE, stringsAsFactors = FALSE)
}

common_prefix <- function(strings) {
  strings <- as.character(strings)
  strings <- strings[!is.na(strings)]
  if (length(strings) == 0) return("")
  if (length(strings) == 1) return(strings[[1]])
  first <- min(strings)
  last <- max(strings)
  max_len <- min(nchar(first), nchar(last))
  if (max_len == 0) return("")
  pos <- 1L
  while (pos <= max_len && substr(first, pos, pos) == substr(last, pos, pos)) {
    pos <- pos + 1L
  }
  substr(first, 1L, pos - 1L)
}

strings_df <- read_table("strings.tsv")
strings <- strings_df[[1]]
prefix <- common_prefix(strings)
result <- data.frame(strings = strings, result = rep(prefix, length(strings)), stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Paths
input_path <- file.path("inputs", "strings.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

if (!dir.exists(output_dir)) {
  dir.create(output_dir, recursive = TRUE)
}

# Read input
df <- read_tsv(input_path, col_types = cols(.default = col_character()))

# Function to find longest common prefix
longest_common_prefix <- function(x) {
  if (length(x) == 0) return("")
  if (length(x) == 1) return(x)

  split_chars <- strsplit(x, "", fixed = TRUE)
  min_len <- min(lengths(split_chars))

  prefix_chars <- character(0)
  for (i in seq_len(min_len)) {
    chars_i <- vapply(split_chars, `[`, character(1), i)
    if (length(unique(chars_i)) == 1) {
      prefix_chars <- c(prefix_chars, chars_i[1])
    } else {
      break
    }
  }
  paste(prefix_chars, collapse = "")
}

common_prefix <- longest_common_prefix(df$strings)

result <- df %>%
  mutate(result = common_prefix) %>%
  select(strings, result)

write_csv(result, output_path, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"strings","result"
"AVAL",""
"AVISITN",""
"USUBJID",""
```

### LLM Output
#### `result.csv`


```csv
strings,result
AVAL,
AVISITN,
USUBJID,
```
