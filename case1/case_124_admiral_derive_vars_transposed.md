# Case 124: pharmaverse/admiral/derive_vars_transposed

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_transposed`
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
Write an R script to perform derive vars transposed using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, IDVAR, IDVARVAL, LBDTC, LBFAST
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	IDVAR	IDVARVAL	QNAM	QVAL
01	LBSEQ	1	LBDTC	2020-01-01
01	LBSEQ	1	LBFAST	Y
01	LBSEQ	2	LBDTC	2020-01-15
02	LBSEQ	1	LBDTC	2020-02-01
02	LBSEQ	1	LBFAST	N
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
keys <- unique(dataset[c("USUBJID", "IDVAR", "IDVARVAL")])
keys <- keys[order(keys$USUBJID, keys$IDVARVAL), , drop = FALSE]
qnames <- sort(unique(dataset$QNAM))
result_df <- keys
for (qname in qnames) {
  result_df[[qname]] <- NA_character_
}
for (i in seq_len(nrow(dataset))) {
  hit <- result_df$USUBJID == dataset$USUBJID[i] &
    result_df$IDVAR == dataset$IDVAR[i] &
    result_df$IDVARVAL == dataset$IDVARVAL[i]
  result_df[[dataset$QNAM[i]]][hit] <- dataset$QVAL[i]
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(tidyr)
library(readr)
library(stringr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
adsl_long <- read.delim(
  file = file.path("inputs", "dataset.tsv"),
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE
)

# Pivot to wide (derive vars transposed equivalent)
result <- adsl_long %>%
  mutate(
    QNAM = str_trim(QNAM),
    QVAL = str_trim(QVAL)
  ) %>%
  pivot_wider(
    id_cols = c(USUBJID, IDVAR, IDVARVAL),
    names_from = QNAM,
    values_from = QVAL
  ) %>%
  select(USUBJID, IDVAR, IDVARVAL, LBDTC, LBFAST)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","IDVAR","IDVARVAL","LBDTC","LBFAST"
1,"LBSEQ",1,"2020-01-01","Y"
1,"LBSEQ",2,"2020-01-15",
2,"LBSEQ",1,"2020-02-01","N"
```

### LLM Output
#### `result.csv`


```csv
USUBJID,IDVAR,IDVARVAL,LBDTC,LBFAST
1,LBSEQ,1,2020-01-01,Y
1,LBSEQ,2,2020-01-15,
2,LBSEQ,1,2020-02-01,N
```
