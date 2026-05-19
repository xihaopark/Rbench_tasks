# Case 186: pharmaverse/metatools/create_subgrps

## Metadata
- Task ID: `pharmaverse/metatools/create_subgrps`
- Package: `metatools`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform create subgrps using the metatools clinical task contract.

Input: grp_defs.tsv, grp_labs.tsv, ref_vec.tsv
Output: result.csv


Required columns for result.csv: ref_vec, subgroup
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `grp_defs.tsv`


```text
grp_defs
<= 2
3-10
> 10
```

#### `grp_labs.tsv`


```text
grp_labs
Low
Mid
High
```

#### `ref_vec.tsv`


```text
ref_vec
1
3
5
12
20
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(metatools))

ref_vec_df <- read.delim(
  file.path("inputs", "ref_vec.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)
grp_defs_df <- read.delim(
  file.path("inputs", "grp_defs.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)
grp_labs_df <- read.delim(
  file.path("inputs", "grp_labs.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)

ref_vec <- as.numeric(ref_vec_df$ref_vec)
subgroup <- metatools::create_subgrps(
  ref_vec = ref_vec,
  grp_defs = grp_defs_df$grp_defs,
  grp_labs = grp_labs_df$grp_labs
)

result <- data.frame(
  ref_vec = ref_vec,
  subgroup = subgroup,
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(stringr)
  library(purrr)
})

# Directories
in_dir  <- "inputs"
out_dir <- "outputs"

if (!dir.exists(out_dir)) dir.create(out_dir, recursive = TRUE)

# Read inputs
grp_defs <- read_tsv(file.path(in_dir, "grp_defs.tsv"), show_col_types = FALSE) %>%
  pull(grp_defs)

grp_labs <- read_tsv(file.path(in_dir, "grp_labs.tsv"), show_col_types = FALSE) %>%
  pull(grp_labs)

ref_vec  <- read_tsv(file.path(in_dir, "ref_vec.tsv"), show_col_types = FALSE)

# Build cut points and operators from grp_defs
parse_def <- function(def) {
  def <- str_trim(def)
  if (str_detect(def, "^<=")) {
    value <- as.numeric(str_remove(def, "^<=") %>% str_trim())
    list(type = "upper_le", lower = -Inf, upper = value, incl_lower = FALSE, incl_upper = TRUE)
  } else if (str_detect(def, "^>=")) {
    value <- as.numeric(str_remove(def, "^>=") %>% str_trim())
    list(type = "lower_ge", lower = value, upper = Inf, incl_lower = TRUE, incl_upper = FALSE)
  } else if (str_detect(def, "^<")) {
    value <- as.numeric(str_remove(def, "^<") %>% str_trim())
    list(type = "upper_lt", lower = -Inf, upper = value, incl_lower = FALSE, incl_upper = FALSE)
  } else if (str_detect(def, "^>")) {
    value <- as.numeric(str_remove(def, "^>") %>% str_trim())
    list(type = "lower_gt", lower = value, upper = Inf, incl_lower = FALSE, incl_upper = FALSE)
  } else if (str_detect(def, "^-?\\d+\\s*-\\s*-?\\d+$")) {
    nums <- str_split(def, "-", simplify = TRUE) %>% str_trim()
    lower <- as.numeric(nums[1])
    upper <- as.numeric(nums[2])
    list(type = "range", lower = lower, upper = upper, incl_lower = TRUE, incl_upper = TRUE)
  } else {
    stop(paste("Unrecognized group definition:", def))
  }
}

defs_parsed <- map(grp_defs, parse_def)

assign_group <- function(x, defs, labs) {
  for (i in seq_along(defs)) {
    d <- defs[[i]]
    in_lower <- if (d$incl_lower) x >= d$lower else x > d$lower
    in_upper <- if (d$incl_upper) x <= d$upper else x < d$upper
    if (isTRUE(in_lower && in_upper)) {
      return(labs[[i]])
    }
  }
  NA_character_
}

result <- ref_vec %>%
  mutate(
    subgroup = map_chr(.data$ref_vec, assign_group, defs = defs_parsed, labs = grp_labs)
  ) %>%
  select(ref_vec, subgroup)

write_csv(result, file.path(out_dir, "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"ref_vec","subgroup"
1,"Low"
3,"Mid"
5,"Mid"
12,"High"
20,"High"
```

### LLM Output
#### `result.csv`


```csv
ref_vec,subgroup
1,Low
3,Mid
5,Mid
12,High
20,High
```
