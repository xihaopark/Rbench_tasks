# Case 194: pharmaverse/sdtmchecks/pass

## Metadata
- Task ID: `pharmaverse/sdtmchecks/pass`
- Package: `sdtmchecks`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: result.
```

## Prompt
```text
Write an R script to perform pass using the sdtmchecks clinical task contract.

Input: none
Output: result.csv


Required columns for result.csv: result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
No input files.

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(sdtmchecks))

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

result <- sdtmchecks::pass()
result_df <- data.frame(result = unname(as.logical(result)))

write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required package ---------------------------------------------------
suppressPackageStartupMessages({
  if (!requireNamespace("sdtmchecks", quietly = TRUE)) {
    stop("Package 'sdtmchecks' is required but not installed.")
  }
})

library(sdtmchecks)

# Create outputs directory ------------------------------------------------
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE, showWarnings = FALSE)
}

# Run sdtmchecks contract / task ------------------------------------------
# Assumes that the sdtmchecks clinical task contract is implemented as a
# function that can be called to validate inputs/ and return pass/fail.
# Adapt the function name/usage here if your contract is different.

# Example: using a generic contract runner
result_flag <- "FAIL"

try({
  # Example contract runner call; replace with the actual function
  # for your specific clinical task contract if different.
  #
  # The assumption is that this function:
  #   - reads SDTM data from 'inputs/'
  #   - performs checks defined by the clinical task contract
  #   - returns TRUE/FALSE or "PASS"/"FAIL"
  #
  # Here we try a few reasonable conventions:

  if (exists("run_clinical_task_contract", where = asNamespace("sdtmchecks"))) {
    res <- sdtmchecks::run_clinical_task_contract(input_dir = "inputs")
  } else if (exists("run_contract", where = asNamespace("sdtmchecks"))) {
    res <- sdtmchecks::run_contract(input_dir = "inputs")
  } else if (exists("run_checks", where = asNamespace("sdtmchecks"))) {
    res <- sdtmchecks::run_checks(path = "inputs")
  } else {
    stop("No suitable clinical task contract runner found in 'sdtmchecks'.")
  }

  if (is.logical(res)) {
    result_flag <- ifelse(isTRUE(res), "PASS", "FAIL")
  } else if (is.character(res)) {
    # Normalize string to PASS/FAIL
    res_up <- toupper(trimws(res[1]))
    if (res_up %in% c("PASS", "OK", "SUCCESS")) {
      result_flag <- "PASS"
    } else {
      result_flag <- "FAIL"
    }
  } else {
    # If result object has a status field
    if (is.list(res) && !is.null(res$status)) {
      status_up <- toupper(trimws(as.character(res$status)))
      if (status_up %in% c("PASS", "OK", "SUCCESS")) {
        result_flag <- "PASS"
      } else {
        result_flag <- "FAIL"
      }
    } else {
      result_flag <- "FAIL"
    }
  }
}, silent = TRUE)

# Write result.csv --------------------------------------------------------
result_df <- data.frame(result = result_flag, stringsAsFactors = FALSE)

write.csv(
  result_df,
  file = file.path("outputs", "result.csv"),
  row.names = FALSE,
  quote = TRUE
)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"result"
TRUE
```

### LLM Output
#### `result.csv`


```csv
"result"
"FAIL"
```
