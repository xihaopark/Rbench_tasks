# Case 063: pharmaverse/admiral/compute_egfr

## Metadata
- Task ID: `pharmaverse/admiral/compute_egfr`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric mismatch in column: result
summary.csv: Value mismatch in column: col_names.
```

## Prompt
```text
Write an R script to perform compute egfr using the admiral clinical task contract.

Input: age.tsv, creat.tsv, race.tsv, sex.tsv
Output: result.csv, summary.csv


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
  library(dplyr)
  library(readr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
age_df   <- read_tsv("inputs/age.tsv",   show_col_types = FALSE)
creat_df <- read_tsv("inputs/creat.tsv", show_col_types = FALSE)
race_df  <- read_tsv("inputs/race.tsv",  show_col_types = FALSE)
sex_df   <- read_tsv("inputs/sex.tsv",   show_col_types = FALSE)

# Combine into one data frame
df <- bind_cols(creat_df, age_df, sex_df, race_df)

# Normalize race and sex
df <- df %>%
  mutate(
    race_std = case_when(
      str_to_upper(race) == "BLACK OR AFRICAN AMERICAN" ~ "BLACK",
      TRUE ~ "OTHER"
    ),
    sex_std = case_when(
      str_to_upper(sex) == "F" ~ "F",
      str_to_upper(sex) == "M" ~ "M",
      TRUE ~ NA_character_
    )
  )

# CKD-EPI 2009 creatinine equation (mL/min/1.73m^2)
df <- df %>%
  mutate(
    scr = creat,
    k   = if_else(sex_std == "F", 0.7, 0.9),
    a   = if_else(sex_std == "F", -0.329, -0.411),
    alpha = pmin(scr / k, 1),
    beta  = pmax(scr / k, 1),
    sex_factor  = if_else(sex_std == "F", 1.018, 1.0),
    race_factor = if_else(race_std == "BLACK", 1.159, 1.0),
    result = 141 * (alpha ^ a) * (beta ^ (-1.209)) * (0.993 ^ age) *
      sex_factor * race_factor
  ) %>%
  select(creat, age, sex, race, result)

# Write result.csv
write_csv(df, "outputs/result.csv")

# Prepare and write summary.csv
summary_df <- tibble(
  n_rows   = nrow(df),
  n_cols   = ncol(df),
  col_names = paste(colnames(df), collapse = ",")
)

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
0.8,45,F,OTHER,89.03705956423661
1.1,60,M,BLACK OR AFRICAN AMERICAN,84.11876654652023
1.4,72,F,OTHER,37.44273097839041
```

#### `summary.csv`


```csv
n_rows,n_cols,col_names
3,5,"creat,age,sex,race,result"
```
