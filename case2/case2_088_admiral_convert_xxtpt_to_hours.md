# Case 088: pharmaverse/admiral/convert_xxtpt_to_hours

## Metadata
- Task ID: `pharmaverse/admiral/convert_xxtpt_to_hours`
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
Write an R script for an R package task: perform convert xxtpt to hours using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: Parse timepoint text into hours: numeric hours stay numeric; HH:MM uses hours plus minutes/60; day counts such as 4D or DAY 4 map to 4 * 24 hours; pre-dose/screening style labels map to 0.


Required columns for result.csv: xxtpt, treatment_duration, range_method, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `range_method.tsv`

```text
range_method
midpoint
```

#### `treatment_duration.tsv`

```text
treatment_duration
24
```

#### `xxtpt.tsv`

```text
xxtpt
1 HOURS
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
treatment_duration <- as.numeric(read.delim(file.path("inputs", "treatment_duration.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1])
range_method <- as.character(read.delim(file.path("inputs", "range_method.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1])

convert_one <- function(x) {
  x_trim <- trimws(toupper(x))
  if (grepl("^[0-9.]+\\s*H(OUR|OURS|R)?$", x_trim)) {
    return(as.numeric(sub("^([0-9.]+).*", "\\1", x_trim)))
  }
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
  NA_real_
}
result <- vapply(xxtpt, convert_one, numeric(1))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(xxtpt = xxtpt, treatment_duration = treatment_duration, range_method = range_method, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(admiral)
  library(dplyr)
  library(readr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
range_method <- readr::read_tsv(
  file = file.path("inputs", "range_method.tsv"),
  show_col_types = FALSE
) %>%
  slice(1) %>%
  pull(range_method)

treatment_duration <- readr::read_tsv(
  file = file.path("inputs", "treatment_duration.tsv"),
  show_col_types = FALSE
) %>%
  slice(1) %>%
  pull(treatment_duration)

xxtpt_df <- readr::read_tsv(
  file = file.path("inputs", "xxtpt.tsv"),
  show_col_types = FALSE
)

# Helper to map text timepoints to hours using admiral-style parsing logic
derive_hours_from_xxtpt <- function(xxtpt) {
  xxtpt_clean <- str_trim(toupper(xxtpt))

  # Pre-dose / screening labels -> 0
  ifelse(
    str_detect(xxtpt_clean, "PRE\\s*-?DOSE|SCREEN|BASELINE|CYCLE 1 DAY 1|C1D1|DAY 1 PRE"),
    0,
    NA_real_
  ) %>%
    # Numeric-only -> hours
    coalesce(ifelse(
      str_detect(xxtpt_clean, "^\\d+(\\.\\d+)?$"),
      as.numeric(xxtpt_clean),
      NA_real_
    )) %>%
    # HH:MM or H:MM -> hours + minutes/60
    coalesce(ifelse(
      str_detect(xxtpt_clean, "^\\d{1,2}:\\d{2}$"),
      {
        h <- as.numeric(str_extract(xxtpt_clean, "^\\d{1,2}(?=:)"))
        m <- as.numeric(str_extract(xxtpt_clean, "(?<=:)\\d{2}$"))
        h + m / 60
      },
      NA_real_
    )) %>%
    # Patterns with hours and minutes, e.g. "2H 30MIN", "2 HOURS 30 MINUTES"
    coalesce(ifelse(
      str_detect(xxtpt_clean, "\\d+\\s*H|HOUR"),
      {
        h <- as.numeric(str_extract(xxtpt_clean, "\\d+(?=\\s*H|\\s*HOUR)"))
        m <- ifelse(
          str_detect(xxtpt_clean, "\\d+\\s*MIN"),
          as.numeric(str_extract(xxtpt_clean, "\\d+(?=\\s*MIN)")),
          0
        )
        h + m / 60
      },
      NA_real_
    )) %>%
    # Day counts, e.g. "3 DAYS", "DAY 2", "4D"
    coalesce(ifelse(
      str_detect(xxtpt_clean, "\\bDAY\\b|\\bDAYS\\b|\\d+\\s*D\\b|^DAY\\s*\\d+"),
      {
        d1 <- as.numeric(str_extract(xxtpt_clean, "(?<=DAY\\s*)\\d+"))
        d2 <- as.numeric(str_extract(xxtpt_clean, "\\d+(?=\\s*D\\b)"))
        d3 <- as.numeric(str_extract(xxtpt_clean, "\\d+(?=\\s*DAYS\\b)"))
        d <- coalesce(d1, d2, d3)
        d * 24
      },
      NA_real_
    ))
}

# Apply conversion
result_df <- xxtpt_df %>%
  mutate(
    result = derive_hours_from_xxtpt(xxtpt),
    treatment_duration = treatment_duration,
    range_method = range_method
  ) %>%
  select(xxtpt, treatment_duration, range_method, result)

# Write output
readr::write_csv(result_df, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"xxtpt","treatment_duration","range_method","result"
"1 HOURS",24,"midpoint",1
"2H 30MIN",24,"midpoint",2.5
"3 DAYS",24,"midpoint",72
"DAY 2",24,"midpoint",48
"4D",24,"midpoint",96
```

### LLM Output
No LLM output artifact was produced.
