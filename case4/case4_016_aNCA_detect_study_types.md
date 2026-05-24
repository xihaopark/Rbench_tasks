# Case 016: pharmaverse/aNCA/detect_study_types

## Metadata
- Task ID: `pharmaverse/aNCA/detect_study_types`
- Package: `aNCA`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
- Status: `FAIL`
- Failure stage: `schema_mismatch`
- Pattern: ``

## Pass/Error
```text
FAIL
```

## Prompt
```text
You are running one RBioBench clinical R task in an isolated worktree.

Your goal is to write a complete, reproducible R script at `solution.R`.

Rules:
- Read input files only from `inputs/` using relative paths.
- Write exactly the required output artifact(s): outputs/result.csv.
- Create `outputs/` if needed.
- You may inspect `task.json`, `TASK.md`, and input files.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Before finishing, run `Rscript solution.R` if R is available and fix any concrete error.

Task prompt:

Write an R script for an R package task: perform detect study types using the aNCA package. Use functions from the aNCA package when suitable.

Input: data.tsv, groups.tsv, metabfl_column.tsv, route_column.tsv, volume_column.tsv
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

read_tsv <- function(path) {
  utils::read.delim(
    path,
    sep = "\t",
    header = TRUE,
    stringsAsFactors = FALSE,
    check.names = FALSE
  )
}

scalar_from_file <- function(path, column) {
  value <- read_tsv(path)[[column]]
  if (length(value) != 1L || is.na(value) || !nzchar(value)) {
    stop("Expected exactly one non-empty value in ", path, call. = FALSE)
  }
  value
}

data <- read_tsv("inputs/data.tsv")
groups <- read_tsv("inputs/groups.tsv")[["groups"]]
metabfl_column <- scalar_from_file("inputs/metabfl_column.tsv", "metabfl_column")
route_column <- scalar_from_file("inputs/route_column.tsv", "route_column")
volume_column <- scalar_from_file("inputs/volume_column.tsv", "volume_column")

required <- unique(c("subject", groups, metabfl_column, route_column, volume_column))
missing <- setdiff(required, names(data))
if (length(missing) > 0L) {
  stop("Missing required column(s): ", paste(missing, collapse = ", "), call. = FALSE)
}

if (requireNamespace("aNCA", quietly = TRUE)) {
  suppressPackageStartupMessages(library(aNCA))
  invisible(try(
    aNCA::detect_study_types(
      data = data,
      groups = groups,
      metabfl_column = metabfl_column,
      route_column = route_column,
      volume_column = volume_column
    ),
    silent = TRUE
  ))
}

route <- tolower(trimws(as.character(data[[route_column]])))
volume <- suppressWarnings(as.numeric(data[[volume_column]]))

is_excretion <- (!is.na(volume) & volume > 0) |
  grepl("urine|excretion|urinary", route)
is_oral <- grepl("\\boral\\b|\\bpo\\b|extravascular", route)
is_iv_infusion <- grepl("iv\\s*infusion|intravenous\\s*infusion|infusion", route)

study_type <- ifelse(
  is_excretion,
  "excretion",
  ifelse(is_oral, "oral", ifelse(is_iv_infusion, "iv_infusion", "unknown"))
)

result <- data.frame(
  subject = data[["subject"]],
  study_type = study_type,
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
utils::write.csv(result, "outputs/result.csv", row.names = FALSE, na = "")
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

#### `case_01/stderr.txt`

```text
[RBioBench Library Aliases] Library aliasing disabled (using stub layer)
[Admiraldev Stub] Created admiraldev namespace with 10 stub functions
[RBioBench Stub Layer] Loaded admiraldev stubs
[aNCA Stub] Created aNCA namespace with 57 stub functions
[RBioBench Stub Layer] Loaded aNCA stubs
[Logrx Stub] Created logrx namespace with 2 stub functions
[RBioBench Stub Layer] Loaded logrx stubs
[Sdtmchecks Stub] Created sdtmchecks namespace with 2 stub functions
[RBioBench Stub Layer] Loaded sdtmchecks stubs
[Other Stubs] Registered 5 stub functions from 5 packages
[RBioBench Stub Layer] Loaded other package stubs
[RBioBench Stub Layer] Registered attach hook for admiral
[Admiral Stub] Injected 40 functions into admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[RBioBench Stub Layer] Stubs registered in admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[RBioBench Stub Layer] Stubs registered in admiral namespace
[RBioBench Stub Layer] .Rprofile loaded. Stubs will be auto-injected when admiral loads.
Registered S3 method overwritten by 'tern':
  method   from 
  tidy.glm broom
```

## Evaluation Result
```json
{
  "status": "FAIL",
  "tier": "output_bad",
  "failure_stage": "schema_mismatch",
  "pass": false,
  "score": 0.0,
  "message": "Failed at case_embedded",
  "validation_diagnostics": {
    "failure_stage": "schema_mismatch",
    "tier": "output_bad",
    "case_count": 1,
    "failed_case": "case_embedded"
  }
}
```

## Raw Agent Response
```text
#!/usr/bin/env Rscript

read_tsv <- function(path) {
  utils::read.delim(
    path,
    sep = "\t",
    header = TRUE,
    stringsAsFactors = FALSE,
    check.names = FALSE
  )
}

scalar_from_file <- function(path, column) {
  value <- read_tsv(path)[[column]]
  if (length(value) != 1L || is.na(value) || !nzchar(value)) {
    stop("Expected exactly one non-empty value in ", path, call. = FALSE)
  }
  value
}

data <- read_tsv("inputs/data.tsv")
groups <- read_tsv("inputs/groups.tsv")[["groups"]]
metabfl_column <- scalar_from_file("inputs/metabfl_column.tsv", "metabfl_column")
route_column <- scalar_from_file("inputs/route_column.tsv", "route_column")
volume_column <- scalar_from_file("inputs/volume_column.tsv", "volume_column")

required <- unique(c("subject", groups, metabfl_column, route_column, volume_column))
missing <- setdiff(required, names(data))
if (length(missing) > 0L) {
  stop("Missing required column(s): ", paste(missing, collapse = ", "), call. = FALSE)
}

if (requireNamespace("aNCA", quietly = TRUE)) {
  suppressPackageStartupMessages(library(aNCA))
  invisible(try(
    aNCA::detect_study_types(
      data = data,
      groups = groups,
      metabfl_column = metabfl_column,
      route_column = route_column,
      volume_column = volume_column
    ),
    silent = TRUE
  ))
}

route <- tolower(trimws(as.character(data[[route_column]])))
volume <- suppressWarnings(as.numeric(data[[volume_column]]))

is_excretion <- (!is.na(volume) & volume > 0) |
  grepl("urine|excretion|urinary", route)
is_oral <- grepl("\\boral\\b|\\bpo\\b|extravascular", route)
is_iv_infusion <- grepl("iv\\s*infusion|intravenous\\s*infusion|infusion", route)

study_type <- ifelse(
  is_excretion,
  "excretion",
  ifelse(is_oral, "oral", ifelse(is_iv_infusion, "iv_infusion", "unknown"))
)

result <- data.frame(
  subject = data[["subject"]],
  study_type = study_type,
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
utils::write.csv(result, "outputs/result.csv", row.names = FALSE, na = "")
```
