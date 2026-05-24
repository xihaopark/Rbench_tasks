# Case 195: pharmaverse/tidytlg/add_indent

## Metadata
- Task ID: `pharmaverse/tidytlg/add_indent`
- Package: `tidytlg`
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
Write an R script to perform add indent using the tidytlg clinical task contract.

Input: df.tsv
Output: result.csv


Required columns for result.csv: anbr, row_type, label, nested_level, group_level, indent
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `df.tsv`


```text
anbr	row_type	label	nested_level	group_level
1	HEADER	Demographics	0	0
1	N	Total	0	0
1	VALUE	Age	0	0
2	TABLE_BY_HEADER	Treatment	0	1
2	VALUE	Drug A	0	1
```

## Code
### Ground Truth Code

```r
df <- read.delim(file.path("inputs", "df.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
df$nested_level <- as.integer(df$nested_level)
df$group_level <- as.integer(df$group_level)
table_offset <- as.integer(any(df$row_type == "TABLE_BY_HEADER"))
df$indent <- ifelse(
  grepl("BY_HEADER", df$row_type) | df$row_type == "HEADER",
  df$group_level + table_offset,
  ifelse(
    df$row_type == "N",
    1L + df$group_level + table_offset,
    ifelse(
      df$row_type == "VALUE",
      2L + df$group_level + table_offset,
      ifelse(df$row_type == "NESTED", df$nested_level + df$group_level + table_offset, 0L)
    )
  )
)
dir.create("outputs", showWarnings = FALSE)
write.csv(df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(tidyr)
  library(tidytlg)
  library(stringr)
})

# ensure outputs directory exists
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# read input
df <- read_tsv(
  file = file.path("inputs", "df.tsv"),
  col_types = cols(
    anbr         = col_double(),
    row_type     = col_character(),
    label        = col_character(),
    nested_level = col_double(),
    group_level  = col_double()
  )
)

# perform add_indent using tidytlg clinical task contract
# infer indenting columns and labels based on provided structure
df_indented <- df %>%
  # ensure correct ordering if not already
  arrange(anbr, group_level, nested_level, row_type) %>%
  add_indent(
    col_label = "label",
    nested_indent = TRUE,
    nested_level = "nested_level",
    group_level = "group_level"
  )

# keep only required columns in required order
result <- df_indented %>%
  select(anbr, row_type, label, nested_level, group_level, indent)

# write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"anbr","row_type","label","nested_level","group_level","indent"
1,"HEADER","Demographics",0,0,1
1,"N","Total",0,0,2
1,"VALUE","Age",0,0,3
2,"TABLE_BY_HEADER","Treatment",0,1,2
2,"VALUE","Drug A",0,1,4
```

### LLM Output
No LLM output artifact was produced.
