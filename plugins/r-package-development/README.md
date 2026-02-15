# R Package Development Plugin

A comprehensive Claude Code plugin for creating, testing, documenting, and releasing production-quality R packages.

## Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| **package-architect** | haiku | Router — entry point for all R package tasks |
| **scaffold-engineer** | sonnet | Creates package skeleton (DESCRIPTION, NAMESPACE, etc.) |
| **code-engineer** | sonnet | Writes R/ code with roxygen2 documentation |
| **test-engineer** | sonnet | Creates testthat test suites |
| **docs-engineer** | sonnet | Vignettes, README, pkgdown websites |
| **check-doctor** | sonnet | Diagnoses and fixes R CMD check issues |
| **release-manager** | sonnet | CRAN/Bioconductor submission preparation |

## Commands

| Command | Description |
|---------|-------------|
| `/create-package` | Full package creation workflow |
| `/add-function` | Add a function with tests and documentation |
| `/add-data` | Add a bundled dataset to the package |
| `/setup-website` | Configure pkgdown + GitHub Pages |
| `/run-check` | Run R CMD check and auto-fix issues |
| `/prepare-release` | Prepare for CRAN/Bioconductor submission |

## Skills (12)

Knowledge base covering all aspects of R package development:

- **package-structure** — DESCRIPTION, NAMESPACE, .Rbuildignore, directory layout
- **r-code-patterns** — No library(), state management, ASCII-only, defensive coding
- **roxygen2-documentation** — @param, @export, @examples, @importFrom patterns
- **testing-patterns** — testthat 3e, fixtures, snapshots, skip patterns
- **data-in-packages** — data/, data-raw/, inst/extdata/, R/sysdata.rda
- **dependencies** — Imports vs Suggests vs Depends, NAMESPACE workflow
- **pkgdown-website** — _pkgdown.yml, Bootstrap 5, GitHub Pages deployment
- **github-actions** — R-CMD-check, pkgdown, test-coverage workflows
- **cran-submission** — CRAN policies, pre-submission checklist, cran-comments.md
- **bioconductor-submission** — BiocCheck, biocViews, S4 classes, vignette requirements
- **licensing** — MIT, GPL, Apache, CC compatibility
- **hex-logo** — hexSticker design patterns

## Based On

- [R Packages (2e)](https://r-pkgs.org/) by Hadley Wickham & Jenny Bryan
- Hands-on experience building CRAN-ready R packages with 0/0/0 check results
