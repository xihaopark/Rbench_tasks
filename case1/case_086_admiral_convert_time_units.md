# Case 086: pharmaverse/admiral/convert_time_units

## Metadata
- Task ID: `pharmaverse/admiral/convert_time_units`
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
Write an R script to perform convert time units using the admiral clinical task contract.

Input: na_idx.tsv, result.tsv, xxtpt.tsv
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
4 POST DOSE
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

library(readr)
library(dplyr)
library(stringr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
na_idx   <- read_tsv("inputs/na_idx.tsv", show_col_types = FALSE)$na_idx
result   <- read_tsv("inputs/result.tsv", show_col_types = FALSE)
xxtpt_df <- read_tsv("inputs/xxtpt.tsv", show_col_types = FALSE)

# Helper: convert xxtpt string to numeric time in hours
convert_time_to_hours <- function(x) {
  x_clean <- str_trim(toupper(x))

  # Initialize
  out <- rep(NA_real_, length(x_clean))

  # Pattern 1: "2H 30MIN", "5H", "30MIN" etc.
  hrs  <- ifelse(str_detect(x_clean, "\\d+\\s*H"),
                 as.numeric(str_extract(x_clean, "(?<=^|\\s)\\d+(?=\\s*H)")),
                 0)
  mins <- ifelse(str_detect(x_clean, "\\d+\\s*MIN"),
                 as.numeric(str_extract(x_clean, "(?<=^|\\s)\\d+(?=\\s*MIN)")),
                 0)
  any_hm <- (hrs + mins) > 0
  out[any_hm] <- hrs[any_hm] + mins[any_hm] / 60

  # Pattern 2: "3 DAYS", "DAY 2"
  days1_idx <- str_detect(x_clean, "^\\d+\\s*DAYS?$")
  if (any(days1_idx & is.na(out))) {
    d <- as.numeric(str_extract(x_clean[days1_idx], "^\\d+"))
    out[days1_idx & is.na(out)] <- d * 24
  }

  days2_idx <- str_detect(x_clean, "^DAY\\s*\\d+")
  if (any(days2_idx & is.na(out))) {
    d <- as.numeric(str_extract(x_clean[days2_idx], "\\d+"))
    out[days2_idx & is.na(out)] <- d * 24
  }

  # Pattern 3: ASSUMPTION for "4 POST DOSE" → 4 HOURS POST DOSE
  postdose_idx <- str_detect(x_clean, "^\\d+\\s+POST\\s+DOSE")
  if (any(postdose_idx & is.na(out))) {
    h <- as.numeric(str_extract(x_clean[postdose_idx], "^\\d+"))
    out[postdose_idx & is.na(out)] <- h
  }

  out
}

# Apply conversion where result is NA and not excluded by na_idx
result_new <- result$result

convert_idx <- is.na(result_new) & !na_idx
if (any(convert_idx)) {
  converted <- convert_time_to_hours(xxtpt_df$xxtpt[convert_idx])
  result_new[convert_idx] <- converted
}

# Build output data frame with required columns
out_df <- tibble(
  xxtpt  = xxtpt_df$xxtpt,
  result = result_new
)

# Write CSV
write_csv(out_df, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"xxtpt","result"
"2H 30MIN",2.5
"3 DAYS",72
"DAY 2",48
"4 POST DOSE",96
```

### LLM Output
#### `result.csv`


```csv
xxtpt,result
2H 30MIN,2.5
3 DAYS,72
DAY 2,48
4 POST DOSE,4
```
