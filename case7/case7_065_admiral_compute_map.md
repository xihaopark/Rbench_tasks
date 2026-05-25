# Case 065: pharmaverse/admiral/compute_map

## Metadata
- Task ID: `pharmaverse/admiral/compute_map`
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

Write an R script for an R package task: perform compute map using the admiral package. Use functions from the admiral package when suitable.

Input: diabp.tsv, hr.tsv, sysbp.tsv
Output: result.csv

Computation: Call admiral::compute_map with diabp, sysbp, and hr from the aligned input vectors; include diabp, sysbp, hr, and result in result.csv.


Required columns for result.csv: diabp, sysbp, hr, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### diabp.tsv
diabp
80
85
70
95

### hr.tsv
hr
60
75
90
110

### sysbp.tsv
sysbp
120
135
110
160
```

## Input
#### `diabp.tsv`

```text
diabp
80
85
70
95
```

#### `hr.tsv`

```text
hr
60
75
90
110
```

#### `sysbp.tsv`

```text
sysbp
120
135
110
160
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

diabp <- as.numeric(read.delim(file.path("inputs", "diabp.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$diabp)
sysbp <- as.numeric(read.delim(file.path("inputs", "sysbp.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$sysbp)
hr <- as.numeric(read.delim(file.path("inputs", "hr.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$hr)
result <- admiral::compute_map(diabp = diabp, sysbp = sysbp, hr = hr)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(diabp = diabp, sysbp = sysbp, hr = hr, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(admiral)

diabp <- read.delim("inputs/diabp.tsv")$diabp
hr    <- read.delim("inputs/hr.tsv")$hr
sysbp <- read.delim("inputs/sysbp.tsv")$sysbp

result <- admiral::compute_map(diabp = diabp, sysbp = sysbp, hr = hr)

dir.create("outputs", showWarnings = FALSE)

output <- data.frame(diabp = diabp, sysbp = sysbp, hr = hr, result = result)

write.csv(output, "outputs/result.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"diabp","sysbp","hr","result"
80,120,60,92.7395237619899
85,135,75,103.240653638352
70,110,90,85.9752811815739
95,160,110,123.186790537475
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
library(admiral)

diabp <- read.delim("inputs/diabp.tsv")$diabp
hr    <- read.delim("inputs/hr.tsv")$hr
sysbp <- read.delim("inputs/sysbp.tsv")$sysbp

result <- admiral::compute_map(diabp = diabp, sysbp = sysbp, hr = hr)

dir.create("outputs", showWarnings = FALSE)

output <- data.frame(diabp = diabp, sysbp = sysbp, hr = hr, result = result)

write.csv(output, "outputs/result.csv", row.names = FALSE)
```
