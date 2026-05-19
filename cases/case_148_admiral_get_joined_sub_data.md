# Case 148: pharmaverse/admiral/get_joined_sub_data

## Metadata
- Task ID: `pharmaverse/admiral/get_joined_sub_data`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(3, 3) vs llm=(2, 3).
```

## Prompt
```text
Write an R script to perform get joined sub data using the admiral clinical task contract.

Input: left.tsv, right.tsv
Output: result.csv


Required columns for result.csv: USUBJID, AVAL, TRT01A
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `left.tsv`


```text
USUBJID	AVAL
01	10
02	20
03	30
```

#### `right.tsv`


```text
USUBJID	TRT01A
01	Drug A
02	Placebo
```

## Code
### Ground Truth Code

```r
left <- read.delim(file.path("inputs", "left.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
right <- read.delim(file.path("inputs", "right.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df <- merge(left, right, by = "USUBJID", all.x = TRUE, sort = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input data
left <- read_tsv(
  file = file.path("inputs", "left.tsv"),
  col_types = cols(.default = col_character())
)

right <- read_tsv(
  file = file.path("inputs", "right.tsv"),
  col_types = cols(.default = col_character())
)

# Perform left join by USUBJID (admiral-like join on key)
result <- left %>%
  inner_join(right, by = "USUBJID") %>%
  select(USUBJID, AVAL, TRT01A)

# Write result
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","AVAL","TRT01A"
1,10,"Drug A"
2,20,"Placebo"
3,30,NA
```

### LLM Output
#### `result.csv`


```csv
USUBJID,AVAL,TRT01A
01,10,Drug A
02,20,Placebo
```
