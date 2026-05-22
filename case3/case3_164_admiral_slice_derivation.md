# Case 164: pharmaverse/admiral/slice_derivation

## Metadata
- Task ID: `pharmaverse/admiral/slice_derivation`
- Package: `admiral`
- Model: `openai/gpt-5.5`
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
library(admiral)
library(dplyr)
library(readr)
library(rlang)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

dataset <- read_tsv("inputs/dataset.tsv", show_col_types = FALSE)
filter_df <- read_tsv("inputs/filter.tsv", show_col_types = FALSE)

filter_condition <- filter_df$filter[1]
filter_expr <- parse_expr(filter_condition)

cutoff_visits <- dataset %>%
  filter(!!filter_expr) %>%
  arrange(USUBJID, AVISITN) %>%
  group_by(USUBJID) %>%
  slice_head(n = 1) %>%
  ungroup() %>%
  transmute(USUBJID, CUTOFF_AVISITN = AVISITN)

result <- derive_vars_merged(
  dataset = dataset,
  dataset_add = cutoff_visits,
  by_vars = exprs(USUBJID),
  new_vars = exprs(CUTOFF_AVISITN)
) %>%
  filter(!is.na(CUTOFF_AVISITN), AVISITN <= CUTOFF_AVISITN) %>%
  select(USUBJID, AVISITN, AVAL)

write_csv(result, "outputs/result.csv")
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
