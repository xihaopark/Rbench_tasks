# Case 052: pharmaverse/aNCA/validate_pk

## Metadata
- Task ID: `pharmaverse/aNCA/validate_pk`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform validate PK using the aNCA clinical task contract.

Input: pk_data.tsv
Output: result.csv


Required columns for result.csv: id, value, group, category
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `pk_data.tsv`


```text
id	value	group	category
1	10.5	A	Type1
2	20.3	B	Type2
3	30.7	A	Type1
4	40.2	B	Type2
5	50.9	A	Type1
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages({
  has_anca <- requireNamespace("aNCA", quietly = TRUE)
})

pk_data <- read.delim(
  file.path("inputs", "pk_data.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)

if (has_anca && "validate_pk" %in% getNamespaceExports("aNCA")) {
  result <- aNCA::validate_pk(pk_data)
} else {
  if (!is.data.frame(pk_data)) {
    stop("pk_data must be a data.frame")
  }
  if (nrow(pk_data) == 0) {
    stop("pk_data must contain at least one row")
  }
  result <- pk_data
}

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
unlink(file.path("outputs", c("summary.csv", "result.rds")))
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
pk_path <- file.path("inputs", "pk_data.tsv")
pk_data <- read_tsv(pk_path, col_types = cols())

# Basic PK validation according to aNCA clinical task contract could include:
# - Checking required columns
# - Ensuring no missing IDs
# - Ensuring numeric values for PK measurements
# - Keeping structure required for downstream aNCA

required_cols <- c("id", "value", "group", "category")
missing_cols <- setdiff(required_cols, colnames(pk_data))
if (length(missing_cols) > 0) {
  stop(paste("Missing required columns:", paste(missing_cols, collapse = ", ")))
}

# Coerce/validate columns
pk_validated <- pk_data %>%
  mutate(
    id = as.character(id),
    value = as.numeric(value),
    group = as.character(group),
    category = as.character(category)
  ) %>%
  filter(!is.na(id), !is.na(value), !is.na(group), !is.na(category))

# Write result
out_path <- file.path("outputs", "result.csv")
write_csv(pk_validated[, required_cols], out_path)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","value","group","category"
1,10.5,"A","Type1"
2,20.3,"B","Type2"
3,30.7,"A","Type1"
4,40.2,"B","Type2"
5,50.9,"A","Type1"
```

### LLM Output
#### `result.csv`


```csv
id,value,group,category
1,10.5,A,Type1
2,20.3,B,Type2
3,30.7,A,Type1
4,40.2,B,Type2
5,50.9,A,Type1
```
