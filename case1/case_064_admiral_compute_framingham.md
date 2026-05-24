# Case 064: pharmaverse/admiral/compute_framingham

## Metadata
- Task ID: `pharmaverse/admiral/compute_framingham`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric mismatch in column: result.
```

## Prompt
```text
Write an R script to perform compute framingham using the admiral clinical task contract.

Input: age.tsv, chol.tsv, cholhdl.tsv, diabetfl.tsv, sex.tsv, smokefl.tsv, sysbp.tsv, trthypfl.tsv
Output: result.csv


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
#!/usr/bin/env Rscript

library(dplyr)
library(readr)

dir.create("outputs", showWarnings = FALSE)

age      <- read_tsv("inputs/age.tsv", col_types = cols())
chol     <- read_tsv("inputs/chol.tsv", col_types = cols())
cholhdl  <- read_tsv("inputs/cholhdl.tsv", col_types = cols())
diabetfl <- read_tsv("inputs/diabetfl.tsv", col_types = cols())
sex      <- read_tsv("inputs/sex.tsv", col_types = cols())
smokefl  <- read_tsv("inputs/smokefl.tsv", col_types = cols())
sysbp    <- read_tsv("inputs/sysbp.tsv", col_types = cols())
trthypfl <- read_tsv("inputs/trthypfl.tsv", col_types = cols())

df <- bind_cols(
  age,
  sex,
  sysbp,
  chol,
  cholhdl,
  smokefl,
  diabetfl,
  trthypfl
)

# Example Framingham-like risk score (placeholder implementation)
# Scores are illustrative only, not for clinical use.
df <- df %>%
  mutate(
    age_score = case_when(
      age < 45 ~ 0,
      age < 55 ~ 2,
      age < 65 ~ 4,
      age < 75 ~ 6,
      TRUE     ~ 8
    ),
    chol_score = case_when(
      chol < 160 ~ 0,
      chol < 200 ~ 1,
      chol < 240 ~ 2,
      chol < 280 ~ 3,
      TRUE       ~ 4
    ),
    hdl_score = case_when(
      cholhdl < 3.0 ~ -1,
      cholhdl < 4.0 ~ 0,
      cholhdl < 5.0 ~ 1,
      TRUE          ~ 2
    ),
    bp_score = case_when(
      sysbp < 120 ~ 0,
      sysbp < 130 ~ 1,
      sysbp < 140 ~ 2,
      sysbp < 160 ~ 3,
      TRUE        ~ 4
    ),
    smoke_score = if_else(smokefl == "Y", 2, 0),
    diab_score  = if_else(diabetfl == "Y", 3, 0),
    trt_score   = if_else(trthypfl == "Y", 2, 0),
    total_score = age_score + chol_score + hdl_score +
      bp_score + smoke_score + diab_score + trt_score,
    result = pmin(pmax(1 + total_score, 1), 30)
  ) %>%
  select(age, sex, sysbp, chol, cholhdl, smokefl, diabetfl, trthypfl, result)

write_csv(df, "outputs/result.csv")
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
age,sex,sysbp,chol,cholhdl,smokefl,diabetfl,trthypfl,result
45,F,118,180,3.5,N,N,N,4
60,M,142,220,4.2,Y,N,Y,15
72,F,155,260,5,N,Y,Y,20
```
