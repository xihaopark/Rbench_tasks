# Case 070: pharmaverse/admiral/compute_scale

## Metadata
- Task ID: `pharmaverse/admiral/compute_scale`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform compute scale using the admiral clinical task contract.

Input: flip_direction.tsv, source.tsv, source_range.tsv, target_range.tsv
Output: result.csv


Required columns for result.csv: source, source_min, source_max, target_min, target_max, flip_direction, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `flip_direction.tsv`


```text
flip_direction
FALSE
```

#### `source.tsv`


```text
source
0
25
50
100
```

#### `source_range.tsv`


```text
bound	value
min	0
max	100
```

#### `target_range.tsv`


```text
bound	value
min	0
max	1
```

## Code
### Ground Truth Code

```r
source <- as.numeric(read.delim(file.path("inputs", "source.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$source)
source_range <- read.delim(file.path("inputs", "source_range.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
target_range <- read.delim(file.path("inputs", "target_range.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
flip_direction <- as.logical(read.delim(file.path("inputs", "flip_direction.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$flip_direction[1])
get_bound <- function(tbl, name) as.numeric(tbl$value[tbl$bound == name][1])
source_min <- get_bound(source_range, "min")
source_max <- get_bound(source_range, "max")
target_min <- get_bound(target_range, "min")
target_max <- get_bound(target_range, "max")
if (flip_direction) {
  tmp <- target_min
  target_min <- target_max
  target_max <- tmp
}
result <- target_min + (source - source_min) / (source_max - source_min) * (target_max - target_min)
result_df <- data.frame(source = source, source_min = source_min, source_max = source_max, target_min = target_min, target_max = target_max, flip_direction = flip_direction, result = result, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
flip_direction <- read_tsv(
  file = "inputs/flip_direction.tsv",
  col_types = cols(
    flip_direction = col_logical()
  )
) %>%
  pull(flip_direction) %>%
  .[1]

source_df <- read_tsv(
  file = "inputs/source.tsv",
  col_types = cols(
    source = col_double()
  )
)

source_range <- read_tsv(
  file = "inputs/source_range.tsv",
  col_types = cols(
    bound = col_character(),
    value = col_double()
  )
)

target_range <- read_tsv(
  file = "inputs/target_range.tsv",
  col_types = cols(
    bound = col_character(),
    value = col_double()
  )
)

source_min <- source_range %>% filter(bound == "min") %>% pull(value) %>% .[1]
source_max <- source_range %>% filter(bound == "max") %>% pull(value) %>% .[1]

target_min <- target_range %>% filter(bound == "min") %>% pull(value) %>% .[1]
target_max <- target_range %>% filter(bound == "max") %>% pull(value) %>% .[1]

# Compute scale
result_df <- source_df %>%
  mutate(
    source_min = source_min,
    source_max = source_max,
    target_min = target_min,
    target_max = target_max
  ) %>%
  mutate(
    result = if (!flip_direction) {
      # Standard scaling: map source_min->target_min, source_max->target_max
      target_min + (source - source_min) *
        (target_max - target_min) / (source_max - source_min)
    } else {
      # Flip scaling: map source_min->target_max, source_max->target_min
      target_max - (source - source_min) *
        (target_max - target_min) / (source_max - source_min)
    }
  ) %>%
  mutate(
    flip_direction = flip_direction
  ) %>%
  select(
    source,
    source_min,
    source_max,
    target_min,
    target_max,
    flip_direction,
    result
  )

# Write output
write_csv(result_df, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"source","source_min","source_max","target_min","target_max","flip_direction","result"
0,0,100,0,1,FALSE,0
25,0,100,0,1,FALSE,0.25
50,0,100,0,1,FALSE,0.5
100,0,100,0,1,FALSE,1
```

### LLM Output
#### `result.csv`


```csv
source,source_min,source_max,target_min,target_max,flip_direction,result
0,0,100,0,1,FALSE,0
25,0,100,0,1,FALSE,0.25
50,0,100,0,1,FALSE,0.5
100,0,100,0,1,FALSE,1
```
