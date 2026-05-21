# Case 063: pharmaverse/admiral/compute_egfr

## Metadata
- Task ID: `pharmaverse/admiral/compute_egfr`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the ground truth.

Main signal:
summary.csv: Value mismatch in column: col_names
```

## Prompt
```text
Write an R script for an R package task: perform compute egfr using the admiral package. Use functions from the admiral package when suitable.

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
library(dplyr)
library(readr)
library(purrr)
library(stringr)
library(tidyr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
age_df   <- read_tsv("inputs/age.tsv", col_types = cols())
creat_df <- read_tsv("inputs/creat.tsv", col_types = cols())
race_df  <- read_tsv("inputs/race.tsv", col_types = cols())
sex_df   <- read_tsv("inputs/sex.tsv", col_types = cols())

# Combine inputs
adat <- 
  bind_cols(
    creat_df,
    age_df,
    sex_df,
    race_df
  ) %>%
  # Ensure correct column order
  select(creat, age, sex, race)

# Compute eGFR using MDRD-style equation
# 175 * creat^(-1.154) * age^(-0.203) * sex_factor * race_factor
# sex_factor: 0.742 if F, 1 otherwise
# race_factor: 1.212 if BLACK OR AFRICAN AMERICAN, 1 otherwise

result_df <- 
  adat %>%
  mutate(
    sex_factor = case_when(
      sex == "F" ~ 0.742,
      TRUE       ~ 1
    ),
    race_factor = case_when(
      race == "BLACK OR AFRICAN AMERICAN" ~ 1.212,
      TRUE                                ~ 1
    ),
    result = 175 * (creat ^ (-1.154)) * (age ^ (-0.203)) * sex_factor * race_factor
  ) %>%
  select(creat, age, sex, race, result)

# Write result.csv
write_csv(result_df, "outputs/result.csv")

# Create summary
summary_df <- 
  result_df %>%
  summarise(
    n_rows = n(),
    n_cols = ncol(.),
    col_names = paste(colnames(.), collapse = ",")
  )

# Write summary.csv
write_csv(summary_df, "outputs/summary.csv")
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
creat,age,sex,race,result
0.8,45,F,OTHER,77.566459506687
1.1,60,M,BLACK OR AFRICAN AMERICAN,82.75777503293826
1.4,72,F,OTHER,36.96338229464978
```

#### `summary.csv`

```csv
n_rows,n_cols,col_names
3,5,"creat,age,sex,race,result"
```
