# rpkg_agent

A Claude Code plugin for building production-quality R packages. Provides 7 specialized agents, 6 workflow commands, and 12 methodology skills covering the full R package lifecycle from creation to CRAN/Bioconductor submission.

## Installation

```bash
claude plugin add choxos/rpkg_agent
```

## What It Does

- **Creates R packages** from scratch with proper structure (DESCRIPTION, NAMESPACE, tests, documentation)
- **Writes R code** following R Packages book best practices (no `library()`, proper `@importFrom`, state management)
- **Generates tests** using testthat 3rd edition with edge cases, error conditions, and fixtures
- **Builds documentation** including roxygen2, vignettes, pkgdown websites, and hex logos
- **Fixes R CMD check** issues systematically to achieve 0 errors, 0 warnings, 0 notes
- **Prepares CRAN/Bioconductor submissions** with multi-platform checks and proper cran-comments.md

## Commands

| Command | What It Does |
|---------|-------------|
| `/create-package` | Full package creation workflow (scaffold + code + tests + docs + check) |
| `/add-function` | Add a new function with roxygen2 docs, tests, and examples |
| `/add-data` | Bundle a dataset with data-raw/ script and documentation |
| `/setup-website` | Configure pkgdown site with GitHub Pages deployment |
| `/run-check` | Run R CMD check, diagnose issues, and auto-fix |
| `/prepare-release` | Version bump, NEWS.md, multi-platform checks, CRAN submission |

## Agents

| Agent | Role |
|-------|------|
| `package-architect` | Router that gathers requirements and delegates to specialists |
| `scaffold-engineer` | Creates the package skeleton and file structure |
| `code-engineer` | Writes R functions with proper roxygen2 documentation |
| `test-engineer` | Creates testthat test suites with comprehensive coverage |
| `docs-engineer` | Writes vignettes, README, pkgdown config, hex logos |
| `check-doctor` | Diagnoses and fixes all R CMD check issues |
| `release-manager` | Handles CRAN/Bioconductor submission preparation |

## Skills (Knowledge Base)

12 comprehensive skill files covering:

- Package structure and DESCRIPTION/NAMESPACE configuration
- R code patterns (forbidden patterns, dependency usage, state management)
- roxygen2 documentation syntax and best practices
- testthat 3rd edition testing patterns
- Bundling data in packages (data/, data-raw/, inst/extdata/)
- Managing Imports vs Suggests vs Depends
- pkgdown website configuration (Bootstrap 5, GitHub Pages)
- GitHub Actions CI/CD workflows (R-CMD-check, pkgdown, coverage)
- CRAN submission policies and pre-submission checklist
- Bioconductor submission requirements (BiocCheck, biocViews, S4)
- Licensing (MIT, GPL, Apache, CC compatibility)
- Hex logo design with hexSticker

## Built With Knowledge From

- [R Packages (2e)](https://r-pkgs.org/) by Hadley Wickham & Jenny Bryan
- Hands-on experience building CRAN-ready R packages

## License

MIT
