# Case 008: pharmaverse/aNCA/apply_labels

## Metadata
- Task ID: `pharmaverse/aNCA/apply_labels`
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
Write an R script to perform apply labels using the aNCA clinical task contract.

Input: data.tsv, labels_df.tsv, type.tsv
Output: result.csv


Required columns for result.csv: id, value, group, category
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `data.tsv`


```text
id	value	group	category
1	10.5	A	Type1
2	20.3	B	Type2
3	30.7	A	Type1
4	40.2	B	Type2
5	50.9	A	Type1
```

#### `labels_df.tsv`


```text
Dataset	Variable	Label
ADPC	id	Subject ID
ADPC	value	Concentration
ADPC	group	Treatment Group
ADPC	category	Analyte Category
```

#### `type.tsv`


```text
type
ADPC
```

## Code
### Ground Truth Code

```r
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
read_tsv <- function(name) read.delim(file.path("inputs", paste0(name, ".tsv")), check.names = FALSE, stringsAsFactors = FALSE)
data <- read_tsv("data")
# Applied labels are R attributes and do not survive CSV serialization; the tabular data is unchanged.
write.csv(data, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required package
suppressWarnings({
  if (!requireNamespace("readr", quietly = TRUE)) {
    install.packages("readr", repos = "https://cloud.r-project.org")
  }
  if (!requireNamespace("dplyr", quietly = TRUE)) {
    install.packages("dplyr", repos = "https://cloud.r-project.org")
  }
})

library(readr)
library(dplyr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input files
data_path   <- file.path("inputs", "data.tsv")
labels_path <- file.path("inputs", "labels_df.tsv")
type_path   <- file.path("inputs", "type.tsv")

data_df   <- read_tsv(data_path, col_types = cols())
labels_df <- read_tsv(labels_path, col_types = cols())
type_df   <- read_tsv(type_path, col_types = cols())

# Determine dataset type
current_type <- type_df$type[1]

# Filter labels for current dataset
labels_filtered <- labels_df %>%
  filter(Dataset == current_type)

# Ensure only required columns in result
required_cols <- c("id", "value", "group", "category")
data_df <- data_df %>% select(all_of(required_cols))

# Optionally, you could apply labels as attributes (not visible in CSV)
for (var in required_cols) {
  label_val <- labels_filtered %>%
    filter(Variable == var) %>%
    pull(Label)
  if (length(label_val) == 1) {
    attr(data_df[[var]], "label") <- label_val
  }
}

# Write output
write_csv(data_df, file.path("outputs", "result.csv"))
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
