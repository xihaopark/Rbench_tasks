# Case 064: pharmaverse/admiral/compute_framingham

## Metadata
- Task ID: `pharmaverse/admiral/compute_framingham`
- Package: `admiral`
- Model: `openai/gpt-5.5`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
```

## Prompt
```text
Write an R script for an R package task: perform compute framingham using the admiral package. Use functions from the admiral package when suitable.

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

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

read_input <- function(filename) {
  read.delim(file.path("inputs", filename), stringsAsFactors = FALSE, check.names = FALSE)
}

age <- read_input("age.tsv")
chol <- read_input("chol.tsv")
cholhdl <- read_input("cholhdl.tsv")
diabetfl <- read_input("diabetfl.tsv")
sex <- read_input("sex.tsv")
smokefl <- read_input("smokefl.tsv")
sysbp <- read_input("sysbp.tsv")
trthypfl <- read_input("trthypfl.tsv")

adsl <- data.frame(
  age = as.numeric(age$age),
  sex = sex$sex,
  sysbp = as.numeric(sysbp$sysbp),
  chol = as.numeric(chol$chol),
  cholhdl = as.numeric(cholhdl$cholhdl),
  smokefl = smokefl$smokefl,
  diabetfl = diabetfl$diabetfl,
  trthypfl = trthypfl$trthypfl,
  stringsAsFactors = FALSE
)

adsl$result <- with(
  adsl,
  0.1 * age +
    0.02 * sysbp +
    0.01 * chol +
    0.5 * cholhdl +
    2 * (smokefl == "Y") +
    2 * (diabetfl == "Y") +
    (trthypfl == "Y")
)

result <- adsl[, c(
  "age",
  "sex",
  "sysbp",
  "chol",
  "cholhdl",
  "smokefl",
  "diabetfl",
  "trthypfl",
  "result"
)]

write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
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
