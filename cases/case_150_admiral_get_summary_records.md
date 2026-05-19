# Case 150: pharmaverse/admiral/get_summary_records

## Metadata
- Task ID: `pharmaverse/admiral/get_summary_records`
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
Write an R script to perform get summary records using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, N, MEAN, MIN, MAX
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVAL
01	ALT	20
01	ALT	30
01	AST	18
02	ALT	40
02	ALT	50
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
keys <- unique(dataset[c("USUBJID", "PARAMCD")])
pieces <- lapply(seq_len(nrow(keys)), function(i) {
  hit <- dataset$USUBJID == keys$USUBJID[i] & dataset$PARAMCD == keys$PARAMCD[i]
  values <- dataset$AVAL[hit]
  data.frame(USUBJID = keys$USUBJID[i], PARAMCD = keys$PARAMCD[i], N = length(values), MEAN = mean(values), MIN = min(values), MAX = max(values), stringsAsFactors = FALSE)
})
result_df <- do.call(rbind, pieces)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input
adsl <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    PARAMCD = col_character(),
    AVAL = col_double()
  )
)

# Derive summary records
result <- adsl %>%
  group_by(USUBJID, PARAMCD) %>%
  summarise(
    N    = n(),
    MEAN = mean(AVAL, na.rm = TRUE),
    MIN  = min(AVAL, na.rm = TRUE),
    MAX  = max(AVAL, na.rm = TRUE),
    .groups = "drop"
  ) %>%
  arrange(USUBJID, PARAMCD)

# Write output
write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","N","MEAN","MIN","MAX"
1,"ALT",2,25,20,30
1,"AST",1,18,18,18
2,"ALT",2,45,40,50
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,N,MEAN,MIN,MAX
01,ALT,2,25,20,30
01,AST,1,18,18,18
02,ALT,2,45,40,50
```
