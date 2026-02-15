---
name: Data in R Packages
description: Comprehensive guide to including data in R packages, covering exported data, internal data, raw files, documentation, and CRAN size limits
---

# Data in R Packages

## Overview

R packages can include data in several forms: exported datasets for users, internal data for package functions, raw data files, and dynamic package state. This skill covers all data types, documentation requirements, and CRAN restrictions.

## Four Types of Package Data

### 1. Exported Data (data/)

User-accessible datasets loaded with `data()` or direct reference.

**Location**: `data/` directory
**Format**: `.rda` or `.RData` files
**Access**: `data(dataset_name)` or direct reference (if LazyData: true)
**Documentation**: Required in `R/data.R`

### 2. Internal Data (R/sysdata.rda)

Data used by package functions, not accessible to users.

**Location**: `R/sysdata.rda`
**Format**: Single `.rda` file containing multiple objects
**Access**: Direct reference in package code
**Documentation**: Not required (internal only)

### 3. Raw Data Files (inst/extdata/)

Non-R data files (CSV, JSON, images, etc.) for users to access.

**Location**: `inst/extdata/`
**Format**: Any file format
**Access**: `system.file("extdata", "file.ext", package = "pkg")`
**Documentation**: Optional, usually in vignettes/examples

### 4. Dynamic Package State (environments)

Runtime state stored in package environments.

**Location**: Environment created in R code
**Format**: In-memory R objects
**Access**: Getter/setter functions
**Documentation**: Document the getter/setter functions

## Exported Data (data/)

### Creating Exported Data

```r
# Prepare your data:
my_dataset <- data.frame(
  id = 1:100,
  value = rnorm(100),
  category = sample(LETTERS[1:3], 100, replace = TRUE)
)

# Save to data/:
usethis::use_data(my_dataset, overwrite = TRUE)
```

This creates `data/my_dataset.rda`.

### Multiple Datasets

```r
# Save multiple datasets:
dataset1 <- mtcars[1:10, ]
dataset2 <- iris[1:50, ]

usethis::use_data(dataset1, dataset2, overwrite = TRUE)
# Creates data/dataset1.rda and data/dataset2.rda
```

### Compression Options

```r
# Default compression (gzip):
usethis::use_data(my_dataset)

# Maximum compression (xz - slowest, smallest):
usethis::use_data(my_dataset, compress = "xz")

# Faster compression (bzip2):
usethis::use_data(my_dataset, compress = "bzip2")

# No compression (largest):
usethis::use_data(my_dataset, compress = FALSE)
```

**CRAN recommendation**: Use `compress = "xz"` for data >1MB.

### LazyData

Add to DESCRIPTION to make data available without `data()` call:

```dcf
LazyData: true
```

```r
# Without LazyData:
library(mypackage)
data(my_dataset)  # Required
head(my_dataset)

# With LazyData:
library(mypackage)
head(my_dataset)  # Direct access
```

**Note**: LazyData loads datasets into namespace but keeps them on disk until accessed (lazy loading).

## Internal Data (R/sysdata.rda)

### Creating Internal Data

Internal data is for package functions only, not exported to users.

```r
# Create internal lookup tables, constants, etc.:
internal_lookup <- list(
  codes = c(A = 1, B = 2, C = 3),
  thresholds = c(low = 0.05, high = 0.95)
)

internal_constants <- list(
  api_version = "v2",
  default_timeout = 30
)

# Save to R/sysdata.rda:
usethis::use_data(
  internal_lookup,
  internal_constants,
  internal = TRUE,
  overwrite = TRUE
)
```

**All objects saved with `internal = TRUE` go into a single file**: `R/sysdata.rda`

### Using Internal Data

```r
# In package functions, reference directly:
my_function <- function(code) {
  value <- internal_lookup$codes[code]
  # ... use value ...
}

# No need for pkg:::sysdata syntax
# Objects are available in package namespace
```

### When to Use Internal Data

**Good uses:**
- Lookup tables
- Large constants
- Pre-computed values (avoid recomputation)
- Default configurations

**Avoid:**
- Data that changes (use environments instead)
- User-facing data (use data/ instead)
- Very large objects (consider lazy loading strategies)

## Raw Data Files (inst/extdata/)

### Adding Raw Data Files

```r
# Create inst/extdata/ directory:
dir.create("inst/extdata", recursive = TRUE)

# Add files manually or:
usethis::use_directory("inst/extdata")

# Then copy files into inst/extdata/
```

**Common file types:**
- CSV, TSV, Excel files
- JSON, XML, YAML
- Images (PNG, JPEG)
- Shapefiles, GeoJSON
- Text files, logs
- Binary formats

### Accessing Raw Data Files

```r
# In package code:
get_example_file <- function(filename) {
  system.file("extdata", filename, package = "mypackage")
}

# Usage:
csv_path <- system.file("extdata", "example.csv", package = "mypackage")
data <- read.csv(csv_path)

# Or in exported function:
#' @examples
#' file <- system.file("extdata", "example.csv", package = "mypackage")
#' data <- read_my_data(file)
read_my_data <- function(file) {
  # ...
}
```

### inst/ vs data/

```
inst/extdata/          # Raw files, any format
├── example.csv        # Access with system.file()
├── sample.json
└── image.png

data/                  # R objects only
├── dataset1.rda       # Access with data() or direct reference
└── dataset2.rda
```

**Use inst/extdata/ when:**
- Non-R formats (CSV, JSON, etc.)
- Files users need paths to
- Multiple related files
- Files for examples/vignettes

**Use data/ when:**
- R objects for analysis
- Data ready to use in R
- Common datasets for package functions

## Documenting Data

### Documenting Exported Data

Create `R/data.R` to document all datasets:

```r
# R/data.R

#' World Health Organization TB data
#'
#' A subset of data from the World Health Organization Global Tuberculosis
#' Report, containing TB cases by country, year, age, and sex.
#'
#' @format A data frame with 7,240 rows and 60 columns:
#' \describe{
#'   \item{country}{Character. Country name}
#'   \item{iso2}{Character. 2-letter ISO country code}
#'   \item{iso3}{Character. 3-letter ISO country code}
#'   \item{year}{Integer. Year of observation (1995-2013)}
#'   \item{new_sp_m014}{Integer. New smear-positive cases in males aged 0-14}
#'   \item{new_sp_m1524}{Integer. New smear-positive cases in males aged 15-24}
#' }
#'
#' @source World Health Organization Global Tuberculosis Report
#'   \url{https://www.who.int/teams/global-tuberculosis-programme/data}
#'
#' @examples
#' head(who)
#' summary(who$year)
#' table(who$country)
"who"
```

**Required tags:**
- `@format` - describe structure and columns
- Title and description (always)

**Recommended tags:**
- `@source` - where data came from
- `@examples` - how to use the data

### Data Documentation Templates

#### Data Frame

```r
#' Customer transaction data
#'
#' Sample transaction data for 1,000 customers over one year,
#' including purchase amounts, dates, and categories.
#'
#' @format A data frame with 1,000 rows and 5 columns:
#' \describe{
#'   \item{customer_id}{Character. Unique customer identifier}
#'   \item{transaction_date}{Date. Date of transaction}
#'   \item{amount}{Numeric. Transaction amount in USD}
#'   \item{category}{Factor. Product category (Electronics, Clothing, Food)}
#'   \item{region}{Character. Customer region (North, South, East, West)}
#' }
#'
#' @details
#' Data was generated synthetically to represent typical e-commerce
#' transaction patterns. Amounts range from $5 to $500.
#'
#' @source Generated using simulation based on real e-commerce patterns
#'
#' @examples
#' head(transactions)
#'
#' # Summary by category
#' aggregate(amount ~ category, data = transactions, FUN = mean)
#'
#' # Transactions over time
#' plot(transactions$transaction_date, transactions$amount)
"transactions"
```

#### List

```r
#' Configuration defaults
#'
#' Default configuration settings for the package.
#'
#' @format A list with components:
#' \describe{
#'   \item{api}{List. API configuration:}
#'     \itemize{
#'       \item \code{endpoint}: Character. Base API URL
#'       \item \code{timeout}: Numeric. Request timeout in seconds
#'       \item \code{retries}: Integer. Number of retry attempts
#'     }
#'   \item{cache}{List. Cache settings:}
#'     \itemize{
#'       \item \code{enabled}: Logical. Whether caching is enabled
#'       \item \code{max_size}: Numeric. Maximum cache size in MB
#'     }
#' }
#'
#' @examples
#' config_defaults$api$endpoint
#' config_defaults$cache$enabled
"config_defaults"
```

#### Vector

```r
#' Built-in color palette
#'
#' A vector of hex color codes for data visualization.
#'
#' @format A named character vector of length 12:
#' \describe{
#'   \item{Names}{Color names (red, blue, green, etc.)}
#'   \item{Values}{Hex color codes}
#' }
#'
#' @examples
#' palette_colors
#' palette_colors["blue"]
#'
#' # Use in plot
#' plot(1:12, col = palette_colors, pch = 16, cex = 2)
"palette_colors"
```

#### Matrix

```r
#' Correlation matrix example
#'
#' Sample correlation matrix for demonstration purposes.
#'
#' @format A 10x10 numeric matrix with row and column names
#'   representing variables v1 through v10. Values are correlations
#'   ranging from -1 to 1.
#'
#' @examples
#' correlation_matrix
#' diag(correlation_matrix)  # All 1s (self-correlation)
"correlation_matrix"
```

## data-raw/ Workflow

Keep data preparation scripts separate from package code.

### Setup

```r
# Create data-raw/ directory and template script:
usethis::use_data_raw("dataset_name")
```

This creates:
- `data-raw/` directory
- `data-raw/dataset_name.R` script
- Adds `^data-raw$` to `.Rbuildignore`

### Data Preparation Script

```r
# data-raw/customer_data.R

## Code to prepare `customer_data` dataset

library(dplyr)
library(lubridate)

# Read raw data:
raw_data <- read.csv("~/Downloads/raw_customer_data.csv")

# Clean and process:
customer_data <- raw_data %>%
  # Clean column names:
  janitor::clean_names() %>%
  # Parse dates:
  mutate(
    transaction_date = ymd(transaction_date),
    signup_date = ymd(signup_date)
  ) %>%
  # Filter to relevant period:
  filter(
    transaction_date >= "2020-01-01",
    transaction_date <= "2023-12-31"
  ) %>%
  # Select and rename:
  select(
    customer_id = id,
    transaction_date,
    amount = transaction_amount,
    category = product_category,
    region = customer_region
  ) %>%
  # Remove duplicates:
  distinct() %>%
  # Sort:
  arrange(transaction_date)

# Save to package:
usethis::use_data(customer_data, overwrite = TRUE, compress = "xz")
```

### Benefits of data-raw/

- **Reproducibility**: Anyone can recreate the data
- **Documentation**: Scripts document data transformations
- **Version control**: Track changes to data preparation
- **Separation**: Keep raw data separate from package
- **Updates**: Easy to update data with new source files

### Multiple Datasets

```r
# data-raw/prepare_all_data.R

# Prepare dataset 1:
source("data-raw/dataset1.R")

# Prepare dataset 2:
source("data-raw/dataset2.R")

# Or in single file:
dataset1 <- prepare_dataset1()
usethis::use_data(dataset1, overwrite = TRUE)

dataset2 <- prepare_dataset2()
usethis::use_data(dataset2, overwrite = TRUE)
```

## CRAN Size Limits

### Size Restrictions

**CRAN limits:**
- Total package size: **5 MB** (compressed)
- Per subdirectory: **1 MB** (recommended)
- Larger packages require justification

**Check size:**
```r
# Build package:
pkgbuild::build()

# Check .tar.gz size:
file.size("../mypackage_1.0.0.tar.gz") / 1024^2  # MB
```

### Reducing Data Size

#### 1. Compression

```r
# Use maximum compression:
usethis::use_data(dataset, compress = "xz")

# Compare sizes:
save(dataset, file = "test_gzip.rda", compress = "gzip")
save(dataset, file = "test_bzip2.rda", compress = "bzip2")
save(dataset, file = "test_xz.rda", compress = "xz")

file.size("test_gzip.rda") / 1024   # KB
file.size("test_bzip2.rda") / 1024
file.size("test_xz.rda") / 1024
```

#### 2. Subsetting

```r
# Instead of full dataset:
# full_data <- huge_dataset  # 10 MB

# Use representative sample:
sample_data <- huge_dataset %>%
  sample_n(1000) %>%
  select(key_columns)  # Select only essential columns

usethis::use_data(sample_data, compress = "xz")
```

#### 3. Move to inst/extdata/

```r
# Instead of:
# usethis::use_data(large_dataset)  # data/large_dataset.rda

# Use raw format in inst/extdata/:
write.csv(large_dataset, "inst/extdata/large_dataset.csv.gz")

# Users access with:
#' @examples
#' data_path <- system.file("extdata", "large_dataset.csv.gz", package = "pkg")
#' data <- read.csv(data_path)
```

#### 4. External Data Packages

For very large data, create separate data package:

```
mypackage/          # Main package
mypackage.data/     # Data-only package (optional dependency)
```

```r
# In main package:
# Add to Suggests:
Suggests: mypackage.data

# In functions:
get_full_data <- function() {
  if (!requireNamespace("mypackage.data", quietly = TRUE)) {
    stop("Install mypackage.data: install.packages('mypackage.data')")
  }
  mypackage.data::full_dataset
}
```

#### 5. Download on Demand

```r
# Store data externally, download as needed:
get_external_data <- function(cache = TRUE) {
  cache_file <- rappdirs::user_cache_dir("mypackage")
  data_file <- file.path(cache_file, "data.rds")

  if (cache && file.exists(data_file)) {
    return(readRDS(data_file))
  }

  # Download:
  url <- "https://example.com/data.rds"
  data <- readRDS(url(url))

  if (cache) {
    dir.create(cache_file, recursive = TRUE, showWarnings = FALSE)
    saveRDS(data, data_file)
  }

  data
}
```

## Dynamic Package State

Use environments for runtime state, not exported data.

### Package Environment Pattern

```r
# R/state.R

# Create package environment:
pkg_env <- new.env(parent = emptyenv())

# Initialize in .onLoad():
.onLoad <- function(libname, pkgname) {
  pkg_env$cache <- new.env(parent = emptyenv())
  pkg_env$config <- list(
    api_key = NULL,
    verbose = FALSE
  )
}

# Getters and setters:
#' Get configuration value
#' @param key Configuration key
#' @export
get_config <- function(key) {
  pkg_env$config[[key]]
}

#' Set configuration value
#' @param key Configuration key
#' @param value New value
#' @export
set_config <- function(key, value) {
  pkg_env$config[[key]] <- value
  invisible(value)
}

# Cache functions:
cache_get <- function(key) {
  pkg_env$cache[[key]]
}

cache_set <- function(key, value) {
  pkg_env$cache[[key]] <- value
  invisible(value)
}

cache_clear <- function() {
  rm(list = ls(pkg_env$cache), envir = pkg_env$cache)
  invisible(NULL)
}
```

### When to Use Environments vs data/

**Use environments for:**
- Runtime configuration
- Caches
- Mutable state
- Session-specific data

**Use data/ for:**
- Static datasets
- Examples and documentation
- Reference data
- Immutable package constants

## Complete Example

### Full Data Package Setup

```r
# 1. Create data-raw/ script:
usethis::use_data_raw("customers")
```

```r
# data-raw/customers.R

## Code to prepare `customers` dataset

library(dplyr)

# Read and clean:
customers <- read.csv("raw/customers.csv") %>%
  filter(active == TRUE) %>%
  select(id, name, region, signup_date) %>%
  arrange(signup_date)

# Save:
usethis::use_data(customers, overwrite = TRUE, compress = "xz")
```

```r
# 2. Document in R/data.R:

#' Customer data
#'
#' Sample customer records for demonstration and testing.
#'
#' @format A data frame with 500 rows and 4 columns:
#' \describe{
#'   \item{id}{Character. Unique customer ID}
#'   \item{name}{Character. Customer name}
#'   \item{region}{Factor. Geographic region (North, South, East, West)}
#'   \item{signup_date}{Date. Account creation date}
#' }
#'
#' @source Generated from internal CRM system (anonymized)
#'
#' @examples
#' head(customers)
#' table(customers$region)
"customers"
```

```r
# 3. Add LazyData to DESCRIPTION:
LazyData: true
```

```r
# 4. Build and check:
devtools::document()
devtools::check()
```

## Common Pitfalls

### 1. Forgetting LazyData

Without LazyData, users must call `data()` explicitly:

```r
# Add to DESCRIPTION:
LazyData: true
```

### 2. Not Documenting Data

CRAN requires documentation for all exported data:

```r
# Create R/data.R with documentation for each dataset
```

### 3. Exceeding Size Limits

```r
# Check package size:
devtools::build()
# Look at .tar.gz file size

# Reduce with:
# - compress = "xz"
# - Subset data
# - Move to inst/extdata/
# - External data package
```

### 4. Wrong File Extension

```r
# WRONG:
usethis::use_data(data, file = "data.RData")

# RIGHT:
usethis::use_data(data)  # Automatically creates .rda
```

### 5. Including data-raw/ in Package

Should be in `.Rbuildignore`:

```
^data-raw$
```

`usethis::use_data_raw()` adds this automatically.

### 6. Using save() Instead of usethis::use_data()

```r
# AVOID:
save(dataset, file = "data/dataset.rda")

# PREFER:
usethis::use_data(dataset)  # Better defaults, automatic compression
```

### 7. Not Using system.file() for inst/extdata/

```r
# WRONG:
data <- read.csv("inst/extdata/example.csv")  # Doesn't work after installation

# RIGHT:
path <- system.file("extdata", "example.csv", package = "mypackage")
data <- read.csv(path)
```

### 8. Documenting Internal Data

Internal data (R/sysdata.rda) should NOT be documented:

```r
# No documentation needed for internal data
usethis::use_data(internal_obj, internal = TRUE)
```

### 9. Missing @format Tag

```r
# INCOMPLETE:
#' My data
"mydata"

# COMPLETE:
#' My data
#' @format A data frame with 100 rows and 5 columns
"mydata"
```

### 10. Hardcoding Paths in data-raw/

```r
# FRAGILE:
raw <- read.csv("C:/Users/Me/Desktop/data.csv")

# BETTER:
raw <- read.csv("raw_data/data.csv")  # Relative to project root

# BEST:
raw <- read.csv(here::here("raw_data", "data.csv"))
```

## Quick Reference

### Creating Data

```r
# Exported data:
usethis::use_data(dataset, compress = "xz")

# Internal data:
usethis::use_data(internal_obj, internal = TRUE)

# Data preparation:
usethis::use_data_raw("dataset")

# Raw files:
# Manually copy to inst/extdata/
```

### Accessing Data

```r
# Exported data:
data(dataset)          # Explicit load
dataset                # Direct (if LazyData: true)

# Internal data:
internal_obj           # Direct in package code

# Raw files:
system.file("extdata", "file.csv", package = "pkg")
```

### Documentation Template

```r
#' Dataset title
#'
#' Description of what the dataset contains.
#'
#' @format A data frame with X rows and Y columns:
#' \describe{
#'   \item{col1}{Type. Description}
#'   \item{col2}{Type. Description}
#' }
#'
#' @source Where the data came from
#'
#' @examples
#' head(dataset)
"dataset"
```

### Size Management

```r
# Check size:
devtools::build()

# Optimize compression:
usethis::use_data(data, compress = "xz")

# Subset:
data_sample <- data[1:1000, ]
```

## Resources

- [R Packages: Data](https://r-pkgs.org/data.html)
- [Writing R Extensions: Data](https://cran.r-project.org/doc/manuals/R-exts.html#Data-in-packages)
- usethis package documentation
