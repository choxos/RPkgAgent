---
name: add-function
description: Add a new function to an existing R package with documentation and tests
---

# Add Function to Package

This command adds a new function to an existing R package, including complete roxygen2 documentation, tests, and NAMESPACE updates.

## Workflow

### Step 1: Understand Function Requirements

Gather information from the user:

- **Function name**: Use snake_case convention (e.g., `clean_text`, `remove_stopwords`)
- **Purpose**: What does this function do? One-sentence summary
- **Parameters**: What arguments does it take?
  - Name, type, default values, constraints
- **Return value**: What does it return? Type and structure
- **Export status**: Should this be user-facing (`@export`) or internal?
- **Dependencies**: Does it use functions from other packages?
- **Related functions**: Does it work with existing functions in the package?

### Step 2: Create Function File with Roxygen Documentation

Create `R/function_name.R` with complete documentation:

```r
#' One-line title for the function
#'
#' More detailed description of what the function does, how it works,
#' and when you might use it. Can span multiple paragraphs.
#'
#' @param arg1 A character vector. Description of what this parameter does
#'   and any constraints (e.g., "must be non-empty", "case-sensitive").
#' @param arg2 A numeric value. Default is 10. Description of this parameter.
#' @param verbose Logical. If \code{TRUE}, prints progress messages. Default
#'   is \code{FALSE}.
#'
#' @returns A data frame with columns \code{id}, \code{value}, and \code{status}.
#'   Returns \code{NULL} if no matches found.
#'
#' @export
#'
#' @examples
#' # Basic usage
#' result <- function_name("example", arg2 = 5)
#'
#' # With verbose output
#' function_name("example", verbose = TRUE)
#'
#' # Edge case
#' function_name(character(0))  # Returns NULL
#'
#' @importFrom dplyr filter mutate
#' @importFrom rlang .data
function_name <- function(arg1, arg2 = 10, verbose = FALSE) {
  # Validate inputs
  if (!is.character(arg1)) {
    stop("`arg1` must be a character vector", call. = FALSE)
  }
  if (length(arg1) == 0) {
    if (verbose) message("Empty input, returning NULL")
    return(NULL)
  }

  # Function logic here
  result <- data.frame(
    id = seq_along(arg1),
    value = arg1,
    status = "processed"
  )

  result
}
```

Key roxygen tags to include:

- `@title` (optional, first line is used if omitted)
- `@description` (optional, paragraph after title if omitted)
- `@param` for EVERY parameter, including dots (`...`)
- `@returns` describing the return value (NOT `@return`)
- `@export` if function should be in NAMESPACE
- `@examples` with executable examples (not wrapped in `\dontrun{}` unless necessary)
- `@importFrom pkg fun` for each external function used
- Optional: `@seealso`, `@references`, `@family`

### Step 3: Handle Package Dependencies

If the function uses external packages:

**Option A**: Add `@importFrom` to the function file (recommended for few imports)

```r
#' @importFrom dplyr filter mutate select
#' @importFrom tidyr pivot_longer
```

**Option B**: Add to `R/packagename-package.R` (recommended for package-wide imports)

```r
#' @importFrom dplyr filter mutate select
#' @importFrom rlang .data .env
#' @keywords internal
"_PACKAGE"
```

Update `DESCRIPTION` to include dependencies:

```
Imports:
    dplyr (>= 1.0.0),
    tidyr
```

If using tidyverse NSE (non-standard evaluation):

- Use `.data$column_name` instead of bare column names
- Or add `utils::globalVariables(c("column1", "column2"))` to `R/packagename-package.R`

### Step 4: Create Comprehensive Tests

Create `tests/testthat/test-function_name.R`:

```r
test_that("function_name works with valid input", {
  result <- function_name("test", arg2 = 5)

  expect_s3_class(result, "data.frame")
  expect_equal(nrow(result), 1)
  expect_equal(result$value, "test")
  expect_equal(result$status, "processed")
})

test_that("function_name handles edge cases", {
  # Empty input
  expect_null(function_name(character(0)))

  # Single element
  result <- function_name("a")
  expect_equal(nrow(result), 1)

  # Multiple elements
  result <- function_name(c("a", "b", "c"))
  expect_equal(nrow(result), 3)
})

test_that("function_name validates inputs", {
  # Wrong type
  expect_error(
    function_name(123),
    "`arg1` must be a character vector"
  )

  # Invalid arg2
  expect_error(
    function_name("test", arg2 = "wrong"),
    class = "error"
  )
})

test_that("function_name verbose mode works", {
  expect_message(
    function_name(character(0), verbose = TRUE),
    "Empty input"
  )
})
```

Cover these test scenarios:

- **Normal inputs**: Typical use cases with expected outputs
- **Edge cases**: Empty, single element, large inputs, NAs
- **Invalid inputs**: Wrong types, out-of-range values
- **Special behaviors**: verbose mode, optional parameters
- **Integration**: Works with other package functions if applicable

### Step 5: Update Documentation

Run `devtools::document()` to:

- Update `NAMESPACE` with exports and imports
- Generate `man/function_name.Rd` help file
- Update package documentation

Verify the generated documentation:

```r
?function_name  # Should display help page correctly
```

### Step 6: Run Package Check

Run `devtools::check()` to verify no new issues:

Look for:

- ✓ No errors about undocumented parameters
- ✓ No warnings about missing `@returns`
- ✓ No notes about undefined global variables
- ✓ Examples run without errors
- ✓ Tests pass

Fix any issues that arise:

- Missing docs → add roxygen tags
- Example errors → fix or wrap in `\dontrun{}`
- Global variables → use `.data$` or add `globalVariables()`
- Unused imports → remove or add explicit usage

### Step 7: Update Reference Documentation

If the package has `_pkgdown.yml`, add the function to the appropriate section:

```yaml
reference:
  - title: "Text Cleaning"
    desc: "Functions for cleaning and preprocessing text"
    contents:
      - clean_text
      - remove_stopwords
      - function_name  # <-- Add here
```

Run `pkgdown::build_reference()` to preview the updated documentation site.

### Final Checklist

Before completing:

- [ ] Function file created in `R/` with roxygen documentation
- [ ] All parameters documented with `@param`
- [ ] Return value documented with `@returns`
- [ ] Working examples included (run without errors)
- [ ] Test file created in `tests/testthat/`
- [ ] Tests cover normal, edge, and error cases
- [ ] `devtools::document()` run successfully
- [ ] `devtools::check()` shows no new errors/warnings
- [ ] Function appears in package help (`?function_name`)
- [ ] `_pkgdown.yml` updated if applicable

Report to user:

```
✓ Function 'function_name' added successfully!

Created:
  - R/function_name.R (45 lines, fully documented)
  - tests/testthat/test-function_name.R (4 tests)
  - man/function_name.Rd (generated)

Quality:
  - R CMD check: 0 new errors ✓ | 0 new warnings ✓
  - Tests: 4/4 passing
  - Documentation: Complete with 3 examples

The function is now exported and ready to use.
```

## Example Interaction

User: "Add a function called 'remove_urls' that strips URLs from text"