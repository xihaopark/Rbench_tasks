# Case 120: pharmaverse/admiral/derive_vars_merged_lookup

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_merged_lookup`
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
Write an R script to perform derive vars merged lookup using the admiral clinical task contract.

Input: dataset.tsv, lookup.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVAL, PARAM
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVAL
01	ALT	10
02	AST	20
03	BILI	1.2
```

#### `lookup.tsv`


```text
PARAMCD	PARAM
ALT	Alanine aminotransferase
AST	Aspartate aminotransferase
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
lookup <- read.delim(file.path("inputs", "lookup.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df <- merge(dataset, lookup, by = "PARAMCD", all.x = TRUE, sort = FALSE)
result_df <- result_df[, c("USUBJID", "PARAMCD", "AVAL", "PARAM")]

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input datasets
dataset <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols()
)

lookup <- read_tsv(
  file = file.path("inputs", "lookup.tsv"),
  col_types = cols()
)

# Derive variables via merged lookup
result <- dataset %>%
  left_join(lookup, by = "PARAMCD") %>%
  select(USUBJID, PARAMCD, AVAL, PARAM)

# Write result to outputs
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVAL","PARAM"
1,"ALT",10,"Alanine aminotransferase"
2,"AST",20,"Aspartate aminotransferase"
3,"BILI",1.2,NA
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,AVAL,PARAM
01,ALT,10,Alanine aminotransferase
02,AST,20,Aspartate aminotransferase
03,BILI,1.2,
```
