# Case 086: pharmaverse/admiral/convert_time_units

## Metadata
- Task ID: `pharmaverse/admiral/convert_time_units`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`
- Failure pattern: `package_api_hallucination`
- Failure pattern note: ! 'convert_dur_to_unit' is not an exported object from 'namespace:admiral'

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
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
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(admiral)
  library(readr)
  library(dplyr)
  library(stringr)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
na_idx  <- readr::read_tsv("inputs/na_idx.tsv", show_col_types = FALSE)
result  <- readr::read_tsv("inputs/result.tsv", show_col_types = FALSE)
xxtpt   <- readr::read_tsv("inputs/xxtpt.tsv", show_col_types = FALSE)

# Clean and standardize time strings to a parsable format
xxtpt_clean <- xxtpt %>%
  mutate(
    xxtpt_std = xxtpt %>%
      str_to_upper() %>%
      str_trim() %>%
      # Normalize "DAY 2" -> "2D"
      str_replace_all("^DAY\\s*([0-9]+)$", "\\1D") %>%
      # Normalize "3 DAYS" -> "3D"
      str_replace_all("^([0-9]+)\\s*DAYS?$", "\\1D") %>%
      # Add separators so that cu_to_daytime can parse more easily
      str_replace_all("H", "H ") %>%
      str_replace_all("MIN", "MIN ")
  )

# Convert character time units into hours using admiral::convert_dtc_to_dtm() helpers
# We first convert to a duration in days using admiral::derive_vars_dur(), then map to hours.
# To leverage admiral utilities, create a dummy date-time baseline and use dur_unit = "H".
refdt <- as.Date("2000-01-01")

xxtpt_conv <- xxtpt_clean %>%
  mutate(
    # Build a dummy ISO-8601 like representation that admiral can work with
    # Example: "2D 3H 30MIN" -> P2DT3H30M
    iso_dur = xxtpt_std %>%
      str_replace_all("([0-9]+)D", "P\\1D") %>%
      str_replace_all("([0-9]+)H", "T\\1H") %>%
      str_replace_all("([0-9]+) MIN", "\\1M") %>%
      # If there is a 'P' but no 'T' we keep as is, else if only time, add 'PT'
      { ifelse(str_detect(., "P") & str_detect(., "T"),
               .,
               ifelse(str_detect(., "P") & !str_detect(., "T"),
                      .,
                      paste0("PT", .))) }
  ) %>%
  # Convert ISO-8601 duration to hours using admiral::convert_dur_to_unit
  rowwise() %>%
  mutate(
    result = admiral::convert_dur_to_unit(
      dur = iso_dur,
      target_unit = "H"
    )
  ) %>%
  ungroup() %>%
  select(xxtpt, result)

# Overwrite with NA where na_idx indicates missing
if (nrow(na_idx) == nrow(xxtpt_conv)) {
  xxtpt_conv$result[na_idx$na_idx] <- NA_real_
}

# Write output
readr::write_csv(xxtpt_conv, "outputs/result.csv", na = "")
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
No LLM output artifact was produced.
