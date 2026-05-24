# Case 016: pharmaverse/aNCA/detect_study_types

## Metadata
- Task ID: `pharmaverse/aNCA/detect_study_types`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: study_type.
```

## Prompt
```text
Write an R script to perform detect study types using the aNCA clinical task contract.

Input: data.tsv, groups.tsv, metabfl_column.tsv, route_column.tsv, volume_column.tsv
Output: result.csv


Required columns for result.csv: subject, study_type
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `data.tsv`


```text
subject	period	route	urine_volume	metabfl
101	1	oral	0	parent
101	2	oral	0	parent
102	1	iv infusion	0	parent
103	1	urine	150	metabolite
```

#### `groups.tsv`


```text
groups
subject
```

#### `metabfl_column.tsv`


```text
metabfl_column
metabfl
```

#### `route_column.tsv`


```text
route_column
route
```

#### `volume_column.tsv`


```text
volume_column
urine_volume
```

## Code
### Ground Truth Code

```r
read_table <- function(name) {
  read.delim(file.path("inputs", name), check.names = FALSE, stringsAsFactors = FALSE)
}

first_value <- function(df, default = "") {
  if (nrow(df) == 0 || ncol(df) == 0 || is.na(df[[1]][[1]])) return(default)
  as.character(df[[1]][[1]])
}

is_extravascular <- function(route) {
  grepl("^(extravascular|oral|po|subcutaneous|sc|intramuscular|im)$", tolower(route))
}

classify_group <- function(df, route_col, volume_col) {
  if (volume_col %in% names(df)) {
    volume <- suppressWarnings(as.numeric(df[[volume_col]]))
    if (any(!is.na(volume) & volume > 0)) return("Excretion Data")
  }
  route <- if (route_col %in% names(df)) df[[route_col]][[1]] else NA_character_
  multiple <- ("TRTRINT" %in% names(df) && any(!is.na(df$TRTRINT))) || ("DOSNO" %in% names(df) && length(unique(df$DOSNO)) > 1)
  bolus <- "ADOSEDUR" %in% names(df) && suppressWarnings(as.numeric(df$ADOSEDUR[[1]])) == 0
  if (is.na(route)) return("Unknown")
  if (is_extravascular(route)) return(ifelse(multiple, "Multiple Extravascular Doses", "Single Extravascular Dose"))
  if (bolus) return(ifelse(multiple, "Multiple IV Bolus Doses", "Single IV Bolus Dose"))
  ifelse(multiple, "Multiple IV Infusion Doses", "Single IV Infusion Dose")
}

data <- read_table("data.tsv")
groups <- as.character(read_table("groups.tsv")[[1]])
groups <- groups[groups %in% names(data)]
route_col <- first_value(read_table("route_column.tsv"), "route")
volume_col <- first_value(read_table("volume_column.tsv"), "volume")

if (!(route_col %in% names(data)) && !(volume_col %in% names(data))) {
  result <- data.frame(study_type = "unknown", stringsAsFactors = FALSE)
} else if (length(groups) == 0) {
  result <- data.frame(study_type = classify_group(data, route_col, volume_col), stringsAsFactors = FALSE)
} else {
  keys <- unique(data[groups])
  study_type <- character(nrow(keys))
  for (i in seq_len(nrow(keys))) {
    keep <- rep(TRUE, nrow(data))
    for (col in groups) keep <- keep & data[[col]] == keys[[col]][[i]]
    study_type[[i]] <- classify_group(data[keep, , drop = FALSE], route_col, volume_col)
  }
  result <- cbind(keys, study_type = study_type)
}

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

data_path   <- file.path("inputs", "data.tsv")
groups_path <- file.path("inputs", "groups.tsv")
metab_path  <- file.path("inputs", "metabfl_column.tsv")
route_path  <- file.path("inputs", "route_column.tsv")
vol_path    <- file.path("inputs", "volume_column.tsv")

data   <- read_tsv(data_path, show_col_types = FALSE)
groups <- read_tsv(groups_path, show_col_types = FALSE)
metab_col <- read_tsv(metab_path, show_col_types = FALSE)$metabfl_column[1]
route_col <- read_tsv(route_path, show_col_types = FALSE)$route_column[1]
vol_col   <- read_tsv(vol_path, show_col_types = FALSE)$volume_column[1]

subject_col <- groups$groups[1]

if (!subject_col %in% names(data)) stop("Subject column not in data")
if (!metab_col %in% names(data))   stop("Metab flag column not in data")
if (!route_col %in% names(data))   stop("Route column not in data")
if (!vol_col %in% names(data))     stop("Volume column not in data")

d <- data %>%
  mutate(
    .subject = .data[[subject_col]],
    .metabfl = as.character(.data[[metab_col]]),
    .route   = tolower(as.character(.data[[route_col]])),
    .volume  = suppressWarnings(as.numeric(.data[[vol_col]]))
  )

# Heuristic for aNCA study types (clinical task contract style)
# PK: parent only, non-urine routes, volumes zero or NA
# CL: urine route with metabolite data and positive volume
# PKCL: mixture (both IV/oral parent and urine metabolite with volume)
study_summary <- d %>%
  group_by(.subject) %>%
  summarise(
    any_urine_route = any(.route %in% c("urine", "urinary", "urine collection"), na.rm = TRUE),
    any_pos_volume  = any(!is.na(.volume) & .volume > 0),
    any_parent      = any(tolower(.metabfl) %in% c("parent", "p"), na.rm = TRUE),
    any_metabolite  = any(tolower(.metabfl) %in% c("metabolite", "m"), na.rm = TRUE),
    .groups = "drop"
  ) %>%
  mutate(
    study_type = case_when(
      any_urine_route & any_pos_volume & any_metabolite & any_parent ~ "PKCL",
      any_urine_route & any_pos_volume & any_metabolite              ~ "CL",
      TRUE                                                           ~ "PK"
    )
  )

result <- study_summary %>%
  transmute(
    subject = .subject,
    study_type
  ) %>%
  arrange(subject)

write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"subject","study_type"
101,"Single Extravascular Dose"
102,"Single IV Infusion Dose"
103,"Excretion Data"
```

### LLM Output
#### `result.csv`


```csv
subject,study_type
101,PK
102,PK
103,CL
```
