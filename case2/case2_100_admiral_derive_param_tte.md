# Case 100: pharmaverse/admiral/derive_param_tte

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_tte`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`
- Failure pattern: `package_api_hallucination`
- Failure pattern note: unused arguments (date = DTHDT, filter = DTHFL == "Y")

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

library(readr)
library(dplyr)
library(tidyr)
library(lubridate)
library(admiral)

# Input/Output paths
infile <- file.path("inputs", "dataset_adsl.tsv")
outdir <- "outputs"
outfile <- file.path(outdir, "result.csv")

if (!dir.exists(outdir)) {
  dir.create(outdir, recursive = TRUE)
}

# Read ADSL
adsl <- read_tsv(
  infile,
  col_types = cols(
    USUBJID = col_character(),
    STUDYID = col_character(),
    TRTSDT  = col_date(format = ""),
    DTHFL   = col_character(),
    DTHDT   = col_date(format = ""),
    LSALVDT = col_date(format = "")
  )
)

# Prepare ADSL for admiral::derive_param_tte
adsl <- adsl %>%
  mutate(
    DTHFL   = if_else(is.na(DTHFL), "N", DTHFL),
    DTHDT   = DTHDT,
    LSALVDT = LSALVDT
  )

# Derive death event
adtte_death <- derive_param_tte(
  dataset_adsl = adsl,
  start_date   = TRTSDT,
  event_conditions = list(
    event(
      description = "Death",
      date        = DTHDT,
      set_values_to = exprs(
        SRCDOM = "ADSL",
        SRCVAR = "DTHDT"
      ),
      filter = DTHFL == "Y"
    )
  ),
  censor_conditions = list(
    censor(
      description = "Last alive",
      date        = LSALVDT,
      set_values_to = exprs(
        SRCDOM = "ADSL",
        SRCVAR = "LSALVDT"
      )
    )
  ),
  subject_keys = exprs(STUDYID, USUBJID),
  set_values_to = exprs(
    PARAMCD = "OS",
    PARAM   = "Overall Survival"
  )
)

# Simplify to required columns and schema
result <- adtte_death %>%
  transmute(
    USUBJID,
    STUDYID,
    EVNTDESC = EVNTDESC,
    SRCDOM,
    SRCVAR,
    CNSR,
    ADT,
    STARTDT = TRTSDT,
    PARAMCD,
    PARAM
  ) %>%
  arrange(STUDYID, USUBJID)

# Write output
write_csv(result, outfile, na = "")
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
