---
name: add-data
description: Add a bundled dataset to an R package with documentation
---

# Add Dataset to Package

This command adds a dataset to an R package, including the raw data processing script, the bundled .rda file, and complete documentation.

## Workflow

### Step 1: Understand Data Requirements

Gather information from the user:

- **Dataset name**: Use lowercase with underscores (e.g., `customer_data`, `gene_expression`)
- **Data source**: Where does the data come from? File path, URL, API?
- **Data format**: CSV, Excel, JSON, database, etc.?
- **Data structure**: data.frame, tibble, matrix, list, vector?
- **Purpose**: What is this data used for? What examples will it enable?
- **Size**: How large is the dataset? (Keep under 5 MB for CRAN packages)
- **License**: Can this data be redistributed? Any attribution required?
- **Documentation needs**: What should users know about each variable?

### Step 2: Create Data Preparation Script

Create `data-raw/prepare_dataset_name.R`:

This script documents the complete process of creating the dataset from raw sources.

```r
## Code to prepare `dataset_name` dataset goes here

# Load required packages
library(dplyr)
library(readr)
library(here)

# Read raw data
# Option 1: From local file
raw_data <- read_csv(here("data-raw", "source_file.csv"))

# Option 2: From URL
# url <- "https://example.com/data.csv"
# raw_data <- read_csv(url)

# Option 3: From package data
# raw_data <- readRDS(here("data-raw", "raw_data.rds"))

# Clean and process data
dataset_name <- raw_data %>%
  # Remove missing values
  filter(!is.na(important_column)) %>%
  # Rename columns to snake_case
  rename(
    new_name = OldName,
    another_name = AnotherOldName
  ) %>%
  # Select relevant columns
  select(id, name, value, category) %>%
  # Create new variables
  mutate(
    value_scaled = scale(value),
    category = factor(category, levels = c("low", "medium", "high"))
  ) %>%
  # Sort for consistency
  arrange(id)

# Validate data
stopifnot(
  nrow(dataset_name) > 0,
  !any(is.na(dataset_name$id)),
  all(dataset_name$value >= 0)
)

# Save as package data
usethis::use_data(dataset_name, overwrite = TRUE)

# Optional: Save as CSV for inst/extdata (for examples)
# write_csv(dataset_name, here("inst", "extdata", "dataset_name.csv"))
```

If the directory doesn't exist, create it:

```r
usethis::use_data_raw("dataset_name")  # Creates data-raw/ and adds to .Rbuildignore
```

### Step 3: Run the Preparation Script

Execute the script to generate `data/dataset_name.rda`:

```r
source("data-raw/prepare_dataset_name.R")
```

This creates:

- `data/dataset_name.rda` - The compressed R data file that ships with the package

Verify the data:

```r
load("data/dataset_name.rda")
str(dataset_name)
head(dataset_name)
```

### Step 4: Optional - Add Raw Data to inst/extdata

If you want to include raw/example data files that users can access:

```r
# In the preparation script or manually:
dir.create("inst/extdata", recursive = TRUE, showWarnings = FALSE)

# Save CSV version
write_csv(dataset_name, "inst/extdata/dataset_name.csv")

# Or copy original raw file
file.copy("data-raw/source.csv", "inst/extdata/dataset_name_raw.csv")
```

Users can then access these with:

```r
system.file("extdata", "dataset_name.csv", package = "yourpackage")
```

### Step 5: Document the Dataset

Create or append to `R/data.R`:

```r
#' Customer transaction data
#'
#' A dataset containing transaction records from an e-commerce platform.
#' This data is useful for demonstrating data cleaning and analysis workflows.
#'
#' @format A data frame with 1,234 rows and 6 variables:
#' \describe{
#'   \item{id}{Unique transaction identifier, integer}
#'   \item{customer_name}{Customer name, character}
#'   \item{product}{Product purchased, character}
#'   \item{quantity}{Number of items, integer}
#'   \item{price}{Price per item in USD, numeric}
#'   \item{date}{Transaction date, Date class}
#' }
#'
#' @source Simulated data based on typical e-commerce patterns.
#'   Generated using \code{data-raw/prepare_customer_data.R}.
#'
#' @examples
#' # Load the data
#' data(customer_data)
#'
#' # View structure
#' str(customer_data)
#'
#' # Basic summary
#' summary(customer_data$price)
#'
#' # Use in analysis
#' library(dplyr)
#' customer_data %>%
#'   group_by(product) %>%
#'   summarise(total_sales = sum(quantity * price))
"customer_data"
```

Key documentation elements:

- **Title**: Short description of the dataset
- **Description**: What the data contains, what it's useful for
- **`@format`**: Exact structure (rows, columns, and variable descriptions)
- **`\describe{}`**: Document EVERY variable with name, type, and meaning
- **`@source`**: Where the data came from, citation if published
- **`@examples`**: Show how to load and use the data
- **Final line**: Must be the dataset name in quotes

For multiple datasets, document each in the same `R/data.R` file:

```r
#' First dataset
#' ...
"dataset_one"

#' Second dataset
#' ...
"dataset_two"
```

### Step 6: Enable Lazy Data Loading

Add to `DESCRIPTION` if not already present:

```
LazyData: true
```

This allows users to access the data by name without calling `data()`:

```r
library(yourpackage)
head(customer_data)  # Works immediately
```

Without `LazyData: true`, users must call:

```r
data(customer_data)
head(customer_data)
```

### Step 7: Update .Rbuildignore

Ensure `data-raw/` is excluded from the built package:

Add to `.Rbuildignore`:

```
^data-raw$
```

This is automatically done if you used `usethis::use_data_raw()`.

### Step 8: Generate Documentation and Check

Run documentation and checks:

```r
# Generate help file
devtools::document()

# Check help page
?customer_data

# Run R CMD check
devtools::check()
```

Look for:

- ✓ No warnings about undocumented data
- ✓ No notes about large data files (keep under 5 MB for CRAN)
- ✓ Data loads correctly in examples
- ✓ Help page renders properly

Common issues:

- **Undocumented data**: Add documentation to `R/data.R`
- **Large data warning**: Compress better, reduce size, or move to separate package
- **LazyData note**: Add `LazyData: true` to DESCRIPTION

### Step 9: Add Example Usage to README/Vignettes

Update `README.md` to showcase the data:

```markdown
## Example Data

The package includes the `customer_data` dataset for demonstration:

```r
library(yourpackage)
data(customer_data)

# Quick summary
summary(customer_data)
```
```

Add to vignettes where the data helps illustrate package functionality.

### Final Checklist

Before completing:

- [ ] `data-raw/prepare_dataset_name.R` created with full processing pipeline
- [ ] `data/dataset_name.rda` generated (under 5 MB)
- [ ] `R/data.R` contains complete documentation
- [ ] Every variable documented in `@format` section
- [ ] `@source` indicates data origin
- [ ] `@examples` show how to use the data
- [ ] `LazyData: true` in DESCRIPTION
- [ ] `^data-raw$` in .Rbuildignore
- [ ] `devtools::document()` ran successfully
- [ ] `devtools::check()` shows no data-related warnings
- [ ] Data loads correctly: `data(dataset_name)` or just `dataset_name`

Report to user:

```
✓ Dataset 'customer_data' added successfully!

Created:
  - data-raw/prepare_customer_data.R (processing script)
  - data/customer_data.rda (1,234 rows × 6 columns, 45 KB)
  - R/data.R (documentation)
  - man/customer_data.Rd (generated help file)

Dataset info:
  - Format: data.frame
  - Size: 45 KB (well under CRAN 5 MB limit)
  - Variables: 6 (all documented)
  - LazyData: enabled

Quality:
  - R CMD check: 0 errors ✓ | 0 warnings ✓
  - Documentation: Complete
  - Reproducible: Full preparation script available

Users can access with: data(customer_data) or just customer_data
```

## Example Interaction

User: "Add the 'iris' dataset but filtered to just setosa species"