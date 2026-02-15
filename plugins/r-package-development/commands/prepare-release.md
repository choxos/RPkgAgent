---
name: prepare-release
description: Prepare package for CRAN or Bioconductor submission with all required checks
---

# Prepare Package Release

This command prepares your R package for submission to CRAN or Bioconductor, running all required checks and creating necessary documentation.

## Workflow

### Step 1: Determine Release Target

Ask the user:

- **CRAN**: General-purpose R packages
- **Bioconductor**: Genomics/bioinformatics packages
- **Internal**: Private repository only

Requirements differ significantly between targets.

## CRAN Release Path

### Step 2: Bump Version Number

Determine version increment using semantic versioning:

- **Major** (1.0.0 → 2.0.0): Breaking changes
- **Minor** (1.0.0 → 1.1.0): New features, backwards compatible
- **Patch** (1.0.0 → 1.0.1): Bug fixes only

Use usethis helper:

```r
# For release to CRAN
usethis::use_version("minor")  # or "major", "patch"
```

This updates:

- `DESCRIPTION` Version field
- Prompts to update `NEWS.md`

Or manually update `DESCRIPTION`:

```
Version: 0.1.0  # First release
Version: 0.1.1  # Patch release
Version: 1.0.0  # Major release
```

### Step 3: Update NEWS.md

Document all changes since last version:

```markdown
# packagename 0.1.0

## New features

* Added `clean_text()` for text preprocessing (#12)
* Added `remove_urls()` to strip URLs from text (#15)
* Included `customer_data` example dataset

## Bug fixes

* Fixed encoding issue in `clean_text()` (#18)
* Corrected documentation for `remove_stopwords()` (#20)

## Minor improvements

* Improved performance of `count_words()` by 30%
* Updated vignettes with more examples
* Added hex logo and pkgdown website
```

Format:

- Use `# packagename version` for the header
- Group changes: New features, Bug fixes, Breaking changes, etc.
- Reference GitHub issues with `(#123)`
- User-facing changes only (not internal refactoring)

### Step 4: Verify Documentation Quality

Check all exported functions have:

**Required roxygen tags**:

- `@param` for every parameter
- `@returns` describing return value (NOT `@return`)
- `@examples` with working, executable code
- `@export` if user-facing

Run:

```r
# Check for missing documentation
pkgload::dev_topic_index()  # Lists all documented topics

# Check specific function
?function_name
```

**Examples requirements**:

- Must run without errors
- Should demonstrate typical usage
- Avoid `\dontrun{}` if possible (CRAN dislikes it)
- Use `\donttest{}` for slow examples (>5 sec)
- Examples with external dependencies:

```r
#' @examples
#' if (requireNamespace("optionalpackage", quietly = TRUE)) {
#'   function_name()
#' }
```

### Step 5: Check URLs

Run URL checker to find broken links:

```r
urlchecker::url_check()
```

This checks URLs in:

- README.md
- Documentation (.Rd files)
- Vignettes
- DESCRIPTION

Fix or remove broken links:

- Update changed URLs
- Remove dead links
- Use archived versions if appropriate

Common issues:

- DOIs: Use `https://doi.org/...` format
- GitHub links: Ensure repositories still exist
- Personal websites: Check still active

### Step 6: Run Local R CMD Check

Run comprehensive local check:

```r
devtools::check()
```

Must achieve: **0 errors | 0 warnings | 0 notes**

If any issues, fix with `/run-check` command first.

Pay special attention to:

- Examples run successfully
- Tests pass
- Vignettes build
- No undefined globals
- No undeclared imports

### Step 7: Run Remote Check with Manual

Check with additional options enabled:

```r
devtools::check(
  remote = TRUE,    # Use CRAN incoming standards
  manual = TRUE     # Build PDF manual
)
```

This:

- Checks as CRAN would
- Builds PDF manual (requires LaTeX)
- Verifies package can be installed from tarball

If PDF manual fails:

```r
# Install tinytex for PDF support
tinytex::install_tinytex()
```

### Step 8: Test on Windows (if on Mac/Linux)

CRAN tests on Windows, so you should too:

```r
devtools::check_win_devel()  # Windows R-devel
devtools::check_win_release() # Windows R-release
devtools::check_win_oldrelease() # Windows R-oldrelease
```

This:

- Uploads package to win-builder server
- Runs R CMD check on Windows
- Emails results (wait 10-30 minutes)

Check email for results, fix any Windows-specific issues.

### Step 9: Test on R-hub (Multiple Platforms)

Test on various operating systems and R versions:

```r
# Check on multiple platforms
rhub::check_for_cran()
```

Or specify platforms:

```r
rhub::check(
  platform = c(
    "ubuntu-gcc-release",
    "fedora-clang-devel",
    "macos-highsierra-release-cran"
  )
)
```

Wait for results (can take 30+ minutes), review for platform-specific issues.

### Step 10: Check Reverse Dependencies

If your package is already on CRAN and other packages depend on it:

```r
revdepcheck::revdep_check(num_workers = 4)
```

This checks that your changes don't break packages that depend on yours.

If first release, skip this step.

### Step 11: Spell Check

Check spelling in documentation:

```r
spelling::spell_check_package()
```

Add legitimate words to `inst/WORDLIST`:

```
packagename
preprocessing
tokenization
```

Fix actual typos in documentation.

### Step 12: Update cran-comments.md

Create or update `cran-comments.md`:

```markdown
## Resubmission

This is a resubmission. In this version I have:

* Fixed the Title field in DESCRIPTION (removed period)
* Added \value sections to all exported functions
* Reduced example runtime to under 5 seconds

## R CMD check results

0 errors | 0 warnings | 0 notes

## Downstream dependencies

I have run R CMD check on downstream dependencies.
All packages passed.
```

For first submission:

```markdown
## First submission

This is the first submission of packagename to CRAN.

## R CMD check results

0 errors | 0 warnings | 1 note

* This is a new release.

## Test environments

* local: macOS 13.0, R 4.3.1
* win-builder: Windows, R-devel and R-release
* R-hub: Ubuntu, Fedora, macOS

All checks passed successfully.
```

Add `^cran-comments\.md$` to `.Rbuildignore`.

### Step 13: Final Pre-Submission Checklist

Verify:

- [ ] Version bumped in DESCRIPTION
- [ ] NEWS.md updated with all changes
- [ ] All functions have `@param`, `@returns`, `@examples`
- [ ] Examples run without errors
- [ ] `devtools::check()`: 0 errors, 0 warnings, 0 notes
- [ ] `devtools::check(remote=TRUE, manual=TRUE)` passes
- [ ] `check_win_devel()` passes (check email)
- [ ] `urlchecker::url_check()` passes
- [ ] `spelling::spell_check_package()` clean
- [ ] cran-comments.md created
- [ ] LICENSE file present and correct
- [ ] All changes committed to git

### Step 14: Submit to CRAN

Use devtools to submit:

```r
devtools::submit_cran()
```

This:

- Builds the package tarball
- Uploads to CRAN
- Submits for review

**IMPORTANT**: Ask user for explicit confirmation before submitting:

"Ready to submit packagename 0.1.0 to CRAN. This will upload the package for review. Confirm?"

Wait for user: "yes", "confirmed", "proceed"

Alternative manual submission:

1. Build tarball: `devtools::build()`
2. Upload at: https://cran.r-project.org/submit.html
3. Confirm email submission

### Step 15: Post-Submission

After submission:

1. **Wait for email**: CRAN auto-checks (30 minutes to 2 hours)
2. **If issues found**: Fix and resubmit
3. **If accepted**:
   - Wait for human review (1-10 days)
   - Respond promptly to reviewer comments
   - Make requested changes and resubmit

### Step 16: Post-Acceptance

Once accepted to CRAN:

Create GitHub release:

```r
usethis::use_github_release()
```

Switch to development version:

```r
usethis::use_dev_version()
```

This bumps version to `0.1.0.9000` for continued development.

Update README with CRAN installation:

```r
install.packages("packagename")
```

## Bioconductor Release Path

For Bioconductor packages, the process differs:

### Step 2 (Bioconductor): Run BiocCheck

```r
BiocCheck::BiocCheck(".")
```

This checks Bioconductor-specific requirements:

- Version numbering (x.y.z where y is even for release)
- biocViews terms in DESCRIPTION
- Vignette requirements
- Coding style
- S4 class usage

### Step 3 (Bioconductor): Verify biocViews

DESCRIPTION must include biocViews:

```
biocViews: Preprocessing, Sequencing, RNASeq, QualityControl
```

Choose from: https://bioconductor.org/packages/release/BiocViews.html

### Step 4 (Bioconductor): Version Scheme

Use Bioconductor versioning:

- Development: x.y.z where y is odd (e.g., 1.1.0)
- Release: x.y.z where y is even (e.g., 1.2.0)

```
Version: 0.99.0  # Pre-release (for new submissions)
```

### Step 5 (Bioconductor): Submit via GitHub

1. Create repository on GitHub
2. Add to Bioconductor:
   - Open issue: https://github.com/Bioconductor/Contributions/issues/new
   - Follow new package template
   - Provide repository URL

Bioconductor will:

- Review package automatically
- Provide feedback
- Accept when requirements met

Report to user:

```
✓ Package ready for CRAN submission!

Pre-submission checks:
  - Version: 0.1.0
  - R CMD check: 0 errors ✓ | 0 warnings ✓ | 0 notes ✓
  - Windows check: PASS ✓
  - URL check: All links valid ✓
  - Spell check: Clean ✓

Documentation:
  - All 12 exported functions have complete docs
  - All functions have @returns and @examples
  - NEWS.md updated
  - cran-comments.md created

Test platforms:
  - Local macOS: PASS
  - win-builder (devel): PASS
  - win-builder (release): PASS
  - R-hub Ubuntu: PASS

Ready to submit!

Next step: Run devtools::submit_cran() when you're ready.
After submission, watch your email for CRAN auto-check results.
```

## Example Interaction

User: "Prepare my package for CRAN release"