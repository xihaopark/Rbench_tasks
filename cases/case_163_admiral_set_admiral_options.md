# Case 163: pharmaverse/admiral/set_admiral_options

## Metadata
- Task ID: `pharmaverse/admiral/set_admiral_options`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(4, 2) vs llm=(3, 2).
```

## Prompt
```text
Write an R script to perform set admiral options using the admiral clinical task contract.

Input: save_memory.tsv, signif_digits.tsv, subject_keys.tsv
Output: result.csv


Required columns for result.csv: option, value
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `save_memory.tsv`


```text
save_memory
1
```

#### `signif_digits.tsv`


```text
signif_digits
8
```

#### `subject_keys.tsv`


```text
subject_keys
"STUDYID"
"USUBJID"
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

subject_keys_df <- read.delim(file.path("inputs", "subject_keys.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
subject_key_names <- as.character(subject_keys_df[[1]])
subject_key_names <- subject_key_names[nzchar(subject_key_names)]
if (!length(subject_key_names)) {
  stop("subject_keys.tsv must contain at least one subject key")
}
subject_keys <- rlang::exprs(!!!rlang::syms(subject_key_names))

signif_digits_df <- read.delim(file.path("inputs", "signif_digits.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
signif_digits <- suppressWarnings(as.integer(signif_digits_df[[1]][1]))
if (is.na(signif_digits) || signif_digits <= 0) {
  stop("signif_digits.tsv must contain a positive integer")
}

save_memory_df <- read.delim(file.path("inputs", "save_memory.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
save_memory_value <- tolower(as.character(save_memory_df[[1]][1]))
save_memory <- save_memory_value %in% c("true", "t", "1", "yes", "y")

admiral::set_admiral_options(subject_keys = subject_keys, signif_digits = signif_digits, save_memory = save_memory)

result_subject_keys <- vapply(admiral::get_admiral_option("subject_keys"), as.character, character(1))
result_df <- data.frame(
  option = c(rep("subject_keys", length(result_subject_keys)), "signif_digits", "save_memory"),
  value = c(
    result_subject_keys,
    as.character(admiral::get_admiral_option("signif_digits")),
    as.character(admiral::get_admiral_option("save_memory"))
  ),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(stringr)
library(purrr)
library(fs)

# Ensure outputs directory exists
if (!dir_exists("outputs")) {
  dir_create("outputs")
}

# Read inputs
save_memory <- read_tsv(
  file = "inputs/save_memory.tsv",
  col_types = cols(.default = col_character())
) %>%
  slice(1) %>%
  pull(1)

signif_digits <- read_tsv(
  file = "inputs/signif_digits.tsv",
  col_types = cols(.default = col_character())
) %>%
  slice(1) %>%
  pull(1)

subject_keys <- read_tsv(
  file = "inputs/subject_keys.tsv",
  col_types = cols(.default = col_character())
) %>%
  pull(1) %>%
  # remove surrounding quotes if present
  str_replace_all('^"(.*)"$', "\\1") %>%
  paste(collapse = ",")

# Construct result
result <- tibble(
  option = c("save_memory", "signif_digits", "subject_keys"),
  value  = c(save_memory, signif_digits, subject_keys)
)

# Write output
write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"option","value"
"subject_keys","STUDYID"
"subject_keys","USUBJID"
"signif_digits","8"
"save_memory","TRUE"
```

### LLM Output
#### `result.csv`


```csv
option,value
save_memory,1
signif_digits,8
subject_keys,"STUDYID,USUBJID"
```
