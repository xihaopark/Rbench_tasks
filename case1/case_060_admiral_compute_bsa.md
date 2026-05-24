# Case 060: pharmaverse/admiral/compute_bsa

## Metadata
- Task ID: `pharmaverse/admiral/compute_bsa`
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
Write an R script to perform compute BSA using the admiral clinical task contract.

Input: height.tsv, method.tsv, weight.tsv
Output: result.csv


Required columns for result.csv: height, weight, method, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `height.tsv`


```text
height
170
180
160
```

#### `method.tsv`


```text
method
Mosteller
```

#### `weight.tsv`


```text
weight
70
80
55
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

height <- as.numeric(read.delim(file.path("inputs", "height.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$height)
weight <- as.numeric(read.delim(file.path("inputs", "weight.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$weight)
method <- as.character(read.delim(file.path("inputs", "method.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$method[[1]])
result <- admiral::compute_bsa(height = height, weight = weight, method = method)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(height = height, weight = weight, method = method, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
height_df <- read_tsv(
  "inputs/height.tsv",
  show_col_types = FALSE,
  progress = FALSE
)

weight_df <- read_tsv(
  "inputs/weight.tsv",
  show_col_types = FALSE,
  progress = FALSE
)

method_df <- read_tsv(
  "inputs/method.tsv",
  show_col_types = FALSE,
  progress = FALSE
)

# Prepare vectors
heights <- height_df$height
weights <- weight_df$weight
method  <- method_df$method[1]

# Recycle height to match length of weight if needed
if (length(heights) != length(weights)) {
  heights <- rep(heights, length.out = length(weights))
}

# Supported methods (admiral clinical task contract: Mosteller)
compute_bsa <- function(height_cm, weight_kg, method) {
  if (method == "Mosteller") {
    return(sqrt(height_cm * weight_kg / 3600))
  } else {
    stop("Unsupported method: ", method)
  }
}

result_vec <- compute_bsa(heights, weights, method)

result_df <- tibble(
  height = heights,
  weight = weights,
  method = method,
  result = result_vec
)

write_csv(result_df, "outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"height","weight","method","result"
170,70,"Mosteller",1.81811868577262
180,80,"Mosteller",2
160,55,"Mosteller",1.56347191994114
```

### LLM Output
#### `result.csv`


```csv
height,weight,method,result
170,70,Mosteller,1.818118685772619
180,80,Mosteller,2
160,55,Mosteller,1.5634719199411433
```
