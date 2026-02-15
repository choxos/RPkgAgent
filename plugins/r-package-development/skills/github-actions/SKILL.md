---
name: github-actions
description: Automated CI/CD workflows for R packages including R CMD check across platforms, test coverage reporting, and pkgdown deployment
---

# GitHub Actions for R Package Development

This skill covers setting up comprehensive GitHub Actions workflows for R package development, including cross-platform testing, continuous integration, automated documentation deployment, and test coverage reporting.

## Rules

1. **Use r-lib/actions v2** for all R-related workflow steps
2. **Test on multiple platforms**: macOS, Windows, Ubuntu (latest + devel + oldrel)
3. **Always include permissions** blocks with minimal required scopes
4. **Use concurrency groups** to cancel outdated workflow runs
5. **Cache dependencies** for faster builds (handled by setup-r-dependencies)
6. **Set GITHUB_PAT** environment variable for private repos or rate limits
7. **Use checkout@v4** (not v2 or v3) for repository checkout
8. **Deploy pkgdown only on push** to main/master, not on PRs
9. **Report test coverage** to Codecov on successful test runs
10. **Use workflow_dispatch** for manual workflow triggering

## Essential Workflows

Every R package should have these three workflows:
1. **R-CMD-check.yaml** - Cross-platform package checking
2. **pkgdown.yaml** - Documentation website deployment
3. **test-coverage.yaml** - Code coverage reporting

## 1. R CMD Check Workflow

### Complete R-CMD-check.yaml

Create `.github/workflows/R-CMD-check.yaml`:

```yaml
# Workflow derived from https://github.com/r-lib/actions/tree/v2/examples
# Need help debugging build failures? Start at https://github.com/r-lib/actions#where-to-find-help
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

name: R-CMD-check

permissions: read-all

jobs:
  R-CMD-check:
    runs-on: ${{ matrix.config.os }}

    name: ${{ matrix.config.os }} (${{ matrix.config.r }})

    strategy:
      fail-fast: false
      matrix:
        config:
          - {os: macos-latest,   r: 'release'}
          - {os: windows-latest, r: 'release'}
          - {os: ubuntu-latest,  r: 'devel', http-user-agent: 'release'}
          - {os: ubuntu-latest,  r: 'release'}
          - {os: ubuntu-latest,  r: 'oldrel-1'}

    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
      R_KEEP_PKG_SOURCE: yes

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Pandoc
        uses: r-lib/actions/setup-pandoc@v2

      - name: Setup R
        uses: r-lib/actions/setup-r@v2
        with:
          r-version: ${{ matrix.config.r }}
          http-user-agent: ${{ matrix.config.http-user-agent }}
          use-public-rspm: true

      - name: Install system dependencies
        if: runner.os == 'Linux'
        run: |
          sudo apt-get update -y
          sudo apt-get install -y libcurl4-openssl-dev

      - name: Install dependencies
        uses: r-lib/actions/setup-r-dependencies@v2
        with:
          extra-packages: any::rcmdcheck
          needs: check

      - name: Check package
        uses: r-lib/actions/check-r-package@v2
        with:
          upload-snapshots: true
          build_args: 'c("--no-manual","--compact-vignettes=gs+qpdf")'
```

### Minimal R-CMD-check.yaml

For simpler packages, you can use fewer test configurations:

```yaml
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

name: R-CMD-check

permissions: read-all

jobs:
  R-CMD-check:
    runs-on: ${{ matrix.config.os }}
    name: ${{ matrix.config.os }} (${{ matrix.config.r }})
    strategy:
      fail-fast: false
      matrix:
        config:
          - {os: macos-latest,   r: 'release'}
          - {os: windows-latest, r: 'release'}
          - {os: ubuntu-latest,  r: 'release'}

    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
      R_KEEP_PKG_SOURCE: yes

    steps:
      - uses: actions/checkout@v4
      - uses: r-lib/actions/setup-pandoc@v2
      - uses: r-lib/actions/setup-r@v2
        with:
          r-version: ${{ matrix.config.r }}
          use-public-rspm: true
      - uses: r-lib/actions/setup-r-dependencies@v2
        with:
          extra-packages: any::rcmdcheck
          needs: check
      - uses: r-lib/actions/check-r-package@v2
```

### With System Dependencies

For packages with system dependencies (e.g., gdal, jags, gsl):

```yaml
      - name: Install system dependencies on macOS
        if: runner.os == 'macOS'
        run: |
          brew install gdal proj geos

      - name: Install system dependencies on Linux
        if: runner.os == 'Linux'
        run: |
          sudo apt-get update
          sudo apt-get install -y \
            libgdal-dev \
            libproj-dev \
            libgeos-dev \
            libudunits2-dev

      - name: Install system dependencies on Windows
        if: runner.os == 'Windows'
        run: |
          # Usually handled by rtools or binary packages
```

### Build Arguments Explained

```yaml
- uses: r-lib/actions/check-r-package@v2
  with:
    upload-snapshots: true
    build_args: 'c("--no-manual","--compact-vignettes=gs+qpdf")'
    error-on: '"error"'  # Can be "warning", "note", or "error"
```

**build_args options**:
- `--no-manual`: Skip PDF manual creation (faster, requires less LaTeX)
- `--compact-vignettes=gs+qpdf`: Compress vignettes with Ghostscript and qpdf
- `--no-build-vignettes`: Skip vignette building entirely
- `--resave-data`: Optimize data file compression

## 2. pkgdown Deployment Workflow

### Complete pkgdown.yaml

Create `.github/workflows/pkgdown.yaml`:

```yaml
# Workflow derived from https://github.com/r-lib/actions/tree/v2/examples
# Need help debugging build failures? Start at https://github.com/r-lib/actions#where-to-find-help
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]
  release:
    types: [published]
  workflow_dispatch:

name: pkgdown

# Limit concurrency: cancel in-progress runs when new ones are triggered
concurrency:
  group: pkgdown-${{ github.event_name != 'pull_request' || github.run_id }}

permissions:
  contents: write

jobs:
  pkgdown:
    runs-on: ubuntu-latest
    # Only restrict concurrency for non-PR jobs
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Pandoc
        uses: r-lib/actions/setup-pandoc@v2

      - name: Setup R
        uses: r-lib/actions/setup-r@v2
        with:
          use-public-rspm: true

      - name: Install dependencies
        uses: r-lib/actions/setup-r-dependencies@v2
        with:
          extra-packages: any::pkgdown, local::.
          needs: website

      - name: Build site
        run: pkgdown::build_site_github_pages(new_process = FALSE, install = FALSE)
        shell: Rscript {0}

      - name: Deploy to GitHub Pages
        if: github.event_name != 'pull_request'
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          folder: docs
          branch: gh-pages
          clean: true
          single-commit: false
```

### Using deploy_to_branch Alternative

```yaml
      - name: Build and deploy
        if: github.event_name != 'pull_request'
        run: |
          git config --local user.name "$GITHUB_ACTOR"
          git config --local user.email "$GITHUB_ACTOR@users.noreply.github.com"
          Rscript -e 'pkgdown::deploy_to_branch(new_process = FALSE)'
```

### Build on PR (Preview Only)

```yaml
      - name: Build site for preview
        if: github.event_name == 'pull_request'
        run: |
          pkgdown::build_site(preview = FALSE, install = FALSE)
        shell: Rscript {0}

      - name: Upload preview
        if: github.event_name == 'pull_request'
        uses: actions/upload-artifact@v3
        with:
          name: pkgdown-preview
          path: docs/
```

## 3. Test Coverage Workflow

### Complete test-coverage.yaml

Create `.github/workflows/test-coverage.yaml`:

```yaml
# Workflow derived from https://github.com/r-lib/actions/tree/v2/examples
# Need help debugging build failures? Start at https://github.com/r-lib/actions#where-to-find-help
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

name: test-coverage

permissions: read-all

jobs:
  test-coverage:
    runs-on: ubuntu-latest
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup R
        uses: r-lib/actions/setup-r@v2
        with:
          use-public-rspm: true

      - name: Install dependencies
        uses: r-lib/actions/setup-r-dependencies@v2
        with:
          extra-packages: any::covr, any::xml2
          needs: coverage

      - name: Test coverage
        run: |
          cov <- covr::package_coverage(
            quiet = FALSE,
            clean = FALSE,
            install_path = file.path(normalizePath(Sys.getenv("RUNNER_TEMP"), winslash = "/"), "package")
          )
          covr::to_cobertura(cov)
        shell: Rscript {0}

      - name: Upload coverage reports to Codecov
        uses: codecov/codecov-action@v4
        with:
          fail_ci_if_error: true
          files: ./cobertura.xml
          token: ${{ secrets.CODECOV_TOKEN }}
          verbose: true

      - name: Show testthat output
        if: always()
        run: |
          ## --------------------------------------------------------------------
          find '${{ runner.temp }}/package' -name 'testthat.Rout*' -exec cat '{}' \; || true
        shell: bash
```

### Codecov Setup

1. Sign up at https://codecov.io with your GitHub account
2. Add your repository
3. Add `CODECOV_TOKEN` to GitHub secrets (Settings > Secrets and variables > Actions)
4. Add badge to README.md:

```markdown
[![codecov](https://codecov.io/gh/username/packagename/branch/main/graph/badge.svg)](https://codecov.io/gh/username/packagename)
```

### Alternative: Coveralls

```yaml
      - name: Upload coverage to Coveralls
        uses: coverallsapp/github-action@v2
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          path-to-lcov: ./coverage.lcov
```

## Additional Useful Workflows

### 4. PR Commands Workflow

Respond to comments like `/document` or `/style`:

Create `.github/workflows/pr-commands.yaml`:

```yaml
on:
  issue_comment:
    types: [created]

name: Commands

jobs:
  document:
    if: startsWith(github.event.comment.body, '/document')
    runs-on: ubuntu-latest
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: actions/checkout@v4
      - uses: r-lib/actions/pr-fetch@v2
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
      - uses: r-lib/actions/setup-r@v2
      - uses: r-lib/actions/setup-r-dependencies@v2
        with:
          extra-packages: any::roxygen2
      - name: Document
        run: roxygen2::roxygenise()
        shell: Rscript {0}
      - name: Commit
        run: |
          git config --local user.name "$GITHUB_ACTOR"
          git config --local user.email "$GITHUB_ACTOR@users.noreply.github.com"
          git add man/\* NAMESPACE
          git commit -m 'Document' || echo "No changes to commit"
      - uses: r-lib/actions/pr-push@v2
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}

  style:
    if: startsWith(github.event.comment.body, '/style')
    runs-on: ubuntu-latest
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: actions/checkout@v4
      - uses: r-lib/actions/pr-fetch@v2
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
      - uses: r-lib/actions/setup-r@v2
      - name: Install styler
        run: install.packages("styler")
        shell: Rscript {0}
      - name: Style
        run: styler::style_pkg()
        shell: Rscript {0}
      - name: Commit
        run: |
          git config --local user.name "$GITHUB_ACTOR"
          git config --local user.email "$GITHUB_ACTOR@users.noreply.github.com"
          git add \*.R
          git commit -m 'Style' || echo "No changes to commit"
      - uses: r-lib/actions/pr-push@v2
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
```

### 5. Lint Workflow

Create `.github/workflows/lint.yaml`:

```yaml
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

name: lint

permissions: read-all

jobs:
  lint:
    runs-on: ubuntu-latest
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: actions/checkout@v4
      - uses: r-lib/actions/setup-r@v2
        with:
          use-public-rspm: true
      - uses: r-lib/actions/setup-r-dependencies@v2
        with:
          extra-packages: any::lintr, local::.
          needs: lint
      - name: Lint
        run: lintr::lint_package()
        shell: Rscript {0}
        env:
          LINTR_ERROR_ON_LINT: true
```

## Workflow Management Commands

### Setup with usethis

```r
# Add R-CMD-check workflow
usethis::use_github_action("check-standard")

# Add pkgdown workflow
usethis::use_github_action("pkgdown")

# Add test-coverage workflow
usethis::use_github_action("test-coverage")

# Add lint workflow
usethis::use_github_action("lint")

# List available workflows
usethis::use_github_actions()

# Browse workflow files
usethis::use_github_action()
```

### Manual Workflow Files

Place workflows in `.github/workflows/` directory:
```
.github/
  workflows/
    R-CMD-check.yaml
    pkgdown.yaml
    test-coverage.yaml
    lint.yaml
```

## Common Pitfalls

### 1. Permissions Errors

**Problem**: Workflow can't push to gh-pages or create releases

**Solution**: Add permissions block:
```yaml
permissions:
  contents: write  # For pushing to gh-pages
  # Or for fine-grained control:
  # contents: read
  # pages: write
  # id-token: write
```

### 2. GITHUB_PAT Not Set

**Problem**: Rate limited or can't access private repos

**Solution**: Set environment variable:
```yaml
env:
  GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
```

For private dependencies, create a Personal Access Token:
1. GitHub Settings > Developer settings > Personal access tokens
2. Generate token with `repo` scope
3. Add to repo secrets as `GH_PAT`
4. Use: `GITHUB_PAT: ${{ secrets.GH_PAT }}`

### 3. Vignette Build Failures

**Problem**: vignettes fail on Ubuntu devel

**Solution**: Skip vignettes on devel:
```yaml
- uses: r-lib/actions/check-r-package@v2
  with:
    build_args: 'c("--no-manual", if (Sys.getenv("R_VERSION") != "devel") "--compact-vignettes=gs+qpdf")'
```

Or add system dependencies:
```yaml
- name: Install system dependencies
  if: runner.os == 'Linux'
  run: |
    sudo apt-get update
    sudo apt-get install -y \
      texlive-latex-base \
      texlive-fonts-recommended \
      texlive-fonts-extra \
      texlive-latex-extra
```

### 4. macOS Binary Issues

**Problem**: macOS builds fail with missing system libraries

**Solution**: Use Homebrew:
```yaml
- name: Install system dependencies
  if: runner.os == 'macOS'
  run: |
    brew install pkg-config
    brew install openssl
    brew install libgit2
```

### 5. Concurrent Builds Conflict

**Problem**: Multiple pushes create conflicting deployments

**Solution**: Use concurrency groups:
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

For pkgdown specifically:
```yaml
concurrency:
  group: pkgdown-${{ github.event_name != 'pull_request' || github.run_id }}
```

### 6. Cached Dependencies Out of Date

**Problem**: Old package versions used despite new releases

**Solution**: Dependencies are cached by setup-r-dependencies. To force refresh:
- Update your DESCRIPTION file
- Or manually clear cache in GitHub Actions settings
- Or increment cache version:

```yaml
- uses: r-lib/actions/setup-r-dependencies@v2
  with:
    cache-version: 2  # Increment this to bust cache
```

### 7. Windows LaTeX Issues

**Problem**: PDF manual fails on Windows

**Solution**: Skip manual on Windows:
```yaml
- uses: r-lib/actions/check-r-package@v2
  with:
    build_args: 'c(if (.Platform$OS.type != "windows") "--no-manual", "--compact-vignettes=gs+qpdf")'
```

Or install TinyTeX:
```yaml
- name: Install TinyTeX
  if: runner.os == 'Windows'
  run: |
    install.packages("tinytex")
    tinytex::install_tinytex()
  shell: Rscript {0}
```

### 8. Test Snapshots Don't Update

**Problem**: Snapshot tests fail but snapshots aren't committed

**Solution**: Use `upload-snapshots: true`:
```yaml
- uses: r-lib/actions/check-r-package@v2
  with:
    upload-snapshots: true
```

Then download artifacts and commit manually, or use pr-commands workflow.

### 9. Workflow Doesn't Trigger

**Problem**: Push to main but workflow doesn't run

**Solution**: Check:
- Branch name matches trigger (`main` vs `master`)
- Workflow file is in `.github/workflows/`
- YAML is valid (use yamllint)
- File has `.yaml` or `.yml` extension
- You have Actions enabled in repo settings

### 10. Badge Shows "unknown"

**Problem**: README badge doesn't show status

**Solution**: Ensure badge URL matches workflow name:
```markdown
<!-- Workflow name: R-CMD-check -->
[![R-CMD-check](https://github.com/username/repo/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/username/repo/actions/workflows/R-CMD-check.yaml)
```

Get badge from GitHub:
Repository > Actions > Select workflow > ... > Create status badge

## Badge Examples

### R-CMD-check Badge

```markdown
[![R-CMD-check](https://github.com/username/packagename/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/username/packagename/actions/workflows/R-CMD-check.yaml)
```

### Test Coverage Badge

```markdown
[![codecov](https://codecov.io/gh/username/packagename/branch/main/graph/badge.svg)](https://codecov.io/gh/username/packagename)
```

### Lifecycle Badge

```markdown
[![Lifecycle: experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)
```

### CRAN Badge

```markdown
[![CRAN status](https://www.r-pkg.org/badges/version/packagename)](https://CRAN.R-project.org/package=packagename)
```

## Advanced Configuration

### Matrix Strategy Customization

```yaml
strategy:
  fail-fast: false  # Don't cancel all if one fails
  matrix:
    config:
      - {os: macos-latest,   r: 'release'}
      - {os: windows-latest, r: 'release'}
      - {os: ubuntu-latest,  r: 'devel'}
      - {os: ubuntu-latest,  r: 'release'}
      - {os: ubuntu-latest,  r: 'oldrel-1'}
    exclude:  # Skip specific combinations
      - os: windows-latest
        r: 'devel'
```

### Conditional Steps

```yaml
- name: Install LaTeX
  if: runner.os == 'Linux' && matrix.config.r == 'release'
  run: sudo apt-get install texlive

- name: Build PDF manual
  if: matrix.config.r != 'devel'
  run: R CMD Rd2pdf .
```

### Environment Variables

```yaml
env:
  # Global (all jobs)
  GLOBAL_VAR: value

jobs:
  check:
    env:
      # Job-level
      JOB_VAR: value
    steps:
      - name: Step with env
        env:
          # Step-level
          STEP_VAR: value
        run: echo $STEP_VAR
```

### Scheduled Workflows

Run checks daily to catch upstream breakage:

```yaml
on:
  schedule:
    - cron: '0 8 * * *'  # 8 AM UTC daily
  workflow_dispatch:  # Allow manual trigger
```

This comprehensive setup ensures robust CI/CD for your R package.
