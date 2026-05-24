# Case 088: pharmaverse/admiral/convert_xxtpt_to_hours

## Metadata
- Task ID: `pharmaverse/admiral/convert_xxtpt_to_hours`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric mismatch in column: result.
```

## Prompt
```text
Write an R script to perform convert xxtpt to hours using the admiral clinical task contract.

Input: range_method.tsv, treatment_duration.tsv, xxtpt.tsv
Output: result.csv


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
4 POST DOSE
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
  library(readr)
  library(dplyr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
range_method <- read_tsv("inputs/range_method.tsv", show_col_types = FALSE) %>%
  slice(1) %>%
  pull(range_method)

treatment_duration <- read_tsv("inputs/treatment_duration.tsv", show_col_types = FALSE) %>%
  slice(1) %>%
  pull(treatment_duration)

xxtpt_df <- read_tsv("inputs/xxtpt.tsv", show_col_types = FALSE)

# Helper to convert xxtpt to hours
convert_xxtpt_to_hours <- function(x) {
  x_trim <- str_trim(x)
  x_up   <- toupper(x_trim)

  # 1) Patterns with explicit HOURS / H / MIN
  # e.g. "1 HOURS", "2H 30MIN"
  # Extract days, hours, minutes generically where present
  days <- ifelse(str_detect(x_up, "\\bDAY\\b|\\bDAYS\\b"),
                 as.numeric(str_extract(x_up, "(?<![0-9])[0-9]+(?=\\s*DAY|\\s*DAYS)")),
                 NA_real_)
  hours <- ifelse(str_detect(x_up, "\\bHOURS\\b|\\bHOUR\\b|[0-9]+\\s*H\\b"),
                  as.numeric(str_extract(x_up, "(?<![0-9])[0-9]+(?=\\s*HOURS|\\s*HOUR|\\s*H\\b)")),
                  NA_real_)
  mins <- ifelse(str_detect(x_up, "MIN\\b|MINS\\b|MINUTE\\b|MINUTES\\b"),
                 as.numeric(str_extract(x_up, "(?<![0-9])[0-9]+(?=\\s*MIN\\b|\\s*MINS\\b|\\s*MINUTE\\b|\\s*MINUTES\\b)")),
                 NA_real_)

  # Initialize result as NA
  res <- rep(NA_real_, length(x_up))

  # If explicit days/hours/mins present, compute
  has_any_time <- !is.na(days) | !is.na(hours) | !is.na(mins)
  res[has_any_time] <-
    coalesce(days[has_any_time], 0) * 24 +
    coalesce(hours[has_any_time], 0) +
    coalesce(mins[has_any_time], 0) / 60

  # 2) DAY n pattern (study day) if not already converted
  # Assume DAY n refers to n-1 days after first dose -> hours = (n-1)*24
  day_only_idx <- which(is.na(res) & str_detect(x_up, "^DAY\\s*[0-9]+$"))
  if (length(day_only_idx) > 0) {
    n <- as.numeric(str_extract(x_up[day_only_idx], "(?<![0-9])[0-9]+$"))
    res[day_only_idx] <- (n - 1) * 24
  }

  # 3) POST DOSE pattern if not already converted
  # For "4 POST DOSE": interpret as 4 * treatment_duration hours
  post_dose_idx <- which(is.na(res) & str_detect(x_up, "POST\\s*DOSE"))
  if (length(post_dose_idx) > 0) {
    n <- suppressWarnings(as.numeric(str_extract(x_up[post_dose_idx], "^[0-9]+")))
    res[post_dose_idx] <- n * as.numeric(treatment_duration)
  }

  res
}

result_df <- xxtpt_df %>%
  mutate(
    treatment_duration = treatment_duration,
    range_method = range_method,
    result = convert_xxtpt_to_hours(xxtpt)
  ) %>%
  select(xxtpt, treatment_duration, range_method, result)

write_csv(result_df, "outputs/result.csv", na = "")
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
"4 POST DOSE",24,"midpoint",96
```

### LLM Output
#### `result.csv`


```csv
xxtpt,treatment_duration,range_method,result
1 HOURS,24,midpoint,1
2H 30MIN,24,midpoint,2.5
3 DAYS,24,midpoint,72
DAY 2,24,midpoint,24
4 POST DOSE,24,midpoint,96
```
