# Case 058: pharmaverse/admiral/compute_age_years

## Metadata
- Task ID: `pharmaverse/admiral/compute_age_years`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform compute age years using the admiral clinical task contract.

Input: age.tsv, age_unit.tsv
Output: result.csv


Required columns for result.csv: age, age_unit, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `age.tsv`


```text
age
18
24
104
365
```

#### `age_unit.tsv`


```text
age_unit
years
months
weeks
days
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

age <- as.numeric(read.delim(file.path("inputs", "age.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$age)
age_unit <- as.character(read.delim(file.path("inputs", "age_unit.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$age_unit)
result <- admiral::compute_age_years(age, age_unit)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(age = age, age_unit = age_unit, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input files
age_df <- read_tsv(
  file = file.path("inputs", "age.tsv"),
  col_types = cols(
    age = col_double()
  )
)

age_unit_df <- read_tsv(
  file = file.path("inputs", "age_unit.tsv"),
  col_types = cols(
    age_unit = col_character()
  )
)

# Combine inputs assuming row-wise correspondence
df <- bind_cols(age_df, age_unit_df)

# Compute age in years according to unit
df_res <- df %>%
  mutate(
    result = case_when(
      age_unit == "years"  ~ age,
      age_unit == "months" ~ age / 12,
      age_unit == "weeks"  ~ age / (365.25 / 7),
      age_unit == "days"   ~ age / 365.25,
      TRUE ~ NA_real_
    )
  ) %>%
  select(age, age_unit, result)

# Write result
write_csv(df_res, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"age","age_unit","result"
18,"years",18
24,"months",2
104,"weeks",1.99315537303217
365,"days",0.999315537303217
```

### LLM Output
#### `result.csv`


```csv
age,age_unit,result
18,years,18
24,months,2
104,weeks,1.9931553730321696
365,days,0.999315537303217
```
