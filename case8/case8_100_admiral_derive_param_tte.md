# Case 100: pharmaverse/admiral/derive_param_tte

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_tte`
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

Write an R script for an R package task: perform derive param tte using the admiral package. Use functions from the admiral package when suitable.

Input: dataset_adsl.tsv
Output: result.csv

Computation: Derive the simplified overall-survival parameter: death records are events with CNSR=0, otherwise use the last-alive date as censored with CNSR=1, and set the fixed OS parameter fields expected by the schema.


Required columns for result.csv: USUBJID, STUDYID, EVNTDESC, SRCDOM, SRCVAR, CNSR, ADT, STARTDT, PARAMCD, PARAM
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dataset_adsl.tsv
USUBJID	STUDYID	TRTSDT	DTHFL	DTHDT	LSALVDT
01	STUDY1	2021-01-01	Y	2021-04-01	2021-04-01
02	STUDY1	2021-01-02	N		2021-03-15
03	STUDY1	2021-01-05	N		2021-02-20
```

## Input
#### `dataset_adsl.tsv`

```text
USUBJID	STUDYID	TRTSDT	DTHFL	DTHDT	LSALVDT
01	STUDY1	2021-01-01	Y	2021-04-01	2021-04-01
02	STUDY1	2021-01-02	N		2021-03-15
03	STUDY1	2021-01-05	N		2021-02-20
```

## Code
### Ground Truth Code

```r
adsl <- read.delim(
  file.path("inputs", "dataset_adsl.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE,
  na.strings = c("", "NA")
)
for (col in c("TRTSDT", "DTHDT", "LSALVDT")) {
  adsl[[col]] <- as.Date(adsl[[col]])
}

is_death <- adsl$DTHFL == "Y"
result <- data.frame(
  USUBJID = adsl$USUBJID,
  STUDYID = adsl$STUDYID,
  EVNTDESC = ifelse(is_death, "DEATH", "LAST KNOWN ALIVE"),
  SRCDOM = "ADSL",
  SRCVAR = ifelse(is_death, "DTHDT", "LSALVDT"),
  CNSR = ifelse(is_death, 0L, 1L),
  ADT = ifelse(is_death, as.character(adsl$DTHDT), as.character(adsl$LSALVDT)),
  STARTDT = as.character(adsl$TRTSDT),
  PARAMCD = "OS",
  PARAM = "Overall Survival",
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(lubridate)

# Create outputs directory if needed
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Read input
adsl <- read.delim("inputs/dataset_adsl.tsv", stringsAsFactors = FALSE, na.strings = "")

# Parse dates
adsl <- adsl %>%
  mutate(
    TRTSDT  = as.Date(TRTSDT),
    DTHDT   = as.Date(DTHDT),
    LSALVDT = as.Date(LSALVDT)
  )

# Derive OS TTE parameter
#   - Death (DTHFL == "Y"): event, CNSR=0, ADT=DTHDT
#   - Otherwise: censored, CNSR=1, ADT=LSALVDT
result <- adsl %>%
  mutate(
    CNSR     = if_else(!is.na(DTHFL) & DTHFL == "Y", 0L, 1L),
    ADT      = if_else(!is.na(DTHFL) & DTHFL == "Y", DTHDT, LSALVDT),
    EVNTDESC = if_else(!is.na(DTHFL) & DTHFL == "Y",
                       "DEATH",
                       "LAST KNOWN ALIVE DATE"),
    SRCDOM   = "ADSL",
    SRCVAR   = if_else(!is.na(DTHFL) & DTHFL == "Y", "DTHDT", "LSALVDT"),
    STARTDT  = TRTSDT,
    PARAMCD  = "OS",
    PARAM    = "Overall Survival"
  ) %>%
  select(USUBJID, STUDYID, EVNTDESC, SRCDOM, SRCVAR, CNSR, ADT, STARTDT, PARAMCD, PARAM)

# Write output
write.csv(result, "outputs/result.csv", row.names = FALSE)

cat("Done. Rows written:", nrow(result), "\n")
print(result)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","STUDYID","EVNTDESC","SRCDOM","SRCVAR","CNSR","ADT","STARTDT","PARAMCD","PARAM"
1,"STUDY1","DEATH","ADSL","DTHDT",0,"2021-04-01","2021-01-01","OS","Overall Survival"
2,"STUDY1","LAST KNOWN ALIVE","ADSL","LSALVDT",1,"2021-03-15","2021-01-02","OS","Overall Survival"
3,"STUDY1","LAST KNOWN ALIVE","ADSL","LSALVDT",1,"2021-02-20","2021-01-05","OS","Overall Survival"
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
#!/usr/bin/env Rscript

library(dplyr)
library(lubridate)

# Create outputs directory if needed
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Read input
adsl <- read.delim("inputs/dataset_adsl.tsv", stringsAsFactors = FALSE, na.strings = "")

# Parse dates
adsl <- adsl %>%
  mutate(
    TRTSDT  = as.Date(TRTSDT),
    DTHDT   = as.Date(DTHDT),
    LSALVDT = as.Date(LSALVDT)
  )

# Derive OS TTE parameter
#   - Death (DTHFL == "Y"): event, CNSR=0, ADT=DTHDT
#   - Otherwise: censored, CNSR=1, ADT=LSALVDT
result <- adsl %>%
  mutate(
    CNSR     = if_else(!is.na(DTHFL) & DTHFL == "Y", 0L, 1L),
    ADT      = if_else(!is.na(DTHFL) & DTHFL == "Y", DTHDT, LSALVDT),
    EVNTDESC = if_else(!is.na(DTHFL) & DTHFL == "Y",
                       "DEATH",
                       "LAST KNOWN ALIVE DATE"),
    SRCDOM   = "ADSL",
    SRCVAR   = if_else(!is.na(DTHFL) & DTHFL == "Y", "DTHDT", "LSALVDT"),
    STARTDT  = TRTSDT,
    PARAMCD  = "OS",
    PARAM    = "Overall Survival"
  ) %>%
  select(USUBJID, STUDYID, EVNTDESC, SRCDOM, SRCVAR, CNSR, ADT, STARTDT, PARAMCD, PARAM)

# Write output
write.csv(result, "outputs/result.csv", row.names = FALSE)

cat("Done. Rows written:", nrow(result), "\n")
print(result)
```
