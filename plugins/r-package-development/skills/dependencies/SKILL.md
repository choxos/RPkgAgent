---
name: Package Dependencies
description: Complete guide to managing R package dependencies, including Imports vs Suggests, namespace imports, and handling dependencies in code, tests, examples, and vignettes
---

# Package Dependencies

## Overview

Managing dependencies correctly is critical for R packages. This skill covers the different dependency types, how to declare and use them, and common patterns for conditional dependencies.

## CRITICAL Concept: Listing vs Importing

**Most important rule**: Listing a package in `Imports:` does NOT make its functions available!

```r
# DESCRIPTION:
Imports:
  dplyr

# This does NOT work:
my_function <- function(data) {
  filter(data, value > 0)  # ERROR: object 'filter' not found
}

# You must ALSO either:

# Option 1: Use explicit namespace (RECOMMENDED for most cases):
my_function <- function(data) {
  dplyr::filter(data, value > 0)
}

# Option 2: Import to namespace via roxygen2:
#' @importFrom dplyr filter
my_function <- function(data) {
  filter(data, value > 0)
}
```

**Listing in DESCRIPTION ensures the package is installed.**
**Importing to NAMESPACE makes functions available in your code.**

## Dependency Types

### Imports

Packages required for your package to work.

```dcf
# DESCRIPTION:
Imports:
    dplyr (>= 1.0.0),
    rlang (>= 1.0.0),
    tidyr
```

**Guarantees:**
- Installed when your package is installed
- Loaded when your package is loaded (but not attached)
- Functions available via `pkg::fun()`

**Use for:**
- Packages your code depends on
- Packages used in most functions
- Critical dependencies

**Usage pattern:**
```r
# Default: Use pkg::fun()
my_function <- function(x) {
  dplyr::mutate(x, new_col = value * 2)
}

# Or import specific functions:
#' @importFrom dplyr mutate select filter
my_function <- function(x) {
  x %>%
    filter(value > 0) %>%
    mutate(doubled = value * 2)
}
```

### Suggests

Optional packages for enhanced functionality, tests, or documentation.

```dcf
# DESCRIPTION:
Suggests:
    ggplot2,
    testthat (>= 3.0.0),
    knitr,
    rmarkdown,
    covr
```

**No guarantees:**
- May or may not be installed
- Must check availability before use
- Cannot use `pkg::fun()` without checking

**Use for:**
- Packages for optional features
- Testing frameworks (testthat, covr)
- Vignette builders (knitr, rmarkdown)
- Packages for examples only
- Heavy dependencies users may not need

**Usage pattern:**
```r
# MUST check availability:
my_plot <- function(data) {
  if (!requireNamespace("ggplot2", quietly = TRUE)) {
    stop("Package 'ggplot2' required but not installed.\n",
         "Install with: install.packages('ggplot2')",
         call. = FALSE)
  }

  ggplot2::ggplot(data, ggplot2::aes(x, y)) +
    ggplot2::geom_point()
}

# Better: Use rlang::check_installed()
my_plot <- function(data) {
  rlang::check_installed("ggplot2", reason = "to create plots")

  ggplot2::ggplot(data, ggplot2::aes(x, y)) +
    ggplot2::geom_point()
}
```

### Depends

Makes another package's functions available in user's workspace (rarely recommended).

```dcf
# DESCRIPTION:
Depends:
    R (>= 4.1.0),
    methods
```

**Effects:**
- Package is attached when yours is attached
- Functions available in user's search path
- Modifies user's environment

**Modern usage:**
- `R (>= version)` - minimum R version (ALWAYS use this)
- `methods` - if defining S4 classes
- Almost never use for other packages

**Why avoid:**
```r
# If you use Depends: dplyr:
library(mypackage)  # Also attaches dplyr

# Now user's environment has all dplyr functions:
filter  # Available (might conflict with stats::filter)
```

**Better approach:**
```r
# Use Imports + explicit namespace:
Imports: dplyr

# In code:
dplyr::filter(...)
```

### LinkingTo

For packages with C/C++ code using headers from other packages.

```dcf
# DESCRIPTION:
LinkingTo:
    Rcpp,
    RcppArmadillo
```

**Use for:**
- Rcpp packages
- Packages providing C++ headers
- Compiled code dependencies

**Often combined with Imports:**
```dcf
Imports:
    Rcpp (>= 1.0.0)
LinkingTo:
    Rcpp
```

### Config/Needs/*

Dependencies for development tools, not package functionality.

```dcf
# DESCRIPTION:
Config/Needs/website:
    pkgdown
Config/Needs/coverage:
    covr
Config/Needs/development:
    devtools,
    usethis,
    roxygen2
```

**Use for:**
- pkgdown for website
- covr for coverage
- Development tools
- CI-specific packages

**Not installed by default:**
```r
# Install with:
pak::pak("mypackage", dependencies = TRUE)  # Includes Config/Needs/*
```

## Adding Dependencies

### Using usethis Helpers

```r
# Add to Imports:
usethis::use_package("dplyr")
usethis::use_package("rlang", min_version = "1.0.0")

# Add to Suggests:
usethis::use_package("ggplot2", type = "Suggests")

# Add to Imports with @importFrom:
usethis::use_import_from("dplyr", c("filter", "mutate", "select"))
# Adds to DESCRIPTION AND creates roxygen2 skeleton

# Add minimum R version:
usethis::use_package("R", min_version = "4.1.0", type = "Depends")
```

### Manual Addition

```dcf
# DESCRIPTION:
Imports:
    dplyr (>= 1.1.0),
    rlang (>= 1.0.0),
    tidyr,
    purrr
Suggests:
    ggplot2 (>= 3.4.0),
    testthat (>= 3.0.0)
```

**Version specifications:**
```dcf
dplyr                    # Any version
dplyr (>= 1.0.0)        # At least 1.0.0
dplyr (>= 1.0.0, < 2.0.0)  # Rarely used, not recommended
```

## Importing Functions to Namespace

### Pattern 1: Explicit Namespace (Recommended)

```r
# No NAMESPACE imports needed
# Just use pkg::fun() everywhere:

my_function <- function(data) {
  data %>%
    dplyr::filter(value > 0) %>%
    dplyr::mutate(doubled = value * 2) %>%
    dplyr::select(id, doubled)
}
```

**Advantages:**
- Clear where functions come from
- No NAMESPACE management needed
- Easy to understand
- No import conflicts

**Disadvantages:**
- More typing
- Slightly verbose

### Pattern 2: Selective Import (@importFrom)

```r
# Import specific functions:
#' @importFrom dplyr filter mutate select
#' @importFrom rlang .data .env
my_function <- function(data) {
  data %>%
    filter(value > 0) %>%
    mutate(doubled = .data$value * 2) %>%
    select(id, doubled)
}
```

**When to use:**
- Functions used many times
- Infix operators (%>%, %||%, :=)
- Core package dependencies
- Reduces verbosity

**Where to put @importFrom:**
```r
# Option 1: In function documentation:
#' My function
#' @importFrom dplyr filter mutate
my_function <- function() { ... }

# Option 2: In package-level doc (R/mypackage-package.R):
#' @importFrom dplyr filter mutate select arrange
#' @importFrom rlang .data .env %||%
"_PACKAGE"

# Option 3: Dedicated imports file (R/aaa-imports.R):
#' @importFrom dplyr filter mutate select
#' @importFrom rlang .data %||%
NULL
```

### Pattern 3: Full Import (@import) - Rare

```r
#' @import rlang
```

**Only for:**
- rlang (if building tidy evaluation package)
- Your own internal package

**Avoid for most packages:**
- Namespace pollution
- Potential conflicts
- Unclear provenance

### Operators and Infix Functions

**Always import operators:**

```r
# WRONG - doesn't work:
data %>% dplyr::filter(x > 0)  # Error: %>% not found

# RIGHT:
#' @importFrom magrittr %>%
data %>% dplyr::filter(x > 0)

# Or use base pipe (R >= 4.1):
data |> dplyr::filter(x > 0)  # No import needed
```

**Common operators to import:**
```r
#' @importFrom magrittr %>%
#' @importFrom rlang %||% !! !!!
#' @importFrom data.table := .N .SD
```

## Using Dependencies in Different Contexts

### In Package Code (R/)

```r
# Imports dependencies - use pkg::fun() or @importFrom:
#' @importFrom dplyr filter
my_function <- function(data) {
  filter(data, value > 0)  # OK: imported
}

# Or:
my_function <- function(data) {
  dplyr::filter(data, value > 0)  # OK: explicit namespace
}

# Suggests dependencies - MUST check first:
my_optional_feature <- function(data) {
  rlang::check_installed("ggplot2", reason = "for plotting")
  ggplot2::ggplot(data, ggplot2::aes(x, y)) +
    ggplot2::geom_point()
}
```

### In Examples (@examples)

```r
# Imports - can use freely:
#' @examples
#' my_function(mtcars)

# Suggests - must check or wrap:
#' @examples
#' \dontrun{
#' # Requires ggplot2
#' my_plot(mtcars)
#' }
#'
#' @examplesIf requireNamespace("ggplot2", quietly = TRUE)
#' my_plot(mtcars)
```

### In Tests (tests/testthat/)

```r
# Imports - can use freely:
test_that("function works", {
  result <- my_function(data)
  expect_equal(result$value, expected)
})

# Suggests - MUST skip if not available:
test_that("plotting works", {
  skip_if_not_installed("ggplot2")

  plot <- my_plot(data)
  expect_s3_class(plot, "gg")
})

test_that("integration with optional package", {
  skip_if_not_installed("dplyr")

  library(dplyr)
  result <- data %>%
    my_transform() %>%
    summarize(mean = mean(value))

  expect_equal(result$mean, 5)
})
```

### In Vignettes (vignettes/)

```r
# YAML header for Suggests dependency:
# ---
# title: "My Vignette"
# vignette: >
#   %\VignetteIndexEntry{My Vignette}
#   %\VignetteEngine{knitr::rmarkdown}
# ---

# First chunk - setup with conditional evaluation:
# ```{r setup, include=FALSE}
# knitr::opts_chunk$set(
#   eval = requireNamespace("ggplot2", quietly = TRUE)
# )
# ```

# Now code chunks only run if ggplot2 available:
# ```{r}
# library(ggplot2)
# ggplot(data, aes(x, y)) + geom_point()
# ```
```

**Alternative: Use separate vignettes:**
```dcf
# DESCRIPTION:
Suggests:
    knitr,
    rmarkdown,
    ggplot2

# vignettes/basic-usage.Rmd - no optional deps
# vignettes/advanced-plotting.Rmd - requires ggplot2
```

### In Documentation (roxygen2)

```r
# References to Suggests packages:
#' @description
#' This function provides plotting capabilities. Requires the
#' \pkg{ggplot2} package to be installed.
#'
#' @seealso [ggplot2::ggplot()] for more plotting options
```

## Minimum Version Specifications

### When to Specify Versions

```dcf
# Always specify if you need specific features:
Imports:
    dplyr (>= 1.1.0),    # Uses .by argument
    rlang (>= 1.0.0),    # Uses check_installed()
    tidyr (>= 1.3.0)     # Uses separate_wider_*

# Don't specify if any version works:
Imports:
    jsonlite,  # Basic read/write - any version fine
    httr       # Standard requests - any version fine
```

### Finding Minimum Versions

```r
# Check when feature was introduced:
# 1. Check package NEWS file
# 2. Check function documentation
# 3. Look at GitHub releases/tags

# Use version where feature appeared:
usethis::use_package("dplyr", min_version = "1.1.0")
```

### R Version

Always specify minimum R version:

```dcf
# DESCRIPTION:
Depends:
    R (>= 4.1.0)
```

**Why specify:**
- Uses R 4.1+ features (native pipe |>, lambda \(x))
- Uses R 4.0+ features (stringsAsFactors = FALSE default)
- Package developed/tested on specific version

**How to choose:**
```r
# Conservative (wide compatibility):
R (>= 4.0.0)

# Modern (allows new features):
R (>= 4.1.0)  # Native pipe, lambda syntax

# Latest stable:
R (>= 4.3.0)
```

## Advanced Patterns

### Soft Dependencies

Functions work differently with/without optional package:

```r
my_function <- function(data, use_fast = TRUE) {
  if (use_fast && requireNamespace("data.table", quietly = TRUE)) {
    # Fast path with data.table:
    dt <- data.table::as.data.table(data)
    result <- dt[, .(mean = mean(value)), by = group]
    return(as.data.frame(result))
  }

  # Fallback to base R:
  aggregate(value ~ group, data, FUN = mean)
}
```

### Conditional Method Registration

```r
# In .onLoad() (R/zzz.R):
.onLoad <- function(libname, pkgname) {
  # Register S3 method only if package available:
  if (requireNamespace("dplyr", quietly = TRUE)) {
    s3_register("dplyr::dplyr_reconstruct", "myclass")
  }
}

# Helper function (from vctrs):
s3_register <- function(generic, class, method = NULL) {
  stopifnot(is.character(generic), length(generic) == 1)
  stopifnot(is.character(class), length(class) == 1)

  pieces <- strsplit(generic, "::")[[1]]
  stopifnot(length(pieces) == 2)
  package <- pieces[[1]]
  generic <- pieces[[2]]

  caller <- parent.frame()

  get_method_env <- function() {
    top <- topenv(caller)
    if (isNamespace(top)) {
      asNamespace(environmentName(top))
    } else {
      caller
    }
  }
  get_method <- function(method, env) {
    if (is.null(method)) {
      get(paste0(generic, ".", class), envir = get_method_env())
    } else {
      method
    }
  }

  method_fn <- get_method(method)
  stopifnot(is.function(method_fn))

  setHook(
    packageEvent(package, "onLoad"),
    function(...) {
      ns <- asNamespace(package)
      method_name <- paste0(generic, ".", class)
      env <- get_method_env()
      registerS3method(generic, class, method_fn, envir = ns)
    }
  )

  if (!isNamespaceLoaded(package)) {
    return(invisible())
  }

  envir <- asNamespace(package)
  if (exists(generic, envir)) {
    registerS3method(generic, class, method_fn, envir = envir)
  }

  invisible()
}
```

### Feature Flags

```r
# Check if optional features available:
has_plotting <- function() {
  requireNamespace("ggplot2", quietly = TRUE)
}

has_fast_processing <- function() {
  requireNamespace("data.table", quietly = TRUE)
}

# Inform users:
package_capabilities <- function() {
  list(
    plotting = has_plotting(),
    fast_processing = has_fast_processing()
  )
}

#' @examples
#' package_capabilities()
#' # $plotting
#' # [1] TRUE
#' # $fast_processing
#' # [1] FALSE
```

### Deprecation Paths

```r
# Gradually move from Imports to Suggests:

# Version 1.0.0:
# Imports: oldpkg
my_function <- function() {
  oldpkg::old_way()
}

# Version 1.1.0:
# Imports: oldpkg
my_function <- function() {
  lifecycle::deprecate_soft(
    "1.1.0", "my_function()",
    details = "oldpkg will become optional in next version"
  )
  oldpkg::old_way()
}

# Version 2.0.0:
# Suggests: oldpkg (moved from Imports)
my_function <- function(use_old = FALSE) {
  if (use_old) {
    lifecycle::deprecate_warn(
      "2.0.0", "my_function(use_old)",
      details = "Old method will be removed in next version"
    )
    rlang::check_installed("oldpkg")
    return(oldpkg::old_way())
  }
  new_way()
}

# Version 3.0.0:
# (oldpkg removed entirely)
my_function <- function() {
  new_way()
}
```

## Common Pitfalls

### 1. Listing Package But Not Using It

**Problem**: Package in Imports but functions not accessible.

```r
# DESCRIPTION:
Imports: dplyr

# Code:
my_function <- function(data) {
  filter(data, x > 0)  # ERROR: object 'filter' not found
}

# Fix:
my_function <- function(data) {
  dplyr::filter(data, x > 0)
}
```

### 2. Using Suggests Without Checking

**Problem**: Assumes suggested package is installed.

```r
# DESCRIPTION:
Suggests: ggplot2

# WRONG:
my_plot <- function(data) {
  ggplot2::ggplot(data, ggplot2::aes(x, y))  # Fails if not installed
}

# RIGHT:
my_plot <- function(data) {
  rlang::check_installed("ggplot2")
  ggplot2::ggplot(data, ggplot2::aes(x, y))
}
```

### 3. Using Depends Unnecessarily

```r
# AVOID:
Depends: dplyr

# PREFER:
Imports: dplyr
# And use dplyr::filter() or @importFrom
```

### 4. Not Specifying Minimum Versions

```r
# RISKY:
Imports: dplyr

my_function <- function(data) {
  dplyr::filter(data, x > 0, .by = group)  # .by added in 1.1.0!
}

# SAFER:
Imports: dplyr (>= 1.1.0)
```

### 5. Importing Entire Packages

```r
# AVOID:
#' @import dplyr
#' @import tidyr
#' @import stringr

# PREFER:
#' @importFrom dplyr filter mutate select
#' @importFrom tidyr pivot_longer
#' @importFrom stringr str_detect
```

### 6. Missing skip_if_not_installed() in Tests

```r
# WRONG:
test_that("optional feature works", {
  library(ggplot2)  # Fails if not installed
  # ...
})

# RIGHT:
test_that("optional feature works", {
  skip_if_not_installed("ggplot2")
  library(ggplot2)
  # ...
})
```

### 7. Hard-Coding Package Checks

```r
# CLUNKY:
if (!requireNamespace("pkg", quietly = TRUE)) {
  stop("Please install pkg")
}

# BETTER:
rlang::check_installed(
  "pkg",
  reason = "to use this feature"
)
# Provides helpful install message
```

### 8. Circular Dependencies

```r
# Package A depends on Package B
# Package B depends on Package A
# = CIRCULAR DEPENDENCY (not allowed!)

# Solution: Extract shared code to third package C
# Both A and B depend on C
```

### 9. Not Using Config/Needs/*

```r
# WRONG - puts dev tools in Suggests:
Suggests:
    testthat,
    devtools,
    usethis,
    pkgdown,
    covr

# RIGHT - separates dev tools:
Suggests:
    testthat
Config/Needs/website:
    pkgdown
Config/Needs/development:
    devtools,
    usethis
Config/Needs/coverage:
    covr
```

### 10. Forgetting :: for Suggests

```r
# DESCRIPTION:
Suggests: ggplot2

# WRONG - even with requireNamespace():
if (requireNamespace("ggplot2", quietly = TRUE)) {
  ggplot(data)  # ERROR: ggplot not found
}

# RIGHT:
if (requireNamespace("ggplot2", quietly = TRUE)) {
  ggplot2::ggplot(data)  # OK: explicit namespace
}
```

## Quick Reference

### Dependency Checklist

```r
# 1. Add to DESCRIPTION:
usethis::use_package("dplyr")              # Imports
usethis::use_package("ggplot2", "Suggests") # Suggests

# 2. Use in code:
# Imports - either:
dplyr::filter(...)           # Explicit (recommended)
# Or:
#' @importFrom dplyr filter
filter(...)                  # Imported

# Suggests - always check:
rlang::check_installed("ggplot2")
ggplot2::ggplot(...)

# 3. In tests:
skip_if_not_installed("ggplot2")

# 4. In examples:
#' @examplesIf requireNamespace("ggplot2", quietly = TRUE)

# 5. Document minimum versions:
Imports: dplyr (>= 1.1.0)
```

### DESCRIPTION Template

```dcf
Package: mypackage
Version: 0.1.0
Depends:
    R (>= 4.1.0)
Imports:
    dplyr (>= 1.1.0),
    rlang (>= 1.0.0),
    tidyr (>= 1.3.0)
Suggests:
    ggplot2 (>= 3.4.0),
    testthat (>= 3.0.0),
    knitr,
    rmarkdown
Config/Needs/website:
    pkgdown
Config/Needs/development:
    devtools,
    usethis
```

### Common Import Patterns

```r
# Package-level imports (R/mypackage-package.R):
#' @importFrom rlang .data .env %||% !! !!!
#' @importFrom magrittr %>%
#' @importFrom dplyr filter mutate select arrange
"_PACKAGE"

# Or use base pipe (R >= 4.1):
# No import needed for |>
```

## Resources

- [R Packages: Dependencies](https://r-pkgs.org/dependencies-mindset-background.html)
- [Writing R Extensions: Package dependencies](https://cran.r-project.org/doc/manuals/R-exts.html#Package-dependencies)
- usethis package documentation
- rlang::check_installed() documentation
