---
name: Testing Patterns with testthat
description: Comprehensive guide to testing R packages using testthat 3rd edition, including test structure, expectations, fixtures, and snapshot testing
---

# Testing Patterns with testthat

## Overview

Testing is essential for reliable R packages. This skill covers testthat 3rd edition, the standard testing framework for R packages, including test structure, expectations, fixtures, and advanced patterns.

## Setup

### Initial Setup

```r
# Setup testthat 3rd edition:
usethis::use_testthat(3)
```

This creates:
```
tests/
├── testthat/
│   └── (test files will go here)
└── testthat.R
```

And adds to DESCRIPTION:
```dcf
Suggests:
  testthat (>= 3.0.0)
Config/testthat/edition: 3
```

### Key Differences: Edition 3 vs 2

**Edition 3 changes:**
- `context()` deprecated (use file names)
- More informative error messages
- Better snapshot testing
- Improved parallel test support
- Stricter comparison defaults

```r
# OLD (Edition 2):
context("My feature tests")
expect_equal(x, y, tolerance = 1e-8)

# NEW (Edition 3):
# No context() - file name is context
expect_equal(x, y, tolerance = 1e-8)  # Same, but better errors
```

## Test File Structure

### Basic Structure

```r
# tests/testthat/test-my-feature.R

test_that("basic functionality works", {
  result <- my_function(1:10)
  expect_equal(result, (1:10) * 2)
  expect_length(result, 10)
})

test_that("handles edge cases", {
  expect_equal(my_function(numeric(0)), numeric(0))
  expect_error(my_function(NULL), class = "error")
})

test_that("parameter validation works", {
  expect_error(my_function("not numeric"), "must be numeric")
  expect_warning(my_function(c(1, NA)), "NA values detected")
})
```

### File Naming Convention

Mirror your R/ file structure:

```
R/
├── data-processing.R
├── visualization.R
└── utils.R

tests/testthat/
├── test-data-processing.R
├── test-visualization.R
└── test-utils.R
```

**Rules:**
- Files MUST start with `test-`
- Use descriptive names matching R/ files
- One test file per R source file (usually)
- Or organize by feature/functionality

### Test Organization Within Files

```r
# tests/testthat/test-statistics.R

# Group related tests
test_that("mean calculation is correct", {
  expect_equal(my_mean(1:10), 5.5)
  expect_equal(my_mean(c(1, 2, 3)), 2)
})

test_that("mean handles NA values", {
  expect_equal(my_mean(c(1, NA, 3), na.rm = TRUE), 2)
  expect_true(is.na(my_mean(c(1, NA, 3), na.rm = FALSE)))
})

test_that("mean validates input", {
  expect_error(my_mean("not numeric"))
  expect_error(my_mean(list(1, 2, 3)))
})

# Separate feature
test_that("median calculation is correct", {
  expect_equal(my_median(1:10), 5.5)
  expect_equal(my_median(1:11), 6)
})
```

## Core Expectations

### expect_equal()

Tests near equality (with tolerance for numerics).

```r
test_that("numeric equality works", {
  # Exact equality:
  expect_equal(1 + 1, 2)

  # With tolerance (default 1.5e-8):
  expect_equal(sqrt(2)^2, 2)

  # Custom tolerance:
  expect_equal(1.00001, 1, tolerance = 1e-4)

  # Vectors:
  expect_equal(1:5, c(1, 2, 3, 4, 5))

  # Data frames:
  expect_equal(
    data.frame(x = 1:3, y = 4:6),
    data.frame(x = 1:3, y = 4:6)
  )

  # Ignore attributes:
  expect_equal(
    c(a = 1, b = 2),
    c(1, 2),
    ignore_attr = TRUE
  )
})
```

### expect_identical()

Tests exact identity (no tolerance).

```r
test_that("exact identity works", {
  # Integers:
  expect_identical(1L, 1L)

  # But not with doubles:
  expect_failure(expect_identical(1, 1L))  # Different types

  # Attributes matter:
  expect_failure(
    expect_identical(
      c(a = 1, b = 2),
      c(1, 2)
    )
  )

  # Use for reference comparisons:
  x <- 1:10
  y <- x
  expect_identical(x, y)

  # Type checking:
  expect_identical(class(x), "integer")
})
```

### expect_error()

Tests that code throws an error.

```r
test_that("errors are thrown correctly", {
  # Basic error:
  expect_error(stop("oops"))

  # Error with specific message (regex):
  expect_error(
    stop("value must be numeric"),
    "must be numeric"
  )

  # Error with specific class (PREFERRED):
  expect_error(
    my_function(invalid_input),
    class = "invalid_input_error"
  )

  # Both class and message:
  expect_error(
    my_function(NULL),
    "cannot be NULL",
    class = "null_input_error"
  )
})
```

**Best practice:** Use custom error classes and test them:

```r
# In your package:
validate_input <- function(x) {
  if (!is.numeric(x)) {
    rlang::abort(
      "Input must be numeric",
      class = "invalid_input_error"
    )
  }
}

# In tests:
test_that("validation errors have correct class", {
  expect_error(
    validate_input("text"),
    class = "invalid_input_error"
  )
})
```

### expect_warning()

Tests that code produces warnings.

```r
test_that("warnings are issued correctly", {
  # Basic warning:
  expect_warning(warning("careful!"))

  # Warning with message:
  expect_warning(
    my_function(c(1, NA)),
    "NA values detected"
  )

  # Warning with class:
  expect_warning(
    my_function(x),
    class = "deprecated_argument"
  )
})
```

### expect_message()

Tests that code produces messages.

```r
test_that("messages are printed correctly", {
  expect_message(message("Processing..."))

  expect_message(
    my_function(verbose = TRUE),
    "Starting computation"
  )

  # Multiple messages:
  expect_message(
    expect_message(
      my_verbose_function(),
      "Step 1"
    ),
    "Step 2"
  )
})
```

### expect_no_error() / expect_no_warning() / expect_no_message()

Tests that code runs without conditions.

```r
test_that("clean execution", {
  # No errors:
  expect_no_error(my_function(valid_input))

  # No warnings:
  expect_no_warning(my_function(good_data))

  # No messages:
  expect_no_message(my_function(verbose = FALSE))
})
```

### Other Useful Expectations

```r
test_that("various expectations work", {
  # Truth values:
  expect_true(2 + 2 == 4)
  expect_false(2 + 2 == 5)

  # NULL:
  expect_null(NULL)
  expect_null(my_function_returning_null())

  # Type checks:
  expect_type(1:10, "integer")
  expect_type(letters, "character")

  # Class checks:
  expect_s3_class(lm(y ~ x, data), "lm")
  expect_s4_class(object, "myS4class")

  # Length/dimensions:
  expect_length(1:10, 10)
  expect_length(list(a = 1, b = 2), 2)

  # Named:
  expect_named(c(a = 1, b = 2), c("a", "b"))

  # Vector type and length:
  expect_vector(1:10, ptype = integer(), size = 10)

  # Matching:
  expect_match("hello world", "hello")
  expect_match("abc123", "\\d+")  # Regex

  # Set operations:
  expect_setequal(c(1, 2, 3), c(3, 2, 1))  # Order doesn't matter
  expect_contains(1:10, c(5, 7, 9))  # Subset

  # Invisible return:
  expect_invisible(invisible(42))

  # Output:
  expect_output(print("hello"), "hello")
})
```

## Snapshot Testing

Test output that's hard to describe with expectations.

### expect_snapshot()

Captures printed output, messages, warnings, and errors.

```r
test_that("function output is correct", {
  expect_snapshot({
    my_complex_function()
  })
})
```

First run creates `tests/testthat/_snaps/my-test.md`:
```md
# function output is correct

    Code
      my_complex_function()
    Output
      Processing data...
      Results:
        Mean: 5.5
        SD: 2.87
```

Subsequent runs compare against snapshot. Update with:
```r
testthat::snapshot_review()
# Or:
testthat::snapshot_accept()
```

### Snapshot Variants

```r
test_that("snapshots capture different outputs", {
  # Just messages:
  expect_snapshot(
    message("Hello"),
    cnd_class = TRUE  # Include condition class
  )

  # Errors (with class):
  expect_snapshot(
    error = TRUE,
    my_function(invalid)
  )

  # Transforming output:
  expect_snapshot(
    my_function(),
    transform = scrub_randomness
  )

  # Multiple variants:
  expect_snapshot({
    cat("Output 1\n")
    message("Message 1")
    cat("Output 2\n")
  })
})
```

### expect_snapshot_output()

Specifically for printed output (deprecated, use expect_snapshot()).

```r
test_that("print methods work", {
  expect_snapshot_output(print(my_object))
})
```

### expect_snapshot_value()

Captures R object structure.

```r
test_that("complex return values are correct", {
  result <- my_complex_function()

  expect_snapshot_value(
    result,
    style = "json2"  # or "serialize", "deparse"
  )
})
```

### When to Use Snapshots

**Good for:**
- Complex printed output (print methods, summaries)
- Error messages (ensures consistent UX)
- Multi-line formatted output
- Plots (as text representation)

**Avoid for:**
- Simple values (use expect_equal())
- When snapshots are hard to review
- When output changes frequently

## Test Helpers and Fixtures

### Helper Files

Files starting with `helper-` run before tests and make utilities available:

```r
# tests/testthat/helper-data.R

# Create test data used across multiple test files:
make_test_data <- function(n = 100) {
  data.frame(
    id = seq_len(n),
    value = rnorm(n),
    category = sample(LETTERS[1:3], n, replace = TRUE)
  )
}

# Create test fixtures:
sample_data <- make_test_data()

# Utility assertions:
expect_valid_output <- function(x) {
  expect_s3_class(x, "data.frame")
  expect_true(nrow(x) > 0)
  expect_named(x, c("id", "result"))
}
```

Use in tests:
```r
# tests/testthat/test-analysis.R
test_that("analysis works with test data", {
  result <- analyze(sample_data)
  expect_valid_output(result)
})
```

### Setup Files

`setup.R` runs once before all tests:

```r
# tests/testthat/setup.R

# Create temporary directory for test outputs:
test_dir <- tempfile("test_outputs_")
dir.create(test_dir)

# Register cleanup:
withr::defer(
  unlink(test_dir, recursive = TRUE),
  teardown_env()
)
```

### Teardown Files

`teardown.R` runs once after all tests:

```r
# tests/testthat/teardown.R

# Cleanup if needed (but prefer withr::defer)
```

### Using withr for Test Isolation

```r
test_that("tests are isolated", {
  # Temporary file:
  withr::local_file("temp.txt")
  writeLines("test", "temp.txt")
  # Automatically deleted after test

  # Temporary directory:
  withr::local_tempdir()  # Creates and returns path, deletes after

  # Options:
  withr::local_options(list(width = 120))
  # Restored after test

  # Environment variables:
  withr::local_envvar(list(MY_VAR = "test_value"))
  # Restored after test

  # Working directory:
  withr::local_dir(tempdir())
  # Restored after test

  # Random seed:
  withr::local_seed(123)
  # Seed state restored after test
})
```

## Conditional Tests

### skip_if_not_installed()

Skip if suggested package not available.

```r
test_that("integration with ggplot2 works", {
  skip_if_not_installed("ggplot2")

  library(ggplot2)
  plot <- my_plot_function(data)
  expect_s3_class(plot, "gg")
})
```

### skip_on_cran()

Skip slow or fragile tests on CRAN.

```r
test_that("slow integration test", {
  skip_on_cran()

  # Test that takes >1 second or requires internet:
  result <- very_slow_operation()
  expect_equal(result$status, "success")
})
```

### skip_on_ci() / skip_on_os()

```r
test_that("platform-specific test", {
  skip_on_ci()  # Skip on CI
  skip_on_os("windows")  # Skip on Windows
  skip_on_os("mac")  # Skip on macOS

  # Test requiring specific platform
})
```

### Custom Skip Conditions

```r
test_that("requires special environment", {
  skip_if(
    Sys.getenv("RUN_FULL_TESTS") != "true",
    "Skipping: RUN_FULL_TESTS not set"
  )

  # Comprehensive test
})

test_that("requires API access", {
  skip_if_offline()  # testthat built-in

  # Test requiring internet
})
```

## Testing Patterns

### Testing Errors and Validation

```r
# In your package (R/validate.R):
validate_input <- function(x) {
  if (!is.numeric(x)) {
    rlang::abort(
      "Input must be numeric",
      class = "invalid_type_error",
      x = x
    )
  }
  if (any(x < 0)) {
    rlang::abort(
      "Input must be non-negative",
      class = "invalid_value_error",
      x = x
    )
  }
  x
}

# In tests (tests/testthat/test-validate.R):
test_that("validation catches type errors", {
  expect_error(
    validate_input("text"),
    class = "invalid_type_error"
  )
  expect_error(
    validate_input(list(1, 2)),
    class = "invalid_type_error"
  )
})

test_that("validation catches value errors", {
  expect_error(
    validate_input(c(1, -1, 3)),
    class = "invalid_value_error"
  )
})

test_that("validation passes valid input", {
  expect_no_error(validate_input(1:10))
  expect_equal(validate_input(1:5), 1:5)
})
```

### Testing with Temporary Files

```r
test_that("file operations work correctly", {
  # Create temporary file:
  temp_file <- withr::local_tempfile(fileext = ".csv")

  # Write data:
  write_my_data(data, temp_file)

  # Test file exists and has content:
  expect_true(file.exists(temp_file))
  expect_gt(file.size(temp_file), 0)

  # Read back and verify:
  result <- read_my_data(temp_file)
  expect_equal(result, data)

  # File automatically deleted after test
})
```

### Testing Side Effects

```r
test_that("function produces expected output", {
  # Capture printed output:
  expect_output(
    my_print_function(data),
    "Summary statistics:"
  )

  # Capture multiple lines:
  expect_snapshot({
    my_print_function(data)
  })
})

test_that("function produces plots", {
  # For base R plots:
  expect_silent({
    plot_data(data)
  })

  # For ggplot2:
  skip_if_not_installed("ggplot2")
  plot <- plot_data(data)
  expect_s3_class(plot, "gg")
})
```

### Testing S3 Methods

```r
test_that("S3 methods work correctly", {
  # Create object:
  obj <- structure(
    list(x = 1:10, y = letters[1:10]),
    class = "myclass"
  )

  # Test print method:
  expect_output(print(obj), "myclass object")

  # Test summary method:
  summ <- summary(obj)
  expect_s3_class(summ, "summary_myclass")
  expect_named(summ, c("n", "mean"))

  # Test subset method:
  expect_equal(obj[1:5], structure(
    list(x = 1:5, y = letters[1:5]),
    class = "myclass"
  ))
})
```

### Testing for Specific Warnings/Messages

```r
test_that("deprecation warnings work", {
  # Function should warn about deprecated argument:
  expect_warning(
    my_function(old_arg = TRUE),
    "old_arg.*deprecated",
    class = "lifecycle_warning_deprecated"
  )

  # Function should message about progress:
  expect_message(
    my_function(verbose = TRUE),
    "Processing 100 items"
  )

  # No warnings with new interface:
  expect_no_warning(my_function(new_arg = TRUE))
})
```

### Testing Randomness

```r
test_that("random functions are reproducible", {
  # Set seed and test:
  withr::local_seed(123)
  result1 <- my_random_function()

  withr::local_seed(123)
  result2 <- my_random_function()

  expect_equal(result1, result2)
})

test_that("random functions vary without seed", {
  results <- replicate(100, my_random_function(), simplify = FALSE)

  # Results should not all be identical:
  expect_gt(length(unique(results)), 1)
})
```

## Complete Test File Template

```r
# tests/testthat/test-my-feature.R

# Basic functionality tests
test_that("basic operations work", {
  result <- my_function(1:10)
  expect_equal(result, (1:10) * 2)
  expect_length(result, 10)
  expect_type(result, "integer")
})

# Edge case tests
test_that("edge cases are handled", {
  # Empty input:
  expect_equal(my_function(numeric(0)), numeric(0))

  # Single element:
  expect_equal(my_function(5), 10)

  # Large input:
  big_input <- 1:10000
  expect_equal(my_function(big_input), big_input * 2)
})

# Error handling tests
test_that("invalid inputs produce errors", {
  expect_error(
    my_function(NULL),
    "cannot be NULL",
    class = "null_input_error"
  )

  expect_error(
    my_function("text"),
    "must be numeric",
    class = "invalid_type_error"
  )

  expect_error(
    my_function(list(1, 2)),
    "must be atomic",
    class = "invalid_type_error"
  )
})

# Warning tests
test_that("warnings are issued appropriately", {
  expect_warning(
    my_function(c(1, NA, 3)),
    "NA values detected"
  )

  expect_warning(
    my_function(1:5, deprecated_arg = TRUE),
    "deprecated",
    class = "lifecycle_warning_deprecated"
  )
})

# Parameter tests
test_that("parameters work correctly", {
  # Default parameters:
  expect_equal(
    my_function(1:5),
    my_function(1:5, multiplier = 2)
  )

  # Non-default parameters:
  expect_equal(
    my_function(1:5, multiplier = 3),
    (1:5) * 3
  )
})

# Integration tests
test_that("integration with other functions works", {
  skip_if_not_installed("dplyr")

  data <- data.frame(x = 1:10)
  result <- data %>%
    dplyr::mutate(y = my_function(x))

  expect_equal(result$y, (1:10) * 2)
})

# Snapshot tests
test_that("output format is stable", {
  expect_snapshot({
    print_my_result(my_function(1:5))
  })
})
```

## Common Pitfalls

### 1. Not Setting Config/testthat/edition

```r
# Add to DESCRIPTION:
Config/testthat/edition: 3
```

### 2. Tests Not Starting with "test-"

```r
# WRONG:
tests/testthat/my-tests.R

# RIGHT:
tests/testthat/test-my-feature.R
```

### 3. Using context() in Edition 3

```r
# WRONG (Edition 3):
context("My tests")

# RIGHT:
# Just use descriptive file names
```

### 4. Not Testing Error Classes

```r
# LESS INFORMATIVE:
expect_error(my_function(bad_input))

# BETTER:
expect_error(my_function(bad_input), "specific message")

# BEST:
expect_error(
  my_function(bad_input),
  class = "specific_error_class"
)
```

### 5. Forgetting skip_if_not_installed()

```r
# WRONG - test fails if package not installed:
test_that("works with ggplot2", {
  library(ggplot2)
  # ...
})

# RIGHT:
test_that("works with ggplot2", {
  skip_if_not_installed("ggplot2")
  library(ggplot2)
  # ...
})
```

### 6. Not Using withr for Cleanup

```r
# RISKY - file might not be deleted if test fails:
test_that("file operations work", {
  file.create("temp.txt")
  # ... test code ...
  unlink("temp.txt")
})

# SAFE - guaranteed cleanup:
test_that("file operations work", {
  withr::local_file("temp.txt")
  file.create("temp.txt")
  # ... test code ...
  # Automatic cleanup
})
```

### 7. Testing Implementation Instead of Interface

```r
# WRONG - tests internal details:
test_that("uses correct algorithm", {
  expect_true(grepl("quicksort", deparse(my_sort)))
})

# RIGHT - tests behavior:
test_that("sorts correctly", {
  expect_equal(my_sort(c(3, 1, 2)), c(1, 2, 3))
})
```

### 8. Tests with Side Effects Between test_that() Blocks

```r
# WRONG - shared state:
x <- NULL
test_that("test 1", {
  x <<- 5
  expect_equal(x, 5)
})
test_that("test 2", {
  expect_equal(x, 5)  # Depends on test 1!
})

# RIGHT - independent:
test_that("test 1", {
  x <- 5
  expect_equal(x, 5)
})
test_that("test 2", {
  x <- 5
  expect_equal(x, 5)
})
```

### 9. Not Running Tests Before Commit

```r
# Always run tests:
devtools::test()

# Or:
testthat::test_local()

# Or full check:
devtools::check()
```

### 10. Slow Tests Without skip_on_cran()

```r
# WRONG - slows down CRAN checks:
test_that("comprehensive test", {
  result <- very_slow_function()  # Takes 10 seconds
  expect_equal(result, expected)
})

# RIGHT:
test_that("comprehensive test", {
  skip_on_cran()
  result <- very_slow_function()
  expect_equal(result, expected)
})
```

## Quick Reference

### Running Tests

```r
# Run all tests:
devtools::test()

# Run specific file:
testthat::test_file("tests/testthat/test-my-feature.R")

# Run with coverage:
covr::package_coverage()

# Interactive testing:
devtools::load_all()
# ... manually test functions ...
```

### Common Expectations

```r
expect_equal(x, y)              # Near equality
expect_identical(x, y)          # Exact identity
expect_true(x) / expect_false(x)
expect_null(x)
expect_error(code, class = "error_class")
expect_warning(code, "message pattern")
expect_message(code, "message pattern")
expect_no_error(code)
expect_snapshot({code})
```

### Test Organization

```r
tests/testthat/
├── helper-*.R        # Helpers (run first)
├── setup.R           # One-time setup
├── test-*.R          # Test files
└── teardown.R        # One-time cleanup
```

## Resources

- [testthat documentation](https://testthat.r-lib.org/)
- [R Packages: Testing](https://r-pkgs.org/testing-basics.html)
- [testthat 3e announcement](https://www.tidyverse.org/blog/2021/10/testthat-3-1-0/)
