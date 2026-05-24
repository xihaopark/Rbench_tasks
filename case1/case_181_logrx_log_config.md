# Case 181: pharmaverse/logrx/log_config

## Metadata
- Task ID: `pharmaverse/logrx/log_config`
- Package: `logrx`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(5, 2) vs llm=(6, 2).
```

## Prompt
```text
Write an R script to perform log config using the logrx clinical task contract.

Input: extra_info.tsv, file.tsv, log_name.tsv, log_path.tsv
Output: result.csv


Required columns for result.csv: key, value
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `extra_info.tsv`


```text
key	value
project	clinical_pilot
run_type	reference
```

#### `file.tsv`


```text
file
analysis.R
```

#### `log_name.tsv`


```text
log_name
analysis_run
```

#### `log_path.tsv`


```text
log_path
outputs/logs
```

## Code
### Ground Truth Code

```r
file <- read.delim(file.path("inputs", "file.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$file[1]
log_name <- read.delim(file.path("inputs", "log_name.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$log_name[1]
log_path <- read.delim(file.path("inputs", "log_path.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$log_path[1]
extra <- read.delim(file.path("inputs", "extra_info.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result <- rbind(
  data.frame(key = c("file", "log_name", "log_path"), value = c(file, log_name, log_path), stringsAsFactors = FALSE),
  data.frame(key = paste0("extra.", extra$key), value = extra$value, stringsAsFactors = FALSE)
)
dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

extra_info <- read_tsv(
  file = file.path("inputs", "extra_info.tsv"),
  col_types = cols(.default = col_character())
)

file_df <- read_tsv(
  file = file.path("inputs", "file.tsv"),
  col_types = cols(.default = col_character())
)

log_name_df <- read_tsv(
  file = file.path("inputs", "log_name.tsv"),
  col_types = cols(.default = col_character())
)

log_path_df <- read_tsv(
  file = file.path("inputs", "log_path.tsv"),
  col_types = cols(.default = col_character())
)

project <- extra_info %>% filter(key == "project") %>% pull(value)
run_type <- extra_info %>% filter(key == "run_type") %>% pull(value)
script_file <- file_df$file[1]
log_name <- log_name_df$log_name[1]
log_path <- log_path_df$log_path[1]

log_file_full <- file.path(log_path, paste0(log_name, ".log"))

result <- tibble(
  key = c(
    "project",
    "run_type",
    "script_file",
    "log_name",
    "log_path",
    "log_file"
  ),
  value = c(
    project,
    run_type,
    script_file,
    log_name,
    log_path,
    log_file_full
  )
)

write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"key","value"
"file","analysis.R"
"log_name","analysis_run"
"log_path","outputs/logs"
"extra.project","clinical_pilot"
"extra.run_type","reference"
```

### LLM Output
#### `result.csv`


```csv
key,value
project,clinical_pilot
run_type,reference
script_file,analysis.R
log_name,analysis_run
log_path,outputs/logs
log_file,outputs/logs/analysis_run.log
```
