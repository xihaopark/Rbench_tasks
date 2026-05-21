# Case 100: pharmaverse/admiral/derive_param_tte

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_tte`
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
Write an R script for an R package task: perform derive param tte using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: Derive the simplified overall-survival parameter: death records are events with CNSR=0, otherwise use the last-alive date as censored with CNSR=1, and set the fixed OS parameter fields expected by the schema.


Required columns for result.csv: USUBJID, STUDYID, EVNTDESC, SRCDOM, SRCVAR, CNSR, ADT, STARTDT, PARAMCD, PARAM
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset_adsl.tsv`

```text
USUBJID	STUDYID	TRTSDT	DTHFL	DTHDT	LSALVDT
01	STUDY1	2021-01-01	Y	2021-04-01	2021-04-01
02	STUDY1	2021-01-02	N		2021-03-15
03	STUDY1	2021-01-05	N		2021-02-20
```

## Code
### Ground Truth Code

```r
adsl <- read.delim(
  file.path("inputs", "dataset_adsl.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE,
  na.strings = c("", "NA")
)
for (col in c("TRTSDT", "DTHDT", "LSALVDT")) {
  adsl[[col]] <- as.Date(adsl[[col]])
}

is_death <- adsl$DTHFL == "Y"
result <- data.frame(
  USUBJID = adsl$USUBJID,
  STUDYID = adsl$STUDYID,
  EVNTDESC = ifelse(is_death, "DEATH", "LAST KNOWN ALIVE"),
  SRCDOM = "ADSL",
  SRCVAR = ifelse(is_death, "DTHDT", "LSALVDT"),
  CNSR = ifelse(is_death, 0L, 1L),
  ADT = ifelse(is_death, as.character(adsl$DTHDT), as.character(adsl$LSALVDT)),
  STARTDT = as.character(adsl$TRTSDT),
  PARAMCD = "OS",
  PARAM = "Overall Survival",
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(readr)
library(lubridate)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
adsl <- readr::read_tsv(
  file = file.path("inputs", "dataset_adsl.tsv"),
  col_types = cols(.default = col_character())
) %>%
  mutate(
    TRTSDT = ymd(TRTSDT),
    DTHDT = ymd(DTHDT),
    LSALVDT = ymd(LSALVDT)
  )

# Derive TTE for simplified overall survival
adtte <- admiral::derive_param_tte(
  dataset_adsl = adsl,
  start_date = TRTSDT,
  event_conditions = list(
    admiral::event_source(
      dataset_name = "adsl",
      filter = DTHFL == "Y",
      date = DTHDT,
      set_values_to = exprs(
        EVNTDESC = "Death",
        SRCDOM   = "ADSL",
        SRCVAR   = "DTHDT"
      )
    )
  ),
  censor_conditions = list(
    admiral::censor_source(
      dataset_name = "adsl",
      filter = is.na(DTHDT),
      date = LSALVDT,
      set_values_to = exprs(
        EVNTDESC = "Last known alive",
        SRCDOM   = "ADSL",
        SRCVAR   = "LSALVDT"
      )
    )
  ),
  subject_keys = exprs(STUDYID, USUBJID)
) %>%
  mutate(
    PARAMCD = "OS",
    PARAM   = "Overall Survival"
  )

# Prepare output with required columns and rename as needed
result <- adtte %>%
  transmute(
    USUBJID,
    STUDYID,
    EVNTDESC,
    SRCDOM,
    SRCVAR,
    CNSR,
    ADT,
    STARTDT
      = STARTDT,
    PARAMCD,
    PARAM
  )

# Write output
readr::write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","STUDYID","EVNTDESC","SRCDOM","SRCVAR","CNSR","ADT","STARTDT","PARAMCD","PARAM"
1,"STUDY1","DEATH","ADSL","DTHDT",0,"2021-04-01","2021-01-01","OS","Overall Survival"
2,"STUDY1","LAST KNOWN ALIVE","ADSL","LSALVDT",1,"2021-03-15","2021-01-02","OS","Overall Survival"
3,"STUDY1","LAST KNOWN ALIVE","ADSL","LSALVDT",1,"2021-02-20","2021-01-05","OS","Overall Survival"
```

### LLM Output
No LLM output artifact was produced.
