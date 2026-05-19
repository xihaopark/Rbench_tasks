# Case 006: pharmaverse/aNCA/adjust_class_and_length

## Metadata
- Task ID: `pharmaverse/aNCA/adjust_class_and_length`
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
Write an R script to perform adjust class and length using the aNCA clinical task contract.

Input: adjust_length.tsv, df.tsv, metadata.tsv
Output: result.csv


Required columns for result.csv: id, value, group, category
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `adjust_length.tsv`


```text
adjust_length
"value"
"group"
"category"
```

#### `df.tsv`


```text
id	value	group	category
1	10.5	A	Type1
2	20.3	B	Type2
3	30.7	A	Type1
4	40.2	B	Type2
5	50.9	A	Type1
```

#### `metadata.tsv`


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
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
read_tsv <- function(name) read.delim(file.path("inputs", paste0(name, ".tsv")), check.names = FALSE, stringsAsFactors = FALSE)
df <- read_tsv("df")
# Class and length metadata adjustments are not represented in CSV; preserve the aligned data table.
write.csv(df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
adjust <- read_tsv("inputs/adjust_length.tsv", show_col_types = FALSE)
df <- read_tsv("inputs/df.tsv", show_col_types = FALSE)
meta <- read_tsv("inputs/metadata.tsv", show_col_types = FALSE)

# Ensure required columns exist and classes/lengths are adjusted based on metadata
common_cols <- intersect(colnames(df), colnames(meta))
df <- df[, common_cols]
meta <- meta[, common_cols]

# Coerce df column classes to match metadata
for (col in common_cols) {
  target_class <- class(meta[[col]])[1]
  if (!inherits(df[[col]], target_class)) {
    if (target_class %in% c("numeric", "integer", "double")) {
      df[[col]] <- suppressWarnings(as.numeric(df[[col]]))
    } else if (target_class == "character") {
      df[[col]] <- as.character(df[[col]])
    } else if (target_class == "factor") {
      df[[col]] <- factor(df[[col]], levels = levels(meta[[col]]))
    } else if (target_class == "logical") {
      df[[col]] <- as.logical(df[[col]])
    } else {
      mode(df[[col]]) <- mode(meta[[col]])
    }
  }
}

# Adjust length according to adjust_length contract (use metadata as reference)
# Here we align df to metadata by id, keeping only ids present in metadata
if ("id" %in% common_cols) {
  df <- df %>%
    semi_join(meta %>% select(id), by = "id")
}

# Keep only required output columns
required_cols <- c("id", "value", "group", "category")
result <- df %>% select(all_of(required_cols))

# Write result
write_csv(result, "outputs/result.csv")
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
