# Case 012: pharmaverse/aNCA/convert_volume_units

## Metadata
- Task ID: `pharmaverse/aNCA/convert_volume_units`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric mismatch in column: volume.
```

## Prompt
```text
Write an R script to perform convert volume units using the aNCA clinical task contract.

Input: avalu.tsv, df.tsv, volume.tsv, volumeu.tsv
Output: result.csv


Required columns for result.csv: id, avalu, volume, volumeu
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `avalu.tsv`


```text
avalu
avalu
```

#### `df.tsv`


```text
id	avalu	volume	volumeu
1	10.5	1.0	L
2	20.3	500.0	mL
3	30.7	0.75	L
```

#### `volume.tsv`


```text
volume
volume
```

#### `volumeu.tsv`


```text
volumeu
volumeu
```

## Code
### Ground Truth Code

```r
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
read_tsv <- function(name) read.delim(file.path("inputs", paste0(name, ".tsv")), check.names = FALSE, stringsAsFactors = FALSE)
df <- read_tsv("df")
write.csv(df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required package
suppressPackageStartupMessages({
  if (!requireNamespace("readr", quietly = TRUE)) {
    stop("Package 'readr' is required.")
  }
  if (!requireNamespace("dplyr", quietly = TRUE)) {
    stop("Package 'dplyr' is required.")
  }
})

library(readr)
library(dplyr)

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input files
df      <- read_tsv("inputs/df.tsv", show_col_types = FALSE)
avalu   <- read_tsv("inputs/avalu.tsv", show_col_types = FALSE)
volume  <- read_tsv("inputs/volume.tsv", show_col_types = FALSE)
volumeu <- read_tsv("inputs/volumeu.tsv", show_col_types = FALSE)

# aNCA clinical task contract: normalize volume units to liters (L)
# Assume 'volume' column is numeric and 'volumeu' has units like 'L', 'mL'
convert_to_L <- function(vol, unit) {
  factor <- dplyr::case_when(
    unit == "L"  ~ 1,
    unit == "mL" ~ 1 / 1000,
    TRUE         ~ NA_real_
  )
  vol * factor
}

df_converted <- df %>%
  mutate(
    volume = convert_to_L(volume, volumeu),
    volumeu = "L"
  ) %>%
  select(id, avalu, volume, volumeu)

# Write output
write_csv(df_converted, "outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","avalu","volume","volumeu"
1,10.5,1,"L"
2,20.3,500,"mL"
3,30.7,0.75,"L"
```

### LLM Output
#### `result.csv`


```csv
id,avalu,volume,volumeu
1,10.5,1,L
2,20.3,0.5,L
3,30.7,0.75,L
```
