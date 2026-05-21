# Case 097: pharmaverse/admiral/derive_param_map

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_map`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`
- Failure pattern: `package_api_hallucination`
- Failure pattern note: ! Each element of the list in argument `by_vars` must be class/type

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script for an R package task: perform derive param map using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: Derive only MAP rows from paired DIABP and SYSBP records using (2 * DIABP + SYSBP) / 3, and set PARAMCD to MAP.


Required columns for result.csv: USUBJID, PARAMCD, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVAL
01	SYSBP	120
01	DIABP	80
02	SYSBP	135
02	DIABP	85
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
wide <- reshape(dataset, idvar = "USUBJID", timevar = "PARAMCD", direction = "wide")
result_df <- data.frame(
  USUBJID = wide$USUBJID,
  PARAMCD = "MAP",
  AVAL = (2 * as.numeric(wide$AVAL.DIABP) + as.numeric(wide$AVAL.SYSBP)) / 3,
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(readr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
adat <- readr::read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    PARAMCD = col_character(),
    AVAL = col_double()
  )
)

# Derive MAP parameter from paired SYSBP and DIABP using admiral::derive_param_map()
# Expect: PARAMCD == "SYSBP", "DIABP"; AVAL numeric; USUBJID identifier
adat_map <- derive_param_map(
  dataset = adat,
  by_vars = vars(USUBJID),
  set_values_to = exprs(PARAMCD = "MAP")
)

# Keep only required columns
result <- adat_map %>%
  select(USUBJID, PARAMCD, AVAL)

# Write output
readr::write_csv(
  result,
  file = file.path("outputs", "result.csv"),
  na = ""
)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL"
1,"MAP",93.3333333333333
2,"MAP",101.666666666667
```

### LLM Output
No LLM output artifact was produced.
