# Case 088: pharmaverse/admiral/convert_xxtpt_to_hours

## Metadata
- Task ID: `pharmaverse/admiral/convert_xxtpt_to_hours`
- Package: `admiral`
- Model: `openai/gpt-5.5`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
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
library(readr)
library(dplyr)
library(stringr)
library(admiral)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

range_method <- read_tsv("inputs/range_method.tsv", show_col_types = FALSE) %>%
  admiral::convert_blanks_to_na()

treatment_duration <- read_tsv("inputs/treatment_duration.tsv", show_col_types = FALSE) %>%
  admiral::convert_blanks_to_na()

xxtpt <- read_tsv("inputs/xxtpt.tsv", show_col_types = FALSE) %>%
  admiral::convert_blanks_to_na()

parse_xxtpt_to_hours <- function(x) {
  txt <- str_squish(str_to_upper(as.character(x)))

  case_when(
    is.na(txt) ~ NA_real_,

    str_detect(txt, "^(PRE[- ]?DOSE|PREDOSE|SCREENING|SCREEN|BASELINE|BL|PRIOR)") ~ 0,

    str_detect(txt, "^\\d+(\\.\\d+)?$") ~ as.numeric(txt),

    str_detect(txt, "^\\d{1,2}:\\d{1,2}$") ~ {
      parts <- str_split_fixed(txt, ":", 2)
      as.numeric(parts[, 1]) + as.numeric(parts[, 2]) / 60
    },

    str_detect(txt, "^DAY\\s*\\d+(\\.\\d+)?$") ~ {
      as.numeric(str_extract(txt, "\\d+(\\.\\d+)?")) * 24
    },

    str_detect(txt, "^\\d+(\\.\\d+)?\\s*D(AY|AYS)?$") ~ {
      as.numeric(str_extract(txt, "\\d+(\\.\\d+)?")) * 24
    },

    str_detect(txt, "^\\d+(\\.\\d+)?\\s*H(OUR|OURS|R|RS)?(\\s+\\d+(\\.\\d+)?\\s*M(IN|INS|INUTE|INUTES)?)?$") ~ {
      hrs <- as.numeric(str_extract(txt, "\\d+(\\.\\d+)?(?=\\s*H)"))
      mins <- str_extract(txt, "\\d+(\\.\\d+)?(?=\\s*M)")
      hrs + if_else(is.na(mins), 0, as.numeric(mins) / 60)
    },

    str_detect(txt, "^\\d+(\\.\\d+)?\\s*M(IN|INS|INUTE|INUTES)?$") ~ {
      as.numeric(str_extract(txt, "\\d+(\\.\\d+)?")) / 60
    },

    TRUE ~ NA_real_
  )
}

result <- xxtpt %>%
  mutate(
    treatment_duration = treatment_duration$treatment_duration[1],
    range_method = range_method$range_method[1],
    result = parse_xxtpt_to_hours(xxtpt)
  ) %>%
  select(xxtpt, treatment_duration, range_method, result)

write_csv(result, "outputs/result.csv")
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
2H 30MIN,24,midpoint,2.5
3 DAYS,24,midpoint,72
DAY 2,24,midpoint,48
4D,24,midpoint,96
```
