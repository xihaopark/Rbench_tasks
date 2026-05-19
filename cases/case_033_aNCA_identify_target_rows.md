# Case 033: pharmaverse/aNCA/identify_target_rows

## Metadata
- Task ID: `pharmaverse/aNCA/identify_target_rows`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(2, 3) vs llm=(10, 3).
```

## Prompt
```text
Write an R script to perform identify target rows using the aNCA clinical task contract.

Input: after.tsv, data.tsv, target_groups.tsv, target_impute.tsv, target_params.tsv
Output: result.csv


Required columns for result.csv: target_params, after, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `after.tsv`


```text
after
42
```

#### `data.tsv`


```text
id	value	group	category
1	10.5	A	Type1
2	20.3	B	Type2
3	30.7	A	Type1
4	40.2	B	Type2
5	50.9	A	Type1
```

#### `target_groups.tsv`


```text
target_groups
"ARM"
"PARAM"
```

#### `target_impute.tsv`


```text
target_impute
start_conc0
```

#### `target_params.tsv`


```text
target_params
"AUC"
"CMAX"
```

## Code
### Ground Truth Code

```r
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
read_tsv <- function(name) read.delim(file.path("inputs", paste0(name, ".tsv")), check.names = FALSE, stringsAsFactors = FALSE)
target_params <- as.character(read_tsv("target_params")[[1]])
after <- read_tsv("after")[[1]][1]
write.csv(data.frame(target_params = target_params, after = rep(after, length(target_params)), result = TRUE, stringsAsFactors = FALSE), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(data.table)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

after <- fread("inputs/after.tsv")
data <- fread("inputs/data.tsv")
target_groups <- fread("inputs/target_groups.tsv")
target_impute <- fread("inputs/target_impute.tsv")
target_params <- fread("inputs/target_params.tsv")

target_params_vec <- target_params[[1]]
after_val <- after[[1]][1]

result_dt <- data.table(
  target_params = rep(target_params_vec, each = nrow(data)),
  after = after_val,
  result = NA_character_
)

fwrite(result_dt, "outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"target_params","after","result"
"AUC",42,TRUE
"CMAX",42,TRUE
```

### LLM Output
#### `result.csv`


```csv
target_params,after,result
AUC,42,
AUC,42,
AUC,42,
AUC,42,
AUC,42,
CMAX,42,
CMAX,42,
CMAX,42,
CMAX,42,
CMAX,42,
```
