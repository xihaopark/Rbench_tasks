# Case 164: pharmaverse/admiral/slice_derivation

## Metadata
- Task ID: `pharmaverse/admiral/slice_derivation`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output files were produced, but the schema or output shape differed from the ground truth.

Main signal:
result.csv: Shape mismatch: ref=(4, 3) vs llm=(2, 3)
```

## Prompt
```text
Write an R script for an R package task: perform slice derivation using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: Within each subject, use the filter condition to identify the first selected visit, then keep rows through that selected visit rather than only the matching row.


Required columns for result.csv: USUBJID, AVISITN, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
USUBJID	AVISITN	AVAL
01	1	10
01	2	12
02	1	9
02	2	11
```

#### `filter.tsv`

```text
filter
AVISITN == 1
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(rlang))

dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
filter_expr <- read.delim(file.path("inputs", "filter.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$filter[1]
slice <- admiral::derivation_slice(!!parse_expr(filter_expr))
result <- admiral::slice_derivation(dataset, derivation = dplyr::filter, slice)
dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(readr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input data
dataset <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols()
)

flt <- read_tsv(
  file = file.path("inputs", "filter.tsv"),
  col_types = cols()
)

# Extract filter condition as character
filter_condition <- flt$filter[1]

# Apply filter within each subject to identify first selected visit
# Then keep all rows up to and including that visit
result <- dataset %>%
  group_by(USUBJID) %>%
  arrange(AVISITN, .by_group = TRUE) %>%
  mutate(
    .match = !! rlang::parse_expr(filter_condition),
    .first_match_visit = if_else(any(.match), min(AVISITN[.match]), NA_real_)
  ) %>%
  filter(!is.na(.first_match_visit) & AVISITN <= .first_match_visit) %>%
  ungroup() %>%
  select(USUBJID, AVISITN, AVAL)

# Write result
write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","AVISITN","AVAL"
1,1,10
2,1,9
1,2,12
2,2,11
```

### LLM Output
#### `result.csv`

```csv
USUBJID,AVISITN,AVAL
01,1,10
02,1,9
```
