# Case 119: pharmaverse/admiral/derive_vars_merged

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_merged`
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
Write an R script to perform derive vars merged using the admiral clinical task contract.

Input: dataset.tsv, source.tsv
Output: result.csv


Required columns for result.csv: USUBJID, AVISIT, AVAL, SEX, AGE
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	AVISIT	AVAL
01	Week 1	10
02	Week 1	20
03	Week 1	30
```

#### `source.tsv`


```text
USUBJID	SEX	AGE
01	F	45
02	M	50
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
source <- read.delim(file.path("inputs", "source.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df <- merge(dataset, source, by = "USUBJID", all.x = TRUE, sort = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required package
if (!requireNamespace("admiral", quietly = TRUE)) {
  install.packages("admiral", repos = "https://cloud.r-project.org")
}
library(admiral)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input data
dataset <- read.delim(
  file = "inputs/dataset.tsv",
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE
)

source <- read.delim(
  file = "inputs/source.tsv",
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE
)

# Derive merged variables using admiral-style derive_vars_merged
result <- derive_vars_merged(
  dataset = dataset,
  dataset_add = source,
  by_vars = exprs(USUBJID),
  new_vars = exprs(SEX, AGE)
)

# Keep only required columns
result_out <- result[, c("USUBJID", "AVISIT", "AVAL", "SEX", "AGE")]

# Write output
write.csv(
  result_out,
  file = "outputs/result.csv",
  row.names = FALSE,
  na = ""
)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","AVISIT","AVAL","SEX","AGE"
1,"Week 1",10,"F",45
2,"Week 1",20,"M",50
3,"Week 1",30,NA,NA
```

### LLM Output
#### `result.csv`


```csv
"USUBJID","AVISIT","AVAL","SEX","AGE"
1,"Week 1",10,"F",45
2,"Week 1",20,"M",50
3,"Week 1",30,,
```
