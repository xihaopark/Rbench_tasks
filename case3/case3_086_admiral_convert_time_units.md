# Case 086: pharmaverse/admiral/convert_time_units

## Metadata
- Task ID: `pharmaverse/admiral/convert_time_units`
- Package: `admiral`
- Model: `openai/gpt-5.5`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the ground truth.

Main signal:
result.csv: Numeric mismatch in column: result
```

## Prompt
```text
Write an R script for an R package task: perform convert time units using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv


Required columns for result.csv: xxtpt, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `na_idx.tsv`

```text
na_idx
FALSE
FALSE
FALSE
FALSE
```

#### `result.tsv`

```text
result
NA
NA
NA
NA
```

#### `xxtpt.tsv`

```text
xxtpt
2H 30MIN
3 DAYS
DAY 2
4D
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

xxtpt <- as.character(read.delim(file.path("inputs", "xxtpt.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
result <- suppressWarnings(as.numeric(read.delim(file.path("inputs", "result.tsv"), check.names = FALSE, stringsAsFactors = FALSE, na.strings = c("", "NA"))[[1]]))
na_idx <- as.logical(read.delim(file.path("inputs", "na_idx.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])

convert_one <- function(x, current, is_na) {
  if (!is.na(current) || isTRUE(is_na)) return(current)
  x_trim <- trimws(toupper(x))
  if (grepl("^[0-9.]+\\s*H(OUR|OURS|R)?S?\\s+[0-9.]+\\s*M(IN|INUTE)?S?$", x_trim)) {
    nums <- as.numeric(regmatches(x_trim, gregexpr("[0-9.]+", x_trim))[[1]])
    return(nums[1] + nums[2] / 60)
  }
  if (grepl("^[0-9.]+\\s*D(AY|AYS)?$", x_trim)) {
    return(as.numeric(sub("^([0-9.]+).*", "\\1", x_trim)) * 24)
  }
  if (grepl("^DAY\\s+[0-9.]+$", x_trim)) {
    return(as.numeric(sub("^DAY\\s+([0-9.]+)$", "\\1", x_trim)) * 24)
  }
  if (grepl("^[0-9.]+\\s+POST\\s*DOSE$", x_trim)) {
    return(as.numeric(sub("^([0-9.]+).*", "\\1", x_trim)) * 24)
  }
  current
}
result <- mapply(convert_one, xxtpt, result, na_idx)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(xxtpt = xxtpt, result = as.numeric(result)), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

na_idx_df <- read.delim("inputs/na_idx.tsv", stringsAsFactors = FALSE)
result_df <- read.delim("inputs/result.tsv", stringsAsFactors = FALSE, na.strings = c("", "NA"))
xxtpt_df <- read.delim("inputs/xxtpt.tsv", stringsAsFactors = FALSE)

convert_with_admiral <- function(value, in_unit, out_unit = "days") {
  converted <- tryCatch(
    as.numeric(admiral::convert_unit(value, in_unit, out_unit)),
    error = function(e) NA_real_
  )

  if (!is.na(converted)) {
    return(converted)
  }

  unit_factor_days <- c(
    seconds = 1 / 86400,
    minutes = 1 / 1440,
    hours = 1 / 24,
    days = 1,
    weeks = 7,
    months = 365.25 / 12,
    years = 365.25
  )

  as.numeric(value) * unit_factor_days[[in_unit]] / unit_factor_days[[out_unit]]
}

standardize_unit <- function(unit) {
  unit <- toupper(unit)

  if (unit %in% c("D", "DAY", "DAYS")) {
    "days"
  } else if (unit %in% c("H", "HR", "HRS", "HOUR", "HOURS")) {
    "hours"
  } else if (unit %in% c("MIN", "MINS", "MINUTE", "MINUTES")) {
    "minutes"
  } else if (unit %in% c("S", "SEC", "SECS", "SECOND", "SECONDS")) {
    "seconds"
  } else {
    NA_character_
  }
}

parse_time_to_days <- function(x) {
  if (is.na(x) || !nzchar(trimws(x))) {
    return(NA_real_)
  }

  txt <- toupper(trimws(x))
  unit_re <- "(?:DAYS?|D|HOURS?|HRS?|HR|H|MINUTES?|MINS?|MIN|SECONDS?|SECS?|SEC|S)"
  num_re <- "[0-9]+(?:\\.[0-9]+)?"

  total <- 0
  found <- FALSE

  pattern_num_unit <- paste0("\\b", num_re, "\\s*", unit_re, "\\b")
  hits <- regmatches(txt, gregexpr(pattern_num_unit, txt, perl = TRUE))[[1]]

  if (length(hits) > 0 && !identical(hits, character(0)) && hits[1] != "-1") {
    for (hit in hits) {
      value <- as.numeric(sub(paste0("^\\s*(", num_re, ").*$"), "\\1", hit, perl = TRUE))
      unit <- sub(paste0("^\\s*", num_re, "\\s*(", unit_re, ").*$"), "\\1", hit, perl = TRUE)
      unit <- standardize_unit(unit)

      if (!is.na(value) && !is.na(unit)) {
        total <- total + convert_with_admiral(value, unit, "days")
        found <- TRUE
      }
    }
  }

  pattern_unit_num <- paste0("\\b", unit_re, "\\s*", num_re, "\\b")
  hits <- regmatches(txt, gregexpr(pattern_unit_num, txt, perl = TRUE))[[1]]

  if (length(hits) > 0 && !identical(hits, character(0)) && hits[1] != "-1") {
    for (hit in hits) {
      unit <- sub(paste0("^\\s*(", unit_re, ")\\s*", num_re, ".*$"), "\\1", hit, perl = TRUE)
      value <- as.numeric(sub(paste0("^\\s*", unit_re, "\\s*(", num_re, ").*$"), "\\1", hit, perl = TRUE))
      unit <- standardize_unit(unit)

      if (!is.na(value) && !is.na(unit)) {
        total <- total + convert_with_admiral(value, unit, "days")
        found <- TRUE
      }
    }
  }

  if (found) total else NA_real_
}

result <- vapply(xxtpt_df$xxtpt, parse_time_to_days, numeric(1))

if ("na_idx" %in% names(na_idx_df)) {
  result[as.logical(na_idx_df$na_idx)] <- NA_real_
}

out <- data.frame(
  xxtpt = xxtpt_df$xxtpt,
  result = result
)

write.csv(out, "outputs/result.csv", row.names = FALSE, na = "NA")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"xxtpt","result"
"2H 30MIN",2.5
"3 DAYS",72
"DAY 2",48
"4D",96
```

### LLM Output
#### `result.csv`

```csv
"xxtpt","result"
"2H 30MIN",0.104166666666667
"3 DAYS",3
"DAY 2",2
"4D",4
```
