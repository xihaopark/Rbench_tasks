# Case 108: pharmaverse/admiral/derive_vars_atc

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_atc`
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
Write an R script to perform derive vars atc using the admiral clinical task contract.

Input: cm.tsv, facm.tsv
Output: result.csv


Required columns for result.csv: USUBJID, CMREFID, CMTRT, ATC1, ATC2
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `cm.tsv`


```text
USUBJID	CMREFID	CMTRT
01	CM1	ASPIRIN
01	CM2	METFORMIN
02	CM3	ATORVASTATIN
```

#### `facm.tsv`


```text
USUBJID	FAREFID	FATESTCD	FASTRESC
01	CM1	ATC1	B01AC06
01	CM1	ATC2	ASPIRIN
01	CM2	ATC1	A10BA02
02	CM3	ATC1	C10AA05
```

## Code
### Ground Truth Code

```r
cm <- read.delim(file.path("inputs", "cm.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
facm <- read.delim(file.path("inputs", "facm.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df <- cm
for (code in sort(unique(facm$FATESTCD))) {
  result_df[[code]] <- NA_character_
}
for (i in seq_len(nrow(facm))) {
  hit <- result_df$USUBJID == facm$USUBJID[i] & result_df$CMREFID == facm$FAREFID[i]
  result_df[[facm$FATESTCD[i]]][hit] <- facm$FASTRESC[i]
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(tidyr)
library(stringr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input data
cm <- read_tsv(
  file = file.path("inputs", "cm.tsv"),
  col_types = cols(.default = col_character())
)

facm <- read_tsv(
  file = file.path("inputs", "facm.tsv"),
  col_types = cols(.default = col_character())
)

# Prepare FACM: keep only ATC1/ATC2 and pivot to wide
facm_wide <- facm %>%
  filter(FATESTCD %in% c("ATC1", "ATC2")) %>%
  select(USUBJID, FAREFID, FATESTCD, FASTRESC) %>%
  mutate(FATESTCD = str_trim(FATESTCD)) %>%
  pivot_wider(
    id_cols = c(USUBJID, FAREFID),
    names_from = FATESTCD,
    values_from = FASTRESC
  ) %>%
  rename(
    CMREFID = FAREFID
  )

# Merge with CM and select required columns
result <- cm %>%
  left_join(facm_wide, by = c("USUBJID", "CMREFID")) %>%
  select(USUBJID, CMREFID, CMTRT, ATC1, ATC2)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","CMREFID","CMTRT","ATC1","ATC2"
1,"CM1","ASPIRIN","B01AC06","ASPIRIN"
1,"CM2","METFORMIN","A10BA02",
2,"CM3","ATORVASTATIN","C10AA05",
```

### LLM Output
#### `result.csv`


```csv
USUBJID,CMREFID,CMTRT,ATC1,ATC2
01,CM1,ASPIRIN,B01AC06,ASPIRIN
01,CM2,METFORMIN,A10BA02,
02,CM3,ATORVASTATIN,C10AA05,
```
