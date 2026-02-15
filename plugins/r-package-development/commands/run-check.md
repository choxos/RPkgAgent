---
name: run-check
description: Run R CMD check and automatically fix common errors, warnings, and notes
---

# Run Package Check

This command runs `R CMD check` on your package and automatically fixes common issues to achieve 0 errors, 0 warnings, 0 notes.

## Workflow

### Step 1: Run Initial Check

Execute the comprehensive package check:

```r
devtools::check()
```

This runs:

- Package installation
- Documentation check
- Code syntax check
- Examples execution
- Tests execution
- Vignette building
- R CMD check with ~50 different checks

Capture the output and parse for:

- **Errors**: Must fix (blocking issues)
- **Warnings**: Should fix (CRAN will reject)
- **Notes**: May fix (CRAN may question)

### Step 2: Parse Check Output

Extract specific issues from the output:

Look for patterns like:

```
* checking DESCRIPTION meta-information ... WARNING
Non-standard license specification:
  MIT + file LICENSE
Standardized: MIT

* checking for missing documentation entries ... WARNING
Undocumented code objects:
  'function_name'

* checking examples ... ERROR
Running examples in 'package-Ex.R' failed

* checking dependencies in R code ... NOTE
'::' or ':::' imports not declared from:
  'dplyr' 'rlang'

* checking R code for possible problems ... NOTE
function_name: no visible binding for global variable 'column_name'
```

Categorize each issue by type.

### Step 3: Fix DESCRIPTION Issues

Common DESCRIPTION problems:

**Issue**: Non-standard license specification

Fix:

```
# Before
License: MIT + file LICENSE

# After
License: MIT + file LICENSE
Authors@R: person("First", "Last", email = "email@example.com",
                  role = c("aut", "cre"))
```

**Issue**: Title or Description formatting

Fix:

```
# Title should be in title case, no period
Title: Tools For Text Preprocessing  # Wrong
Title: Tools for Text Preprocessing  # Correct

# Description should be longer than Title, end with period
Description: Text preprocessing.  # Too short
Description: A comprehensive toolkit for preprocessing text data,
    including functions for cleaning, tokenization, and normalization.  # Good
```

**Issue**: Missing fields

Add:

```
Encoding: UTF-8
Roxygen: list(markdown = TRUE)
RoxygenNote: 7.2.3
```

### Step 4: Fix Missing Documentation

**Issue**: Undocumented code objects

Find undocumented exports:

```r
# Check NAMESPACE
cat(readLines("NAMESPACE"), sep = "\n")
```

For each undocumented function, add roxygen2 documentation:

```r
#' Title for the function
#'
#' Description of what it does.
#'
#' @param arg Description
#' @returns Description of return value
#' @export
#' @examples
#' function_name("example")
function_name <- function(arg) { }
```

Then regenerate docs:

```r
devtools::document()
```

**Issue**: Missing `@returns` tag

Add to every exported function:

```r
#' @returns A character vector of cleaned text.
```

Or for invisible returns:

```r
#' @returns Invisibly returns the input object.
```

### Step 5: Fix Import/Dependency Issues

**Issue**: `'::' or ':::' imports not declared`

Solution A: Add to DESCRIPTION Imports:

```
Imports:
    dplyr,
    rlang
```

Solution B: Add `@importFrom` to R files:

```r
#' @importFrom dplyr filter mutate
#' @importFrom rlang .data
```

Then run:

```r
devtools::document()  # Updates NAMESPACE
```

**Issue**: Unused imports in NAMESPACE

Find what's imported but not used. Either:

- Add explicit usage in code
- Remove from `@importFrom` statements

**Issue**: Depends/Imports/Suggests confusion

Guidelines:

- **Imports**: Packages used in your functions (most common)
- **Suggests**: Packages used in examples/vignettes/tests (optional)
- **Depends**: Rarely used (only for package extensions)

Move packages between sections as needed.

### Step 6: Fix Global Variable Notes

**Issue**: `no visible binding for global variable 'column_name'`

This occurs with NSE (non-standard evaluation) in tidyverse code.

Solution A: Use `.data` pronoun (recommended):

```r
# Before
df %>% filter(column_name == "value")

# After
df %>% filter(.data$column_name == "value")

# Add to roxygen
#' @importFrom rlang .data
```

Solution B: Declare global variables:

In `R/packagename-package.R`:

```r
#' @keywords internal
"_PACKAGE"

# Suppress R CMD check notes for NSE
utils::globalVariables(c(
  "column_name",
  "another_column",
  "third_column"
))
```

Solution C: Use string-based indexing:

```r
df %>% filter(.data[["column_name"]] == "value")
```

### Step 7: Fix Example Errors

**Issue**: Examples fail during check

Strategies:

**Option 1**: Fix the example code

```r
#' @examples
#' # This should work
#' result <- function_name("valid input")
#' str(result)
```

**Option 2**: Add error handling to examples

```r
#' @examples
#' # Gracefully handle missing dependencies
#' if (requireNamespace("optionalpackage", quietly = TRUE)) {
#'   function_name("example")
#' }
```

**Option 3**: Use `\dontrun{}` (sparingly!)

```r
#' @examples
#' \dontrun{
#'   # Requires API key
#'   function_name(api_key = Sys.getenv("API_KEY"))
#' }
#'
#' # But include runnable examples too
#' function_name("demo_mode")
```

**Option 4**: Use `\donttest{}` for slow examples

```r
#' @examples
#' # Quick example
#' function_name(n = 10)
#'
#' \donttest{
#'   # Slow example (>5 seconds)
#'   function_name(n = 1000000)
#' }
```

CRAN preference: Runnable examples > `\donttest{}` > `\dontrun{}`

### Step 8: Fix Non-ASCII Characters

**Issue**: Non-ASCII characters in code or docs

Find non-ASCII:

```r
tools::showNonASCII(readLines("R/function.R"))
```

Replace with Unicode escapes:

```r
# Before
"café"  # Contains é (non-ASCII)

# After
"caf\u00e9"  # Unicode escape
```

For roxygen documentation:

```r
# Before
#' Measure in Ångströms

# After
#' Measure in \u00c5ngstr\u00f6ms
```

### Step 9: Fix LICENSE Issues

**Issue**: License file doesn't match DESCRIPTION

For MIT license, create `LICENSE` file:

```
YEAR: 2024
COPYRIGHT HOLDER: Your Name
```

For GPL-3:

```
DESCRIPTION:
License: GPL-3

No LICENSE file needed (standard license)
```

For custom licenses, ensure LICENSE.md is complete.

### Step 10: Fix Test Issues

**Issue**: Tests fail during check

Run tests locally:

```r
devtools::test()
```

Fix failing tests:

- Update expected values
- Handle edge cases
- Skip tests conditionally if needed:

```r
test_that("function works with optional dependency", {
  skip_if_not_installed("optionalpackage")
  expect_equal(function_name(), expected)
})
```

### Step 11: Fix Vignette Build Issues

**Issue**: Vignette fails to build

Check vignette locally:

```r
devtools::build_vignettes()
```

Common fixes:

- Add missing Suggests packages to DESCRIPTION
- Fix code errors in vignette chunks
- Add `eval=FALSE` to chunks that require user setup

### Step 12: Regenerate Documentation

After all fixes, regenerate:

```r
devtools::document()  # Updates NAMESPACE and .Rd files
```

Verify:

```r
devtools::check_man()  # Check documentation consistency
```

### Step 13: Re-run Check

Run check again:

```r
devtools::check()
```

Iterate through steps 3-12 until achieving:

```
── R CMD check results ───── packagename 0.1.0 ────
Duration: 1m 23s

0 errors ✔ | 0 warnings ✔ | 0 notes ✔
```

### Step 14: Handle Acceptable Notes

Some notes are acceptable for CRAN:

**Acceptable notes**:

- "New submission" (for first CRAN submission)
- "Maintainer: ..." (informational)
- Installed size notes if genuinely needed
- "GNU make is a SystemRequirements" (if actually required)

**Not acceptable**:

- Undocumented objects
- Undefined global variables (can fix)
- Missing imports (can fix)
- Examples with errors

Document acceptable notes in `cran-comments.md`:

```markdown
## R CMD check results

0 errors | 0 warnings | 1 note

* This is a new release.
```

### Common Issue Reference

Quick fixes for frequent problems:

| Issue | Fix |
|-------|-----|
| Undocumented function | Add roxygen docs, run `devtools::document()` |
| Missing `@returns` | Add `#' @returns` tag to exported functions |
| NSE variable note | Use `.data$var` or `globalVariables()` |
| Unused import | Remove from DESCRIPTION or add usage |
| Missing import | Add to DESCRIPTION Imports |
| Non-ASCII character | Replace with `\uXXXX` escape |
| Example error | Fix code or use `\dontrun{}` |
| LICENSE format | Create LICENSE file with YEAR and COPYRIGHT HOLDER |
| Title format | Use title case, no period |
| Description short | Expand to 2+ sentences, end with period |

### Final Checklist

- [ ] `devtools::check()` runs without errors
- [ ] 0 errors, 0 warnings, 0 notes (or only acceptable notes)
- [ ] All functions documented with roxygen2
- [ ] All `@param` and `@returns` tags present
- [ ] Examples run successfully
- [ ] Tests pass
- [ ] NAMESPACE correctly generated
- [ ] No undefined global variables
- [ ] No undeclared imports
- [ ] DESCRIPTION properly formatted
- [ ] LICENSE file correct

Report to user:

```
✓ Package check complete!

Check results:
  - R CMD check: 0 errors ✓ | 0 warnings ✓ | 0 notes ✓
  - Duration: 1m 23s

Fixes applied:
  - Added @returns to 3 functions
  - Fixed 2 NSE global variable notes (added .data$)
  - Added missing import: dplyr to DESCRIPTION
  - Fixed DESCRIPTION Title formatting
  - Regenerated NAMESPACE and documentation

Your package is ready for release!

Next steps:
  - Run /prepare-release for CRAN submission
  - Or deploy to GitHub/internal repository
```

## Example Interaction

User: "Run check on my package and fix any issues"