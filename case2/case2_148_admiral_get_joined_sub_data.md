# Case 148: pharmaverse/admiral/get_joined_sub_data

## Metadata
- Task ID: `pharmaverse/admiral/get_joined_sub_data`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`
- Failure pattern: `package_api_hallucination`
- Failure pattern note: unused arguments (join_vars = expr_c(join_vars, intersect(unname(extract_vars(new_vars)), chr2vars(colnames(dataset_add)))), first_cond_lower = !!first_cond_lower, first_cond_upper = !!first_cond_upper, order = order, tmp_obs_nr_var = !!tmp_obs_nr_var, filter_add = !!filter_add, filter_join = !!filter_join, check_type = check_type)

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script for an R package task: perform get joined sub data using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: Use left.tsv as the primary dataset and left-join right.tsv by USUBJID, preserving unmatched left rows.


Required columns for result.csv: USUBJID, AVAL, TRT01A
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `left.tsv`

```text
USUBJID	AVAL
01	10
02	20
03	30
```

#### `right.tsv`

```text
USUBJID	TRT01A
01	Drug A
02	Placebo
```

## Code
### Ground Truth Code

```r
left <- read.delim(file.path("inputs", "left.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
right <- read.delim(file.path("inputs", "right.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df <- merge(left, right, by = "USUBJID", all.x = TRUE, sort = FALSE)

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
left <- readr::read_tsv(
  file = file.path("inputs", "left.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    AVAL    = col_double()
  )
)

right <- readr::read_tsv(
  file = file.path("inputs", "right.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    TRT01A  = col_character()
  )
)

# Perform left join using admiral helper
result <- admiral::derive_vars_joined(
  dataset = left,
  dataset_add = right,
  by_vars = exprs(USUBJID),
  join_type = "left",
  filter_add = TRUE
) %>%
  select(USUBJID, AVAL, TRT01A)

# Write output
readr::write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","AVAL","TRT01A"
1,10,"Drug A"
2,20,"Placebo"
3,30,NA
```

### LLM Output
No LLM output artifact was produced.
