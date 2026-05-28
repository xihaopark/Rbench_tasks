# Case 050: pharmaverse/aNCA/translate_terms

## Metadata
- Task ID: `pharmaverse/aNCA/translate_terms`
- Package: `aNCA`
- Model: `claude-code/claude-sonnet-4-6`
- Agent: `Claude Code`
- Status: `PASS`
- Failure stage: `pass`
- Pattern: ``

## Pass/Error
```text
PASS
```

## Prompt
```text
You are running one RBioBench clinical R task in an isolated worktree.

Your goal is to write a complete, reproducible R script at `solution.R`.

Rules:
- `TASK.md` is the authoritative task contract. `task.json` is sanitized metadata only.
- Read input files only from `inputs/` using relative paths.
- Write exactly the required output artifact(s): outputs/result.csv.
- Create `outputs/` if needed.
- You may inspect `task.json`, `TASK.md`, and input files.
- You may consult public R package documentation, including CRAN, r-universe,
  and GitHub repository documentation, to verify normal exported package APIs.
- Do not infer package function names from task metadata. Use a package API only when
  it is a normal exported function you can verify; otherwise implement the required
  transformation directly from the inputs.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Do NOT attempt to install R packages. All required packages are pre-installed in the
  evaluation Docker environment. Public documentation is useful for API lookup, but
  the final solution must run with the packages already installed in that environment.
  If a package is not available in the current shell, write the solution assuming it
  is available and move on.
- You may run `Rscript solution.R` to do a quick syntax check, but do not spend time
  debugging runtime errors caused by missing packages or system libraries.

Task prompt:

Write an R script for an R package task: perform translate terms using the aNCA package. Use functions from the aNCA package when suitable.

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

Reference package function list:
The hidden reference solution's R package function calls are listed below. If the list is empty, the reference solution does not call package functions.
- package_functions: []
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

dir.create('outputs', showWarnings = FALSE, recursive = TRUE)

input_terms <- read.delim("inputs/input_terms.tsv", stringsAsFactors = FALSE)
mapping_col <- read.delim("inputs/mapping_col.tsv", stringsAsFactors = FALSE)
metadata    <- read.delim("inputs/metadata.tsv",    stringsAsFactors = FALSE)
target_col  <- read.delim("inputs/target_col.tsv",  stringsAsFactors = FALSE)

mc <- mapping_col[[1]]
tc <- target_col[[1]]

result <- data.frame(
  input_terms = input_terms[[1]],
  mapping_col = mc,
  result = metadata[[tc]][match(input_terms[[1]], metadata[[mc]])],
  stringsAsFactors = FALSE
)

write.csv(result, "outputs/result.csv", row.names = FALSE)
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
#### `result.csv`

```csv
"input_terms","mapping_col","result"
"AVAL","Variable","Analysis Value"
"PARAMCD","Variable","Parameter Code"
```

#### `case_01/stderr.txt`

```text
[RBioBench Library Aliases] Library aliasing disabled (using stub layer)
[Admiraldev Stub] Created admiraldev namespace with 10 stub functions
[RBioBench Stub Layer] Loaded admiraldev stubs
[aNCA Stub] Created aNCA namespace with 57 stub functions
[RBioBench Stub Layer] Loaded aNCA stubs
[Logrx Stub] Created logrx namespace with 2 stub functions
[RBioBench Stub Layer] Loaded logrx stubs
[Sdtmchecks Stub] Created sdtmchecks namespace with 2 stub functions
[RBioBench Stub Layer] Loaded sdtmchecks stubs
[Other Stubs] Registered 5 stub functions from 5 packages
[RBioBench Stub Layer] Loaded other package stubs
[RBioBench Stub Layer] Registered attach hook for admiral
[Admiral Stub] Injected 40 functions into admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[RBioBench Stub Layer] Stubs registered in admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[RBioBench Stub Layer] Stubs registered in admiral namespace
[RBioBench Stub Layer] .Rprofile loaded. Stubs will be auto-injected when admiral loads.
```

## Evaluation Result
```json
{
  "status": "PASS",
  "tier": "pass",
  "failure_stage": "pass",
  "pass": true,
  "score": 1.0,
  "message": "",
  "validation_diagnostics": {
    "failure_stage": "pass",
    "tier": "pass",
    "case_count": 1,
    "failed_case": null
  }
}
```

## Raw Agent Response
```text
#!/usr/bin/env Rscript

dir.create('outputs', showWarnings = FALSE, recursive = TRUE)

input_terms <- read.delim("inputs/input_terms.tsv", stringsAsFactors = FALSE)
mapping_col <- read.delim("inputs/mapping_col.tsv", stringsAsFactors = FALSE)
metadata    <- read.delim("inputs/metadata.tsv",    stringsAsFactors = FALSE)
target_col  <- read.delim("inputs/target_col.tsv",  stringsAsFactors = FALSE)

mc <- mapping_col[[1]]
tc <- target_col[[1]]

result <- data.frame(
  input_terms = input_terms[[1]],
  mapping_col = mc,
  result = metadata[[tc]][match(input_terms[[1]], metadata[[mc]])],
  stringsAsFactors = FALSE
)

write.csv(result, "outputs/result.csv", row.names = FALSE)
```
