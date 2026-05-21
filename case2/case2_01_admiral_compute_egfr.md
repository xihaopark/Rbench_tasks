# Case2 01: pharmaverse/admiral/compute_egfr

## Metadata
- Task ID: `pharmaverse/admiral/compute_egfr`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv, summary.csv.
```

## Prompt
```text
Write an R script for an R package task: perform compute egfr using the admiral package. The function(s) you use to solve the task must come from the admiral package.

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

library(admiral)

# Create output directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
age   <- read.delim("inputs/age.tsv",   stringsAsFactors = FALSE)
creat <- read.delim("inputs/creat.tsv", stringsAsFactors = FALSE)
race  <- read.delim("inputs/race.tsv",  stringsAsFactors = FALSE)
sex   <- read.delim("inputs/sex.tsv",   stringsAsFactors = FALSE)

# Combine into a single dataset
adsl <- cbind(age, creat, sex, race)

# Derive MDRD-style eGFR using admiral
adsl <- admiral::derive_var_egfr(
  adsl,
  method = "MDRD",
  creat = CREAT,
  age = AGE,
  sex = SEX,
  race = RACE
)

# Align column names to match function arguments
names(adsl)[names(adsl) == "age"]   <- "AGE"
names(adsl)[names(adsl) == "creat"] <- "CREAT"
names(adsl)[names(adsl) == "sex"]   <- "SEX"
names(adsl)[names(adsl) == "race"]  <- "RACE"

# Re-run derive_var_egfr after renaming
adsl <- admiral::derive_var_egfr(
  adsl,
  method = "MDRD",
  creat = CREAT,
  age = AGE,
  sex = SEX,
  race = RACE
)

# Prepare result.csv
result <- data.frame(
  creat = adsl$CREAT,
  age   = adsl$AGE,
  sex   = adsl$SEX,
  race  = adsl$RACE,
  result = adsl$EGFR
)

write.csv(result, "outputs/result.csv", row.names = FALSE, quote = TRUE)

# Prepare summary.csv
summary_df <- data.frame(
  n_rows   = nrow(result),
  n_cols   = ncol(result),
  col_names = I(list(colnames(result)))
)

write.csv(summary_df, "outputs/summary.csv", row.names = FALSE, quote = TRUE)
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
No LLM output artifact was produced.
