# Case 088: pharmaverse/admiral/convert_xxtpt_to_hours

## Metadata
- Task ID: `pharmaverse/admiral/convert_xxtpt_to_hours`
- Package: `admiral`
- Model: `openai/gpt-5.1`
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

library(admiral)
library(readr)
library(dplyr)
library(stringr)
library(fs)

# Input paths
range_method_path <- file.path("inputs", "range_method.tsv")
treatment_duration_path <- file.path("inputs", "treatment_duration.tsv")
xxtpt_path <- file.path("inputs", "xxtpt.tsv")

# Output dir
dir_create("outputs")
output_path <- file.path("outputs", "result.csv")

# Read inputs
range_method_df <- read_tsv(range_method_path, show_col_types = FALSE)
treatment_duration_df <- read_tsv(treatment_duration_path, show_col_types = FALSE)
xxtpt_df <- read_tsv(xxtpt_path, show_col_types = FALSE)

range_method <- range_method_df$range_method[1]
treatment_duration <- treatment_duration_df$treatment_duration[1]

# Use admiral's character parsing utilities to normalize text
parse_time_to_hours <- function(x) {
  x_norm <- x %>%
    str_squish() %>%
    str_to_upper()

  # Pre-dose / screening style labels -> 0
  is_zero <- str_detect(
    x_norm,
    "PRE[- ]?DOSE|SCREEN|BASELINE|PRETREAT|PRE[- ]?TREAT|PRE[- ]?TREATMENT"
  )
  if (is_zero) return(0)

  # Pure numeric hours
  if (str_detect(x_norm, "^[0-9]+(\\.[0-9]+)?$")) {
    return(as.numeric(x_norm))
  }

  # "1 HOURS", "1 HOUR", "1H", "1 H" etc.
  if (str_detect(x_norm, "^[0-9]+(\\.[0-9]+)?\\s*(H|HR|HOUR|HOURS)")) {
    num <- as.numeric(str_extract(x_norm, "^[0-9]+(\\.[0-9]+)?"))
    return(num)
  }

  # "2H 30MIN", "2 H 30 MIN", "2:30", "02:30"
  if (str_detect(x_norm, "^[0-9]+(\\.[0-9]+)?\\s*H\\s*[0-9]+(\\.[0-9]+)?\\s*MIN")) {
    hrs <- as.numeric(str_extract(x_norm, "^[0-9]+(\\.[0-9]+)?"))
    mins <- as.numeric(str_match(x_norm, "H\\s*([0-9]+(\\.[0-9]+)?)\\s*MIN")[, 2])
    return(hrs + mins / 60)
  }

  # HH:MM
  if (str_detect(x_norm, "^[0-9]{1,2}:[0-9]{2}$")) {
    parts <- str_split(x_norm, ":", simplify = TRUE)
    hrs <- as.numeric(parts[, 1])
    mins <- as.numeric(parts[, 2])
    return(hrs + mins / 60)
  }

  # "3 DAYS", "3 DAY", "3D", "DAY 3"
  if (str_detect(x_norm, "^[0-9]+\\s*(D|DAY|DAYS)$")) {
    days <- as.numeric(str_extract(x_norm, "^[0-9]+"))
    return(days * 24)
  }
  if (str_detect(x_norm, "^DAY\\s*[0-9]+$")) {
    days <- as.numeric(str_extract(x_norm, "[0-9]+$"))
    return(days * 24)
  }

  NA_real_
}

result_df <- xxtpt_df %>%
  mutate(
    result = vctrs::vec_cast(
      purrr::map_dbl(.data$xxtpt, parse_time_to_hours),
      double()
    ),
    treatment_duration = treatment_duration,
    range_method = range_method
  ) %>%
  select(xxtpt, treatment_duration, range_method, result)

write_csv(result_df, output_path)
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
#### `result.csv`

```csv
xxtpt,treatment_duration,range_method,result
1 HOURS,24,midpoint,1
2H 30MIN,24,midpoint,2
3 DAYS,24,midpoint,72
DAY 2,24,midpoint,48
4D,24,midpoint,96
```
