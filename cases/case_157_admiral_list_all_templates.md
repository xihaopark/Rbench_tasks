# Case 157: pharmaverse/admiral/list_all_templates

## Metadata
- Task ID: `pharmaverse/admiral/list_all_templates`
- Package: `admiral`
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
Write an R script to perform list all templates using the admiral clinical task contract.

Input: package.tsv
Output: result.csv


Required columns for result.csv: template
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `package.tsv`


```text
package
"admiral"
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

package_df <- read.delim(file.path("inputs", "package.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
package <- as.character(package_df[[ncol(package_df)]][1])
if (package == "test_value" || !nzchar(package)) package <- "admiral"

result <- admiral::list_all_templates(package = package)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(template = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(jsonlite)
  library(tools)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read package.tsv
pkg_df <- read.delim(
  file = file.path("inputs", "package.tsv"),
  header = TRUE,
  sep = "\t",
  quote = "\"",
  stringsAsFactors = FALSE
)

# Get package name
pkg <- pkg_df$package[1]

# Ensure package is installed and load it
if (!requireNamespace(pkg, quietly = TRUE)) {
  install.packages(pkg, repos = "https://cloud.r-project.org")
}
suppressPackageStartupMessages(library(pkg, character.only = TRUE))

# Use admiral clinical task contract to list templates
# Assuming admiral exports this via admiraldev-style contract helpers
# and that the function is named "clinical_task_contract"
ctc_fun <- get("clinical_task_contract", asNamespace(pkg))

# Get contract information
ctc <- ctc_fun()

# Expecting templates listed in a component named "template" or "templates"
if (!is.null(ctc$templates)) {
  templates <- ctc$templates
} else if (!is.null(ctc$template)) {
  templates <- ctc$template
} else if (!is.null(ctc[["templates"]])) {
  templates <- ctc[["templates"]]
} else if (!is.null(ctc[["template"]])) {
  templates <- ctc[["template"]]
} else {
  # Fallback: try to infer from any character vector element named like templates
  char_elems <- Filter(is.character, ctc)
  if (length(char_elems) == 0) {
    templates <- character(0)
  } else if (!is.null(names(char_elems))) {
    idx <- grep("template", names(char_elems), ignore.case = TRUE)
    if (length(idx) > 0) {
      templates <- char_elems[[idx[1]]]
    } else {
      templates <- unlist(char_elems, use.names = FALSE)
    }
  } else {
    templates <- unlist(char_elems, use.names = FALSE)
  }
}

templates <- unique(as.character(templates))
templates <- templates[!is.na(templates) & nzchar(templates)]

result <- data.frame(
  template = templates,
  stringsAsFactors = FALSE
)

write.csv(
  result,
  file = file.path("outputs", "result.csv"),
  row.names = FALSE,
  quote = TRUE
)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"template"
"ADAB"
"ADAE"
"ADCM"
"ADEG"
"ADEX"
"ADLB"
"ADLBHY"
"ADMH"
"ADPC"
"ADPP"
"ADPPK"
"ADSL"
"ADVS"
```

### LLM Output
No LLM output artifact was produced.
