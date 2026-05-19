# Case 137: pharmaverse/admiral/filter_relative

## Metadata
- Task ID: `pharmaverse/admiral/filter_relative`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(3, 4) vs llm=(2, 4).
```

## Prompt
```text
Write an R script to perform filter relative using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, AVISITN, PARAMCD, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	AVISITN	PARAMCD	AVAL
01	-1	SCREEN	8
01	1	BASE	10
01	2	POST	12
02	-1	SCREEN	9
02	1	POST	11
02	2	POST	13
03	1	BASE	20
03	2	POST	22
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(rlang))

dataset <- read.delim(
  file.path("inputs", "dataset.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)

result <- admiral::filter_relative(
  dataset = dataset,
  by_vars = exprs(USUBJID),
  order = exprs(AVISITN),
  condition = PARAMCD == "BASE",
  mode = "first",
  selection = "before",
  inclusive = TRUE,
  keep_no_ref_groups = FALSE,
  check_type = "warning"
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
adsl <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols()
)

# Apply "filter relative" per admiral clinical task contract:
# Keep records with AVISITN >= 1 and where a baseline (AVISITN == 1 & PARAMCD == "BASE")
# exists for the same USUBJID and PARAMCD
baseline <- adsl %>%
  filter(AVISITN == 1, PARAMCD == "BASE") %>%
  select(USUBJID, PARAMCD)

result <- adsl %>%
  semi_join(baseline, by = c("USUBJID", "PARAMCD")) %>%
  filter(AVISITN >= 1) %>%
  select(USUBJID, AVISITN, PARAMCD, AVAL)

# Write output
write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","AVISITN","PARAMCD","AVAL"
1,-1,"SCREEN",8
1,1,"BASE",10
3,1,"BASE",20
```

### LLM Output
#### `result.csv`


```csv
USUBJID,AVISITN,PARAMCD,AVAL
01,1,BASE,10
03,1,BASE,20
```
