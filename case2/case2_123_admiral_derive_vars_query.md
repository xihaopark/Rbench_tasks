# Case 123: pharmaverse/admiral/derive_vars_query

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_query`
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

library(readr)
library(dplyr)
library(tidyr)
library(stringr)
library(admiral)

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

# Ensure required columns exist
required_ae <- c("USUBJID", "AETERM", "AEREL", "AESEV")
if (!all(required_ae %in% names(ae))) {
  stop("Input AE dataset is missing required columns.")
}

required_queries <- c("QUERY_ID", "SRCVAR", "TERMCHAR", "NEWVAR")
if (!all(required_queries %in% names(queries))) {
  stop("Input queries dataset is missing required columns.")
}

# Initialize result data
result <- ae

# For each query, derive the corresponding flag variable using admiral-style derivation
for (i in seq_len(nrow(queries))) {
  q <- queries[i, ]

  src_var <- q$SRCVAR
  term_char <- q$TERMCHAR
  new_var <- q$NEWVAR

  # Skip if source variable is not in the dataset
  if (!src_var %in% names(result)) next

  # Build condition: source variable equals TERMCHAR
  cond <- expr_text(!!sym(src_var) == !!term_char)

  # Use admiral::derive_var_flagvisits-like pattern via derive_vars_merged
  # Here, we define a simple condition-based derivation using dplyr within a loop
  result <- result %>%
    mutate(
      !!sym(new_var) := if_else(
        .data[[src_var]] == term_char,
        "Y",
        "N"
      )
    )
}

# Keep only required columns for output
required_output <- c("USUBJID", "AETERM", "AEREL", "AESEV",
                     "CQ01FL", "CQ02FL")

# Ensure all required flag columns exist, even if no matching query was applied
for (v in required_output) {
  if (!v %in% names(result)) {
    result[[v]] <- NA_character_
  }
}

result_out <- result %>%
  select(all_of(required_output))

# Write result to CSV
write_csv(result_out, file.path("outputs", "result.csv"), na = "")
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
No LLM output artifact was produced.
