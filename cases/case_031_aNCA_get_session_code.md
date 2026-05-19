# Case 031: pharmaverse/aNCA/get_session_code

## Metadata
- Task ID: `pharmaverse/aNCA/get_session_code`
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
Write an R script to perform get session code using the aNCA clinical task contract.

Input: output_path.tsv, session.tsv, template_path.tsv
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

#### `session.tsv`


```text
session
clinical_report_session
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

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
output_path_df <- read_tsv("inputs/output_path.tsv", show_col_types = FALSE)
session_df     <- read_tsv("inputs/session.tsv", show_col_types = FALSE)
template_df    <- read_tsv("inputs/template_path.tsv", show_col_types = FALSE)

# Extract values
output_path   <- output_path_df$output_path[1]
session_code  <- session_df$session[1]
template_path <- template_df$template_path[1]

# Here we "perform get session code using the aNCA clinical task contract"
# For this task, we assume the session code itself is the required result
result_df <- tibble(
  id     = "session_code",
  result = session_code
)

# Write result
write_csv(result_df, "outputs/result.csv", na = "")
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
session_code,clinical_report_session
```
