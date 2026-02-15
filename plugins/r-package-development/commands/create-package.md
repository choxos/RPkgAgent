---
name: create-package
description: Create a new R package from scratch with full development infrastructure
---

# Create R Package

This command guides you through creating a complete R package from scratch, including code, tests, documentation, and a pkgdown website.

## Workflow

### Step 1: Gather Requirements

Ask the user for the following information:

- **Package name**: Must be a valid R package name (letters, numbers, periods only; start with letter)
- **Package purpose**: One-sentence description of what the package does
- **Author information**: Name and email for DESCRIPTION file
- **License**: MIT, GPL-3, Apache-2.0, etc.
- **Target platform**: CRAN, Bioconductor, or internal use only
- **Existing code**: Does the user have R scripts to convert into package functions?
- **Dependencies**: What packages does this depend on?

### Step 2: Route to @scaffold-engineer

The scaffold engineer creates the basic package structure:

- Run `usethis::create_package("/path/to/packagename")` or manually create structure
- Create `DESCRIPTION` file with Package, Title, Version (0.0.0.9000), Authors@R, Description, License, Encoding (UTF-8), Roxygen (list(markdown = TRUE)), RoxygenNote
- Create `LICENSE` file (if MIT/Apache-2.0) or `LICENSE.md`
- Create `R/packagename-package.R` with package-level documentation
- Create `tests/testthat.R` with `testthat::test_check("packagename")`
- Create `tests/testthat/` directory
- Create `.Rbuildignore` with patterns for development files
- Create `.gitignore` with `.Rproj.user`, `.Rhistory`, `.RData`, `.Ruserdata`
- Create `packagename.Rproj` if needed

### Step 3: Route to @code-engineer

The code engineer writes R functions:

- For each function identified in requirements:
  - Create `R/function_name.R`
  - Write function with proper argument validation
  - Add complete roxygen2 documentation:
    - `@title` and `@description`
    - `@param` for each parameter with type and description
    - `@returns` describing return value structure
    - `@export` if function should be user-facing
    - `@examples` with working, executable examples
    - `@importFrom pkg function` for external dependencies
  - Follow tidyverse style guide (snake_case, spaces, etc.)

- For packages with dependencies, add to DESCRIPTION:
  - `Imports:` for required packages
  - `Suggests:` for optional packages (used in examples/vignettes)

### Step 4: Route to @test-engineer

The test engineer writes comprehensive tests:

- For each exported function, create `tests/testthat/test-function_name.R`
- Write tests using testthat framework:
  - `test_that("descriptive test name", { ... })`
  - Test normal inputs with `expect_equal()`, `expect_identical()`
  - Test edge cases (empty inputs, NULLs, NAs)
  - Test error conditions with `expect_error()`, `expect_warning()`
  - Aim for >80% code coverage on core functions

- Run `devtools::test()` to verify all tests pass

### Step 5: Route to @docs-engineer

The docs engineer creates user-facing documentation:

- Create `README.md` with:
  - Package overview and purpose
  - Installation instructions (`pak::pak("user/package")` or `install.packages()`)
  - Quick start example showing main functionality
  - Badges for R-CMD-check, codecov, lifecycle

- Create `vignettes/packagename.Rmd`:
  - Introduction vignette showing typical workflows
  - Use `usethis::use_vignette("packagename")`

- Create basic `_pkgdown.yml`:
  - Set template (Bootstrap 5)
  - Organize reference into logical sections

- Create hex logo:
  - `man/figures/logo.png` (240x278 px)
  - Add to README with `usethis::use_logo()`

### Step 6: Route to @check-doctor

The check doctor runs quality checks and fixes issues:

- Run `devtools::document()` to generate NAMESPACE and .Rd files
- Run `devtools::check()` to run R CMD check
- Parse output for errors, warnings, notes:
  - Fix DESCRIPTION formatting issues
  - Add missing documentation
  - Remove unused imports
  - Fix example errors
  - Add `utils::globalVariables()` for NSE notes if needed

- Iterate until achieving 0 errors, 0 warnings, 0 notes (or only acceptable notes)

### Step 7: Final Verification

Confirm the package is complete:

- `devtools::check()` shows 0 errors, 0 warnings, 0 notes
- All exported functions have documentation and examples
- All tests pass with `devtools::test()`
- README renders correctly
- Git repository initialized with clean commit history

Report to user:

```
✓ Package 'packagename' created successfully!

Structure:
  - 5 exported functions in R/
  - 5 test files with 25 tests (100% passing)
  - README.md with quick start
  - 1 vignette
  - DESCRIPTION complete with all metadata

Quality:
  - R CMD check: 0 errors ✓ | 0 warnings ✓ | 0 notes ✓
  - Test coverage: 87%

Next steps:
  - Run /setup-website to create pkgdown site
  - Push to GitHub
  - Consider adding more vignettes or data
```

## Example Interaction

User: "Create a package called 'cleantext' for text preprocessing"