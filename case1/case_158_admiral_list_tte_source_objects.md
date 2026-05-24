# Case 158: pharmaverse/admiral/list_tte_source_objects

## Metadata
- Task ID: `pharmaverse/admiral/list_tte_source_objects`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(12, 6) vs llm=(2, 6).
```

## Prompt
```text
Write an R script to perform list tte source objects using the admiral clinical task contract.

Input: package.tsv
Output: result.csv


Required columns for result.csv: object, dataset_name, filter, date, censor, set_values_to
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `package.tsv`


```text
package
admiral
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

package <- as.character(read.delim(file.path("inputs", "package.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1])
result <- admiral::list_tte_source_objects(package = package)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required packages
suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(stringr)
  library(purrr)
  library(fs)
})

# Create outputs directory if it does not exist
if (!dir_exists("outputs")) dir_create("outputs")

# Read package.tsv
pkg_df <- read_tsv("inputs/package.tsv", show_col_types = FALSE)
pkg <- pkg_df$package[1]

# Load package
suppressPackageStartupMessages(library(pkg, character.only = TRUE))

# Helper to safely get object from namespace
get_obj <- function(x, pkg) {
  getFromNamespace(x, pkg)
}

# List all objects in package namespace
objs <- getNamespaceExports(pkg)

# Identify TTE source objects: assume objects containing "tte" or "TTE"
tte_objs <- objs[str_detect(objs, regex("tte", ignore_case = TRUE))]

# Fallback: if no obvious TTE objects, search all objects for class with "tte"
if (length(tte_objs) == 0) {
  all_objs <- objs
  tte_flag <- map_lgl(all_objs, function(o) {
    obj <- tryCatch(get_obj(o, pkg), error = function(e) NULL)
    if (is.null(obj)) return(FALSE)
    cl <- class(obj)
    any(str_detect(cl, regex("tte", ignore_case = TRUE)))
  })
  tte_objs <- all_objs[tte_flag]
}

# Extract fields for each TTE object
res <- map_dfr(tte_objs, function(o) {
  obj <- tryCatch(get_obj(o, pkg), error = function(e) NULL)
  if (is.null(obj)) {
    tibble(
      object = o,
      dataset_name = NA_character_,
      filter = NA_character_,
      date = NA_character_,
      censor = NA_character_,
      set_values_to = NA_character_
    )
  } else {
    # Try to extract named elements commonly used in TTE source objects
    dataset_name <- tryCatch(obj$dataset_name, error = function(e) NA_character_)
    filter      <- tryCatch(obj$filter,       error = function(e) NA_character_)
    date        <- tryCatch(obj$date,         error = function(e) NA_character_)
    censor      <- tryCatch(obj$censor,       error = function(e) NA_character_)
    set_values_to <- tryCatch(obj$set_values_to, error = function(e) NA_character_)

    # Coerce non-character elements to deparsed text
    to_chr <- function(x) {
      if (is.null(x) || (length(x) == 1 && is.na(x))) return(NA_character_)
      if (is.character(x)) return(paste(x, collapse = " "))
      paste(deparse(x), collapse = " ")
    }

    tibble(
      object = o,
      dataset_name = to_chr(dataset_name),
      filter = to_chr(filter),
      date = to_chr(date),
      censor = to_chr(censor),
      set_values_to = to_chr(set_values_to)
    )
  }
})

# Write result
write_csv(res, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"object","dataset_name","filter","date","censor","set_values_to"
"ae_gr3_event","adae","TRTEMFL == ""Y"" & ATOXGR == ""3""","ASTDT",0,"EVNTDESC: ""GRADE 3 ADVERSE EVENT""<br>SRCDOM: ""ADAE""<br>SRCVAR: ""ASTDT""<br>SRCSEQ: AESEQ"
"ae_wd_event","adae","TRTEMFL == ""Y"" & AEACN == ""DRUG WITHDRAWN""","ASTDT",0,"EVNTDESC: ""ADVERSE EVENT LEADING TO DRUG WITHDRAWAL""<br>SRCDOM: ""ADAE""<br>SRCVAR: ""ASTDT""<br>SRCSEQ: AESEQ"
"ae_gr35_event","adae","TRTEMFL == ""Y"" & ATOXGR %in% c(""3"", ""4"", ""5"")","ASTDT",0,"EVNTDESC: ""GRADE 3-5 ADVERSE EVENT""<br>SRCDOM: ""ADAE""<br>SRCVAR: ""ASTDT""<br>SRCSEQ: AESEQ"
"lastalive_censor","adsl","NULL","LSTALVDT",1,"EVNTDESC: ""ALIVE""<br>SRCDOM: ""ADSL""<br>SRCVAR: ""LSTALVDT"""
"ae_gr1_event","adae","TRTEMFL == ""Y"" & ATOXGR == ""1""","ASTDT",0,"EVNTDESC: ""GRADE 1 ADVERSE EVENT""<br>SRCDOM: ""ADAE""<br>SRCVAR: ""ASTDT""<br>SRCSEQ: AESEQ"
"ae_ser_event","adae","TRTEMFL == ""Y"" & AESER == ""Y""","ASTDT",0,"EVNTDESC: ""SERIOUS ADVERSE EVENT""<br>SRCDOM: ""ADAE""<br>SRCVAR: ""ASTDT""<br>SRCSEQ: AESEQ"
"ae_gr2_event","adae","TRTEMFL == ""Y"" & ATOXGR == ""2""","ASTDT",0,"EVNTDESC: ""GRADE 2 ADVERSE EVENT""<br>SRCDOM: ""ADAE""<br>SRCVAR: ""ASTDT""<br>SRCSEQ: AESEQ"
"ae_event","adae","TRTEMFL == ""Y""","ASTDT",0,"EVNTDESC: ""ADVERSE EVENT""<br>SRCDOM: ""ADAE""<br>SRCVAR: ""ASTDT""<br>SRCSEQ: AESEQ"
"ae_gr4_event","adae","TRTEMFL == ""Y"" & ATOXGR == ""4""","ASTDT",0,"EVNTDESC: ""GRADE 4 ADVERSE EVENT""<br>SRCDOM: ""ADAE""<br>SRCVAR: ""ASTDT""<br>SRCSEQ: AESEQ"
"ae_gr5_event","adae","TRTEMFL == ""Y"" & ATOXGR == ""5""","ASTDT",0,"EVNTDESC: ""GRADE 5 ADVERSE EVENT""<br>SRCDOM: ""ADAE""<br>SRCVAR: ""ASTDT""<br>SRCSEQ: AESEQ"
"ae_sev_event","adae","TRTEMFL == ""Y"" & AESEV == ""SEVERE""","ASTDT",0,"EVNTDESC: ""SEVERE ADVERSE EVENT""<br>SRCDOM: ""ADAE""<br>SRCVAR: ""ASTDT""<br>SRCSEQ: AESEQ"
"death_event","adsl","DTHFL == ""Y""","DTHDT",0,"EVNTDESC: ""DEATH""<br>SRCDOM: ""ADSL""<br>SRCVAR: ""DTHDT"""
```

### LLM Output
#### `result.csv`


```csv
object,dataset_name,filter,date,censor,set_values_to
derive_param_tte,,,,,
list_tte_source_objects,,,,,
```
