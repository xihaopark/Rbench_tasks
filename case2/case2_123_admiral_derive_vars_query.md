# Case 123: pharmaverse/admiral/derive_vars_query

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_query`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
```

## Prompt
```text
Write an R script for an R package task: perform derive vars query using the admiral package. The function(s) you use to solve the task must come from the admiral package.

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

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input data
adsl <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(.default = col_character())
)

queries <- read_tsv(
  file = file.path("inputs", "dataset_queries.tsv"),
  col_types = cols(.default = col_character())
)

# Apply each query using admiral's derive_var_merged_exist_flag()
# to generate Y/N flags per condition
out <- adsl

for (i in seq_len(nrow(queries))) {
  src_var <- queries$SRCVAR[i]
  termchar <- queries$TERMCHAR[i]
  new_var <- queries$NEWVAR[i]

  out <- derive_var_merged_exist_flag(
    dataset = out,
    dataset_add = out,
    by_vars = exprs(USUBJID, AETERM, AEREL, AESEV),
    condition = !!rlang::sym(src_var) == termchar,
    new_var = !!rlang::sym(new_var),
    true_value = "Y",
    false_value = "N"
  )
}

# Keep only required columns in the specified order
result <- out %>%
  select(USUBJID, AETERM, AEREL, AESEV, CQ01FL, CQ02FL)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
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
