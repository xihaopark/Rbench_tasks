# Case2 28: pharmaverse/aNCA/translate_terms

## Metadata
- Task ID: `pharmaverse/aNCA/translate_terms`
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
Write an R script for an R package task: perform translate terms using the aNCA package. The function(s) you use to solve the task must come from the aNCA package.

Input: input_terms.tsv, mapping_col.tsv, metadata.tsv, target_col.tsv
Output: result.csv

Computation: Map each input term by looking it up from mapping_col to target_col, and write a row-preserving table with input_terms, mapping_col, and result.


Required columns for result.csv: input_terms, mapping_col, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### input_terms.tsv
input_terms
AVAL
PARAMCD

### mapping_col.tsv
mapping_col
Variable

### metadata.tsv
Variable	Label
AVAL	Analysis Value
PARAMCD	Parameter Code
USUBJID	Subject ID

### target_col.tsv
target_col
Label
```

## Input
#### `input_terms.tsv`

```text
input_terms
AVAL
PARAMCD
```

#### `mapping_col.tsv`

```text
mapping_col
Variable
```

#### `metadata.tsv`

```text
Variable	Label
AVAL	Analysis Value
PARAMCD	Parameter Code
USUBJID	Subject ID
```

#### `target_col.tsv`

```text
target_col
Label
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)
suppressPackageStartupMessages(library(purrr))

# 1. 读取输入数据 / Read input data
input_terms_path <- file.path("inputs", "input_terms.tsv")
if (!file.exists(input_terms_path)) {
  stop("input_terms.tsv is required input")
}
input_terms_df <- read.delim(input_terms_path, check.names = FALSE, stringsAsFactors = FALSE)
input_terms <- input_terms_df$input_terms
mapping_col_path <- file.path("inputs", "mapping_col.tsv")
if (!file.exists(mapping_col_path)) {
  stop("mapping_col.tsv is required input")
}
mapping_col_df <- read.delim(mapping_col_path, check.names = FALSE, stringsAsFactors = FALSE)
mapping_col <- mapping_col_df$mapping_col
target_col_path <- file.path("inputs", "target_col.tsv")
if (!file.exists(target_col_path)) {
  stop("target_col.tsv is required input")
}
target_col_df <- read.delim(target_col_path, check.names = FALSE, stringsAsFactors = FALSE)
target_col <- target_col_df$target_col
metadata_path <- file.path("inputs", "metadata.tsv")
if (!file.exists(metadata_path)) {
  stop("metadata.tsv is required input")
}
metadata <- read.delim(metadata_path, check.names = FALSE, stringsAsFactors = FALSE)

# 2. 数据验证 / Data validation
# 检查数据框的基本结构
for (df_name in c("metadata")) {
  df <- get(df_name)
  if (nrow(df) == 0) {
    stop(paste("Data frame", df_name, "is empty"))
  }
    if (ncol(df) == 0) {
      stop(paste("Data frame", df_name, "has no columns"))
    }
}

# 3. 执行函数实现 / Execute function implementation
# 提取标量参数
if (is.data.frame(mapping_col) && ncol(mapping_col) > 0 && nrow(mapping_col) > 0) {
  mapping_col <- mapping_col[[1]][1]
} else if (is.character(mapping_col)) {
  mapping_col <- mapping_col[1]
}
if (is.data.frame(target_col) && ncol(target_col) > 0 && nrow(target_col) > 0) {
  target_col <- target_col[[1]][1]
} else if (is.character(target_col)) {
  target_col <- target_col[1]
}

result <- tryCatch({
  if (!has_aNCA) stop("aNCA unavailable")
  aNCA:::translate_terms(
    input_terms = input_terms,
    mapping_col = mapping_col,
    target_col = target_col,
    metadata = metadata
  )
}, error = function(e) {
  # 如果函数调用失败，返回NULL
  NULL
})


# 4. 创建结果数据框 / Create result dataframe
# 将函数结果与输入数据合并（如果可能）
if (is.vector(result) || is.numeric(result) || is.character(result)) {
  result_df <- data.frame(
    input_terms = input_terms, mapping_col = mapping_col,
    result = result,
    stringsAsFactors = FALSE)
  } else if (is.data.frame(result)) {
  result_df <- result
} else {
  result_df <- data.frame(result = result, stringsAsFactors = FALSE)
}
# 6. 保存输出 / Save outputs
outputs_dir <- "outputs"
dir.create(outputs_dir, showWarnings = FALSE)
unlink(file.path(outputs_dir, c("result.rds", "summary.csv")))
if (exists("result_df") && is.data.frame(result_df)) {
  write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else if (exists("result")) {
  if (is.data.frame(result)) {
    write.csv(result, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else if (is.list(result)) {
    saveRDS(result, file.path(outputs_dir, "result.rds"))
  } else if (is.vector(result) || is.numeric(result) || is.character(result)) {
    result_df <- data.frame(result = result, stringsAsFactors = FALSE)
    write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else {
    saveRDS(result, file.path(outputs_dir, "result.rds"))
  }
} else {
  stop("No result variable found. Function implementation may not have returned a value.")
}
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(aNCA)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
input_terms <- read.delim("inputs/input_terms.tsv", sep = "\t", header = TRUE, stringsAsFactors = FALSE)
mapping_col <- read.delim("inputs/mapping_col.tsv", sep = "\t", header = TRUE, stringsAsFactors = FALSE)
metadata    <- read.delim("inputs/metadata.tsv", sep = "\t", header = TRUE, stringsAsFactors = FALSE)
target_col  <- read.delim("inputs/target_col.tsv", sep = "\t", header = TRUE, stringsAsFactors = FALSE)

# Extract column names
mapping_col_name <- mapping_col$mapping_col[1]
target_col_name  <- target_col$target_col[1]

# Prepare metadata for translation
meta_for_translate <- metadata
# Ensure metadata has the expected columns
if (!all(c(mapping_col_name, target_col_name) %in% colnames(meta_for_translate))) {
  stop("Metadata does not contain required mapping/target columns.")
}

# Perform translation using aNCA::translate_terms
res_vec <- aNCA::translate_terms(
  terms      = input_terms$input_terms,
  from       = mapping_col_name,
  to         = target_col_name,
  dictionary = meta_for_translate
)

# Assemble result table
result_df <- data.frame(
  input_terms = input_terms$input_terms,
  mapping_col = mapping_col_name,
  result      = res_vec,
  stringsAsFactors = FALSE
)

# Write output
write.csv(result_df, file = "outputs/result.csv", row.names = FALSE, quote = TRUE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"input_terms","mapping_col","result"
"AVAL","Variable","Analysis Value"
"PARAMCD","Variable","Parameter Code"
```

### LLM Output
No LLM output artifact was produced.
