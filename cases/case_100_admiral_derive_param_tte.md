# Case 100: pharmaverse/admiral/derive_param_tte

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_tte`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: EVNTDESC.
```

## Prompt
```text
Write an R script to perform derive param tte using the admiral clinical task contract.

Input: dataset_adsl.tsv
Output: result.csv


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

library(dplyr)
library(readr)
library(lubridate)

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input ADSL
adsl <- read_tsv(
  file = file.path("inputs", "dataset_adsl.tsv"),
  col_types = cols(.default = col_character())
)

# Convert dates
adsl <- adsl %>%
  mutate(
    TRTSDT   = ymd(TRTSDT),
    DTHDT    = ymd(DTHDT),
    LSALVDT  = ymd(LSALVDT)
  )

# Derive ADT (event/analysis date) and censoring
res <- adsl %>%
  mutate(
    # Event description and source
    EVNTDESC = "Time to Death",
    SRCDOM   = "ADSL",
    SRCVAR   = "DTHDT",
    # Event date: death date if death flag Y, otherwise last alive date
    ADT = case_when(
      DTHFL == "Y" & !is.na(DTHDT) ~ DTHDT,
      TRUE ~ LSALVDT
    ),
    # Censoring: 0 = event (death), 1 = censored
    CNSR = if_else(DTHFL == "Y" & !is.na(DTHDT), 0L, 1L),
    # Start date
    STARTDT = TRTSDT,
    # Parameter
    PARAMCD = "TTEDEATH",
    PARAM   = "Time to Death"
  ) %>%
  select(
    USUBJID,
    STUDYID,
    EVNTDESC,
    SRCDOM,
    SRCVAR,
    CNSR,
    ADT,
    STARTDT,
    PARAMCD,
    PARAM
  )

# Write result
write_csv(res, file.path("outputs", "result.csv"), na = "")
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
#### `result.csv`


```csv
USUBJID,STUDYID,EVNTDESC,SRCDOM,SRCVAR,CNSR,ADT,STARTDT,PARAMCD,PARAM
01,STUDY1,Time to Death,ADSL,DTHDT,0,2021-04-01,2021-01-01,TTEDEATH,Time to Death
02,STUDY1,Time to Death,ADSL,DTHDT,1,2021-03-15,2021-01-02,TTEDEATH,Time to Death
03,STUDY1,Time to Death,ADSL,DTHDT,1,2021-02-20,2021-01-05,TTEDEATH,Time to Death
```
