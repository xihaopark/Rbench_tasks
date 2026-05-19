# Case 005: pharmaverse/aNCA/add_label_attribute

## Metadata
- Task ID: `pharmaverse/aNCA/add_label_attribute`
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
Write an R script to perform add label attribute using the aNCA clinical task contract.

Input: df.tsv, myres.tsv
Output: result.csv


Required columns for result.csv: id, value, group, category
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `df.tsv`


```text
id	value	group	category
1	10.5	A	Type1
2	20.3	B	Type2
3	30.7	A	Type1
4	40.2	B	Type2
5	50.9	A	Type1
```

#### `myres.tsv`


```text
myres
"value"
"group"
"category"
```

## Code
### Ground Truth Code

```r
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
read_tsv <- function(name) read.delim(file.path("inputs", paste0(name, ".tsv")), check.names = FALSE, stringsAsFactors = FALSE)
df <- read_tsv("df")
# Column labels are R attributes and do not survive CSV serialization; the tabular data is unchanged.
write.csv(df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
df_path <- file.path("inputs", "df.tsv")
myres_path <- file.path("inputs", "myres.tsv")

df <- read_tsv(df_path, col_types = cols())
myres <- read_tsv(myres_path, col_types = cols())

# Extract columns to keep from myres (excluding id, which must always be present)
requested_cols <- myres$myres
requested_cols <- requested_cols[requested_cols != "id"]

# Ensure required base columns
base_cols <- c("id", "value", "group", "category")
cols_to_keep <- unique(c("id", intersect(requested_cols, base_cols)))

# Subset df to required columns; ensure all required result columns exist
result <- df %>%
  select(all_of(cols_to_keep))

# Reorder and ensure all required columns exist in final output
for (col in base_cols) {
  if (!col %in% colnames(result)) {
    result[[col]] <- NA
  }
}
result <- result %>%
  select(all_of(base_cols))

# Write output
write_csv(result, file.path("outputs", "result.csv"))
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
