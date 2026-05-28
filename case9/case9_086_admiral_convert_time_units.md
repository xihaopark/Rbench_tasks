# Case 086: pharmaverse/admiral/convert_time_units

## Metadata
- Task ID: `pharmaverse/admiral/convert_time_units`
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

Write an R script for an R package task: perform convert time units using the admiral package. Use functions from the admiral package when suitable.

Input: na_idx.tsv, result.tsv, xxtpt.tsv
Output: result.csv


Required columns for result.csv: xxtpt, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### na_idx.tsv
na_idx
FALSE
FALSE
FALSE
FALSE

### result.tsv
result
NA
NA
NA
NA

### xxtpt.tsv
xxtpt
2H 30MIN
3 DAYS
DAY 2
4D
```

## Input
#### `na_idx.tsv`

```text
na_idx
FALSE
FALSE
FALSE
FALSE
```

#### `result.tsv`

```text
result
NA
NA
NA
NA
```

#### `xxtpt.tsv`

```text
xxtpt
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
result <- suppressWarnings(as.numeric(read.delim(file.path("inputs", "result.tsv"), check.names = FALSE, stringsAsFactors = FALSE, na.strings = c("", "NA"))[[1]]))
na_idx <- as.logical(read.delim(file.path("inputs", "na_idx.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])

convert_one <- function(x, current, is_na) {
  if (!is.na(current) || isTRUE(is_na)) return(current)
  x_trim <- trimws(toupper(x))
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
  current
}
result <- mapply(convert_one, xxtpt, result, na_idx)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(xxtpt = xxtpt, result = as.numeric(result)), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(admiral)

# Read inputs
xxtpt_df  <- read.table("inputs/xxtpt.tsv",  header = TRUE, sep = "\t", stringsAsFactors = FALSE)
na_idx_df <- read.table("inputs/na_idx.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
result_df <- read.table("inputs/result.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)

# Extract vectors
xxtpt  <- xxtpt_df$xxtpt
na_idx <- as.logical(na_idx_df$na_idx)

# Convert CDISC timepoint strings to hours using admiral
result <- convert_xxtpt_to_hours(xxtpt)

# Apply NA where na_idx is TRUE
result[na_idx] <- NA_real_

# Build output data frame
output_df <- data.frame(
  xxtpt  = xxtpt,
  result = result,
  stringsAsFactors = FALSE
)

# Write output
dir.create("outputs", showWarnings = FALSE)
write.csv(output_df, "outputs/result.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"xxtpt","result"
"2H 30MIN",2.5
"3 DAYS",72
"DAY 2",48
"4D",96
```

### LLM Output
No LLM output artifact was produced.

#### `case_01/stderr.txt`

```text
[RBioBench Library Aliases] Library aliasing disabled (using stub layer)
[Admiraldev Stub] Created admiraldev namespace with 10 stub functions
[RBioBench Stub Layer] Loaded admiraldev stubs
[aNCA Stub] Created aNCA namespace with 57 stub functions
[RBioBench Stub Layer] Loaded aNCA stubs
[Logrx Stub] Created logrx namespace with 2 stub functions
[RBioBench Stub Layer] Loaded logrx stubs
[Sdtmchecks Stub] Created sdtmchecks namespace with 2 stub functions
[RBioBench Stub Layer] Loaded sdtmchecks stubs
[Other Stubs] Registered 5 stub functions from 5 packages
[RBioBench Stub Layer] Loaded other package stubs
[RBioBench Stub Layer] Registered attach hook for admiral
[Admiral Stub] Injected 40 functions into admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[RBioBench Stub Layer] Stubs registered in admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[RBioBench Stub Layer] Stubs registered in admiral namespace
[RBioBench Stub Layer] .Rprofile loaded. Stubs will be auto-injected when admiral loads.
Error in convert_special_cases(xxtpt, result, na_idx, treatment_duration) : 
  unused arguments (result, na_idx, treatment_duration)
Calls: convert_xxtpt_to_hours
Execution halted
```

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
xxtpt_df  <- read.table("inputs/xxtpt.tsv",  header = TRUE, sep = "\t", stringsAsFactors = FALSE)
na_idx_df <- read.table("inputs/na_idx.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
result_df <- read.table("inputs/result.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)

# Extract vectors
xxtpt  <- xxtpt_df$xxtpt
na_idx <- as.logical(na_idx_df$na_idx)

# Convert CDISC timepoint strings to hours using admiral
result <- convert_xxtpt_to_hours(xxtpt)

# Apply NA where na_idx is TRUE
result[na_idx] <- NA_real_

# Build output data frame
output_df <- data.frame(
  xxtpt  = xxtpt,
  result = result,
  stringsAsFactors = FALSE
)

# Write output
dir.create("outputs", showWarnings = FALSE)
write.csv(output_df, "outputs/result.csv", row.names = FALSE)
```
