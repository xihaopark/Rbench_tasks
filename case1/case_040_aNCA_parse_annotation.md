# Case 040: pharmaverse/aNCA/parse_annotation

## Metadata
- Task ID: `pharmaverse/aNCA/parse_annotation`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: result.
```

## Prompt
```text
Write an R script to perform parse annotation using the aNCA clinical task contract.

Input: data.tsv, text.tsv
Output: result.csv


Required columns for result.csv: text, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `data.tsv`


```text
PARAMCD	PARAM	UNIT
AUC	Area under the curve	h*ng/mL
CMAX	Maximum concentration	ng/mL
```

#### `text.tsv`


```text
text
Dose-normalized AUC (h*ng/mL)
```

## Code
### Ground Truth Code

```r
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
read_tsv <- function(name) read.delim(file.path("inputs", paste0(name, ".tsv")), check.names = FALSE, stringsAsFactors = FALSE)
text <- as.character(read_tsv("text")[[1]])
write.csv(data.frame(text = text, result = text, stringsAsFactors = FALSE), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(data.table)
  library(jsonlite)
})

# Paths
input_dir  <- "inputs"
output_dir <- "outputs"
data_path  <- file.path(input_dir, "data.tsv")
text_path  <- file.path(input_dir, "text.tsv")
out_path   <- file.path(output_dir, "result.csv")

# Create outputs dir if needed
if (!dir.exists(output_dir)) dir.create(output_dir, recursive = TRUE)

# Read inputs
data_dt <- fread(data_path, sep = "\t", header = TRUE, quote = "")
text_dt <- fread(text_path, sep = "\t", header = TRUE, quote = "")

# Simple parsing: match PARAM and UNIT occurrences in text
parse_one <- function(txt, ref_dt) {
  matches <- list()
  for (i in seq_len(nrow(ref_dt))) {
    row <- ref_dt[i]
    found_param <- grepl(row$PARAM, txt, fixed = TRUE)
    found_unit  <- grepl(row$UNIT,  txt, fixed = TRUE)
    if (found_param || found_unit) {
      matches[[length(matches) + 1]] <- list(
        PARAMCD = row$PARAMCD,
        PARAM   = row$PARAM,
        UNIT    = row$UNIT,
        found_param = found_param,
        found_unit  = found_unit
      )
    }
  }
  if (length(matches) == 0) {
    return(toJSON(list(), auto_unbox = TRUE))
  }
  toJSON(matches, auto_unbox = TRUE)
}

result_dt <- copy(text_dt)
result_dt[, result := vapply(text, parse_one, character(1), ref_dt = data_dt)]

fwrite(result_dt[, .(text, result)], out_path)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"text","result"
"Dose-normalized AUC (h*ng/mL)","Dose-normalized AUC (h*ng/mL)"
```

### LLM Output
#### `result.csv`


```csv
text,result
Dose-normalized AUC (h*ng/mL),"[{""PARAMCD"":""AUC"",""PARAM"":""Area under the curve"",""UNIT"":""h*ng/mL"",""found_param"":false,""found_unit"":true},{""PARAMCD"":""CMAX"",""PARAM"":""Maximum concentration"",""UNIT"":""ng/mL"",""found_param"":false,""found_unit"":true}]"
```
