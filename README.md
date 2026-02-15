# RPkgAgent

> **R Package Development Agent** -- 7 specialized agents, 6 workflow commands, and 12 methodology skills for building production-quality R packages

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) plugin that guides you through the entire R package lifecycle -- from `usethis::create_package()` to `devtools::submit_cran()`. Built on best practices from [R Packages (2e)](https://r-pkgs.org/) by Hadley Wickham & Jenny Bryan.

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

### 3. Add Functions, Tests, and Docs

```
/add-function
/run-check
/setup-website
/prepare-release
```

---

## What It Does

| Stage | What the Agent Handles |
|-------|----------------------|
| **Scaffold** | DESCRIPTION, NAMESPACE, LICENSE, .Rbuildignore, .gitignore, test infrastructure |
| **Code** | R/ functions with roxygen2 docs, `pkg::fun()` imports, state management, ASCII-only source |
| **Test** | testthat 3e suites with fixtures, edge cases, error conditions, snapshots, skip patterns |
| **Document** | README with badges, vignettes (html_vignette), pkgdown site (Bootstrap 5), hex logo, NEWS.md, CITATION |
| **Check** | Systematic diagnosis and fix of all R CMD check errors, warnings, and notes toward 0/0/0 |
| **Release** | Version bumping, cran-comments.md, multi-platform checks (win-builder, R-hub, GHA), CRAN/Bioconductor submission |

---

## Commands

| Command | Description | Agents Involved |
|---------|-------------|-----------------|
| `/create-package` | Full package creation workflow: gather requirements, scaffold structure, add code, tests, docs, and run check | package-architect, scaffold-engineer, code-engineer, test-engineer, docs-engineer, check-doctor |
| `/add-function` | Add a new exported or internal function with roxygen2 documentation, testthat tests, and working examples | code-engineer, test-engineer |
| `/add-data` | Bundle a dataset: create `data-raw/` preparation script, generate `.rda` file, write `R/data.R` documentation | code-engineer, docs-engineer |
| `/setup-website` | Configure pkgdown with `_pkgdown.yml` (Bootstrap 5, light/dark toggle, organized reference), hex logo, and GitHub Actions deployment workflow | docs-engineer |
| `/run-check` | Run `devtools::check()`, categorize issues (errors > warnings > notes), apply targeted fixes, re-check until 0/0/0 | check-doctor |
| `/prepare-release` | Pre-submission checklist, version bump, NEWS.md update, cran-comments.md, multi-platform validation, submission guidance | release-manager |

---

## Agents

### package-architect (haiku) -- Router

Entry point for all tasks. Gathers requirements through structured questions (package name, purpose, author info, license, dependencies, target repository) and delegates to the right specialist. Coordinates multi-step workflows across agents.

### scaffold-engineer (sonnet) -- Package Skeleton

Creates the complete package structure from scratch:

- `DESCRIPTION` with proper `Authors@R` (person()), Title Case title, formatted Description
- `LICENSE` + `LICENSE.md` (MIT, GPL-3, Apache-2.0, CC0)
- `R/pkgname-package.R` with `"_PACKAGE"` documentation and `globalVariables()`
- `.Rbuildignore`, `.gitignore` with standard patterns
- `tests/testthat.R` + `tests/testthat/helper-data.R`
- Git initialization

### code-engineer (sonnet) -- R Code

Writes R functions following strict package rules:

- No `library()`, `require()`, `source()`, `:::`, `T`/`F`, non-ASCII, `setwd()`
- `pkg::fun()` for Imports, `requireNamespace()` for Suggests
- Complete roxygen2: `@param` for every argument, `@returns`, `@export`, `@examples`
- State management with `withr::local_*()` or `on.exit()`
- `utils::globalVariables()` for data.table/ggplot2 NSE

### test-engineer (sonnet) -- Testing

Creates testthat 3rd edition test suites:

- `test-*.R` files mirroring `R/` structure
- Happy path, edge cases (empty, single, large, all-NA), error conditions
- `expect_equal()`, `expect_error()`, `expect_snapshot()`, `expect_s3_class()`
- `skip_if_not_installed()`, `skip_on_cran()` for conditional tests
- Shared fixtures in `helper-data.R`

### docs-engineer (sonnet) -- Documentation

Produces all package documentation:

- **README.md** with badges (R-CMD-check, Codecov, CRAN, License, ORCID), installation, quick start
- **Vignettes** using `rmarkdown::html_vignette` with runnable examples
- **pkgdown** `_pkgdown.yml`: Bootstrap 5, light-switch, Google Fonts, organized reference sections, navbar, footer
- **Hex logo** via `hexSticker` with accessible color palettes
- **NEWS.md** with semantic sections (New Features, Bug Fixes, Breaking Changes)
- **inst/CITATION** with proper `bibentry()`
- **R/data.R** dataset documentation with `@format` and `\describe{}`

### check-doctor (sonnet) -- R CMD Check

Diagnoses and fixes all check issues:

| Category | Common Issues |
|----------|--------------|
| **Errors** | Missing imports, undocumented arguments, example failures, test failures |
| **Warnings** | Missing `@returns`, non-standard license, S3 method inconsistency |
| **Notes** | No visible binding (NSE), undefined globals, package size, non-ASCII, unused Imports, LazyData without data |

Handles platform-specific issues (Windows paths, macOS Xcode, Linux system dependencies) and `.Rbuildignore` configuration.

### release-manager (sonnet) -- CRAN/Bioconductor

Manages the submission process:

- **Version bumping**: `major.minor.patch` semantics, `.9000` dev versions
- **Pre-submission checklist**: 30+ items covering code, DESCRIPTION, docs, tests, multi-platform
- **cran-comments.md**: Test environments, downstream dependencies, resubmission notes
- **Multi-platform testing**: win-builder, R-hub, GitHub Actions matrix (5 configs)
- **Reverse dependency checks** with `revdepcheck`
- **Bioconductor**: BiocCheck, biocViews, version scheme, S4 classes, runnable vignettes

---

## Skills (Knowledge Base)

12 comprehensive methodology files that agents reference when working:

| Skill | What It Covers |
|-------|---------------|
| **package-structure** | 5 package states, required/optional files, directory layout, `.Rbuildignore` patterns |
| **r-code-patterns** | Forbidden patterns (`library()`, `:::`, `T`/`F`), `pkg::fun()` usage, state management with withr, ASCII-only source |
| **roxygen2-documentation** | All tags (`@param`, `@returns`, `@export`, `@examples`, `@family`, `@inheritParams`), Markdown support, auto-linking, cross-references |
| **testing-patterns** | testthat 3e config, `test_that()` patterns, expectations, snapshot tests, fixtures, `skip_*()` functions, coverage goals |
| **data-in-packages** | `data/` (exported), `R/sysdata.rda` (internal), `inst/extdata/` (raw files), `data-raw/` scripts, `LazyData`, CRAN size limits (5 MB), `tools::resaveRdaFiles()` |
| **dependencies** | Imports vs Suggests vs Depends vs LinkingTo, NAMESPACE workflow, `@importFrom` vs `::`, `requireNamespace()` for Suggests |
| **pkgdown-website** | `_pkgdown.yml` config (Bootstrap 5, `light-switch`, Google Fonts, navbar, reference sections, footer), GitHub Pages deployment, custom CSS |
| **github-actions** | R-CMD-check matrix (ubuntu R-release/R-devel/R-oldrel + windows + macOS), pkgdown deployment, test-coverage with Codecov |
| **cran-submission** | CRAN policies (no more than 1 submission per 1-2 weeks), `devtools::submit_cran()`, `\dontrun{}` vs `\donttest{}`, file system rules, version numbering |
| **bioconductor-submission** | BiocCheck, `biocViews` vocabulary, even/odd version scheme, `vapply()` over `sapply()`, runnable vignettes with BiocStyle, S4 preference |
| **licensing** | MIT (+file), GPL-2/3, Apache-2.0, CC0, compatibility matrix, bundled code licensing, COPYRIGHT file |
| **hex-logo** | `hexSticker` API, Google Fonts via `showtext`, accessible color palettes, `man/figures/logo.png` placement, SVG export |

---

## Repository Structure

```
RPkgAgent/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── r-package-development/
│       ├── README.md
│       ├── agents/
│       │   ├── package-architect.md
│       │   ├── scaffold-engineer.md
│       │   ├── code-engineer.md
│       │   ├── test-engineer.md
│       │   ├── docs-engineer.md
│       │   ├── check-doctor.md
│       │   └── release-manager.md
│       ├── commands/
│       │   ├── create-package.md
│       │   ├── add-function.md
│       │   ├── add-data.md
│       │   ├── setup-website.md
│       │   ├── run-check.md
│       │   └── prepare-release.md
│       └── skills/
│           ├── package-structure/SKILL.md
│           ├── r-code-patterns/SKILL.md
│           ├── roxygen2-documentation/SKILL.md
│           ├── testing-patterns/SKILL.md
│           ├── data-in-packages/SKILL.md
│           ├── dependencies/SKILL.md
│           ├── pkgdown-website/SKILL.md
│           ├── github-actions/SKILL.md
│           ├── cran-submission/SKILL.md
│           ├── bioconductor-submission/SKILL.md
│           ├── licensing/SKILL.md
│           └── hex-logo/SKILL.md
├── README.md
└── LICENSE
```

---

## Usage Examples

### Create a new package from scratch

```
I want to create an R package called "csvtools" that provides
functions for reading and writing CSV files with automatic type
detection. Target CRAN, use MIT license.
```

### Add a function to an existing package

```
Add a function called read_csv_smart() that reads a CSV file
and automatically detects column types. It should take file path,
delimiter, and na arguments.
```

### Bundle a dataset

```
Add the iris dataset as sample data to my package with full
documentation including @format and @source tags.
```

### Set up a pkgdown website

```
Set up a pkgdown website for my package with Bootstrap 5,
light/dark mode, and deploy to GitHub Pages.
```

### Fix R CMD check issues

```
I'm getting these warnings from devtools::check():
- Missing \value for 'read_csv_smart'
- no visible binding for global variable 'col_name'
Help me fix them.
```

### Prepare for CRAN submission

```
My package passes local checks with 0/0/0.
Help me prepare for CRAN submission with multi-platform
testing and cran-comments.md.
```

---

## R Dependencies

The agent uses standard R development tools. Install as needed:

```r
install.packages(c(
  "devtools",     # Package development workflow
  "usethis",      # Package setup helpers
  "roxygen2",     # Documentation generation
  "testthat",     # Testing framework
  "pkgdown",      # Website builder
  "rcmdcheck",    # Enhanced R CMD check
  "covr",         # Code coverage
  "hexSticker",   # Hex logo generation
  "rhub",         # Multi-platform checking
  "urlchecker",   # URL validation
  "spelling"      # Spell checking
))
```

---

## Built With Knowledge From

- [R Packages (2e)](https://r-pkgs.org/) by Hadley Wickham & Jenny Bryan
- [Writing R Extensions](https://cran.r-project.org/doc/manuals/R-exts.html) (CRAN official manual)
- [testthat](https://testthat.r-lib.org/) 3rd edition documentation
- [pkgdown](https://pkgdown.r-lib.org/) website builder documentation
- [CRAN Repository Policy](https://cran.r-project.org/web/packages/policies.html)
- [Bioconductor Package Guidelines](https://contributions.bioconductor.org/index.html)
- Hands-on experience building CRAN-ready R packages with 0/0/0 check results

## License

MIT License -- see [LICENSE](LICENSE) for details.
