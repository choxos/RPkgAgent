---
name: R Code Patterns for Packages
description: Critical rules and patterns for writing R code in packages, including forbidden functions, namespace management, and package state handling
---

# R Code Patterns for Packages

## Overview

Writing code for R packages differs significantly from writing R scripts. This skill covers the critical rules, forbidden functions, and best practices that prevent common package development errors.

## CRITICAL: Forbidden Functions in R/

These functions should **NEVER** appear in your package's R/ code:

### NEVER Use library() or require()

**Why**: These modify the user's search path and create dependencies on load order.

```r
# WRONG - DO NOT DO THIS:
my_function <- function(x) {
  library(dplyr)
  x %>% filter(value > 0)
}

# RIGHT - Use explicit namespace:
my_function <- function(x) {
  dplyr::filter(x, value > 0)
}

# ALSO RIGHT - Import in NAMESPACE via roxygen2:
#' @importFrom dplyr filter
my_function <- function(x) {
  filter(x, value > 0)
}
```

**Exception**: `require()` is acceptable for Suggests packages, but use `requireNamespace()` instead:

```r
# ACCEPTABLE but not ideal:
if (require("ggplot2")) {
  # code using ggplot2
}

# BETTER:
if (requireNamespace("ggplot2", quietly = TRUE)) {
  # code using ggplot2
}

# BEST - with helpful error message:
check_suggested <- function(pkg) {
  if (!requireNamespace(pkg, quietly = TRUE)) {
    stop(
      "Package '", pkg, "' required but not installed.\n",
      "Install with: install.packages('", pkg, "')",
      call. = FALSE
    )
  }
}
```

### NEVER Use source()

**Why**: Package code is loaded via the namespace mechanism, not sourcing.

```r
# WRONG:
source("R/utils.R")

# RIGHT:
# Just define functions in R/ files. They're automatically loaded.
# If you need execution order, use alphabetical naming or .onLoad()
```

### NEVER Use ::: (Triple Colon)

**Why**: Accessing internal functions from other packages is fragile and violates encapsulation.

```r
# WRONG:
result <- dplyr:::some_internal_function(x)

# RIGHT:
# Either:
# 1. The other package should export it
# 2. Copy the code (with attribution) if license permits
# 3. Use a different approach
# 4. Ask the maintainer to export it
```

**Exception**: Acceptable in tests to test your own internal functions:

```r
# In tests/testthat/test-internal.R:
test_that("internal function works", {
  expect_equal(mypackage:::.internal_function(5), 10)
})
```

## Namespace Management

### Default Pattern: pkg::fun()

For packages listed in `Imports:`, use explicit namespace references:

```r
#' @param x A data frame
my_function <- function(x) {
  # Explicit namespace - always works
  dplyr::mutate(x, new_col = value * 2)
}
```

**Advantages:**
- Clear where functions come from
- No NAMESPACE imports needed for simple usage
- Easier to understand code provenance

### Import Pattern: @importFrom

For heavily-used functions, import to your namespace:

```r
#' @importFrom dplyr filter mutate select
#' @importFrom rlang .data
my_function <- function(x) {
  x %>%
    filter(.data$value > 0) %>%
    mutate(doubled = .data$value * 2) %>%
    select(.data$id, .data$doubled)
}
```

**When to import:**
- Functions used in many places (reduces typing)
- Infix operators (%>%, %||%, :=)
- Functions from your package's core dependencies
- Performance-critical code (tiny speedup from avoiding :::)

### Full Import: @import (Rare)

Import an entire package namespace (almost never recommended):

```r
#' @import rlang
```

**Only use for:**
- rlang (if building a tidyverse-style package)
- Your own package's internal utilities package

**Why avoid:**
- Imports ALL exported functions (namespace pollution)
- Risk of function name conflicts
- Makes code provenance unclear
- Can break if imported package adds conflicting exports

## R Code Restrictions

### Use TRUE/FALSE, Never T/F

**Why**: T and F are variables that can be reassigned; TRUE and FALSE are reserved words.

```r
# WRONG:
if (x == T) {
  return(F)
}

# RIGHT:
if (x == TRUE) {
  return(FALSE)
}

# Why it matters:
T <- FALSE  # User can do this!
x == T      # Now broken
```

### ASCII-Only in .R Files

**Why**: R CMD check enforces ASCII in package R code for portability.

```r
# WRONG:
message("Processing data → results")
city <- "São Paulo"

# RIGHT:
message("Processing data -> results")  # ASCII arrow
city <- "S\u00e3o Paulo"              # Unicode escape

# For documentation (roxygen2 comments), UTF-8 is fine:
#' @param city City name (e.g., "São Paulo")
```

**Handling Unicode:**

```r
# Use Unicode escapes:
"\u00e9"     # é
"\u2192"     # →
"\u2264"     # ≤

# Or use iconv():
iconv("São Paulo", to = "ASCII//TRANSLIT")
```

### Global State and Options

**CRITICAL**: Never modify global state without restoration.

```r
# WRONG - leaves user's session modified:
my_function <- function() {
  options(stringsAsFactors = FALSE)
  # ... code ...
}

# RIGHT - restore on exit:
my_function <- function() {
  old_opts <- options(stringsAsFactors = FALSE)
  on.exit(options(old_opts), add = TRUE)
  # ... code ...
}

# BETTER - use withr:
my_function <- function() {
  withr::local_options(stringsAsFactors = FALSE)
  # ... code ...
}
```

**Common global state to be careful with:**
- `options()`
- `par()` (graphics parameters)
- Working directory (`setwd()`)
- Random seed (`set.seed()`)
- Environment variables

```r
# Always restore:
plot_something <- function() {
  old_par <- par(mfrow = c(2, 2))
  on.exit(par(old_par), add = TRUE)
  # ... plotting code ...
}

# Or use withr:
plot_something <- function() {
  withr::local_par(mfrow = c(2, 2))
  # ... plotting code ...
}
```

## Build-Time vs Load-Time Code

### Understanding Execution Timing

**CRITICAL**: Top-level code in R/ files executes during `R CMD build`, not when users load your package!

```r
# This runs at BUILD time (on your machine):
message("Building package")  # Users never see this!
current_year <- as.integer(format(Sys.Date(), "%Y"))  # Frozen at build time!

# This runs at LOAD time (on user's machine):
.onLoad <- function(libname, pkgname) {
  message("Loading package")  # Users see this
}

my_function <- function() {
  year <- as.integer(format(Sys.Date(), "%Y"))  # Computed each call
  # ...
}
```

### What Belongs Where

```r
# TOP-LEVEL (Build time):
# - Function definitions (always)
# - Package constants
# - Nothing that queries the system!

# Examples:
PACKAGE_VERSION <- "1.0.0"  # OK - constant
API_ENDPOINT <- "https://api.example.com"  # OK - constant

# WRONG - these run at build time:
USER_HOME <- Sys.getenv("HOME")  # Gets YOUR home, not user's!
N_CORES <- parallel::detectCores()  # Gets YOUR cores!
IS_WINDOWS <- .Platform$OS.type == "windows"  # Gets YOUR OS!

# RIGHT - query at runtime:
get_user_home <- function() {
  Sys.getenv("HOME")
}

get_n_cores <- function() {
  parallel::detectCores()
}

is_windows <- function() {
  .Platform$OS.type == "windows"
}
```

## Package State Management

### Using Environments for State

Packages should not use global variables. Use environments instead:

```r
# Create package environment (at build time):
pkg_env <- new.env(parent = emptyenv())

# Initialize state (at load time):
.onLoad <- function(libname, pkgname) {
  pkg_env$cache <- list()
  pkg_env$config <- list(
    api_key = NULL,
    verbose = FALSE
  )
}

# Access in functions:
get_config <- function(key) {
  pkg_env$config[[key]]
}

set_config <- function(key, value) {
  pkg_env$config[[key]] <- value
  invisible(value)
}

cache_get <- function(key) {
  pkg_env$cache[[key]]
}

cache_set <- function(key, value) {
  pkg_env$cache[[key]] <- value
  invisible(value)
}
```

**Why `parent = emptyenv()`:**
- Prevents accidental variable lookup in global environment
- Keeps package environment isolated
- Makes dependencies explicit

### .onLoad() and .onAttach()

Place these in `R/zzz.R` (alphabetically last, so other code is already sourced):

```r
# R/zzz.R

.onLoad <- function(libname, pkgname) {
  # Run when namespace is loaded
  # Setup package state, register S3 methods, etc.
  # NO messages to user here!

  # Initialize package environment:
  pkg_env$cache <- new.env(parent = emptyenv())

  # Register S3 methods dynamically:
  # s3_register("dplyr::dplyr_reconstruct", "my_class")

  # Set options (with package prefix):
  op <- options()
  op_mypackage <- list(
    mypackage.verbose = FALSE,
    mypackage.api_url = "https://api.example.com"
  )
  toset <- !(names(op_mypackage) %in% names(op))
  if (any(toset)) options(op_mypackage[toset])

  invisible()
}

.onAttach <- function(libname, pkgname) {
  # Run when package is attached (library() call)
  # OK to show startup messages here

  packageStartupMessage(
    "Welcome to mypackage version ",
    utils::packageVersion("mypackage")
  )

  # Check for API key:
  if (is.null(get_api_key())) {
    packageStartupMessage(
      "No API key found. Set one with set_api_key()"
    )
  }

  invisible()
}

.onUnload <- function(libpath) {
  # Cleanup when package is unloaded
  # Close connections, free resources, etc.
  library.dynam.unload("mypackage", libpath)
}
```

**Key differences:**
- `.onLoad()`: Always runs (even with `loadNamespace()`), no messages
- `.onAttach()`: Only with `library()`, messages OK
- Use `.onLoad()` for setup, `.onAttach()` for user communication

## File Organization

### Anti-Pattern: One Function Per File

**Problem**: Creates too many small files, hard to navigate.

```
# AVOID (unless functions are very large):
R/
├── add.R
├── subtract.R
├── multiply.R
├── divide.R
├── mean.R
├── median.R
└── mode.R
```

### Anti-Pattern: All Code in One File

**Problem**: Creates one huge file, hard to maintain.

```
# AVOID:
R/
└── functions.R  # 5000 lines!
```

### Good Pattern: Logical Grouping

```
R/
├── mypackage-package.R  # Package docs and imports
├── arithmetic.R         # add(), subtract(), multiply(), divide()
├── statistics.R         # mean(), median(), mode(), sd()
├── data.R              # Data documentation
├── utils.R             # Internal utilities
└── zzz.R              # .onLoad, .onAttach
```

**Grouping strategies:**
- By feature/domain
- By class (if using S3/R6)
- By workflow step
- utils.R for shared internal functions

### File Naming Conventions

```
R/
├── aaa-imports.R         # Loaded first (package-level imports)
├── class-model.R         # Class definition
├── model-methods.R       # Methods for model class
├── model-utils.R         # Utilities for model class
├── generics.R            # Generic function definitions
├── import-standalone-*.R # Standalone utilities
├── utils.R               # General utilities
├── utils-assertions.R    # Assertion utilities
├── data.R                # Data documentation
├── mypackage-package.R   # Package documentation
└── zzz.R                 # .onLoad, .onAttach (loaded last)
```

## Handling Non-Standard Evaluation

### data.table and dplyr Column References

**Problem**: R CMD check warns about "no visible binding for global variable".

```r
# This triggers warnings:
my_function <- function(data) {
  data %>%
    dplyr::filter(value > 0) %>%
    dplyr::mutate(doubled = value * 2)
}
# Note: value, doubled are column names, not variables
```

**Solution 1: Use .data pronoun (tidyverse)**

```r
#' @importFrom rlang .data
my_function <- function(data) {
  data %>%
    dplyr::filter(.data$value > 0) %>%
    dplyr::mutate(doubled = .data$value * 2)
}
```

**Solution 2: Use globalVariables()**

```r
# In R/globals.R or top of file:
utils::globalVariables(c("value", "doubled"))

my_function <- function(data) {
  data %>%
    dplyr::filter(value > 0) %>%
    dplyr::mutate(doubled = value * 2)
}
```

**Solution 3: Use .SD (data.table)**

```r
my_function <- function(dt) {
  # Instead of:
  # dt[, .(doubled = value * 2)]

  # Use .SD:
  dt[, .(doubled = .SD$value * 2)]
}
```

### Declaring Global Variables

```r
# R/globals.R
utils::globalVariables(c(
  # data.table/dplyr column references:
  "value", "doubled", "id", "name",
  # Other NSE variables:
  ".",  # data.table's . placeholder
  "where"  # tidyselect
))
```

## Performance Considerations

### Vectorization

```r
# SLOW:
sum_slow <- function(x) {
  total <- 0
  for (i in seq_along(x)) {
    total <- total + x[i]
  }
  total
}

# FAST:
sum_fast <- function(x) {
  sum(x)
}
```

### Pre-allocation

```r
# SLOW - grows vector:
result <- c()
for (i in 1:10000) {
  result <- c(result, i^2)
}

# FAST - pre-allocated:
result <- numeric(10000)
for (i in 1:10000) {
  result[i] <- i^2
}

# FASTEST - vectorized:
result <- (1:10000)^2
```

### Avoid Repeated Function Calls

```r
# SLOW:
for (i in 1:length(x)) {  # length() called each iteration
  # ...
}

# FAST:
n <- length(x)
for (i in 1:n) {
  # ...
}

# BETTER - seq_along handles empty vectors:
for (i in seq_along(x)) {
  # ...
}
```

## Common Pitfalls

### 1. Using library() in Package Code

**Problem**: Most common mistake by newcomers.

```r
# WRONG:
my_plot <- function(data) {
  library(ggplot2)  # NO!
  ggplot(data, aes(x, y)) + geom_point()
}

# RIGHT:
#' @import ggplot2
my_plot <- function(data) {
  ggplot(data, aes(x, y)) + geom_point()
}
```

### 2. Forgetting to Restore Options

**Problem**: User's session left in modified state.

```r
# WRONG:
my_function <- function() {
  options(warn = -1)
  # risky operation
}

# RIGHT:
my_function <- function() {
  withr::local_options(warn = -1)
  # risky operation
}
```

### 3. System Queries at Top Level

**Problem**: Captures build-time values, not runtime.

```r
# WRONG:
N_CORES <- parallel::detectCores()  # Build time!

my_parallel <- function() {
  cl <- parallel::makeCluster(N_CORES)  # User gets YOUR core count!
  # ...
}

# RIGHT:
my_parallel <- function(n_cores = parallel::detectCores()) {
  cl <- parallel::makeCluster(n_cores)
  # ...
}
```

### 4. Using T/F Instead of TRUE/FALSE

```r
# WRONG:
if (verbose == T) {  # Can break!
  message("Details...")
}

# RIGHT:
if (verbose == TRUE) {
  message("Details...")
}

# BETTER - isTRUE() is safer:
if (isTRUE(verbose)) {
  message("Details...")
}
```

### 5. Not Using parent = emptyenv()

**Problem**: Package environment can accidentally access globals.

```r
# WRONG:
cache <- new.env()  # parent = parent.frame()

# RIGHT:
cache <- new.env(parent = emptyenv())
```

### 6. Direct UTF-8 in Code

**Problem**: R CMD check fails on ASCII-enforcing systems.

```r
# WRONG:
cities <- c("São Paulo", "Zürich")

# RIGHT:
cities <- c("S\u00e3o Paulo", "Z\u00fcrich")
```

### 7. Using ::: for Other Packages

```r
# WRONG:
result <- otherpkg:::internal_function(x)

# RIGHT:
# Contact maintainer to export it, or find alternative
```

### 8. Namespace Pollution with @import

```r
# WRONG (unless rlang):
#' @import dplyr
#' @import tidyr
#' @import stringr

# RIGHT:
#' @importFrom dplyr filter mutate select
#' @importFrom tidyr pivot_longer
#' @importFrom stringr str_detect str_replace
```

### 9. Missing globalVariables() Declaration

**Problem**: R CMD check warnings about NSE variables.

```r
# Triggers warnings:
summarize_data <- function(df) {
  df %>%
    group_by(category) %>%
    summarize(mean_value = mean(value))
}

# Fix:
utils::globalVariables(c("category", "value", "mean_value"))
```

### 10. Using source() for Code Organization

```r
# WRONG:
# In R/main.R:
source("R/utils.R")

# RIGHT:
# Just define functions in both files
# R/ files are all loaded automatically
```

## Quick Reference

### Package Code Checklist

```r
# ✓ DO:
- Use pkg::fun() for Imports
- Use requireNamespace() for Suggests
- Use TRUE/FALSE (never T/F)
- Use ASCII in .R files (\uXXXX for Unicode)
- Restore global state with on.exit() or withr
- Use new.env(parent = emptyenv()) for state
- Put .onLoad() in R/zzz.R
- Use utils::globalVariables() for NSE

# ✗ DON'T:
- library() or require() (except requireNamespace() for Suggests)
- source()
- ::: to access other packages' internals
- T/F instead of TRUE/FALSE
- UTF-8 directly in .R files
- Modify options/par without restoration
- System queries at top level
- One function per file (usually)
- All code in one file
```

### Template: R/zzz.R

```r
# Package environment
pkg_env <- new.env(parent = emptyenv())

.onLoad <- function(libname, pkgname) {
  # Initialize package state
  pkg_env$cache <- new.env(parent = emptyenv())

  # Set package options
  op <- options()
  op_mypackage <- list(
    mypackage.verbose = FALSE
  )
  toset <- !(names(op_mypackage) %in% names(op))
  if (any(toset)) options(op_mypackage[toset])

  invisible()
}

.onAttach <- function(libname, pkgname) {
  packageStartupMessage("Welcome to mypackage")
  invisible()
}
```

## Resources

- [R Packages: R code](https://r-pkgs.org/code.html)
- [Advanced R: Environments](https://adv-r.hadley.nz/environments.html)
- [Writing R Extensions: Package namespaces](https://cran.r-project.org/doc/manuals/R-exts.html#Package-namespaces)
