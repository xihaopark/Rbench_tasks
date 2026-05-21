# Case 118: pharmaverse/admiral/derive_vars_extreme_event

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_extreme_event`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script for an R package task: perform derive vars extreme event using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: Use source adverse events with AETOXGR >= 2, order by descending AETOXGR then ADT and AESEQ, and return one selected extreme event per STUDYID/USUBJID according to mode.


Required columns for result.csv: STUDYID, USUBJID, TRTSDT, EXTDT, EXTTERM, EXTGR
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
STUDYID	USUBJID	TRTSDT
S1	01	2020-01-01
S1	02	2020-01-05
S1	03	2020-01-10
```

#### `mode.tsv`

```text
mode
first
```

#### `source_datasets.tsv`

```text
STUDYID	USUBJID	ADT	AESEQ	AETERM	AETOXGR
S1	01	2020-01-03	1	HEADACHE	1
S1	01	2020-01-08	2	NAUSEA	3
S1	02	2020-01-07	1	RASH	2
S1	02	2020-01-08	2	FEVER	1
S1	03	2020-01-15	1	COUGH	2
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(rlang))

dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
ae <- read.delim(file.path("inputs", "source_datasets.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
mode <- read.delim(file.path("inputs", "mode.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$mode[1]

dataset$TRTSDT <- as.Date(dataset$TRTSDT)
ae$ADT <- as.Date(ae$ADT)

grade2_event <- admiral::event(
  dataset_name = "ae",
  condition = AETOXGR >= 2,
  set_values_to = rlang::exprs(EVNTDESC = "Grade 2 or higher adverse event")
)

result <- admiral::derive_vars_extreme_event(
  dataset = dataset,
  by_vars = rlang::exprs(STUDYID, USUBJID),
  events = list(grade2_event),
  source_datasets = list(ae = ae),
  order = rlang::exprs(dplyr::desc(AETOXGR), ADT, AESEQ),
  mode = mode,
  new_vars = rlang::exprs(EXTDT = ADT, EXTTERM = AETERM, EXTGR = AETOXGR)
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(readr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
adsl <- read_tsv(
  file = "inputs/dataset.tsv",
  col_types = cols(
    STUDYID = col_character(),
    USUBJID = col_character(),
    TRTSDT = col_date(format = "%Y-%m-%d")
  )
)

mode_df <- read_tsv(
  file = "inputs/mode.tsv",
  col_types = cols(
    mode = col_character()
  )
)

ae <- read_tsv(
  file = "inputs/source_datasets.tsv",
  col_types = cols(
    STUDYID = col_character(),
    USUBJID = col_character(),
    ADT = col_date(format = "%Y-%m-%d"),
    AESEQ = col_double(),
    AETERM = col_character(),
    AETOXGR = col_double()
  )
)

# Determine mode
ext_mode <- mode_df$mode[1]

# Filter source AEs: AETOXGR >= 2
ae_filt <- ae %>%
  filter(AETOXGR >= 2)

# Derive extreme event per STUDYID/USUBJID using admiral
adsl_ext <- derive_vars_extreme_event(
  adsl = adsl,
  dataset_add = ae_filt,
  order = vars(desc(AETOXGR), ADT, AESEQ),
  mode = ext_mode,
  by_vars = vars(STUDYID, USUBJID),
  new_var = EXTDT,
  source_vars = vars(ADT, AETERM, AETOXGR),
  check_type = "none"
)

# Rename / derive required variables
result <- adsl_ext %>%
  mutate(
    EXTTERM = AETERM,
    EXTGR = AETOXGR
  ) %>%
  select(STUDYID, USUBJID, TRTSDT, EXTDT, EXTTERM, EXTGR)

# Write output
write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"STUDYID","USUBJID","TRTSDT","EXTDT","EXTTERM","EXTGR"
"S1",1,2020-01-01,2020-01-08,"NAUSEA",3
"S1",2,2020-01-05,2020-01-07,"RASH",2
"S1",3,2020-01-10,2020-01-15,"COUGH",2
```

### LLM Output
No LLM output artifact was produced.
