# Case 062: pharmaverse/admiral/compute_duration

## Metadata
- Task ID: `pharmaverse/admiral/compute_duration`
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
Write an R script to perform compute duration using the admiral clinical task contract.

Input: add_one.tsv, end_date.tsv, out_unit.tsv, start_date.tsv
Output: result.csv


Required columns for result.csv: start_date, end_date, add_one, out_unit, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `add_one.tsv`


```text
add_one
TRUE
TRUE
FALSE
```

#### `end_date.tsv`


```text
end_date
2020-01-10
2020-02-20
2020-03-01
```

#### `out_unit.tsv`


```text
out_unit
days
days
days
```

#### `start_date.tsv`


```text
start_date
2020-01-01
2020-02-10
2020-03-01
```

## Code
### Ground Truth Code

```r
start_date <- as.Date(read.delim(file.path("inputs", "start_date.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$start_date)
end_date <- as.Date(read.delim(file.path("inputs", "end_date.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$end_date)
add_one <- as.logical(read.delim(file.path("inputs", "add_one.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$add_one)
out_unit <- as.character(read.delim(file.path("inputs", "out_unit.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$out_unit)
result <- as.numeric(end_date - start_date) + ifelse(add_one, 1, 0)
result_df <- data.frame(start_date = as.character(start_date), end_date = as.character(end_date), add_one = add_one, out_unit = out_unit, result = result, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
# Load required packages
suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(lubridate)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input files
add_one <- read_tsv(
  file = "inputs/add_one.tsv",
  col_types = cols(
    add_one = col_logical()
  )
)

end_date <- read_tsv(
  file = "inputs/end_date.tsv",
  col_types = cols(
    end_date = col_date(format = "%Y-%m-%d")
  )
)

out_unit <- read_tsv(
  file = "inputs/out_unit.tsv",
  col_types = cols(
    out_unit = col_character()
  )
)

start_date <- read_tsv(
  file = "inputs/start_date.tsv",
  col_types = cols(
    start_date = col_date(format = "%Y-%m-%d")
  )
)

# Combine inputs
dat <- bind_cols(start_date, end_date, add_one, out_unit)

# Compute duration following admiral clinical task contract logic
dat <- dat %>%
  mutate(
    result = case_when(
      out_unit == "days" ~ as.numeric(end_date - start_date),
      out_unit == "weeks" ~ as.numeric(end_date - start_date) / 7,
      out_unit == "months" ~ interval(start_date, end_date) / months(1),
      out_unit == "years" ~ interval(start_date, end_date) / years(1),
      TRUE ~ NA_real_
    ),
    result = if_else(add_one & !is.na(result), result + 1, result)
  )

# Keep required columns only
result <- dat %>%
  select(start_date, end_date, add_one, out_unit, result)

# Write output
write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"start_date","end_date","add_one","out_unit","result"
"2020-01-01","2020-01-10",TRUE,"days",10
"2020-02-10","2020-02-20",TRUE,"days",11
"2020-03-01","2020-03-01",FALSE,"days",0
```

### LLM Output
#### `result.csv`


```csv
start_date,end_date,add_one,out_unit,result
2020-01-01,2020-01-10,TRUE,days,10
2020-02-10,2020-02-20,TRUE,days,11
2020-03-01,2020-03-01,FALSE,days,0
```
