# Case 063: pharmaverse/admiral/compute_egfr

## Metadata
- Task ID: `pharmaverse/admiral/compute_egfr`
- Package: `admiral`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
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
- Read input files only from `inputs/` using relative paths.
- Write exactly the required output artifact(s): outputs/result.csv, outputs/summary.csv.
- Create `outputs/` if needed.
- You may inspect `task.json`, `TASK.md`, and input files.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Before finishing, run `Rscript solution.R` if R is available and fix any concrete error.

Task prompt:

Write an R script for an R package task: perform compute egfr using the admiral package. Use functions from the admiral package when suitable.

Input: age.tsv, creat.tsv, race.tsv, sex.tsv
Output: result.csv, summary.csv

Computation: Use the MDRD-style eGFR convention: 175 * creat^(-1.154) * age^(-0.203) * sex_factor * race_factor, with F sex_factor 0.742 and BLACK OR AFRICAN AMERICAN race_factor 1.212; also write summary.csv.


Required columns for result.csv: creat, age, sex, race, result
Required columns for summary.csv: n_rows, n_cols, col_names
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `age.tsv`

```text
age
45
60
72
```

#### `creat.tsv`

```text
creat
0.8
1.1
1.4
```

#### `race.tsv`

```text
race
OTHER
BLACK OR AFRICAN AMERICAN
OTHER
```

#### `sex.tsv`

```text
sex
F
M
F
```

## Code
### Ground Truth Code

```r
creat <- as.numeric(read.delim(file.path("inputs", "creat.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$creat)
age <- as.numeric(read.delim(file.path("inputs", "age.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$age)
sex <- as.character(read.delim(file.path("inputs", "sex.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$sex)
race <- as.character(read.delim(file.path("inputs", "race.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$race)
sex_factor <- ifelse(toupper(sex) == "F", 0.742, 1)
race_factor <- ifelse(toupper(race) == "BLACK OR AFRICAN AMERICAN", 1.212, 1)
result <- 175 * creat^(-1.154) * age^(-0.203) * sex_factor * race_factor
result_df <- data.frame(creat = creat, age = age, sex = sex, race = race, result = result, stringsAsFactors = FALSE)
summary_df <- data.frame(n_rows = nrow(result_df), n_cols = ncol(result_df), col_names = paste(names(result_df), collapse = ", "), stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
write.csv(summary_df, file.path("outputs", "summary.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  admiral_available <- requireNamespace("admiral", quietly = TRUE)
  if (admiral_available) {
    library(admiral)
  }
})

read_one_column_tsv <- function(path, column) {
  data <- read.delim(path, sep = "\t", header = TRUE, stringsAsFactors = FALSE)
  if (!identical(names(data), column)) {
    stop(sprintf("Expected column '%s' in %s", column, path), call. = FALSE)
  }
  data[[column]]
}

creat <- read_one_column_tsv("inputs/creat.tsv", "creat")
age <- read_one_column_tsv("inputs/age.tsv", "age")
sex <- read_one_column_tsv("inputs/sex.tsv", "sex")
race <- read_one_column_tsv("inputs/race.tsv", "race")

n <- length(creat)
if (!all(lengths(list(age, sex, race)) == n)) {
  stop("Input files must contain the same number of rows.", call. = FALSE)
}

creat <- as.numeric(creat)
age <- as.numeric(age)
sex <- as.character(sex)
race <- as.character(race)

sex_factor <- ifelse(sex == "F", 0.742, 1)
race_factor <- ifelse(race == "BLACK OR AFRICAN AMERICAN", 1.212, 1)
formula_result <- 175 * creat^(-1.154) * age^(-0.203) * sex_factor * race_factor

result <- formula_result
if (admiral_available && exists("compute_egfr", envir = asNamespace("admiral"))) {
  admiral_result <- tryCatch(
    admiral::compute_egfr(
      creat = creat,
      creatu = rep("mg/dL", n),
      age = age,
      sex = sex,
      race = race,
      method = rep("MDRD", n)
    ),
    error = function(e) NULL
  )
  if (!is.null(admiral_result) && length(admiral_result) == n && all(!is.na(admiral_result))) {
    result <- as.numeric(admiral_result)
  }
}

result_df <- data.frame(
  creat = creat,
  age = age,
  sex = sex,
  race = race,
  result = result,
  check.names = FALSE
)

summary_df <- data.frame(
  n_rows = nrow(result_df),
  n_cols = ncol(result_df),
  col_names = paste(names(result_df), collapse = ","),
  check.names = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result_df, "outputs/result.csv", row.names = FALSE)
write.csv(summary_df, "outputs/summary.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"creat","age","sex","race","result"
0.8,45,"F","OTHER",77.566459506687
1.1,60,"M","BLACK OR AFRICAN AMERICAN",82.7577750329383
1.4,72,"F","OTHER",36.9633822946498
```

#### `summary.csv`

```csv
"n_rows","n_cols","col_names"
3,5,"creat, age, sex, race, result"
```

### LLM Output
#### `result.csv`

```csv
"creat","age","sex","race","result"
0.8,45,"F","OTHER",77.566459506687
1.1,60,"M","BLACK OR AFRICAN AMERICAN",82.7577750329383
1.4,72,"F","OTHER",36.9633822946498
```

#### `summary.csv`

```csv
"n_rows","n_cols","col_names"
3,5,"creat,age,sex,race,result"
```

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
```

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

suppressPackageStartupMessages({
  admiral_available <- requireNamespace("admiral", quietly = TRUE)
  if (admiral_available) {
    library(admiral)
  }
})

read_one_column_tsv <- function(path, column) {
  data <- read.delim(path, sep = "\t", header = TRUE, stringsAsFactors = FALSE)
  if (!identical(names(data), column)) {
    stop(sprintf("Expected column '%s' in %s", column, path), call. = FALSE)
  }
  data[[column]]
}

creat <- read_one_column_tsv("inputs/creat.tsv", "creat")
age <- read_one_column_tsv("inputs/age.tsv", "age")
sex <- read_one_column_tsv("inputs/sex.tsv", "sex")
race <- read_one_column_tsv("inputs/race.tsv", "race")

n <- length(creat)
if (!all(lengths(list(age, sex, race)) == n)) {
  stop("Input files must contain the same number of rows.", call. = FALSE)
}

creat <- as.numeric(creat)
age <- as.numeric(age)
sex <- as.character(sex)
race <- as.character(race)

sex_factor <- ifelse(sex == "F", 0.742, 1)
race_factor <- ifelse(race == "BLACK OR AFRICAN AMERICAN", 1.212, 1)
formula_result <- 175 * creat^(-1.154) * age^(-0.203) * sex_factor * race_factor

result <- formula_result
if (admiral_available && exists("compute_egfr", envir = asNamespace("admiral"))) {
  admiral_result <- tryCatch(
    admiral::compute_egfr(
      creat = creat,
      creatu = rep("mg/dL", n),
      age = age,
      sex = sex,
      race = race,
      method = rep("MDRD", n)
    ),
    error = function(e) NULL
  )
  if (!is.null(admiral_result) && length(admiral_result) == n && all(!is.na(admiral_result))) {
    result <- as.numeric(admiral_result)
  }
}

result_df <- data.frame(
  creat = creat,
  age = age,
  sex = sex,
  race = race,
  result = result,
  check.names = FALSE
)

summary_df <- data.frame(
  n_rows = nrow(result_df),
  n_cols = ncol(result_df),
  col_names = paste(names(result_df), collapse = ","),
  check.names = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result_df, "outputs/result.csv", row.names = FALSE)
write.csv(summary_df, "outputs/summary.csv", row.names = FALSE)
```
