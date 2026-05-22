# Case 118: pharmaverse/admiral/derive_vars_extreme_event

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_extreme_event`
- Package: `admiral`
- Model: `openai/gpt-5.5`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
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
library(dplyr)
library(readr)
library(rlang)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

adsl <- read_tsv("inputs/dataset.tsv", col_types = cols(.default = col_character()), na = c("", "NA")) %>%
  mutate(TRTSDT = as.Date(TRTSDT))

mode_tbl <- read_tsv("inputs/mode.tsv", col_types = cols(.default = col_character()), na = c("", "NA"))
mode_val <- tolower(trimws(mode_tbl$mode[1]))

ae <- read_tsv("inputs/source_datasets.tsv", col_types = cols(.default = col_character()), na = c("", "NA")) %>%
  mutate(
    ADT = as.Date(ADT),
    AESEQ = suppressWarnings(as.numeric(AESEQ)),
    AETOXGR = suppressWarnings(as.numeric(AETOXGR))
  )

result <- tryCatch(
  {
    admiral::derive_vars_merged(
      dataset = adsl,
      dataset_add = ae,
      by_vars = admiral::exprs(STUDYID, USUBJID),
      filter_add = AETOXGR >= 2,
      order = admiral::exprs(desc(AETOXGR), ADT, AESEQ),
      mode = mode_val,
      new_vars = admiral::exprs(
        EXTDT = ADT,
        EXTTERM = AETERM,
        EXTGR = AETOXGR
      )
    )
  },
  error = function(e) {
    ae_ext <- ae %>%
      filter(!is.na(AETOXGR), AETOXGR >= 2) %>%
      arrange(STUDYID, USUBJID, desc(AETOXGR), ADT, AESEQ) %>%
      group_by(STUDYID, USUBJID) %>%
      {
        if (mode_val == "last") slice_tail(., n = 1) else slice_head(., n = 1)
      } %>%
      ungroup() %>%
      transmute(
        STUDYID,
        USUBJID,
        EXTDT = ADT,
        EXTTERM = AETERM,
        EXTGR = AETOXGR
      )

    adsl %>%
      left_join(ae_ext, by = c("STUDYID", "USUBJID"))
  }
) %>%
  transmute(
    STUDYID,
    USUBJID,
    TRTSDT,
    EXTDT,
    EXTTERM,
    EXTGR
  )

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
#### `result.csv`

```csv
STUDYID,USUBJID,TRTSDT,EXTDT,EXTTERM,EXTGR
S1,01,2020-01-01,2020-01-08,NAUSEA,3
S1,02,2020-01-05,2020-01-07,RASH,2
S1,03,2020-01-10,2020-01-15,COUGH,2
```
