# Case 087: pharmaverse/admiral/convert_treatment_patterns

## Metadata
- Task ID: `pharmaverse/admiral/convert_treatment_patterns`
- Package: `admiral`
- Model: `claude-code/claude-sonnet-4-6`
- Agent: `Claude Code`
- Status: `FAIL`
- Failure stage: `value_mismatch`
- Pattern: ``

## Pass/Error
```text
FAIL
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

Write an R script for an R package task: perform convert treatment patterns using the admiral package. Use functions from the admiral package when suitable.

Input: treatment_duration.tsv, xxtpt.tsv
Output: result.csv

Computation: Map treatment-relative text patterns deterministically: treatment start/zero/pre-dose to 0, after-start to positive offsets, before-end/pre-EOT to negative offsets from end, and end/after-end patterns to the treatment end plus any offset.


Required columns for result.csv: xxtpt, treatment_duration, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### treatment_duration.tsv
treatment_duration
72
72
72
72
72
... [2 more lines omitted]

### xxtpt.tsv
xxtpt
START OF TREATMENT
PRE-DOSE
30 MIN AFTER START
2 HOURS AFTER START
END OF TREATMENT
... [2 more lines omitted]
```

## Input
#### `treatment_duration.tsv`

```text
treatment_duration
72
72
72
72
72
72
72
```

#### `xxtpt.tsv`

```text
xxtpt
START OF TREATMENT
PRE-DOSE
30 MIN AFTER START
2 HOURS AFTER START
END OF TREATMENT
4 HOURS BEFORE END
6 HOURS AFTER END
```

## Code
### Ground Truth Code

```r
xxtpt <- as.character(read.delim(file.path("inputs", "xxtpt.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$xxtpt)
treatment_duration <- as.numeric(read.delim(file.path("inputs", "treatment_duration.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$treatment_duration)

parse_hours <- function(label, duration) {
  text <- toupper(trimws(label))
  number <- suppressWarnings(as.numeric(sub(".*?([0-9]+(?:\\.[0-9]+)?).*", "\\1", text)))
  if (text == "START OF TREATMENT") {
    0
  } else if (text == "PRE-DOSE") {
    -1
  } else if (grepl("MIN AFTER START", text)) {
    number / 60
  } else if (grepl("HOURS? AFTER START", text)) {
    number
  } else if (text == "END OF TREATMENT") {
    duration
  } else if (grepl("HOURS? BEFORE END", text)) {
    duration - number
  } else if (grepl("HOURS? AFTER END", text)) {
    duration + number
  } else {
    NA_real_
  }
}

result <- mapply(parse_hours, xxtpt, treatment_duration)
result_df <- data.frame(xxtpt = xxtpt, treatment_duration = treatment_duration, result = as.numeric(result), stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(admiral)

# Read inputs
treatment_duration_df <- read.table("inputs/treatment_duration.tsv", header = TRUE, sep = "\t",
                                    stringsAsFactors = FALSE)
xxtpt_df              <- read.table("inputs/xxtpt.tsv",               header = TRUE, sep = "\t",
                                    stringsAsFactors = FALSE)

# Combine row-wise (one row per timepoint, all share the same treatment duration)
df <- data.frame(
  xxtpt              = xxtpt_df$xxtpt,
  treatment_duration = treatment_duration_df$treatment_duration,
  stringsAsFactors   = FALSE
)

# Helper: convert a numeric value + unit string to hours
parse_to_hours <- function(num_str, unit_str) {
  val  <- as.numeric(num_str)
  unit <- toupper(trimws(unit_str))
  if (grepl("^MIN",  unit)) return(val / 60)
  if (grepl("^H",    unit)) return(val)        # HOUR, HOURS, H
  if (grepl("^DAY",  unit)) return(val * 24)
  if (grepl("^WEEK", unit)) return(val * 168)
  return(val)  # unknown unit → assume hours
}

# Convert a single xxtpt label to hours given a treatment duration value
convert_one <- function(pattern, duration) {
  p <- toupper(trimws(pattern))

  # ── Patterns that map to 0 (treatment start / pre-dose) ──────────────────
  if (grepl("^(START OF TREATMENT|PREDOSE|PRE-DOSE|PRE DOSE|ZERO|BASELINE)$", p)) {
    return(0)
  }

  # ── X UNIT AFTER START  (positive offset from start) ─────────────────────
  m <- regexpr("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)\\s+AFTER\\s+START",
               p, perl = TRUE)
  if (m != -1) {
    sub_str <- regmatches(p, m)
    parts   <- regmatches(sub_str,
                 regexec("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)", sub_str))[[1]]
    return(parse_to_hours(parts[2], parts[3]))
  }

  # ── X UNIT BEFORE END  (offset subtracted from end) ──────────────────────
  m <- regexpr("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)\\s+BEFORE\\s+END",
               p, perl = TRUE)
  if (m != -1) {
    sub_str <- regmatches(p, m)
    parts   <- regmatches(sub_str,
                 regexec("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)", sub_str))[[1]]
    return(duration - parse_to_hours(parts[2], parts[3]))
  }

  # ── PRE-EOT / PRE EOT  (just before end, no explicit offset → end) ───────
  if (grepl("^PRE.?EOT$|^PRE.?END OF TREATMENT$", p)) {
    return(duration)
  }

  # ── END OF TREATMENT / EOT → treatment duration ──────────────────────────
  if (grepl("^(END OF TREATMENT|EOT|EOI|END OF INFUSION)$", p)) {
    return(duration)
  }

  # ── X UNIT AFTER END  (positive offset beyond end) ───────────────────────
  m <- regexpr("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)\\s+AFTER\\s+END",
               p, perl = TRUE)
  if (m != -1) {
    sub_str <- regmatches(p, m)
    parts   <- regmatches(sub_str,
                 regexec("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)", sub_str))[[1]]
    return(duration + parse_to_hours(parts[2], parts[3]))
  }

  # ── Fallback: unrecognised pattern ───────────────────────────────────────
  warning(sprintf("Unrecognised timepoint pattern: '%s'", pattern))
  return(NA_real_)
}

# Apply conversion row-by-row
df$result <- mapply(convert_one, df$xxtpt, df$treatment_duration)

# Write output
dir.create("outputs", showWarnings = FALSE)
write.csv(df, "outputs/result.csv", row.names = FALSE)

# Quick diagnostic print
print(df)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"xxtpt","treatment_duration","result"
"START OF TREATMENT",72,0
"PRE-DOSE",72,-1
"30 MIN AFTER START",72,0.5
"2 HOURS AFTER START",72,2
"END OF TREATMENT",72,72
"4 HOURS BEFORE END",72,68
"6 HOURS AFTER END",72,78
```

### LLM Output
No LLM output artifact was produced.

## Evaluation Result
```json
{
  "status": "FAIL",
  "tier": "schema_ok",
  "failure_stage": "value_mismatch",
  "pass": false,
  "score": 0.0,
  "message": "Failed at case_embedded",
  "validation_diagnostics": {
    "failure_stage": "value_mismatch",
    "tier": "schema_ok",
    "case_count": 1,
    "failed_case": "case_embedded"
  }
}
```

## Raw Agent Response
```text
library(admiral)

# Read inputs
treatment_duration_df <- read.table("inputs/treatment_duration.tsv", header = TRUE, sep = "\t",
                                    stringsAsFactors = FALSE)
xxtpt_df              <- read.table("inputs/xxtpt.tsv",               header = TRUE, sep = "\t",
                                    stringsAsFactors = FALSE)

# Combine row-wise (one row per timepoint, all share the same treatment duration)
df <- data.frame(
  xxtpt              = xxtpt_df$xxtpt,
  treatment_duration = treatment_duration_df$treatment_duration,
  stringsAsFactors   = FALSE
)

# Helper: convert a numeric value + unit string to hours
parse_to_hours <- function(num_str, unit_str) {
  val  <- as.numeric(num_str)
  unit <- toupper(trimws(unit_str))
  if (grepl("^MIN",  unit)) return(val / 60)
  if (grepl("^H",    unit)) return(val)        # HOUR, HOURS, H
  if (grepl("^DAY",  unit)) return(val * 24)
  if (grepl("^WEEK", unit)) return(val * 168)
  return(val)  # unknown unit → assume hours
}

# Convert a single xxtpt label to hours given a treatment duration value
convert_one <- function(pattern, duration) {
  p <- toupper(trimws(pattern))

  # ── Patterns that map to 0 (treatment start / pre-dose) ──────────────────
  if (grepl("^(START OF TREATMENT|PREDOSE|PRE-DOSE|PRE DOSE|ZERO|BASELINE)$", p)) {
    return(0)
  }

  # ── X UNIT AFTER START  (positive offset from start) ─────────────────────
  m <- regexpr("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)\\s+AFTER\\s+START",
               p, perl = TRUE)
  if (m != -1) {
    sub_str <- regmatches(p, m)
    parts   <- regmatches(sub_str,
                 regexec("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)", sub_str))[[1]]
    return(parse_to_hours(parts[2], parts[3]))
  }

  # ── X UNIT BEFORE END  (offset subtracted from end) ──────────────────────
  m <- regexpr("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)\\s+BEFORE\\s+END",
               p, perl = TRUE)
  if (m != -1) {
    sub_str <- regmatches(p, m)
    parts   <- regmatches(sub_str,
                 regexec("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)", sub_str))[[1]]
    return(duration - parse_to_hours(parts[2], parts[3]))
  }

  # ── PRE-EOT / PRE EOT  (just before end, no explicit offset → end) ───────
  if (grepl("^PRE.?EOT$|^PRE.?END OF TREATMENT$", p)) {
    return(duration)
  }

  # ── END OF TREATMENT / EOT → treatment duration ──────────────────────────
  if (grepl("^(END OF TREATMENT|EOT|EOI|END OF INFUSION)$", p)) {
    return(duration)
  }

  # ── X UNIT AFTER END  (positive offset beyond end) ───────────────────────
  m <- regexpr("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)\\s+AFTER\\s+END",
               p, perl = TRUE)
  if (m != -1) {
    sub_str <- regmatches(p, m)
    parts   <- regmatches(sub_str,
                 regexec("^([0-9.]+)\\s*(MIN|HOURS?|H|DAYS?|WEEKS?)", sub_str))[[1]]
    return(duration + parse_to_hours(parts[2], parts[3]))
  }

  # ── Fallback: unrecognised pattern ───────────────────────────────────────
  warning(sprintf("Unrecognised timepoint pattern: '%s'", pattern))
  return(NA_real_)
}

# Apply conversion row-by-row
df$result <- mapply(convert_one, df$xxtpt, df$treatment_duration)

# Write output
dir.create("outputs", showWarnings = FALSE)
write.csv(df, "outputs/result.csv", row.names = FALSE)

# Quick diagnostic print
print(df)
```
