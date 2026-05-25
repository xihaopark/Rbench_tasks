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
library(dplyr)

# Read input files
age      <- read.table("inputs/age.tsv",      header = TRUE, sep = "\t")
chol     <- read.table("inputs/chol.tsv",     header = TRUE, sep = "\t")
cholhdl  <- read.table("inputs/cholhdl.tsv",  header = TRUE, sep = "\t")
diabetfl <- read.table("inputs/diabetfl.tsv", header = TRUE, sep = "\t")
sex      <- read.table("inputs/sex.tsv",      header = TRUE, sep = "\t")
smokefl  <- read.table("inputs/smokefl.tsv",  header = TRUE, sep = "\t")
sysbp    <- read.table("inputs/sysbp.tsv",    header = TRUE, sep = "\t")
trthypfl <- read.table("inputs/trthypfl.tsv", header = TRUE, sep = "\t")

# Combine all inputs by row position
data <- bind_cols(age, sex, sysbp, chol, cholhdl, smokefl, diabetfl, trthypfl)

# Compute the Framingham score
data <- data %>%
  mutate(
    result = 0.1 * age +
             0.02 * sysbp +
             0.01 * chol +
             0.5 * cholhdl +
             2 * (smokefl == "Y") +
             2 * (diabetfl == "Y") +
             1 * (trthypfl == "Y")
  )

# Select required columns
result <- data %>%
  select(age, sex, sysbp, chol, cholhdl, smokefl, diabetfl, trthypfl, result)

# Write output
dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
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
library(dplyr)

# Read input files
age      <- read.table("inputs/age.tsv",      header = TRUE, sep = "\t")
chol     <- read.table("inputs/chol.tsv",     header = TRUE, sep = "\t")
cholhdl  <- read.table("inputs/cholhdl.tsv",  header = TRUE, sep = "\t")
diabetfl <- read.table("inputs/diabetfl.tsv", header = TRUE, sep = "\t")
sex      <- read.table("inputs/sex.tsv",      header = TRUE, sep = "\t")
smokefl  <- read.table("inputs/smokefl.tsv",  header = TRUE, sep = "\t")
sysbp    <- read.table("inputs/sysbp.tsv",    header = TRUE, sep = "\t")
trthypfl <- read.table("inputs/trthypfl.tsv", header = TRUE, sep = "\t")

# Combine all inputs by row position
data <- bind_cols(age, sex, sysbp, chol, cholhdl, smokefl, diabetfl, trthypfl)

# Compute the Framingham score
data <- data %>%
  mutate(
    result = 0.1 * age +
             0.02 * sysbp +
             0.01 * chol +
             0.5 * cholhdl +
             2 * (smokefl == "Y") +
             2 * (diabetfl == "Y") +
             1 * (trthypfl == "Y")
  )

# Select required columns
result <- data %>%
  select(age, sex, sysbp, chol, cholhdl, smokefl, diabetfl, trthypfl, result)

# Write output
dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
```
