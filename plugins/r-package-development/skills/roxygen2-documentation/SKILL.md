---
name: roxygen2 Documentation
description: Complete guide to roxygen2 tags, Markdown support, cross-references, and documentation patterns for functions, data, and packages
---

# roxygen2 Documentation

## Overview

roxygen2 allows you to write documentation directly above your R code using special comments starting with `#'`. This skill covers all essential roxygen2 tags, Markdown formatting, and documentation patterns.

## Core Concepts

### Basic Structure

```r
#' Function title (one line, sentence case)
#'
#' Longer description paragraph providing more details about what
#' the function does, when to use it, and any important context.
#'
#' @param x Description of parameter x
#' @param y Description of parameter y
#' @returns Description of return value
#' @export
#' @examples
#' my_function(1, 2)
my_function <- function(x, y) {
  # implementation
}
```

**Required components:**
1. Title (first line, no blank line before it)
2. Blank line (separates title from description)
3. Description (optional but recommended)
4. Blank line (separates description from tags)
5. Tags (@param, @returns, @export, etc.)

### Generating Documentation

```r
# Generate .Rd files and NAMESPACE:
devtools::document()

# Or:
roxygen2::roxygenise()

# Preview documentation:
?my_function
```

## Essential Tags

### @param

Documents function parameters. One @param per parameter.

```r
#' @param x A numeric vector of values
#' @param y A numeric vector, same length as x
#' @param method Character string specifying the method to use.
#'   Options are "mean" (default), "median", or "mode".
#' @param verbose Logical. If TRUE, prints progress messages.
#'   Default is FALSE.
#' @param ... Additional arguments passed to underlying functions
```

**Patterns:**
- Start with type description ("A numeric vector", "Character string")
- Explain constraints (length, range, allowed values)
- Document defaults if not obvious
- Use `...` for dots

### @returns (Preferred) / @return

Documents what the function returns. Use `@returns` (plural) in new code.

```r
#' @returns A numeric vector of the same length as the input
my_function <- function(x) {
  x * 2
}

#' @returns A data frame with columns:
#'   \describe{
#'     \item{id}{Character. Unique identifier}
#'     \item{value}{Numeric. Measured value}
#'     \item{timestamp}{POSIXct. Time of measurement}
#'   }
my_summarize <- function(data) {
  # ...
}

#' @returns Invisible NULL. Called for side effects (prints plot).
my_plot <- function(x) {
  plot(x)
  invisible(NULL)
}

#' @returns A list with components:
#'   * `estimate`: Numeric vector of estimates
#'   * `std_error`: Standard errors
#'   * `p_value`: P-values
my_test <- function(x, y) {
  list(
    estimate = c(1, 2),
    std_error = c(0.1, 0.2),
    p_value = c(0.01, 0.05)
  )
}
```

**Note**: `@return` (singular) is older but still works. `@returns` is now preferred.

### @export

Marks function for export (makes it available to users).

```r
#' User-facing function
#' @export
public_function <- function() {
  # ...
}

#' Internal function (no @export)
internal_helper <- function() {
  # ...
}
```

**When to export:**
- Functions users should call directly
- S3 methods for generics from other packages (often)
- Datasets (automatically exported from data/)

**When NOT to export:**
- Internal helper functions
- Functions only called by other package functions
- S3 methods for your own generics (use @export with method)

### @examples

Provides executable examples.

```r
#' @examples
#' # Basic usage
#' result <- my_function(1:10)
#'
#' # With different parameters
#' result2 <- my_function(1:10, method = "median")
#'
#' # Handling edge cases
#' my_function(numeric(0))  # Empty input
my_function <- function(x, method = "mean") {
  # ...
}
```

**Best practices:**
- Include comments explaining what each example shows
- Show basic usage first, then advanced
- Demonstrate important edge cases
- Keep examples short and focused
- Examples are run by R CMD check!

### @examplesIf

Conditionally run examples (new in roxygen2 7.2.0).

```r
#' @examplesIf requireNamespace("ggplot2", quietly = TRUE)
#' library(ggplot2)
#' ggplot(data, aes(x, y)) + geom_point()

#' @examplesIf interactive()
#' # Only run in interactive sessions
#' my_interactive_function()

#' @examplesIf identical(Sys.getenv("NOT_CRAN"), "true")
#' # Only on GitHub Actions, not CRAN
#' slow_test()
```

### \dontrun{} vs \donttest{}

Control example execution during R CMD check.

```r
#' @examples
#' # This runs:
#' my_function(1:10)
#'
#' \dontrun{
#' # This is shown but NEVER run (not even by check):
#' # Use for code that requires user interaction or credentials
#' my_function(get_user_input())
#' set_api_key("your-key-here")
#' }
#'
#' \donttest{
#' # This is run by check, but not by example():
#' # Use for slow operations
#' very_slow_operation()
#' }
```

**Guidelines:**
- Prefer runnable examples when possible
- Use `\dontrun{}` for interactive code, credentials, web APIs
- Use `\donttest{}` for slow operations (>5 seconds)
- Use `@examplesIf` instead when appropriate

### @importFrom

Imports specific functions from other packages into your namespace.

```r
#' @importFrom dplyr filter mutate select
#' @importFrom rlang .data %||%
#' @importFrom stats median sd
NULL
```

**Placement:**
- Package-level documentation file (R/mypackage-package.R)
- Or in individual function documentation
- Or in a dedicated R/aaa-imports.R file

```r
# R/aaa-imports.R
#' @importFrom dplyr filter mutate select arrange
#' @importFrom rlang .data .env %||% !! !!!
#' @importFrom magrittr %>%
NULL
```

### @import

Imports entire package namespace (rarely recommended).

```r
#' @import rlang
NULL
```

**Only use for:**
- rlang (if building tidyverse-style package)
- Your own internal utilities package

**Avoid for:**
- Most packages (use @importFrom or pkg::fun() instead)
- Risk of namespace conflicts

### @keywords internal

Marks function as internal (hides from package index).

```r
#' Internal helper function
#'
#' This function is not intended for direct use.
#'
#' @keywords internal
#' @export
internal_but_exported <- function() {
  # Sometimes needed for S3 methods, package architecture
}
```

**Use when:**
- Function must be exported (e.g., S3 method) but isn't user-facing
- Internal APIs for advanced users

## Inheritance and Linking

### @inheritParams

Inherits parameter documentation from another function.

```r
#' Base function
#' @param x A numeric vector
#' @param y A numeric vector
#' @param method Method to use: "mean", "median", or "mode"
base_function <- function(x, y, method = "mean") {
  # ...
}

#' Wrapper function
#' @inheritParams base_function
#' @param z Additional parameter
wrapper_function <- function(x, y, method = "mean", z) {
  base_function(x, y, method)
  # ... use z ...
}
```

**Benefits:**
- Reduces duplication
- Ensures consistency
- Easier maintenance

### @inherit

Inherits various documentation sections.

```r
#' Original function
#' @param x Input
#' @returns Output
#' @examples
#' original(1:10)
original <- function(x) {
  # ...
}

#' @inherit original examples
#' @inheritParams original
#' @returns Modified output
modified <- function(x) {
  # different implementation
}
```

**Can inherit:**
- `@inherit fun params` - all parameters
- `@inherit fun return` - return value
- `@inherit fun examples` - examples
- `@inherit fun description` - description
- `@inherit fun details` - details
- `@inherit fun` - everything

### @family

Creates "See Also" links to related functions.

```r
#' @family statistical functions
mean_wrapper <- function(x) {
  mean(x)
}

#' @family statistical functions
median_wrapper <- function(x) {
  median(x)
}

#' @family statistical functions
mode_wrapper <- function(x) {
  # ...
}
```

Generates links: "Other statistical functions: median_wrapper(), mode_wrapper()"

### @seealso

Manual cross-references and external links.

```r
#' @seealso [related_function()] for related functionality
#' @seealso [other_package::their_function()] for alternative approach
#' @seealso <https://example.com> for more details
#' @seealso `vignette("introduction")` for tutorial
```

### @rdname

Combines documentation of multiple functions on one page.

```r
#' Get or set configuration
#'
#' @param key Configuration key
#' @param value Configuration value
#' @rdname config
#' @export
get_config <- function(key) {
  pkg_env$config[[key]]
}

#' @rdname config
#' @export
set_config <- function(key, value) {
  pkg_env$config[[key]] <- value
  invisible(value)
}
```

**Generates single help page:** `?config` shows both functions.

**Use for:**
- Getter/setter pairs
- Function families that are always used together
- Different interfaces to same functionality

### @describeIn

Documents multiple related items with individual descriptions.

```r
#' Process data
#'
#' @param x Input data
#' @export
process <- function(x) {
  UseMethod("process")
}

#' @describeIn process Process numeric vectors
#' @export
process.numeric <- function(x) {
  # ...
}

#' @describeIn process Process character vectors
#' @export
process.character <- function(x) {
  # ...
}

#' @describeIn process Process data frames
#' @export
process.data.frame <- function(x) {
  # ...
}
```

**Generates:** Single page with subsections for each method.

## Markdown Support

Enable in DESCRIPTION:
```dcf
Roxygen: list(markdown = TRUE)
```

### Text Formatting

```r
#' Function with *italic*, **bold**, and `code` text
#'
#' You can use _italic_ and __bold__ alternatives too.
#'
#' Inline code: `x + y`
#'
#' Code blocks:
#' ```
#' result <- my_function(
#'   x = 1:10,
#'   method = "mean"
#' )
#' ```
```

### Lists

```r
#' Bulleted list:
#' * Item 1
#' * Item 2
#'   * Nested item
#' * Item 3
#'
#' Numbered list:
#' 1. First step
#' 2. Second step
#' 3. Third step
#'
#' Definition list:
#' * `term1`: Definition of term 1
#' * `term2`: Definition of term 2
```

### Links

```r
#' See [my_function()] for details
#'
#' See [other_package::their_function()] for alternatives
#'
#' See [my_function()] and [another_function()] for related work
#'
#' See the vignette: `vignette("introduction", package = "mypackage")`
#'
#' External link: <https://example.com>
#' Or: [Link text](https://example.com)
```

**Auto-linking:**
- `[function_name()]` - links to function in same package
- `[pkg::function_name()]` - links to function in other package
- Backticks alone don't create links: `function_name`

### Sections

```r
#' # Major section
#'
#' Content here
#'
#' ## Subsection
#'
#' More content
#'
#' ### Sub-subsection
#'
#' Even more content
```

## Data Documentation

Document datasets in `R/data.R`:

```r
#' World Health Organization TB data
#'
#' A subset of data from the World Health Organization Global Tuberculosis
#' Report containing tuberculosis cases by country, year, age, and sex.
#'
#' @format A data frame with 7,240 rows and 60 columns:
#' \describe{
#'   \item{country}{Character. Country name}
#'   \item{iso2}{Character. 2-letter ISO country code}
#'   \item{iso3}{Character. 3-letter ISO country code}
#'   \item{year}{Integer. Year of observation (1995-2013)}
#'   \item{new_sp_m014}{Integer. New smear-positive cases, males aged 0-14}
#'   \item{new_sp_f014}{Integer. New smear-positive cases, females aged 0-14}
#' }
#'
#' @source World Health Organization Global Tuberculosis Report
#'   \url{https://www.who.int/teams/global-tuberculosis-programme/data}
#'
#' @examples
#' head(who)
#' summary(who$year)
"who"
```

**Key tags for data:**
- `@format` - describe structure
- `\describe{}` and `\item{}` - describe columns/elements
- `@source` - where data came from
- Quote the dataset name: `"dataset_name"`

### Different Data Types

```r
#' Example vector
#'
#' @format A numeric vector of length 100
"example_vector"

#' Example list
#'
#' @format A list with components:
#' \describe{
#'   \item{x}{Numeric vector}
#'   \item{y}{Character vector}
#'   \item{metadata}{List of metadata}
#' }
"example_list"

#' Example matrix
#'
#' @format A 10x10 numeric matrix
"example_matrix"
```

## Package-Level Documentation

Use `"_PACKAGE"` for package-level documentation:

```r
# R/mypackage-package.R

#' mypackage: Brief Package Description
#'
#' A longer description of what the package does. This should be
#' one paragraph providing enough detail for users to understand
#' the package's purpose and scope.
#'
#' @section Main functions:
#' * [important_function()] - does important things
#' * [another_function()] - does other things
#' * [helper_function()] - helps with things
#'
#' @section Getting started:
#' See `vignette("introduction")` for a tutorial.
#'
#' @keywords internal
#' @importFrom rlang .data %||%
#' @importFrom dplyr filter mutate select
"_PACKAGE"
```

**Generates:** `?mypackage` help page.

**Common pattern:** Put package-level `@importFrom` here.

## Advanced Patterns

### S3 Method Documentation

```r
#' Print method for myclass objects
#'
#' @param x A myclass object
#' @param ... Additional arguments (ignored)
#' @returns Invisible x (called for side effects)
#' @export
print.myclass <- function(x, ...) {
  cat("myclass object with", length(x), "elements\n")
  invisible(x)
}

#' Summary method for myclass objects
#'
#' @inheritParams print.myclass
#' @returns A summary_myclass object
#' @export
summary.myclass <- function(x, ...) {
  structure(
    list(n = length(x), mean = mean(x)),
    class = "summary_myclass"
  )
}
```

### Documenting Multiple Related Functions

```r
#' Arithmetic operations
#'
#' Basic arithmetic operations for custom objects.
#'
#' @param x,y Numeric vectors
#' @name arithmetic
#' @returns Numeric vector
NULL

#' @rdname arithmetic
#' @export
add <- function(x, y) x + y

#' @rdname arithmetic
#' @export
subtract <- function(x, y) x - y

#' @rdname arithmetic
#' @export
multiply <- function(x, y) x * y

#' @rdname arithmetic
#' @export
divide <- function(x, y) x / y
```

### Templates with @template

Create reusable documentation chunks:

```r
# man-roxygen/common-params.R (not R/ directory!)
#' @param verbose Logical. If TRUE, prints progress messages.
#' @param ... Additional arguments passed to underlying functions.

# Then in function documentation:
#' My function
#' @template common-params
#' @export
my_function <- function(verbose = FALSE, ...) {
  # ...
}
```

**Setup:**
1. Create `man-roxygen/` directory
2. Put `.R` files with roxygen blocks
3. Use `@template filename` (without `.R`)

## Complete Templates

### Exported Function

```r
#' Brief title (one line, sentence case)
#'
#' Longer description providing context and details about when to use
#' this function and what it does.
#'
#' @param x A numeric vector. Input values to process.
#' @param method Character string specifying method. Options:
#'   * "mean" (default) - arithmetic mean
#'   * "median" - median value
#'   * "mode" - most common value
#' @param na.rm Logical. Should missing values be removed? Default is FALSE.
#' @param verbose Logical. Print progress messages? Default is FALSE.
#' @param ... Additional arguments passed to underlying functions.
#'
#' @returns A numeric vector of the same length as `x`.
#'
#' @export
#' @examples
#' # Basic usage
#' my_function(1:10)
#'
#' # With different method
#' my_function(1:10, method = "median")
#'
#' # Handling missing values
#' my_function(c(1, NA, 3), na.rm = TRUE)
#'
#' @seealso [related_function()] for related functionality
#' @family processing functions
my_function <- function(x, method = "mean", na.rm = FALSE,
                        verbose = FALSE, ...) {
  # implementation
}
```

### Internal Function

```r
#' Internal helper function
#'
#' This function is not intended for direct use by package users.
#' It provides internal utilities for other package functions.
#'
#' @param x Input data
#' @returns Processed data
#' @keywords internal
#' @noRd
internal_helper <- function(x) {
  # implementation
}
```

**Note**: `@noRd` means "don't create .Rd file" - use for purely internal functions.

### Dataset

```r
#' Example dataset for demonstrations
#'
#' A synthetic dataset containing measurements of various properties
#' across different conditions. Useful for testing and examples.
#'
#' @format A data frame with 150 rows and 5 columns:
#' \describe{
#'   \item{id}{Character. Unique identifier for each observation}
#'   \item{value}{Numeric. Measured value (range: 0-100)}
#'   \item{category}{Factor. Category label (A, B, or C)}
#'   \item{timestamp}{POSIXct. Time of measurement}
#'   \item{valid}{Logical. Whether measurement passed quality checks}
#' }
#'
#' @source Generated synthetically for package examples
#'
#' @examples
#' head(example_data)
#' summary(example_data)
#'
#' # Basic analysis
#' mean(example_data$value)
#' table(example_data$category)
"example_data"
```

### Package Documentation

```r
#' mypackage: Tools for Data Analysis
#'
#' mypackage provides a set of tools for common data analysis tasks,
#' including data cleaning, transformation, and visualization. It is
#' designed to work seamlessly with the tidyverse ecosystem.
#'
#' @section Main functions:
#' Data processing:
#' * [clean_data()] - clean and validate data
#' * [transform_data()] - apply transformations
#'
#' Analysis:
#' * [analyze_data()] - perform statistical analysis
#' * [summarize_results()] - summarize analysis results
#'
#' Visualization:
#' * [plot_data()] - create data visualizations
#'
#' @section Getting started:
#' See `vignette("introduction")` for a comprehensive tutorial.
#'
#' @keywords internal
#' @importFrom rlang .data .env %||% !! !!!
#' @importFrom dplyr filter mutate select arrange
#' @import ggplot2
"_PACKAGE"
```

## Common Pitfalls

### 1. Missing Blank Line After Title

```r
# WRONG:
#' Function title
#' More description
#' @param x Input

# RIGHT:
#' Function title
#'
#' More description
#'
#' @param x Input
```

### 2. Using @return Instead of @returns

Not wrong, but `@returns` is now preferred:

```r
# OLD:
#' @return A numeric vector

# NEW:
#' @returns A numeric vector
```

### 3. Forgetting @export

```r
# Function is internal (not exported):
#' Public-facing function
my_function <- function() {}

# Need to export:
#' Public-facing function
#' @export
my_function <- function() {}
```

### 4. Not Documenting All Parameters

R CMD check will warn if parameters lack documentation:

```r
# WARNING - y not documented:
#' @param x Input
my_function <- function(x, y) {}

# CORRECT:
#' @param x Input
#' @param y Other input
my_function <- function(x, y) {}
```

### 5. Broken Links

```r
# WRONG - no parentheses, no link created:
#' See my_function for details

# RIGHT:
#' See [my_function()] for details
```

### 6. Forgetting @format for Data

```r
# INCOMPLETE:
#' My dataset
"dataset"

# COMPLETE:
#' My dataset
#' @format A data frame with 100 rows and 5 columns
"dataset"
```

### 7. Using \\ Escapes Unnecessarily

With Markdown enabled, use Markdown syntax:

```r
# OLD (without Markdown):
#' \code{x} is a vector

# NEW (with Markdown):
#' `x` is a vector
```

### 8. Examples That Don't Run

```r
# WRONG - example will fail:
#' @examples
#' my_function(nonexistent_object)

# RIGHT:
#' @examples
#' my_function(1:10)
```

### 9. Missing @keywords internal for Internal Exports

```r
# Exported but internal (e.g., S3 method):
#' @export
#' @keywords internal
internal_but_exported <- function() {}
```

### 10. Not Running devtools::document()

```r
# After changing roxygen comments:
devtools::document()

# Or:
roxygen2::roxygenise()
```

## Quick Reference

### Essential Tags

```r
#' @param name Description
#' @returns What the function returns
#' @export
#' @examples code
#' @importFrom pkg fun
#' @import pkg
#' @keywords internal
#' @seealso [other_function()]
#' @family function_family
#' @inheritParams other_function
#' @inherit other_function return examples
#' @rdname combined_topic
#' @describeIn generic_function Method description
```

### Markdown Quick Reference

```r
#' *italic* or _italic_
#' **bold** or __bold__
#' `code`
#' [function_name()] - auto-link
#' [pkg::function()] - cross-package link
#' <https://example.com> - URL
#' * Bullet point
#' 1. Numbered item
```

## Resources

- [roxygen2 documentation](https://roxygen2.r-lib.org/)
- [R Packages: Documentation](https://r-pkgs.org/man.html)
- [Writing R Extensions](https://cran.r-project.org/doc/manuals/R-exts.html#Writing-R-documentation-files)
