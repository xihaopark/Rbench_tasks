# Case 123: pharmaverse/admiral/derive_vars_query

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_query`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: CQ01FL.
```

## Prompt
```text
Write an R script to perform derive vars query using the admiral clinical task contract.

Input: dataset.tsv, dataset_queries.tsv
Output: result.csv


Required columns for result.csv: USUBJID, AETERM, AEREL, AESEV, CQ01FL, CQ02FL
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
QUERY_ID	SRCVAR	TERMCHAR	NEWVAR
Q1	AETERM	HEADACHE	CQ01FL
Q2	AEREL	RELATED	CQ02FL
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
queries <- read.delim(file.path("inputs", "dataset_queries.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
for (i in seq_len(nrow(queries))) {
  src <- queries$SRCVAR[i]
  new_var <- queries$NEWVAR[i]
  result_df[[new_var]] <- ifelse(result_df[[src]] == queries$TERMCHAR[i], "Y", "N")
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(stringr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input datasets
ae <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(.default = col_character())
)

queries <- read_tsv(
  file = file.path("inputs", "dataset_queries.tsv"),
  col_types = cols(.default = col_character())
)

# Initialize result dataset
res <- ae

# Apply queries to derive flag variables
for (i in seq_len(nrow(queries))) {
  q <- queries[i, ]
  srcvar <- q$SRCVAR
  termchar <- q$TERMCHAR
  newvar <- q$NEWVAR

  # Create new variable if it does not exist
  if (!newvar %in% names(res)) {
    res[[newvar]] <- NA_character_
  }

  # Set flag based on query
  res[[newvar]] <- if_else(
    !is.na(res[[srcvar]]) & res[[srcvar]] == termchar,
    "Y",
    res[[newvar]]
  )
}

# Keep only required columns
res_out <- res %>%
  select(USUBJID, AETERM, AEREL, AESEV, CQ01FL, CQ02FL)

# Write result
write_csv(res_out, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","AETERM","AEREL","AESEV","CQ01FL","CQ02FL"
1,"HEADACHE","RELATED","MILD","Y","Y"
2,"NAUSEA","NOT RELATED","MODERATE","N","N"
3,"HEADACHE","RELATED","SEVERE","Y","Y"
```

### LLM Output
#### `result.csv`


```csv
USUBJID,AETERM,AEREL,AESEV,CQ01FL,CQ02FL
01,HEADACHE,RELATED,MILD,Y,Y
02,NAUSEA,NOT RELATED,MODERATE,,
03,HEADACHE,RELATED,SEVERE,Y,Y
```
