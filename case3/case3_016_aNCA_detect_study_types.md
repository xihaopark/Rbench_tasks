# Case 016: pharmaverse/aNCA/detect_study_types

## Metadata
- Task ID: `pharmaverse/aNCA/detect_study_types`
- Package: `aNCA`
- Model: `openai/gpt-5.5`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output files were produced, but the schema or output shape differed from the ground truth.

Main signal:
result.csv: Shape mismatch: ref=(3, 2) vs llm=(4, 2)
```

## Prompt
```text
Write an R script for an R package task: perform detect study types using the aNCA package. Use functions from the aNCA package when suitable.

Output: result.csv

Computation: Classify rows by route/evidence: oral rows as oral, IV infusion rows as iv_infusion, and excretion rows as excretion; preserve one output row per input row.


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
  if (requireNamespace("aNCA", quietly = TRUE)) {
    library(aNCA)
  }
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

read_tsv <- function(path) {
  read.delim(path, header = TRUE, sep = "\t", stringsAsFactors = FALSE, check.names = FALSE)
}

read_first_value <- function(path) {
  x <- read_tsv(path)
  as.character(x[[1]][1])
}

data <- read_tsv(file.path("inputs", "data.tsv"))
groups <- read_tsv(file.path("inputs", "groups.tsv"))[[1]]

route_col <- read_first_value(file.path("inputs", "route_column.tsv"))
metabfl_col <- read_first_value(file.path("inputs", "metabfl_column.tsv"))
volume_col <- read_first_value(file.path("inputs", "volume_column.tsv"))

route <- tolower(trimws(as.character(data[[route_col]])))

volume <- rep(NA_real_, nrow(data))
if (!is.na(volume_col) && volume_col %in% names(data)) {
  volume <- suppressWarnings(as.numeric(data[[volume_col]]))
}

study_type <- rep(NA_character_, nrow(data))

is_excretion <- grepl("urine|feces|faeces|excret|renal|urinary", route) |
  (!is.na(volume) & volume > 0)

is_iv_infusion <- grepl("\\biv\\b|intravenous", route) &
  grepl("infusion|inf", route)

is_oral <- grepl("\\boral\\b|\\bpo\\b|per os", route)

study_type[is_excretion] <- "excretion"
study_type[!is_excretion & is_iv_infusion] <- "iv_infusion"
study_type[!is_excretion & !is_iv_infusion & is_oral] <- "oral"

result <- data.frame(
  subject = data[["subject"]],
  study_type = study_type,
  stringsAsFactors = FALSE,
  check.names = FALSE
)

write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
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
"subject","study_type"
101,"oral"
101,"oral"
102,"iv_infusion"
103,"excretion"
```
