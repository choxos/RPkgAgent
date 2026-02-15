---
name: pkgdown-website
description: Build and deploy professional package documentation websites using pkgdown with Bootstrap 5, GitHub Pages, and automated workflows
---

# pkgdown Website Development

This skill covers creating comprehensive, professional package documentation websites using pkgdown with Bootstrap 5 theming, GitHub Pages deployment, and automated CI/CD workflows.

## Rules

1. **Always use Bootstrap 5** with modern theming capabilities including dark mode
2. **URLs must match** across DESCRIPTION, _pkgdown.yml, and GitHub Pages settings
3. **Use GitHub Actions** for automated deployment (don't build locally and commit)
4. **Organize documentation** with clear reference sections and article groupings
5. **Enable dark mode** using light-switch theme for better user experience
6. **Include social links** in navbar for package promotion and author contact
7. **Math rendering** should use KaTeX for fast, client-side rendering
8. **Custom fonts** via Google Fonts improve branding and readability
9. **Development mode** should be set to auto for version-appropriate messaging
10. **Test locally** with `pkgdown::build_site()` before deploying

## Complete _pkgdown.yml Configuration

### Full Template with All Features

```yaml
url: https://username.github.io/packagename/
template:
  bootstrap: 5
  theme: arrow-light
  bslib:
    base_font: {google: "Atkinson Hyperlegible"}
    heading_font: {google: "Fraunces"}
    code_font: {google: "JetBrains Mono"}
    bg: "#ffffff"
    fg: "#1e1e1e"
    primary: "#0054AD"
    secondary: "#767676"
  light-switch: true
  math-rendering: katex

development:
  mode: auto
  version_label: danger
  version_tooltip: "Development version"

home:
  title: "packagename: An Amazing R Package"
  description: >
    A comprehensive toolkit for doing amazing things in R.
    This package provides intuitive functions and robust workflows.
  links:
  - text: Ask a question
    href: https://github.com/username/packagename/discussions
  - text: Report a bug
    href: https://github.com/username/packagename/issues

authors:
  Your Name:
    href: https://yourwebsite.com
  Another Author:
    href: https://github.com/anotherusername

navbar:
  structure:
    left:  [home, intro, reference, articles, news]
    right: [search, github, linkedin, twitter]
  components:
    home:
      icon: fas fa-home fa-lg
      href: index.html
      aria-label: Home
    intro:
      text: Get started
      href: articles/packagename.html
    reference:
      text: Reference
      href: reference/index.html
    articles:
      text: Articles
      menu:
      - text: "Getting Started"
        href: articles/packagename.html
      - text: "Advanced Usage"
        href: articles/advanced-usage.html
      - text: "Examples"
        href: articles/examples.html
      - text: -------
      - text: "All Articles"
        href: articles/index.html
    news:
      text: News
      href: news/index.html
    github:
      icon: fab fa-github fa-lg
      href: https://github.com/username/packagename
      aria-label: GitHub
    linkedin:
      icon: fab fa-linkedin fa-lg
      href: https://www.linkedin.com/in/yourprofile/
      aria-label: LinkedIn
    twitter:
      icon: fab fa-twitter fa-lg
      href: https://twitter.com/yourhandle
      aria-label: Twitter

reference:
  - title: "Data Import and Export"
    desc: >
      Functions for reading and writing data files in various formats.
    contents:
      - read_data
      - write_data
      - import_csv
      - export_excel

  - title: "Data Manipulation"
    desc: >
      Core functions for transforming and cleaning data.
    contents:
      - clean_data
      - transform_variables
      - filter_rows
      - select_columns

  - title: "Statistical Analysis"
    desc: >
      Functions for statistical modeling and hypothesis testing.
    contents:
      - fit_model
      - test_hypothesis
      - calculate_statistics
      - generate_report

  - title: "Visualization"
    desc: >
      Create publication-quality plots and charts.
    contents:
      - plot_distribution
      - create_scatter
      - visualize_trends
      - export_plot

  - title: "Utilities"
    desc: >
      Helper functions and package configuration.
    contents:
      - validate_input
      - check_dependencies
      - packagename_options
      - print.packagename_object

  - title: "Internal Functions"
    desc: >
      Internal functions not intended for direct use.
    contents:
      - has_keyword("internal")

articles:
  - title: "Tutorials"
    navbar: Tutorials
    contents:
      - packagename
      - getting-started
      - basic-workflow

  - title: "Advanced Topics"
    navbar: Advanced
    contents:
      - advanced-usage
      - performance-optimization
      - extending-packagename

  - title: "Case Studies"
    navbar: ~
    contents:
      - case-study-1
      - case-study-2
      - real-world-examples

footer:
  structure:
    left: developed_by
    right: built_with
  components:
    developed_by: "Developed by [Your Name](https://yourwebsite.com)"
    built_with: "Built with [pkgdown](https://pkgdown.r-lib.org/) and [Bootstrap 5](https://getbootstrap.com/)"
```

### Minimal _pkgdown.yml Template

```yaml
url: https://username.github.io/packagename/
template:
  bootstrap: 5
  light-switch: true

development:
  mode: auto
```

## DESCRIPTION File URL Configuration

The URL field in DESCRIPTION must match _pkgdown.yml:

```
Package: packagename
Title: An Amazing R Package
Version: 0.1.0
URL: https://username.github.io/packagename/, https://github.com/username/packagename
BugReports: https://github.com/username/packagename/issues
```

**Note**: Multiple URLs are comma-separated. Package website comes first, then GitHub repo.

## Initial Setup with usethis

### Complete Setup Process

```r
# 1. Initialize pkgdown structure
usethis::use_pkgdown()

# 2. Configure for GitHub Pages (recommended)
usethis::use_pkgdown_github_pages()

# This function:
# - Adds URL to DESCRIPTION
# - Creates _pkgdown.yml
# - Adds GitHub Actions workflow
# - Configures .gitignore for docs/
# - Sets up gh-pages branch
```

### Manual Setup (if needed)

```r
# Create basic structure
usethis::use_pkgdown()

# Build site locally
pkgdown::build_site()

# Preview in browser
pkgdown::preview_site()

# Build specific components
pkgdown::build_reference()
pkgdown::build_articles()
pkgdown::build_home()
pkgdown::build_news()
```

## GitHub Actions Workflow

### Complete pkgdown.yaml Workflow

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

# Only allow one deployment at a time
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
```

### Alternative: Simple Deployment Workflow

```yaml
on:
  push:
    branches: [main]

name: pkgdown

permissions:
  contents: write

jobs:
  pkgdown:
    runs-on: ubuntu-latest
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: actions/checkout@v4
      - uses: r-lib/actions/setup-pandoc@v2
      - uses: r-lib/actions/setup-r@v2
      - uses: r-lib/actions/setup-r-dependencies@v2
        with:
          extra-packages: any::pkgdown, local::.
          needs: website
      - name: Build and deploy
        run: |
          pkgdown::deploy_to_branch(new_process = FALSE)
        shell: Rscript {0}
```

## Bootstrap 5 Theming

### Built-in Themes

pkgdown includes several Bootstrap 5 themes:
- `arrow-light` / `arrow-dark` (Tidyverse theme)
- Standard Bootstrap 5 themes

```yaml
template:
  bootstrap: 5
  theme: arrow-light  # or arrow-dark
```

### Custom Color Scheme with bslib

```yaml
template:
  bootstrap: 5
  bslib:
    # Base colors
    bg: "#ffffff"           # Background
    fg: "#212529"           # Foreground text
    primary: "#0054AD"      # Primary brand color
    secondary: "#6c757d"    # Secondary color
    success: "#198754"
    info: "#0dcaf0"
    warning: "#ffc107"
    danger: "#dc3545"

    # Typography
    base_font: {google: "Roboto"}
    heading_font: {google: "Roboto Slab"}
    code_font: {google: "Fira Code"}
    font_scale: 1.0

    # Spacing
    spacer: 1rem
```

### Google Fonts Options

Popular font combinations:

```yaml
# Professional
bslib:
  base_font: {google: "Atkinson Hyperlegible"}
  heading_font: {google: "Fraunces"}
  code_font: {google: "JetBrains Mono"}

# Modern
bslib:
  base_font: {google: "Inter"}
  heading_font: {google: "Inter"}
  code_font: {google: "Fira Code"}

# Classic
bslib:
  base_font: {google: "Lato"}
  heading_font: {google: "Merriweather"}
  code_font: {google: "Source Code Pro"}

# Friendly
bslib:
  base_font: {google: "Nunito"}
  heading_font: {google: "Nunito"}
  code_font: {google: "Ubuntu Mono"}
```

## Reference Organization

### Grouping Functions by Purpose

```yaml
reference:
  - title: "Core Functions"
    desc: "Main user-facing functions"
    contents:
      - has_concept("core")

  - title: "Data Processing"
    contents:
      - starts_with("process_")
      - starts_with("transform_")

  - title: "Plotting Functions"
    contents:
      - matches("plot|visualize|graph")

  - title: "S3 Methods"
    contents:
      - matches("\\.")

  - title: "Datasets"
    desc: "Example datasets included with the package"
    contents:
      - has_keyword("datasets")
```

### Using Roxygen Concepts

In your function documentation:

```r
#' @concept core
#' @concept data-import
```

Then in _pkgdown.yml:

```yaml
reference:
  - title: "Core Functions"
    contents:
      - has_concept("core")
  - title: "Data Import"
    contents:
      - has_concept("data-import")
```

## Articles Configuration

### Organizing Vignettes

```yaml
articles:
  - title: "Getting Started"
    navbar: "Get Started"
    desc: "New to the package? Start here!"
    contents:
      - introduction
      - installation
      - quick-start

  - title: "Workflows"
    navbar: "Workflows"
    contents:
      - basic-workflow
      - advanced-workflow
      - batch-processing

  - title: "Reference"
    navbar: ~  # Don't show in navbar
    contents:
      - technical-details
      - algorithm-description
```

## Common Pitfalls

### 1. URL Mismatch

**Problem**: Website deploys but links are broken

**Solution**: Ensure URLs match exactly in three places:
```r
# DESCRIPTION
URL: https://username.github.io/packagename/

# _pkgdown.yml
url: https://username.github.io/packagename/

# GitHub Settings > Pages
Source: gh-pages branch, / (root)
```

**Note**: Trailing slash matters! Be consistent.

### 2. GitHub Pages Not Enabling

**Problem**: Workflow runs but site doesn't appear

**Solution**:
1. Check GitHub repo Settings > Pages
2. Source should be "Deploy from a branch"
3. Branch should be "gh-pages" and "/ (root)"
4. Wait 2-3 minutes after first deployment
5. Check `https://username.github.io/packagename/`

### 3. Missing Dependencies

**Problem**: Build fails with "package not found"

**Solution**: Add to DESCRIPTION:
```
Suggests:
    pkgdown,
    knitr,
    rmarkdown
```

### 4. Math Not Rendering

**Problem**: LaTeX equations show as raw code

**Solution**: Enable math rendering:
```yaml
template:
  math-rendering: katex  # or mathjax
```

In your .Rmd:
```
Inline: $E = mc^2$
Display: $$\int_0^\infty e^{-x^2} dx = \frac{\sqrt{\pi}}{2}$$
```

### 5. Icons Not Showing

**Problem**: Social media icons appear as text

**Solution**: Use FontAwesome classes correctly:
```yaml
navbar:
  components:
    github:
      icon: fab fa-github fa-lg  # fab for brands
      href: https://github.com/user/repo
    home:
      icon: fas fa-home fa-lg    # fas for solid
      href: index.html
```

### 6. Dark Mode Issues

**Problem**: Dark mode has poor contrast

**Solution**: Test both modes and adjust:
```yaml
template:
  light-switch: true
  bslib:
    # Light mode
    bg: "#ffffff"
    fg: "#212529"
    # These will auto-adjust for dark mode
    # Or specify dark mode explicitly
```

### 7. Development Version Badge

**Problem**: Release version shows "dev" badge

**Solution**: Use automatic mode:
```yaml
development:
  mode: auto  # auto-detects based on version number
```

Version numbering:
- Development: 0.1.0.9000
- Release: 0.1.0

### 8. Large Site Build Time

**Problem**: Build takes >10 minutes on GitHub Actions

**Solution**:
```yaml
# In workflow, use cached dependencies
- uses: r-lib/actions/setup-r-dependencies@v2
  with:
    extra-packages: any::pkgdown, local::.
    needs: website

# In _pkgdown.yml, skip heavy computations
template:
  params:
    docsearch:
      api_key: YOUR_KEY
      index_name: YOUR_INDEX
```

### 9. Broken Internal Links

**Problem**: Links between articles don't work

**Solution**: Use relative paths:
```markdown
See the [advanced tutorial](advanced-usage.html) for more.
See [function reference](../reference/my_function.html).
```

### 10. Custom CSS Not Applied

**Problem**: Custom styles don't appear

**Solution**:
```yaml
template:
  includes:
    in_header: |
      <link rel="stylesheet" href="custom.css">
```

Create `pkgdown/extra.css` (automatically included):
```css
.navbar {
  background-color: #0054AD !important;
}
```

## Testing and Validation

### Local Testing

```r
# Full site build
pkgdown::build_site()

# Individual components
pkgdown::build_reference()
pkgdown::build_home()
pkgdown::build_articles()
pkgdown::build_news()

# Preview in browser
pkgdown::preview_site()

# Clean and rebuild
pkgdown::clean_site()
pkgdown::build_site()
```

### Validation Checklist

- [ ] All URLs match (DESCRIPTION, _pkgdown.yml, GitHub)
- [ ] Site builds without errors locally
- [ ] All reference pages render correctly
- [ ] Articles render with proper formatting
- [ ] Links between pages work
- [ ] Dark mode has good contrast
- [ ] Icons display properly
- [ ] Math equations render (if applicable)
- [ ] Code examples run correctly
- [ ] Mobile view is responsive
- [ ] Search functionality works
- [ ] Social links point to correct profiles

## Advanced Features

### Search Integration

Algolia DocSearch (free for open source):

```yaml
template:
  params:
    docsearch:
      api_key: YOUR_API_KEY
      index_name: YOUR_INDEX_NAME
```

### Custom Footer

```yaml
footer:
  structure:
    left: [developed_by, citation]
    right: [built_with, legal]
  components:
    developed_by: "Developed by [Your Name](https://yoursite.com)"
    citation: "Please cite this package as..."
    built_with: "Built with pkgdown"
    legal: "[Privacy Policy](privacy.html) | [Terms](terms.html)"
```

### OpenGraph and Twitter Cards

```yaml
home:
  opengraph:
    image:
      src: man/figures/card.png
      alt: "Package logo"
    twitter:
      creator: "@yourhandle"
      card: summary_large_image
```

This creates rich previews when shared on social media.
