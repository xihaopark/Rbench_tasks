# Case 027: pharmaverse/aNCA/get_code

## Metadata
- Task ID: `pharmaverse/aNCA/get_code`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: id.
```

## Prompt
```text
Write an R script to perform get code using the aNCA clinical task contract.

Input: output_path.tsv, setts_obj.tsv, template_path.tsv
Output: result.csv


Required columns for result.csv: id, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `output_path.tsv`


```text
output_path
"temp_file.txt"
```

#### `setts_obj.tsv`


```text
setts_obj
clinical_report_settings
```

#### `template_path.tsv`


```text
template_path
"temp_template.pptx"
```

## Code
### Ground Truth Code

```r
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
read_tsv <- function(name) read.delim(file.path("inputs", paste0(name, ".tsv")), check.names = FALSE, stringsAsFactors = FALSE)
scalar <- function(x) if (nrow(x) && ncol(x)) as.character(x[[1]][1]) else NA_character_
output_path <- scalar(read_tsv("output_path"))
write.csv(data.frame(id = 1L, result = output_path, stringsAsFactors = FALSE), file.path("outputs", "result.csv"), row.names = FALSE)
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

# Read inputs
output_path_df <- read_tsv(
  file = file.path("inputs", "output_path.tsv"),
  col_types = cols(.default = col_character())
)

setts_obj_df <- read_tsv(
  file = file.path("inputs", "setts_obj.tsv"),
  col_types = cols(.default = col_character())
)

template_path_df <- read_tsv(
  file = file.path("inputs", "template_path.tsv"),
  col_types = cols(.default = col_character())
)

# Extract values
output_path <- gsub('"', "", output_path_df$output_path[1])
setts_obj <- setts_obj_df$setts_obj[1]
template_path <- gsub('"', "", template_path_df$template_path[1])

# Placeholder "get code using the aNCA clinical task contract"
# In absence of specification, just combine inputs into a single result row
result_df <- tibble(
  id = "aNCA_clinical_task",
  result = paste(
    "output_path:", output_path,
    "| setts_obj:", setts_obj,
    "| template_path:", template_path
  )
)

# Write result
write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","result"
1,"temp_file.txt"
```

### LLM Output
#### `result.csv`


```csv
id,result
aNCA_clinical_task,output_path: temp_file.txt | setts_obj: clinical_report_settings | template_path: temp_template.pptx
```
