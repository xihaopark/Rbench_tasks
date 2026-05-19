# Case 152: pharmaverse/admiral/get_vars_query

## Metadata
- Task ID: `pharmaverse/admiral/get_vars_query`
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
Write an R script to perform get vars query using the admiral clinical task contract.

Input: dataset.tsv, dataset_queries.tsv
Output: result.csv


Required columns for result.csv: QUERY_ID, USUBJID, SRCVAR, SRCVAL, TERMCHAR
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	AETERM	AEREL	AESEV
01	HEADACHE	RELATED	MILD
02	NAUSEA	NOT RELATED	MODERATE
03	HEADACHE	RELATED	SEVERE
```

#### `dataset_queries.tsv`


```text
QUERY_ID	SRCVAR	TERMCHAR
Q1	AETERM	HEADACHE
Q2	AEREL	RELATED
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
queries <- read.delim(file.path("inputs", "dataset_queries.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
pieces <- lapply(seq_len(nrow(queries)), function(i) {
  q <- queries[i, , drop = FALSE]
  src <- q$SRCVAR[1]
  hit <- dataset[[src]] == q$TERMCHAR[1]
  data.frame(QUERY_ID = q$QUERY_ID[1], USUBJID = dataset$USUBJID[hit], SRCVAR = src, SRCVAL = dataset[[src]][hit], TERMCHAR = q$TERMCHAR[1], stringsAsFactors = FALSE)
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

# Read input datasets
dataset <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(.default = col_character())
)

queries <- read_tsv(
  file = file.path("inputs", "dataset_queries.tsv"),
  col_types = cols(.default = col_character())
)

# Perform get vars query
results_list <- lapply(seq_len(nrow(queries)), function(i) {
  q <- queries[i, ]
  var <- q$SRCVAR
  term <- q$TERMCHAR

  if (!var %in% names(dataset)) {
    return(NULL)
  }

  ds_sub <- dataset %>%
    filter(.data[[var]] == term) %>%
    mutate(
      QUERY_ID = q$QUERY_ID,
      SRCVAR   = var,
      SRCVAL   = .data[[var]],
      TERMCHAR = term,
      .keep = "all"
    ) %>%
    select(QUERY_ID, USUBJID, SRCVAR, SRCVAL, TERMCHAR)

  if (nrow(ds_sub) == 0) {
    return(NULL)
  }

  ds_sub
})

result <- bind_rows(results_list)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"QUERY_ID","USUBJID","SRCVAR","SRCVAL","TERMCHAR"
"Q1",1,"AETERM","HEADACHE","HEADACHE"
"Q1",3,"AETERM","HEADACHE","HEADACHE"
"Q2",1,"AEREL","RELATED","RELATED"
"Q2",3,"AEREL","RELATED","RELATED"
```

### LLM Output
#### `result.csv`


```csv
QUERY_ID,USUBJID,SRCVAR,SRCVAL,TERMCHAR
Q1,01,AETERM,HEADACHE,HEADACHE
Q1,03,AETERM,HEADACHE,HEADACHE
Q2,01,AEREL,RELATED,RELATED
Q2,03,AEREL,RELATED,RELATED
```
