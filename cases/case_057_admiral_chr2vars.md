# Case 057: pharmaverse/admiral/chr2vars

## Metadata
- Task ID: `pharmaverse/admiral/chr2vars`
- Package: `admiral`
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
Write an R script to perform chr2vars using the admiral clinical task contract.

Input: chr.tsv
Output: result.csv


Required columns for result.csv: chr, symbol
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `chr.tsv`


```text
chr
STUDYID
USUBJID
PARAMCD
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(rlang))

chr_df <- read.delim(file.path("inputs", "chr.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
chr <- as.character(chr_df[[1]])
result <- admiral::chr2vars(chr)

dir.create("outputs", showWarnings = FALSE)
saveRDS(result, file.path("outputs", "result.rds"))
write.csv(data.frame(chr = chr, symbol = vapply(result, as.character, character(1)), stringsAsFactors = FALSE), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(stringr)
library(fs)

# Create outputs directory if it does not exist
if (!dir_exists("outputs")) {
  dir_create("outputs")
}

# Read input
chr <- read_tsv(
  file = "inputs/chr.tsv",
  col_types = cols(.default = col_character())
)

# Derive chr and symbol according to admiral clinical task contract assumptions
# Here we assume PARAMCD contains the "chr" information, and USUBJID the "symbol" placeholder.
# Adapt mapping logic as needed once real metadata is available.
result <- chr %>%
  transmute(
    chr   = PARAMCD,
    symbol = USUBJID
  )

# Keep only required columns and ensure they exist
result <- result %>%
  select(chr, symbol)

# Write output
write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"chr","symbol"
"STUDYID","STUDYID"
"USUBJID","USUBJID"
"PARAMCD","PARAMCD"
```

### LLM Output
No LLM output artifact was produced.
