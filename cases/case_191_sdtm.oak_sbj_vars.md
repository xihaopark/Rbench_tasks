# Case 191: pharmaverse/sdtm.oak/sbj_vars

## Metadata
- Task ID: `pharmaverse/sdtm.oak/sbj_vars`
- Package: `sdtm.oak`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(3, 2) vs llm=(0, 2).
```

## Prompt
```text
Write an R script to perform sbj vars using the sdtm.oak clinical task contract.

Input: none
Output: result.csv


Required columns for result.csv: position, variable
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
No input files.

## Code
### Ground Truth Code

```r
vars <- c("STUDYID", "USUBJID", "SUBJID")
result <- data.frame(position = seq_along(vars), variable = vars, stringsAsFactors = FALSE)
dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Helper to safely list and read SDTM-like files
sdtm_files <- list.files("inputs", pattern = "\\.csv$|\\.xpt$|\\.sas7bdat$|\\.tsv$",
                         full.names = TRUE, ignore.case = TRUE)

# Initialize empty data frame for variables
vars_df <- tibble(position = integer(), variable = character())

# Function to extract variables from a single file
extract_vars_from_file <- function(fpath) {
  fname <- basename(fpath)
  dom <- tools::file_path_sans_ext(fname)

  df <- tryCatch({
    if (grepl("\\.csv$", fpath, ignore.case = TRUE)) {
      suppressWarnings(readr::read_csv(fpath, col_types = cols(.default = col_guess())))
    } else if (grepl("\\.tsv$", fpath, ignore.case = TRUE)) {
      suppressWarnings(readr::read_tsv(fpath, col_types = cols(.default = col_guess())))
    } else if (grepl("\\.xpt$", fpath, ignore.case = TRUE)) {
      if (!requireNamespace("haven", quietly = TRUE)) return(NULL)
      haven::read_xpt(fpath)
    } else if (grepl("\\.sas7bdat$", fpath, ignore.case = TRUE)) {
      if (!requireNamespace("haven", quietly = TRUE)) return(NULL)
      haven::read_sas(fpath)
    } else {
      NULL
    }
  }, error = function(e) NULL)

  if (is.null(df)) return(NULL)

  tibble(
    variable = names(df)
  )
}

# Collect variables from all SDTM-like files
var_list <- lapply(sdtm_files, extract_vars_from_file)
var_list <- var_list[!vapply(var_list, is.null, logical(1))]

if (length(var_list) > 0) {
  vars_df <- bind_rows(var_list) %>%
    distinct(variable) %>%
    arrange(variable) %>%
    mutate(position = dplyr::row_number()) %>%
    select(position, variable)
}

# Ensure at least an empty structure with required columns
if (nrow(vars_df) == 0) {
  vars_df <- tibble(
    position = integer(),
    variable = character()
  )
}

# Write result
write_csv(vars_df, file = file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"position","variable"
1,"STUDYID"
2,"USUBJID"
3,"SUBJID"
```

### LLM Output
#### `result.csv`


```csv
position,variable
```
