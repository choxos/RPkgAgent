---
layout: default
title: RPkgAgent
---

# RPkgAgent

**R Package Development Agent** -- 7 specialized agents, 6 workflow commands, and 12 methodology skills for building production-quality R packages.

A [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) plugin that guides you through the entire R package lifecycle -- from `usethis::create_package()` to `devtools::submit_cran()`. Built on best practices from [R Packages (2e)](https://r-pkgs.org/) by Hadley Wickham & Jenny Bryan.

[View on GitHub](https://github.com/choxos/RPkgAgent){: .btn }

---

## Quick Start

### 1. Install the Plugin

```bash
/plugin marketplace add choxos/RPkgAgent
/plugin install r-package-development
```

### 2. Create a Package

```
/create-package
```

The agent will ask for your package name, purpose, author info, license, and dependencies, then scaffold the entire structure.

### 3. Build Your Package

```
/add-function
/run-check
/setup-website
/prepare-release
```

---

## Pipeline

| Stage | What the Agent Handles |
|:------|:----------------------|
| **Scaffold** | DESCRIPTION, NAMESPACE, LICENSE, .Rbuildignore, .gitignore, test infrastructure |
| **Code** | R/ functions with roxygen2 docs, `pkg::fun()` imports, state management, ASCII-only source |
| **Test** | testthat 3e suites with fixtures, edge cases, error conditions, snapshots, skip patterns |
| **Document** | README with badges, vignettes, pkgdown site (Bootstrap 5), hex logo, NEWS.md, CITATION |
| **Check** | Systematic diagnosis and fix of all R CMD check errors, warnings, and notes toward 0/0/0 |
| **Release** | Version bumping, cran-comments.md, multi-platform checks, CRAN/Bioconductor submission |

---

## Commands

| Command | Description |
|:--------|:------------|
| `/create-package` | Full package creation workflow: scaffold, code, tests, docs, check |
| `/add-function` | Add a new function with roxygen2 docs, tests, and examples |
| `/add-data` | Bundle a dataset with `data-raw/` script and `R/data.R` documentation |
| `/setup-website` | Configure pkgdown with Bootstrap 5, hex logo, and GitHub Pages deployment |
| `/run-check` | Run `devtools::check()`, diagnose issues, auto-fix until 0/0/0 |
| `/prepare-release` | Pre-submission checklist, version bump, multi-platform checks, submission |

---

## Agents

### package-architect (haiku) -- Router

Entry point for all tasks. Gathers requirements through structured questions and delegates to the right specialist. Coordinates multi-step workflows across agents.

### scaffold-engineer (sonnet) -- Package Skeleton

Creates the complete package structure: DESCRIPTION with proper `Authors@R`, LICENSE files, `R/pkgname-package.R`, `.Rbuildignore`, `.gitignore`, and test infrastructure.

### code-engineer (sonnet) -- R Code

Writes R functions following strict package rules: no `library()`, `require()`, `source()`, `:::`, `T`/`F`, non-ASCII, or `setwd()`. Uses `pkg::fun()` for Imports and complete roxygen2 documentation.

### test-engineer (sonnet) -- Testing

Creates testthat 3rd edition test suites with happy path, edge cases, error conditions, snapshot tests, conditional skips, and shared fixtures.

### docs-engineer (sonnet) -- Documentation

Produces README with badges, vignettes, pkgdown `_pkgdown.yml` (Bootstrap 5, light/dark toggle), hex logo via hexSticker, NEWS.md, inst/CITATION, and dataset documentation.

### check-doctor (sonnet) -- R CMD Check

Diagnoses and fixes all check issues systematically: missing imports, undocumented arguments, NSE bindings, package size, non-ASCII characters, unused Imports, and platform-specific problems.

### release-manager (sonnet) -- CRAN/Bioconductor

Manages version bumping, pre-submission checklist (30+ items), cran-comments.md, multi-platform testing (win-builder, R-hub, GHA matrix), reverse dependency checks, and Bioconductor-specific requirements.

---

## Skills (Knowledge Base)

12 comprehensive methodology files that agents reference:

| Skill | Coverage |
|:------|:---------|
| **package-structure** | Package states, required files, directory layout, `.Rbuildignore` |
| **r-code-patterns** | Forbidden patterns, `pkg::fun()`, state management with withr |
| **roxygen2-documentation** | All tags, Markdown support, auto-linking, cross-references |
| **testing-patterns** | testthat 3e, expectations, snapshots, fixtures, skip functions |
| **data-in-packages** | `data/`, `R/sysdata.rda`, `inst/extdata/`, `data-raw/`, size limits |
| **dependencies** | Imports vs Suggests vs Depends, NAMESPACE, `@importFrom` vs `::` |
| **pkgdown-website** | Bootstrap 5, light-switch, Google Fonts, GitHub Pages deployment |
| **github-actions** | R-CMD-check matrix, pkgdown deployment, test-coverage with Codecov |
| **cran-submission** | CRAN policies, `devtools::submit_cran()`, version numbering |
| **bioconductor-submission** | BiocCheck, biocViews, version scheme, S4, runnable vignettes |
| **licensing** | MIT, GPL, Apache, CC0, compatibility matrix, bundled code |
| **hex-logo** | hexSticker API, Google Fonts, accessible colors, SVG export |

---

## Usage Examples

**Create a new package:**
```
I want to create an R package called "csvtools" that provides
functions for reading CSV files. Target CRAN, use MIT license.
```

**Add a function:**
```
Add a function called read_csv_smart() that reads a CSV file
and automatically detects column types.
```

**Fix R CMD check issues:**
```
I'm getting warnings from devtools::check():
- Missing \value for 'read_csv_smart'
- no visible binding for global variable 'col_name'
```

**Prepare for CRAN:**
```
My package passes local checks with 0/0/0.
Help me prepare for CRAN submission.
```

---

## R Dependencies

```r
install.packages(c(
  "devtools", "usethis", "roxygen2", "testthat",
  "pkgdown", "rcmdcheck", "covr", "hexSticker",
  "rhub", "urlchecker", "spelling"
))
```

---

## References

- [R Packages (2e)](https://r-pkgs.org/) by Hadley Wickham & Jenny Bryan
- [Writing R Extensions](https://cran.r-project.org/doc/manuals/R-exts.html)
- [testthat](https://testthat.r-lib.org/) 3rd edition
- [pkgdown](https://pkgdown.r-lib.org/)
- [CRAN Repository Policy](https://cran.r-project.org/web/packages/policies.html)
- [Bioconductor Package Guidelines](https://contributions.bioconductor.org/index.html)

---

**License:** MIT -- see [LICENSE](https://github.com/choxos/RPkgAgent/blob/main/LICENSE) for details.
