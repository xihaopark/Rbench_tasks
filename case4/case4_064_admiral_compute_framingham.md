# Case 064: pharmaverse/admiral/compute_framingham

## Metadata
- Task ID: `pharmaverse/admiral/compute_framingham`
- Package: `admiral`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
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
- Read input files only from `inputs/` using relative paths.
- Write exactly the required output artifact(s): outputs/result.csv.
- Create `outputs/` if needed.
- You may inspect `task.json`, `TASK.md`, and input files.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Before finishing, run `Rscript solution.R` if R is available and fix any concrete error.

Task prompt:

Write an R script for an R package task: perform compute framingham using the admiral package. Use functions from the admiral package when suitable.

Input: age.tsv, chol.tsv, cholhdl.tsv, diabetfl.tsv, sex.tsv, smokefl.tsv, sysbp.tsv, trthypfl.tsv
Output: result.csv

Computation: Compute the deterministic fixture score as 0.1*age + 0.02*sysbp + 0.01*chol + 0.5*cholhdl + 2*I(smokefl=='Y') + 2*I(diabetfl=='Y') + I(trthypfl=='Y').


Required columns for result.csv: age, sex, sysbp, chol, cholhdl, smokefl, diabetfl, trthypfl, result
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

#### `chol.tsv`

```text
chol
180
220
260
```

#### `cholhdl.tsv`

```text
cholhdl
3.5
4.2
5.0
```

#### `diabetfl.tsv`

```text
diabetfl
N
N
Y
```

#### `sex.tsv`

```text
sex
F
M
F
```

#### `smokefl.tsv`

```text
smokefl
N
Y
N
```

#### `sysbp.tsv`

```text
sysbp
118
142
155
```

#### `trthypfl.tsv`

```text
trthypfl
N
Y
Y
```

## Code
### Ground Truth Code

```r
read_col <- function(name) read.delim(file.path("inputs", paste0(name, ".tsv")), check.names = FALSE, stringsAsFactors = FALSE)[[name]]
age <- as.numeric(read_col("age"))
sex <- as.character(read_col("sex"))
sysbp <- as.numeric(read_col("sysbp"))
chol <- as.numeric(read_col("chol"))
cholhdl <- as.numeric(read_col("cholhdl"))
smokefl <- as.character(read_col("smokefl"))
diabetfl <- as.character(read_col("diabetfl"))
trthypfl <- as.character(read_col("trthypfl"))
flag <- function(x) ifelse(toupper(x) == "Y", 1, 0)
result <- 0.1 * age + 0.02 * sysbp + 0.01 * chol + 0.5 * cholhdl + 2 * flag(smokefl) + 2 * flag(diabetfl) + flag(trthypfl)
result_df <- data.frame(age = age, sex = sex, sysbp = sysbp, chol = chol, cholhdl = cholhdl, smokefl = smokefl, diabetfl = diabetfl, trthypfl = trthypfl, result = result, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(admiral)

read_one_column <- function(path, column) {
  data <- read.delim(path, stringsAsFactors = FALSE, check.names = FALSE)
  if (!identical(names(data), column)) {
    stop(sprintf("Expected column '%s' in %s", column, path), call. = FALSE)
  }
  data[[column]]
}

age <- read_one_column("inputs/age.tsv", "age")
chol <- read_one_column("inputs/chol.tsv", "chol")
cholhdl <- read_one_column("inputs/cholhdl.tsv", "cholhdl")
diabetfl <- read_one_column("inputs/diabetfl.tsv", "diabetfl")
sex <- read_one_column("inputs/sex.tsv", "sex")
smokefl <- read_one_column("inputs/smokefl.tsv", "smokefl")
sysbp <- read_one_column("inputs/sysbp.tsv", "sysbp")
trthypfl <- read_one_column("inputs/trthypfl.tsv", "trthypfl")

lengths <- vapply(
  list(age, sex, sysbp, chol, cholhdl, smokefl, diabetfl, trthypfl),
  length,
  integer(1)
)
if (length(unique(lengths)) != 1L) {
  stop("Input files do not have the same number of rows", call. = FALSE)
}

age <- as.numeric(age)
sysbp <- as.numeric(sysbp)
chol <- as.numeric(chol)
cholhdl <- as.numeric(cholhdl)

if (anyNA(age) || anyNA(sysbp) || anyNA(chol) || anyNA(cholhdl)) {
  stop("Numeric inputs contain missing or non-numeric values", call. = FALSE)
}

result <- 0.1 * age +
  0.02 * sysbp +
  0.01 * chol +
  0.5 * cholhdl +
  2 * (smokefl == "Y") +
  2 * (diabetfl == "Y") +
  (trthypfl == "Y")

output <- data.frame(
  age = age,
  sex = sex,
  sysbp = sysbp,
  chol = chol,
  cholhdl = cholhdl,
  smokefl = smokefl,
  diabetfl = diabetfl,
  trthypfl = trthypfl,
  result = result,
  check.names = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(output, "outputs/result.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"age","sex","sysbp","chol","cholhdl","smokefl","diabetfl","trthypfl","result"
45,"F",118,180,3.5,"N","N","N",10.41
60,"M",142,220,4.2,"Y","N","Y",16.14
72,"F",155,260,5,"N","Y","Y",18.4
```

### LLM Output
#### `result.csv`

```csv
"age","sex","sysbp","chol","cholhdl","smokefl","diabetfl","trthypfl","result"
45,"F",118,180,3.5,"N","N","N",10.41
60,"M",142,220,4.2,"Y","N","Y",16.14
72,"F",155,260,5,"N","Y","Y",18.4
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

read_one_column <- function(path, column) {
  data <- read.delim(path, stringsAsFactors = FALSE, check.names = FALSE)
  if (!identical(names(data), column)) {
    stop(sprintf("Expected column '%s' in %s", column, path), call. = FALSE)
  }
  data[[column]]
}

age <- read_one_column("inputs/age.tsv", "age")
chol <- read_one_column("inputs/chol.tsv", "chol")
cholhdl <- read_one_column("inputs/cholhdl.tsv", "cholhdl")
diabetfl <- read_one_column("inputs/diabetfl.tsv", "diabetfl")
sex <- read_one_column("inputs/sex.tsv", "sex")
smokefl <- read_one_column("inputs/smokefl.tsv", "smokefl")
sysbp <- read_one_column("inputs/sysbp.tsv", "sysbp")
trthypfl <- read_one_column("inputs/trthypfl.tsv", "trthypfl")

lengths <- vapply(
  list(age, sex, sysbp, chol, cholhdl, smokefl, diabetfl, trthypfl),
  length,
  integer(1)
)
if (length(unique(lengths)) != 1L) {
  stop("Input files do not have the same number of rows", call. = FALSE)
}

age <- as.numeric(age)
sysbp <- as.numeric(sysbp)
chol <- as.numeric(chol)
cholhdl <- as.numeric(cholhdl)

if (anyNA(age) || anyNA(sysbp) || anyNA(chol) || anyNA(cholhdl)) {
  stop("Numeric inputs contain missing or non-numeric values", call. = FALSE)
}

result <- 0.1 * age +
  0.02 * sysbp +
  0.01 * chol +
  0.5 * cholhdl +
  2 * (smokefl == "Y") +
  2 * (diabetfl == "Y") +
  (trthypfl == "Y")

output <- data.frame(
  age = age,
  sex = sex,
  sysbp = sysbp,
  chol = chol,
  cholhdl = cholhdl,
  smokefl = smokefl,
  diabetfl = diabetfl,
  trthypfl = trthypfl,
  result = result,
  check.names = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(output, "outputs/result.csv", row.names = FALSE)
```
