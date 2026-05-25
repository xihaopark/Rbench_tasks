# Case 071: pharmaverse/admiral/compute_tmf

## Metadata
- Task ID: `pharmaverse/admiral/compute_tmf`
- Package: `admiral`
- Model: `claude-code/claude-sonnet-4-6`
- Agent: `Claude Code`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`
- Pattern: ``

## Pass/Error
```text
NO_OUTPUT
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

Write an R script for an R package task: perform compute tmf using the admiral package. Use functions from the admiral package when suitable.

Input: dtc.tsv, dtm.tsv, ignore_seconds_flag.tsv
Output: result.csv

Computation: For this fixture's partial-time flags, map hour-only imputation to M, minute-only imputation to S, and date-only records to H.


Required columns for result.csv: dtc, dtm, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dtc.tsv
dtc
2020-01-01T12
2020-01-01T12:30
2020-01-01

### dtm.tsv
dtm
2020-01-01 12:00:00
2020-01-01 12:30:00
2020-01-01 00:00:00

### ignore_seconds_flag.tsv
ignore_seconds_flag
FALSE
```

## Input
#### `dtc.tsv`

```text
dtc
2020-01-01T12
2020-01-01T12:30
2020-01-01
```

#### `dtm.tsv`

```text
dtm
2020-01-01 12:00:00
2020-01-01 12:30:00
2020-01-01 00:00:00
```

#### `ignore_seconds_flag.tsv`

```text
ignore_seconds_flag
FALSE
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

dtc <- as.character(read.delim(file.path("inputs", "dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
dtm_chr <- as.character(read.delim(file.path("inputs", "dtm.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
ignore_seconds_flag <- as.logical(read.delim(file.path("inputs", "ignore_seconds_flag.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1])

result <- ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}$", dtc), NA_character_,
          ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}$", dtc), if (ignore_seconds_flag) NA_character_ else "S",
          ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}T\\d{2}$", dtc), "M",
          ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}$", dtc), "H", NA_character_))))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(dtc = dtc, dtm = dtm_chr, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(admiral)

# Read inputs
dtc_df <- read.delim("inputs/dtc.tsv", stringsAsFactors = FALSE)
dtm_df <- read.delim("inputs/dtm.tsv", stringsAsFactors = FALSE)
ignore_seconds_flag_df <- read.delim("inputs/ignore_seconds_flag.tsv", stringsAsFactors = FALSE)

dtc <- dtc_df$dtc
dtm <- as.POSIXct(dtm_df$dtm, format = "%Y-%m-%d %H:%M:%S", tz = "UTC")
ignore_seconds_flag <- as.logical(ignore_seconds_flag_df$ignore_seconds_flag[1])

# Compute TMF using admiral::compute_tmf
result <- compute_tmf(dtc = dtc, dtm = dtm, ignore_seconds_flag = ignore_seconds_flag)

# Build output dataframe
out <- data.frame(
  dtc = dtc,
  dtm = format(dtm, "%Y-%m-%d %H:%M:%S"),
  result = result,
  stringsAsFactors = FALSE
)

# Create outputs directory if needed
dir.create("outputs", showWarnings = FALSE)

# Write result
write.csv(out, "outputs/result.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"dtc","dtm","result"
"2020-01-01T12","2020-01-01 12:00:00","M"
"2020-01-01T12:30","2020-01-01 12:30:00","S"
"2020-01-01","2020-01-01 00:00:00","H"
```

### LLM Output
No LLM output artifact was produced.

## Evaluation Result
```json
{
  "status": "NO_OUTPUT",
  "tier": "exec_fail",
  "failure_stage": "execution_failure",
  "pass": false,
  "score": 0.0,
  "message": "Failed at case_embedded",
  "validation_diagnostics": {
    "failure_stage": "execution_failure",
    "tier": "exec_fail",
    "case_count": 1,
    "failed_case": "case_embedded"
  }
}
```

## Raw Agent Response
```text
library(admiral)

# Read inputs
dtc_df <- read.delim("inputs/dtc.tsv", stringsAsFactors = FALSE)
dtm_df <- read.delim("inputs/dtm.tsv", stringsAsFactors = FALSE)
ignore_seconds_flag_df <- read.delim("inputs/ignore_seconds_flag.tsv", stringsAsFactors = FALSE)

dtc <- dtc_df$dtc
dtm <- as.POSIXct(dtm_df$dtm, format = "%Y-%m-%d %H:%M:%S", tz = "UTC")
ignore_seconds_flag <- as.logical(ignore_seconds_flag_df$ignore_seconds_flag[1])

# Compute TMF using admiral::compute_tmf
result <- compute_tmf(dtc = dtc, dtm = dtm, ignore_seconds_flag = ignore_seconds_flag)

# Build output dataframe
out <- data.frame(
  dtc = dtc,
  dtm = format(dtm, "%Y-%m-%d %H:%M:%S"),
  result = result,
  stringsAsFactors = FALSE
)

# Create outputs directory if needed
dir.create("outputs", showWarnings = FALSE)

# Write result
write.csv(out, "outputs/result.csv", row.names = FALSE)
```
