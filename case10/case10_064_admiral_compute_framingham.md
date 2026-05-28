# Case 064: pharmaverse/admiral/compute_framingham

## Metadata
- Task ID: `pharmaverse/admiral/compute_framingham`
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

Write an R script for an R package task: perform compute framingham using the admiral package. Use functions from the admiral package when suitable.

Input: age.tsv, chol.tsv, cholhdl.tsv, diabetfl.tsv, sex.tsv, smokefl.tsv, sysbp.tsv, trthypfl.tsv
Output: result.csv

Computation: Compute the deterministic fixture score as 0.1*age + 0.02*sysbp + 0.01*chol + 0.5*cholhdl + 2*I(smokefl=='Y') + 2*I(diabetfl=='Y') + I(trthypfl=='Y').


Required columns for result.csv: age, sex, sysbp, chol, cholhdl, smokefl, diabetfl, trthypfl, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### age.tsv
age
45
60
72

### chol.tsv
chol
180
220
260

### cholhdl.tsv
cholhdl
3.5
4.2
5.0

### diabetfl.tsv
diabetfl
N
N
Y

### sex.tsv
sex
F
M
F

### smokefl.tsv
smokefl
N
Y
N

### sysbp.tsv
sysbp
118
142
155

### trthypfl.tsv
trthypfl
N
Y
Y

Reference package function list:
The hidden reference solution's R package function calls are listed below. If the list is empty, the reference solution does not call package functions.
- package_functions: []
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
#!/usr/bin/env Rscript

dir.create('outputs', showWarnings = FALSE, recursive = TRUE)

age      <- read.delim("inputs/age.tsv")
chol     <- read.delim("inputs/chol.tsv")
cholhdl  <- read.delim("inputs/cholhdl.tsv")
diabetfl <- read.delim("inputs/diabetfl.tsv")
sex      <- read.delim("inputs/sex.tsv")
smokefl  <- read.delim("inputs/smokefl.tsv")
sysbp    <- read.delim("inputs/sysbp.tsv")
trthypfl <- read.delim("inputs/trthypfl.tsv")

df <- data.frame(
  age      = age$age,
  sex      = sex$sex,
  sysbp    = sysbp$sysbp,
  chol     = chol$chol,
  cholhdl  = cholhdl$cholhdl,
  smokefl  = smokefl$smokefl,
  diabetfl = diabetfl$diabetfl,
  trthypfl = trthypfl$trthypfl
)

df$result <- 0.1 * df$age +
             0.02 * df$sysbp +
             0.01 * df$chol +
             0.5  * df$cholhdl +
             2    * as.integer(df$smokefl  == "Y") +
             2    * as.integer(df$diabetfl == "Y") +
             1    * as.integer(df$trthypfl == "Y")

write.csv(df, "outputs/result.csv", row.names = FALSE)
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
#!/usr/bin/env Rscript

dir.create('outputs', showWarnings = FALSE, recursive = TRUE)

age      <- read.delim("inputs/age.tsv")
chol     <- read.delim("inputs/chol.tsv")
cholhdl  <- read.delim("inputs/cholhdl.tsv")
diabetfl <- read.delim("inputs/diabetfl.tsv")
sex      <- read.delim("inputs/sex.tsv")
smokefl  <- read.delim("inputs/smokefl.tsv")
sysbp    <- read.delim("inputs/sysbp.tsv")
trthypfl <- read.delim("inputs/trthypfl.tsv")

df <- data.frame(
  age      = age$age,
  sex      = sex$sex,
  sysbp    = sysbp$sysbp,
  chol     = chol$chol,
  cholhdl  = cholhdl$cholhdl,
  smokefl  = smokefl$smokefl,
  diabetfl = diabetfl$diabetfl,
  trthypfl = trthypfl$trthypfl
)

df$result <- 0.1 * df$age +
             0.02 * df$sysbp +
             0.01 * df$chol +
             0.5  * df$cholhdl +
             2    * as.integer(df$smokefl  == "Y") +
             2    * as.integer(df$diabetfl == "Y") +
             1    * as.integer(df$trthypfl == "Y")

write.csv(df, "outputs/result.csv", row.names = FALSE)
```
