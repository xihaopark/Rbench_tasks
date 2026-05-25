# Case 088: pharmaverse/admiral/convert_xxtpt_to_hours

## Metadata
- Task ID: `pharmaverse/admiral/convert_xxtpt_to_hours`
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

Write an R script for an R package task: perform convert xxtpt to hours using the admiral package. Use functions from the admiral package when suitable.

Input: range_method.tsv, treatment_duration.tsv, xxtpt.tsv
Output: result.csv

Computation: Parse timepoint text into hours: numeric hours stay numeric; HH:MM uses hours plus minutes/60; day counts such as 4D or DAY 4 map to 4 * 24 hours; pre-dose/screening style labels map to 0.


Required columns for result.csv: xxtpt, treatment_duration, range_method, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### range_method.tsv
range_method
midpoint

### treatment_duration.tsv
treatment_duration
24

### xxtpt.tsv
xxtpt
1 HOURS
2H 30MIN
3 DAYS
DAY 2
4D
```

## Input
#### `range_method.tsv`

```text
range_method
midpoint
```

#### `treatment_duration.tsv`

```text
treatment_duration
24
```

#### `xxtpt.tsv`

```text
xxtpt
1 HOURS
2H 30MIN
3 DAYS
DAY 2
4D
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

xxtpt <- as.character(read.delim(file.path("inputs", "xxtpt.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
treatment_duration <- as.numeric(read.delim(file.path("inputs", "treatment_duration.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1])
range_method <- as.character(read.delim(file.path("inputs", "range_method.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1])

convert_one <- function(x) {
  x_trim <- trimws(toupper(x))
  if (grepl("^[0-9.]+\\s*H(OUR|OURS|R)?$", x_trim)) {
    return(as.numeric(sub("^([0-9.]+).*", "\\1", x_trim)))
  }
  if (grepl("^[0-9.]+\\s*H(OUR|OURS|R)?S?\\s+[0-9.]+\\s*M(IN|INUTE)?S?$", x_trim)) {
    nums <- as.numeric(regmatches(x_trim, gregexpr("[0-9.]+", x_trim))[[1]])
    return(nums[1] + nums[2] / 60)
  }
  if (grepl("^[0-9.]+\\s*D(AY|AYS)?$", x_trim)) {
    return(as.numeric(sub("^([0-9.]+).*", "\\1", x_trim)) * 24)
  }
  if (grepl("^DAY\\s+[0-9.]+$", x_trim)) {
    return(as.numeric(sub("^DAY\\s+([0-9.]+)$", "\\1", x_trim)) * 24)
  }
  if (grepl("^[0-9.]+\\s+POST\\s*DOSE$", x_trim)) {
    return(as.numeric(sub("^([0-9.]+).*", "\\1", x_trim)) * 24)
  }
  NA_real_
}
result <- vapply(xxtpt, convert_one, numeric(1))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(xxtpt = xxtpt, treatment_duration = treatment_duration, range_method = range_method, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(admiral)

# Read inputs
xxtpt_df <- read.table("inputs/xxtpt.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
treatment_duration_df <- read.table("inputs/treatment_duration.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
range_method_df <- read.table("inputs/range_method.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)

xxtpt <- xxtpt_df$xxtpt
treatment_duration <- treatment_duration_df$treatment_duration[1]
range_method <- range_method_df$range_method[1]

# Convert timepoints to hours using admiral function
result_hours <- convert_xxtpt_to_hours(
  xxtpt = xxtpt,
  treatment_duration = treatment_duration,
  range_method = range_method
)

# Build output data frame
result_df <- data.frame(
  xxtpt = xxtpt,
  treatment_duration = treatment_duration,
  range_method = range_method,
  result = result_hours,
  stringsAsFactors = FALSE
)

# Create outputs directory if needed
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Write output
write.csv(result_df, "outputs/result.csv", row.names = FALSE)

cat("Done. Output written to outputs/result.csv\n")
print(result_df)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"xxtpt","treatment_duration","range_method","result"
"1 HOURS",24,"midpoint",1
"2H 30MIN",24,"midpoint",2.5
"3 DAYS",24,"midpoint",72
"DAY 2",24,"midpoint",48
"4D",24,"midpoint",96
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
xxtpt_df <- read.table("inputs/xxtpt.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
treatment_duration_df <- read.table("inputs/treatment_duration.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
range_method_df <- read.table("inputs/range_method.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)

xxtpt <- xxtpt_df$xxtpt
treatment_duration <- treatment_duration_df$treatment_duration[1]
range_method <- range_method_df$range_method[1]

# Convert timepoints to hours using admiral function
result_hours <- convert_xxtpt_to_hours(
  xxtpt = xxtpt,
  treatment_duration = treatment_duration,
  range_method = range_method
)

# Build output data frame
result_df <- data.frame(
  xxtpt = xxtpt,
  treatment_duration = treatment_duration,
  range_method = range_method,
  result = result_hours,
  stringsAsFactors = FALSE
)

# Create outputs directory if needed
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Write output
write.csv(result_df, "outputs/result.csv", row.names = FALSE)

cat("Done. Output written to outputs/result.csv\n")
print(result_df)
```
