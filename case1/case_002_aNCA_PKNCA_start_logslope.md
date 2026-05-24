# Case 002: pharmaverse/aNCA/PKNCA_impute_method_start_logslope

## Metadata
- Task ID: `pharmaverse/aNCA/PKNCA_impute_method_start_logslope`
- Package: `aNCA`
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
Write an R script to perform PKNCA impute method start logslope using the aNCA clinical task contract.

Input: conc.tsv, end.tsv, options.tsv, start.tsv, time.tsv
Output: result.csv


Required columns for result.csv: conc, time
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `conc.tsv`


```text
conc
0.1
1.5
10.0
100.0
1000.0
```

#### `end.tsv`


```text
end
4.5
```

#### `options.tsv`


```text
options
method=start_logslope
allow_blq=TRUE
```

#### `start.tsv`


```text
start
1.5
```

#### `time.tsv`


```text
time
1.0
2.0
3.0
4.0
5.0
```

## Code
### Ground Truth Code

```r
read_tsv <- function(name) {
  path <- file.path("inputs", name)
  if (!file.exists(path)) {
    stop(name, " is required")
  }
  read.delim(path, check.names = FALSE, stringsAsFactors = FALSE)
}

first_column <- function(df) {
  if (ncol(df) == 0) {
    return(character())
  }
  df[[1]]
}

scalar_numeric <- function(df) {
  values <- suppressWarnings(as.numeric(first_column(df)))
  values <- values[!is.na(values)]
  if (length(values) == 0) NA_real_ else values[[1]]
}

write_result <- function(result) {
  dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
  write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "NA")
}

conc <- suppressWarnings(as.numeric(first_column(read_tsv("conc.tsv"))))
time <- suppressWarnings(as.numeric(first_column(read_tsv("time.tsv"))))
keep <- !is.na(conc) & !is.na(time)
result <- data.frame(conc = conc[keep], time = time[keep])
write_result(result)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(PKNCA)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
conc_df   <- read.table("inputs/conc.tsv",   header = TRUE, sep = "\t", stringsAsFactors = FALSE)
time_df   <- read.table("inputs/time.tsv",   header = TRUE, sep = "\t", stringsAsFactors = FALSE)
start_df  <- read.table("inputs/start.tsv",  header = TRUE, sep = "\t", stringsAsFactors = FALSE)
end_df    <- read.table("inputs/end.tsv",    header = TRUE, sep = "\t", stringsAsFactors = FALSE)
opt_df    <- read.table("inputs/options.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)

# Extract vectors
conc_vec  <- conc_df$conc
time_vec  <- time_df$time
start_val <- start_df$start[1]
end_val   <- end_df$end[1]

# Parse options
method_row <- grepl("^method=", opt_df$options)
allow_row  <- grepl("^allow_blq=", opt_df$options)

method     <- sub("^method=", "", opt_df$options[method_row])
allow_blq  <- as.logical(sub("^allow_blq=", "", opt_df$options[allow_row]))

# Prepare concentration-time data.frame
dat <- data.frame(time = time_vec, conc = conc_vec)

# Create PKNCAdata with impute option
conc_obj <- PKNCAconc(dat, conc ~ time)
intervals <- data.frame(
  start = start_val,
  end   = end_val
)

pknca_data <- PKNCAdata(
  conc = conc_obj,
  intervals = intervals,
  impute = list(
    method = method,
    allow.blq = allow_blq
  )
)

# Run PKNCA simply to trigger/validate imputation settings (not used further)
try(suppressWarnings(PKNCAresults(pknca_data)), silent = TRUE)

# Apply start_logslope imputation manually if needed
# Identify rows within [start, end]
in_range <- dat$time >= start_val & dat$time <= end_val
subdat   <- dat[in_range, , drop = FALSE]

# If first time in subdat is after start, impute concentration at start using log-slope
if (nrow(subdat) >= 2 && subdat$time[1] > start_val) {
  t1 <- subdat$time[1]
  t2 <- subdat$time[2]
  c1 <- subdat$conc[1]
  c2 <- subdat$conc[2]

  # Only perform log-slope if both concentrations are > 0
  if (c1 > 0 && c2 > 0 && t2 != t1) {
    k   <- (log(c2) - log(c1)) / (t2 - t1)
    c0  <- exp(log(c1) + k * (start_val - t1))
    # Prepend imputed row
    imputed_row <- data.frame(time = start_val, conc = c0)
    dat <- rbind(imputed_row, dat)
    dat <- dat[order(dat$time), ]
  }
}

# Write output
out_df <- dat[, c("conc", "time")]
write.csv(out_df, file = "outputs/result.csv", row.names = FALSE, quote = TRUE)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"conc","time"
0.1,1
1.5,2
10,3
100,4
1000,5
```

### LLM Output
No LLM output artifact was produced.
