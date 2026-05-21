# Case2 02: pharmaverse/admiral/compute_framingham

## Metadata
- Task ID: `pharmaverse/admiral/compute_framingham`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script for an R package task: perform compute framingham using the admiral package. The function(s) you use to solve the task must come from the admiral package.

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
#!/usr/bin/env Rscript

library(admiral)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input datasets
age      <- read.delim("inputs/age.tsv",      stringsAsFactors = FALSE)
chol     <- read.delim("inputs/chol.tsv",     stringsAsFactors = FALSE)
cholhdl  <- read.delim("inputs/cholhdl.tsv",  stringsAsFactors = FALSE)
diabetfl <- read.delim("inputs/diabetfl.tsv", stringsAsFactors = FALSE)
sex      <- read.delim("inputs/sex.tsv",      stringsAsFactors = FALSE)
smokefl  <- read.delim("inputs/smokefl.tsv",  stringsAsFactors = FALSE)
sysbp    <- read.delim("inputs/sysbp.tsv",    stringsAsFactors = FALSE)
trthypfl <- read.delim("inputs/trthypfl.tsv", stringsAsFactors = FALSE)

# Combine input datasets using admiral
adsl <- admiral::derive_vars_merged(
  age,
  dataset_add = chol,
  by_vars = NULL
)

adsl <- admiral::derive_vars_merged(
  adsl,
  dataset_add = cholhdl,
  by_vars = NULL
)

adsl <- admiral::derive_vars_merged(
  adsl,
  dataset_add = diabetfl,
  by_vars = NULL
)

adsl <- admiral::derive_vars_merged(
  adsl,
  dataset_add = sex,
  by_vars = NULL
)

adsl <- admiral::derive_vars_merged(
  adsl,
  dataset_add = smokefl,
  by_vars = NULL
)

adsl <- admiral::derive_vars_merged(
  adsl,
  dataset_add = sysbp,
  by_vars = NULL
)

adsl <- admiral::derive_vars_merged(
  adsl,
  dataset_add = trthypfl,
  by_vars = NULL
)

# Compute deterministic fixture score using admiral
adsl <- admiral::derive_vars_merged(
  adsl,
  dataset_add = transform(
    adsl,
    result = 0.1 * age +
             0.02 * sysbp +
             0.01 * chol +
             0.5 * cholhdl +
             2 * (smokefl == "Y") +
             2 * (diabetfl == "Y") +
             (trthypfl == "Y")
  ),
  by_vars = NULL
)

# Keep required columns
result <- adsl[, c("age",
                   "sex",
                   "sysbp",
                   "chol",
                   "cholhdl",
                   "smokefl",
                   "diabetfl",
                   "trthypfl",
                   "result")]

# Write output
write.csv(result, file = "outputs/result.csv", row.names = FALSE)
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
