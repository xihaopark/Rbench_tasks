# Case 102: pharmaverse/admiral/derive_var_base

## Metadata
- Task ID: `pharmaverse/admiral/derive_var_base`
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
Write an R script to perform derive var base using the admiral clinical task contract.

Input: dataset.tsv, new_var.tsv, source_var.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVISITN, AVAL, ABLFL, BASE
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVISITN	AVAL	ABLFL
01	ALT	0	20	Y
01	ALT	1	30	N
01	ALT	2	28	N
02	ALT	0	15	Y
02	ALT	1	22	N
```

#### `new_var.tsv`


```text
new_var
BASE
```

#### `source_var.tsv`


```text
source_var
AVAL
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
source_var <- as.character(read.delim(file.path("inputs", "source_var.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$source_var[1])
new_var <- as.character(read.delim(file.path("inputs", "new_var.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$new_var[1])
key <- paste(result_df$USUBJID, result_df$PARAMCD, sep = "\r")
base_key <- key[result_df$ABLFL == "Y"]
base_value <- result_df[[source_var]][result_df$ABLFL == "Y"]
lookup <- setNames(base_value, base_key)
result_df[[new_var]] <- unname(lookup[key])

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

# Read inputs
dataset <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols()
)

new_var <- read_tsv(
  file = file.path("inputs", "new_var.tsv"),
  col_types = cols()
)

source_var <- read_tsv(
  file = file.path("inputs", "source_var.tsv"),
  col_types = cols()
)

# Expecting single values in new_var and source_var
new_var_name    <- new_var$new_var[1]
source_var_name <- source_var$source_var[1]

# Derive BASE as baseline AVAL (where ABLFL == "Y") per USUBJID, PARAMCD
# and merge back to all records for those subjects/parameters
baseline_vals <- dataset %>%
  filter(ABLFL == "Y") %>%
  select(USUBJID, PARAMCD, !!source_var_name) %>%
  rename(!!new_var_name := !!sym(source_var_name))

result <- dataset %>%
  left_join(baseline_vals, by = c("USUBJID", "PARAMCD")) %>%
  select(USUBJID, PARAMCD, AVISITN, AVAL, ABLFL, BASE = !!sym(new_var_name))

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVISITN","AVAL","ABLFL","BASE"
1,"ALT",0,20,"Y",20
1,"ALT",1,30,"N",20
1,"ALT",2,28,"N",20
2,"ALT",0,15,"Y",15
2,"ALT",1,22,"N",15
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,AVISITN,AVAL,ABLFL,BASE
01,ALT,0,20,Y,20
01,ALT,1,30,N,20
01,ALT,2,28,N,20
02,ALT,0,15,Y,15
02,ALT,1,22,N,15
```
