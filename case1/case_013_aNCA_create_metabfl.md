# Case 013: pharmaverse/aNCA/create_metabfl

## Metadata
- Task ID: `pharmaverse/aNCA/create_metabfl`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(5, 6) vs llm=(15, 6).
```

## Prompt
```text
Write an R script to perform create metabfl using the aNCA clinical task contract.

Input: dataset.tsv, metabolites.tsv
Output: result.csv


Required columns for result.csv: id, value, group, category, PARAM, METABFL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
id	value	group	category
1	10.5	A	Type1
2	20.3	B	Type2
3	30.7	A	Type1
4	40.2	B	Type2
5	50.9	A	Type1
```

#### `metabolites.tsv`


```text
metabolites
"AUC"
"CMAX"
"METABOLITE1"
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

dataset <- read_tsv("dataset.tsv")
metabolites <- as.character(first_column(read_tsv("metabolites.tsv")))
if (!"PARAM" %in% names(dataset)) {
  dataset$PARAM <- "PARAM1"
}
dataset$METABFL <- ifelse(as.character(dataset$PARAM) %in% metabolites, "Y", "N")
write_result(dataset)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(data.table)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
dataset_path <- file.path("inputs", "dataset.tsv")
metabolites_path <- file.path("inputs", "metabolites.tsv")

dat <- fread(dataset_path, sep = "\t", header = TRUE)
mets <- fread(metabolites_path, sep = "\t", header = TRUE)

# Ensure columns
required_cols <- c("id", "value", "group", "category")
missing_cols <- setdiff(required_cols, names(dat))
if (length(missing_cols) > 0) {
  stop("Missing required columns in dataset.tsv: ", paste(missing_cols, collapse = ", "))
}

# aNCA clinical task contract: create METABFL flag
# METABFL = "Y" if PARAM corresponds to a metabolite, else "N"
met_vec <- unique(gsub('"', "", mets$metabolites))

# For demonstration, create records for each PARAM per id
# Here we assign PARAM directly from metabolites list
res <- rbindlist(lapply(met_vec, function(p) {
  tmp <- copy(dat)
  tmp[, PARAM := p]
  tmp
}))

res[, METABFL := ifelse(PARAM %in% met_vec, "Y", "N")]

# Keep required columns in specified order
out <- res[, .(id, value, group, category, PARAM, METABFL)]

# Write result
fwrite(out, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","value","group","category","PARAM","METABFL"
1,10.5,"A","Type1","PARAM1","N"
2,20.3,"B","Type2","PARAM1","N"
3,30.7,"A","Type1","PARAM1","N"
4,40.2,"B","Type2","PARAM1","N"
5,50.9,"A","Type1","PARAM1","N"
```

### LLM Output
#### `result.csv`


```csv
id,value,group,category,PARAM,METABFL
1,10.5,A,Type1,AUC,Y
2,20.3,B,Type2,AUC,Y
3,30.7,A,Type1,AUC,Y
4,40.2,B,Type2,AUC,Y
5,50.9,A,Type1,AUC,Y
1,10.5,A,Type1,CMAX,Y
2,20.3,B,Type2,CMAX,Y
3,30.7,A,Type1,CMAX,Y
4,40.2,B,Type2,CMAX,Y
5,50.9,A,Type1,CMAX,Y
1,10.5,A,Type1,METABOLITE1,Y
2,20.3,B,Type2,METABOLITE1,Y
3,30.7,A,Type1,METABOLITE1,Y
4,40.2,B,Type2,METABOLITE1,Y
5,50.9,A,Type1,METABOLITE1,Y
```
