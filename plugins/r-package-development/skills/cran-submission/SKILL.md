---
name: cran-submission
description: Complete guide to CRAN submission including policies, pre-submission checks, common issues, and the submission process
---

# CRAN Submission Guide

This skill covers the complete CRAN submission process, from pre-submission preparation through to acceptance and post-release tasks. CRAN (Comprehensive R Archive Network) is the primary repository for R packages and has strict quality requirements.

## Rules

1. **All exported functions** must have `@returns` and `@examples` documentation
2. **Version number** must be greater than the current CRAN version
3. **Test on multiple platforms** before submission (Windows, macOS, Linux, R-devel)
4. **Fix all ERRORs and WARNINGs** - CRAN will reject packages with any
5. **NOTEs should be explained** in cran-comments.md
6. **Title case for Title field** with no period at the end, under 65 characters
7. **Use cph role** for copyright holders in Authors@R
8. **Check all URLs** with urlchecker::url_check()
9. **Examples must run** without errors (use `\donttest{}` sparingly)
10. **Respond to CRAN within 2 weeks** or submission will be archived

## Pre-Submission Checklist

### Essential Documentation Requirements

Every exported function must have:

```r
#' Function Title
#'
#' @description
#' Detailed description of what the function does.
#'
#' @param x Description of parameter x
#' @param y Description of parameter y
#'
#' @returns A data.frame with columns:
#'   \item{col1}{Description of column 1}
#'   \item{col2}{Description of column 2}
#'
#' @examples
#' # Basic usage
#' result <- my_function(x = 1, y = 2)
#' print(result)
#'
#' # Advanced usage
#' result2 <- my_function(x = 1:10, y = 20)
#'
#' @export
my_function <- function(x, y) {
  # implementation
}
```

**Critical**: Use `@returns` (not `@return`) for consistency with roxygen2 7.0+.

### DESCRIPTION File Requirements

```
Package: packagename
Title: A Package That Does Amazing Things  # Title case, < 65 chars, no period
Version: 0.1.0  # Must be > current CRAN version
Authors@R: c(
    person("First", "Last", email = "email@example.com",
           role = c("aut", "cre"),
           comment = c(ORCID = "0000-0000-0000-0000")),
    person("Another", "Author", role = "aut"),
    person("Company Name", role = "cph")  # Copyright holder
    )
Description: Provides tools for amazing analysis. This package implements
    novel algorithms and provides intuitive interfaces. Functions include
    data processing, visualization, and reporting capabilities.
License: MIT + file LICENSE
Encoding: UTF-8
LazyData: true
Roxygen: list(markdown = TRUE)
RoxygenNote: 7.3.0
URL: https://username.github.io/packagename/, https://github.com/username/packagename
BugReports: https://github.com/username/packagename/issues
Suggests:
    testthat (>= 3.0.0),
    knitr,
    rmarkdown
VignetteBuilder: knitr
Config/testthat/edition: 3
```

**Title field rules**:
- Title case (capitalize major words)
- No period at end
- Under 65 characters
- Don't start with "A Package for..." (redundant)
- Be specific about what it does

**Description field rules**:
- One paragraph
- Indent continuation lines with 4 spaces
- Don't start with "This package..."
- Explain what the package does, not just list features
- Mention key algorithms or papers if relevant

### Version Numbering

For new submissions:
```
Version: 0.1.0
```

For updates (must be > CRAN version):
```
# If CRAN has 0.1.0, use:
Version: 0.1.1  # Bug fixes
Version: 0.2.0  # New features
Version: 1.0.0  # Major release
```

Development versions (not for CRAN):
```
Version: 0.1.0.9000  # Development version after 0.1.0
```

## Multi-Platform Testing

### Local Testing

```r
# Standard check
devtools::check()

# Check with remote checking (stricter)
devtools::check(remote = TRUE, manual = TRUE)

# Check with --as-cran flag
devtools::check(args = c('--as-cran'))
```

### Windows Testing

```r
# Check on Windows builder (CRAN's Windows server)
devtools::check_win_devel()   # R-devel
devtools::check_win_release() # R-release
devtools::check_win_oldrelease() # R-oldrelease
```

You'll receive an email with results in ~30 minutes.

### R-hub Testing

```r
# Install rhub
install.packages("rhub")

# Validate email first time
rhub::validate_email()

# Check on multiple platforms
rhub::check_for_cran()

# Specific platforms
rhub::check(platform = "windows-x86_64-devel")
rhub::check(platform = "ubuntu-gcc-release")
rhub::check(platform = "macos-highsierra-release-cran")
rhub::check(platform = "solaris-x86-patched")

# List available platforms
rhub::platforms()
```

**Note**: R-hub can be unreliable. GitHub Actions with r-lib/actions is often more reliable.

### GitHub Actions

Most reliable multi-platform testing:

```r
# Set up GitHub Actions
usethis::use_github_action("check-standard")
```

Tests on:
- macOS (release)
- Windows (release)
- Ubuntu (devel, release, oldrel-1)

## URL Checking

```r
# Check all URLs in documentation and vignettes
urlchecker::url_check()

# Fix or explain any broken URLs
```

Common issues:
- DOIs should use `https://doi.org/` not `http://dx.doi.org/`
- Use permanent URLs, not redirects
- Some URLs may be temporarily down (explain in cran-comments.md)

## Creating cran-comments.md

Create `cran-comments.md` in package root:

```markdown
## Test environments
* local R installation, R 4.4.0
* ubuntu 22.04 (on GitHub Actions), R-devel, R-release, R-oldrel-1
* windows-latest (on GitHub Actions), R-release
* macos-latest (on GitHub Actions), R-release
* win-builder (devel and release)

## R CMD check results

0 errors | 0 warnings | 1 note

* This is a new release.

## Downstream dependencies

There are currently no downstream dependencies for this package.
```

### For Updates

```markdown
## Test environments
* local R installation, R 4.4.0
* ubuntu 22.04 (on GitHub Actions), R-devel, R-release, R-oldrel-1
* windows-latest (on GitHub Actions), R-release
* macos-latest (on GitHub Actions), R-release

## R CMD check results

0 errors | 0 warnings | 0 notes

## What's changed

This is a minor release that fixes bugs and adds new features:

* Fixed issue with X (#123)
* Added new function Y
* Improved documentation for Z

## Downstream dependencies

I have run R CMD check on downstream dependencies of my package.
All packages passed.
```

### Explaining NOTEs

```markdown
## R CMD check results

0 errors | 0 warnings | 1 note

* Checking CRAN incoming feasibility ... NOTE

  Maintainer: 'First Last <email@example.com>'

  New submission

  Possibly misspelled words in DESCRIPTION:
    MCMC (3:49)
    PyMC (9:31)

  These are not misspelled:
  * MCMC is a standard acronym for Markov Chain Monte Carlo
  * PyMC is the name of a Python library
```

## Submission Process

### First Submission

```r
# 1. Final checks
devtools::check()
devtools::check_win_devel()
rhub::check_for_cran()
urlchecker::url_check()

# 2. Build the package
devtools::build()

# 3. Submit to CRAN
devtools::submit_cran()

# This will:
# - Build the package
# - Run R CMD check
# - Ask for confirmation
# - Submit to CRAN via web form
```

### Manual Submission

If `submit_cran()` doesn't work:

1. Build package: `devtools::build()`
2. Go to https://cran.r-project.org/submit.html
3. Upload .tar.gz file
4. Fill in maintainer email and comments
5. Submit

### Confirmation Email

You'll receive an email asking to confirm submission:
- Click the confirmation link within 24 hours
- CRAN will then review your package

## CRAN Review Process

### Timeline

1. **Submission**: Upload package
2. **Confirmation** (immediate): Confirm via email
3. **Automated checks** (minutes to hours): CRAN runs checks
4. **Human review** (days to weeks): CRAN volunteer reviews
5. **Acceptance or rejection**: Email notification

**Typical timeline**: 1-2 weeks, but can be longer during busy periods.

### Common Rejection Reasons

1. **ERRORs or WARNINGs** in R CMD check
2. **Missing documentation** (no @returns or @examples)
3. **Examples fail** or take too long (>5 seconds per example)
4. **License issues** (incompatible or missing LICENSE file)
5. **DESCRIPTION problems** (title, description formatting)
6. **Failed tests** on CRAN's systems
7. **Large package size** (>5 MB without justification)
8. **Writing to user's home directory** without permission
9. **Internet resources without checks** (must fail gracefully if offline)
10. **Non-standard file permissions**

## Common R CMD Check Issues and Fixes

### 1. Undefined Global Variables

**Problem**:
```
checking R code for possible problems ... NOTE
my_function: no visible binding for global variable 'x'
my_function: no visible binding for global variable 'y'
```

**Cause**: Using NSE (non-standard evaluation) with dplyr, data.table, etc.

**Solution**:
```r
# In R/globals.R
utils::globalVariables(c("x", "y", "z"))

# Or use .data pronoun
library(dplyr)
my_function <- function(df) {
  df %>%
    mutate(z = .data$x + .data$y)
}
```

### 2. Unused Imports

**Problem**:
```
checking dependencies in R code ... NOTE
Namespace in Imports field not imported from: 'package'
```

**Solution**: Remove from DESCRIPTION or use somewhere:
```r
#' @importFrom package function
```

### 3. Non-ASCII Characters

**Problem**:
```
checking R files for non-ASCII characters ... NOTE
```

**Solution**: Use Unicode escapes:
```r
# Bad
"Müller"

# Good
"M\u00fcller"
```

### 4. Examples Fail

**Problem**:
```
checking examples ... ERROR
```

**Solution**: Wrap problematic examples:
```r
#' @examples
#' # Basic example (always runs)
#' result <- my_function(x = 1)
#'
#' \donttest{
#' # Slow example (skipped on CRAN)
#' slow_result <- slow_function(x = 1:1000000)
#' }
#'
#' \dontrun{
#' # Requires authentication (never runs)
#' api_result <- call_api(key = "your-key")
#' }
```

**Rules**:
- `\donttest{}`: Runs locally and on CI, skipped on CRAN (for slow examples)
- `\dontrun{}`: Never runs automatically (for examples needing setup)
- Don't overuse - most examples should run

### 5. Missing Imports

**Problem**:
```
checking dependencies in R code ... WARNING
'::' or ':::' imports not declared from:
  'package'
```

**Solution**: Add to DESCRIPTION Imports and use:
```r
#' @importFrom package function
```

Or declare in NAMESPACE:
```
importFrom(package, function)
```

### 6. LaTeX/PDF Errors

**Problem**:
```
checking PDF version of manual ... WARNING
```

**Solution**: Fix .Rd formatting issues or use:
```yaml
# In GitHub Actions
build_args: 'c("--no-manual")'
```

### 7. Vignette Build Fails

**Problem**:
```
checking re-building of vignette outputs ... WARNING
```

**Solution**: Ensure vignettes build:
```r
devtools::build_vignettes()

# Check for missing packages in Suggests
```

### 8. File Permissions

**Problem**:
```
checking file modes ... NOTE
```

**Solution**: Fix permissions:
```bash
chmod 644 R/*.R
chmod 644 man/*.Rd
chmod 755 configure
```

### 9. Package Size

**Problem**:
```
checking installed package size ... NOTE
  installed size is X Mb
  sub-directories of 1Mb or more:
    data   Y Mb
```

**Solution**:
- Compress data: `tools::resaveRdaFiles("data")`
- Use `LazyData: true` in DESCRIPTION
- Consider external data package
- Explain large size in cran-comments.md

### 10. Writing to User Directories

**Problem**: Package writes to home directory or modifies environment

**Solution**: Use temporary directories:
```r
# Bad
write.csv(data, "~/mypackage/output.csv")

# Good
tmpdir <- tempdir()
write.csv(data, file.path(tmpdir, "output.csv"))

# For configuration
config_dir <- tools::R_user_dir("mypackage", which = "config")
if (!dir.exists(config_dir)) {
  dir.create(config_dir, recursive = TRUE)
}
```

### 11-20. Additional Common Issues

**11. Suggested packages not declared**:
```r
# Conditional usage
if (requireNamespace("ggplot2", quietly = TRUE)) {
  # Use ggplot2
} else {
  message("Install ggplot2 for better plots")
}
```

**12. Internet resources without checks**:
```r
# Check connectivity
check_connection <- function(url) {
  tryCatch({
    httr::GET(url, httr::timeout(5))
    TRUE
  }, error = function(e) {
    message("Cannot connect to ", url)
    FALSE
  })
}
```

**13. Testing failures**:
```r
# Skip tests on CRAN if they require internet/auth
testthat::skip_on_cran()
testthat::skip_if_offline()
```

**14. Long-running tests**:
```r
# In tests/testthat.R
Sys.setenv("NOT_CRAN" = "true")

# In test files
testthat::skip_on_cran()  # Skip slow tests
```

**15. Missing LICENSE file**:
```r
# For MIT license
usethis::use_mit_license()

# Creates LICENSE and LICENSE.md
```

**16. Undeclared S3 methods**:
```r
#' @export
print.myclass <- function(x, ...) {
  # implementation
}

# In NAMESPACE (auto-generated by roxygen2)
S3method(print, myclass)
```

**17. Examples too long**:
- Keep examples under 5 seconds each
- Use smaller datasets
- Wrap slow parts in `\donttest{}`

**18. Rd cross-references broken**:
```r
#' @seealso \code{\link{other_function}}
#' @seealso \code{\link[otherpackage]{their_function}}
```

**19. Non-standard installation**:
- Don't require user interaction during installation
- Use `configure` script for system dependencies
- Document requirements clearly

**20. Archived dependencies**:
- Check that all dependencies are on CRAN
- Update or remove archived dependencies

### 21-30. More Issues

**21. Data documentation missing**:
```r
#' Dataset Title
#'
#' @description Description of dataset
#'
#' @format A data frame with 100 rows and 5 variables:
#' \describe{
#'   \item{var1}{Description of var1}
#'   \item{var2}{Description of var2}
#' }
#' @source \url{https://example.com/data}
"dataset_name"
```

**22. NAMESPACE conflicts**:
```r
# Don't export everything
#' @keywords internal
internal_function <- function() {}
```

**23. Windows path issues**:
```r
# Use file.path() not paste
file.path("dir", "file.txt")  # Good
# Not: paste("dir", "file.txt", sep = "/")  # Bad
```

**24. Encoding issues**:
Add to DESCRIPTION:
```
Encoding: UTF-8
```

**25. Undeclared attachments**:
```r
# Don't use library() in package code
# Use :: or importFrom instead
dplyr::filter()  # Good
# Not: library(dplyr); filter()  # Bad
```

**26. Examples with \dontrun only**:
- Every function needs runnable examples
- Use `\donttest{}` instead of `\dontrun{}` when possible

**27. Biased package names**:
- Don't use "R" prefix (e.g., "Rpackage")
- Don't use misleading names
- Check name isn't trademarked

**28. Missing return values**:
```r
#' @returns A numeric vector of length n
#' @returns NULL (invisibly) if no output
```

**29. Improper TRUE/FALSE**:
```r
# Use TRUE/FALSE, not T/F
if (condition == TRUE)  # Good
# Not: if (condition == T)  # Bad
```

**30. Unconventional directory structure**:
- Follow standard R package structure
- No spaces in file names
- Use lowercase for .R files

### 31-50. Advanced Issues

**31-40**: Platform-specific code, parallel processing issues, memory leaks, C/C++ code problems, Fortran code issues, Java dependencies, system command calls, graphic device problems, locale-specific code, timezone issues

**41-50**: Partial argument matching, `1:length()` issues, `sapply()` problems, dependency version constraints, circular dependencies, namespace pollution, documentation warnings, alias issues, keyword problems, cross-reference errors

## Resubmission After Rejection

### Responding to CRAN Comments

If rejected, you'll get an email with issues. Fix them and resubmit:

```markdown
## Resubmission

This is a resubmission. In this version I have:

* Fixed the DESCRIPTION title to be in title case
* Added \value sections to all exported functions
* Ensured all examples run without errors
* Fixed the NOTE about global variables by using utils::globalVariables()

## Test environments
...
```

**Important**:
- Address ALL comments from CRAN
- Be polite and professional
- Explain what you changed
- Resubmit within 2 weeks

### Version Bumping

After rejection, bump patch version:
```
# First submission
Version: 0.1.0

# After rejection
Version: 0.1.0  # Keep same version if resubmitting same release

# Or bump if making significant changes
Version: 0.1.1
```

## Reverse Dependency Checking

If your package has downstream dependencies (packages that depend on yours):

```r
# Install revdepcheck
install.packages("revdepcheck")

# Check reverse dependencies
revdepcheck::revdep_check(num_workers = 4)

# View results
revdepcheck::revdep_report()

# If issues found, contact maintainers
```

## Post-Acceptance Tasks

### After CRAN Acceptance

```r
# 1. Create GitHub release
usethis::use_github_release()

# 2. Bump to development version
usethis::use_dev_version()

# This changes:
# Version: 0.1.0  ->  Version: 0.1.0.9000
```

### Announce Release

- Tweet about it
- Post on R-bloggers
- Update website
- Email interested users

### Monitor CRAN

Check CRAN check results daily:
```
https://cran.r-project.org/web/checks/check_results_PACKAGENAME.html
```

Fix any issues that arise on CRAN's servers.

## Common Pitfalls

### 1. Submitting with WARNINGs

**Problem**: Package has warnings but submitter thinks they're acceptable

**Fix**: CRAN will auto-reject. Fix ALL warnings before submission.

### 2. Not Testing on Windows

**Problem**: Package works on Mac/Linux but fails on Windows

**Fix**: Use `check_win_devel()` and GitHub Actions before submission.

### 3. Examples That Require Setup

**Problem**: Examples assume files/data/authentication exist

**Fix**: Examples must be self-contained or use `\dontrun{}`.

### 4. Not Responding Quickly

**Problem**: Wait too long to respond to CRAN

**Fix**: Respond within 2 weeks or submission is archived.

### 5. Arguing with CRAN

**Problem**: Trying to argue that a NOTE is acceptable

**Fix**: Either fix it or provide a clear, technical explanation in cran-comments.md.

### 6. Incomplete Documentation

**Problem**: Missing `@returns` or `@examples`

**Fix**: Every exported function needs both.

### 7. Version Confusion

**Problem**: Trying to submit 0.1.0 when CRAN has 0.1.0

**Fix**: Bump version to 0.1.1 or higher.

### 8. Large Package Size

**Problem**: Package is 10+ MB

**Fix**: Compress data, use external data, or justify in cran-comments.md.

### 9. Too Many Imports

**Problem**: Importing 20+ packages

**Fix**: Consider if all are necessary. Each import is a dependency burden.

### 10. Not Reading CRAN Policies

**Problem**: Violating policies unknowingly

**Fix**: Read https://cran.r-project.org/web/packages/policies.html

## Final Pre-Submission Checklist

- [ ] `devtools::check()` shows 0 errors, 0 warnings, 0 notes (or explained)
- [ ] `check_win_devel()` passes
- [ ] `rhub::check_for_cran()` or GitHub Actions passes
- [ ] All exported functions have `@returns` and `@examples`
- [ ] `urlchecker::url_check()` passes
- [ ] Version number > CRAN version (for updates)
- [ ] DESCRIPTION Title is title case, <65 chars, no period
- [ ] DESCRIPTION Description is properly formatted
- [ ] LICENSE file present and correct
- [ ] cran-comments.md created and up-to-date
- [ ] NEWS.md updated with changes
- [ ] All tests pass
- [ ] Examples run in <5 seconds each
- [ ] No writing to home directory
- [ ] Internet-using functions fail gracefully when offline
- [ ] Package size justified if >5 MB
- [ ] Reverse dependencies checked (if any)

Good luck with your CRAN submission!
