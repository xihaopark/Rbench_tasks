# Case 160: pharmaverse/admiral/propagate_na_values

## Metadata
- Task ID: `pharmaverse/admiral/propagate_na_values`
- Package: `admiral`
- Model: `claude-code/claude-sonnet-4-6`
- Agent: `Claude Code`
- Status: `PASS`
- Failure stage: `pass`
- Pattern: ``

## Pass/Error
```text
PASS
```

## Prompt
```text
You are running one RBioBench clinical R task in an isolated worktree.

Your goal is to write a complete, reproducible R script at `solution.R`.

Rules:
- `TASK.md` is the authoritative task contract. `task.json` is sanitized metadata only.
- Read input files only from `inputs/` using relative paths.
- Write exactly the required output artifact(s): outputs/result.csv.
- Create `outputs/` if needed.
- You may inspect `task.json`, `TASK.md`, and input files.
- You may consult public R package documentation, including CRAN, r-universe,
  and GitHub repository documentation, to verify normal exported package APIs.
- Do not infer package function names from task metadata. Use a package API only when
  it is a normal exported function you can verify; otherwise implement the required
  transformation directly from the inputs.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Do NOT attempt to install R packages. All required packages are pre-installed in the
  evaluation Docker environment. Public documentation is useful for API lookup, but
  the final solution must run with the packages already installed in that environment.
  If a package is not available in the current shell, write the solution assuming it
  is available and move on.
- You may run `Rscript solution.R` to do a quick syntax check, but do not spend time
  debugging runtime errors caused by missing packages or system libraries.

Task prompt:

Write an R script for an R package task: perform propagate na values using the admiral package. Use functions from the admiral package when suitable.

Input: partial.tsv
Output: result.csv

Computation: For ordered datetime components, keep components before the first missing component and blank all later components.


Required columns for result.csv: year, month, day, hour, minute, second
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### partial.tsv
year	month	day	hour	minute	second
2020	05	NA	10	30	45
2021	NA	12	09	15	00
2022	07	04	NA	20	30
```

## Input
#### `partial.tsv`

```text
year	month	day	hour	minute	second
2020	05	NA	10	30	45
2021	NA	12	09	15	00
2022	07	04	NA	20	30
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "partial.tsv"), check.names = FALSE, stringsAsFactors = FALSE, na.strings = c("NA", ""))
cols <- c("year", "month", "day", "hour", "minute", "second")
for (i in seq_len(nrow(result_df))) {
  missing_seen <- FALSE
  for (col in cols) {
    if (missing_seen || is.na(result_df[[col]][i]) || result_df[[col]][i] == "") {
      result_df[[col]][i] <- NA_character_
      missing_seen <- TRUE
    }
  }
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(readr)

# Create outputs directory if needed
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Read input (read all as character to preserve original values like "05", "NA")
partial <- read_tsv("inputs/partial.tsv", col_types = cols(.default = "c"))

# Ordered datetime components
components <- c("year", "month", "day", "hour", "minute", "second")

# Propagate NA: once a component is NA (or "NA" string), blank all subsequent components
propagate_na_row <- function(row) {
  vals <- unlist(row[components])
  # Find the first truly missing position (NA or "NA" string)
  first_na <- which(is.na(vals) | vals == "NA")
  if (length(first_na) > 0) {
    # Set all components from first NA onwards to NA
    row[components[first_na[1]:length(components)]] <- NA_character_
  }
  row
}

# Apply row-wise propagation using purrr-style row iteration
result <- do.call(rbind, lapply(seq_len(nrow(partial)), function(i) {
  row <- partial[i, ]
  vals <- unlist(row[components])
  # Find the first truly missing position (NA or "NA" string)
  first_na <- which(is.na(vals) | vals == "NA")
  if (length(first_na) > 0) {
    # Set all components from first NA onwards to NA
    row[components[first_na[1]:length(components)]] <- NA_character_
  }
  row
})) %>%
  select(all_of(components))

# Write output - use empty string for NA values
write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"year","month","day","hour","minute","second"
2020,"5",,,,
2021,,,,,
2022,"7","4",,,
```

### LLM Output
No LLM output artifact was produced.

## Evaluation Result
```json
{
  "status": "PASS",
  "tier": "pass",
  "failure_stage": "pass",
  "pass": true,
  "score": 1.0,
  "message": "",
  "validation_diagnostics": {
    "failure_stage": "pass",
    "tier": "pass",
    "case_count": 1,
    "failed_case": null
  }
}
```

## Raw Agent Response
```text
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(readr)

# Create outputs directory if needed
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Read input (read all as character to preserve original values like "05", "NA")
partial <- read_tsv("inputs/partial.tsv", col_types = cols(.default = "c"))

# Ordered datetime components
components <- c("year", "month", "day", "hour", "minute", "second")

# Propagate NA: once a component is NA (or "NA" string), blank all subsequent components
propagate_na_row <- function(row) {
  vals <- unlist(row[components])
  # Find the first truly missing position (NA or "NA" string)
  first_na <- which(is.na(vals) | vals == "NA")
  if (length(first_na) > 0) {
    # Set all components from first NA onwards to NA
    row[components[first_na[1]:length(components)]] <- NA_character_
  }
  row
}

# Apply row-wise propagation using purrr-style row iteration
result <- do.call(rbind, lapply(seq_len(nrow(partial)), function(i) {
  row <- partial[i, ]
  vals <- unlist(row[components])
  # Find the first truly missing position (NA or "NA" string)
  first_na <- which(is.na(vals) | vals == "NA")
  if (length(first_na) > 0) {
    # Set all components from first NA onwards to NA
    row[components[first_na[1]:length(components)]] <- NA_character_
  }
  row
})) %>%
  select(all_of(components))

# Write output - use empty string for NA values
write_csv(result, "outputs/result.csv", na = "")
```
