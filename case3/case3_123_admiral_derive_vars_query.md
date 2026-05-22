# Case 123: pharmaverse/admiral/derive_vars_query

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_query`
- Package: `admiral`
- Model: `openai/gpt-5.5`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
```

## Prompt
```text
Write an R script for an R package task: perform derive vars query using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: Evaluate each condition from dataset_queries.tsv against every record and create the corresponding Y/N flag column.


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

library(admiral)
library(dplyr)
library(readr)
library(rlang)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

dataset <- read_tsv(
  file = "inputs/dataset.tsv",
  col_types = cols(.default = col_character()),
  show_col_types = FALSE
)

dataset_queries <- read_tsv(
  file = "inputs/dataset_queries.tsv",
  col_types = cols(.default = col_character()),
  show_col_types = FALSE
)

result <- dataset

for (i in seq_len(nrow(dataset_queries))) {
  srcvar <- dataset_queries$SRCVAR[i]
  termchar <- dataset_queries$TERMCHAR[i]
  newvar <- dataset_queries$NEWVAR[i]

  result <- result %>%
    mutate(
      !!sym(newvar) := if_else(
        .data[[srcvar]] == termchar,
        "Y",
        "N",
        missing = "N"
      )
    )
}

result <- result %>%
  select(USUBJID, AETERM, AEREL, AESEV, CQ01FL, CQ02FL)

write_csv(result, "outputs/result.csv")
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
02,NAUSEA,NOT RELATED,MODERATE,N,N
03,HEADACHE,RELATED,SEVERE,Y,Y
```
