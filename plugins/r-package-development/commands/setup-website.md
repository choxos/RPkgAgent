---
name: setup-website
description: Set up a pkgdown website with GitHub Pages deployment
---

# Setup Package Website

This command creates a complete pkgdown website for your R package with automatic GitHub Pages deployment, custom styling, hex logo, and organized documentation.

## Workflow

### Step 1: Create _pkgdown.yml Configuration

Create `_pkgdown.yml` in the package root:

```yaml
url: https://username.github.io/packagename/

template:
  bootstrap: 5
  bslib:
    preset: bootstrap
    primary: "#0054a6"
    base_font:
      google: "Source Sans Pro"
    code_font:
      google: "JetBrains Mono"
  light-switch: true
  includes:
    in_header: |
      <script defer data-domain="username.github.io/packagename" src="https://plausible.io/js/script.js"></script>

home:
  title: "packagename • Elegant Text Preprocessing"
  description: "Clean and preprocess text data with ease"
  links:
    - text: Report a bug
      href: https://github.com/username/packagename/issues

navbar:
  structure:
    left: [intro, reference, articles, tutorials, news]
    right: [search, github, lightswitch]
  components:
    twitter:
      icon: "fab fa-twitter fa-lg"
      href: https://twitter.com/username
      aria-label: Twitter
    mastodon:
      icon: "fab fa-mastodon fa-lg"
      href: https://fosstodon.org/@username
      aria-label: Mastodon

reference:
  - title: "Text Cleaning"
    desc: "Functions for cleaning and preprocessing text"
    contents:
      - clean_text
      - remove_urls
      - remove_stopwords
  - title: "Text Analysis"
    desc: "Functions for analyzing text data"
    contents:
      - count_words
      - extract_keywords
  - title: "Data"
    desc: "Example datasets included in the package"
    contents:
      - has_keyword("datasets")
  - title: "Utilities"
    desc: "Helper functions and utilities"
    contents:
      - starts_with("utils_")

articles:
  - title: "Getting Started"
    navbar: ~
    contents:
      - packagename
  - title: "Advanced Usage"
    navbar: Advanced
    contents:
      - custom-preprocessing
      - batch-processing

footer:
  structure:
    left: developed_by
    right: built_with
  components:
    developed_by: |
      Developed by [Your Name](https://yourwebsite.com).
    built_with: |
      Built with [pkgdown](https://pkgdown.r-lib.org/) and
      [R](https://www.r-project.org/).
```

Key configuration elements:

- **url**: Must match GitHub Pages URL (username.github.io/packagename)
- **template**: Bootstrap 5 with light/dark mode toggle
- **reference**: Organize functions into logical groups
- **articles**: Organize vignettes
- **navbar**: Social links, search

### Step 2: Create Hex Logo

Create `man/figures/generate_logo.R`:

```r
library(hexSticker)
library(showtext)

# Load Google Font
font_add_google("Source Sans Pro", "source")
showtext_auto()

# Create hex sticker
sticker(
  # Subplot (icon, plot, or image)
  subplot = ~ plot.new(),  # Or use an icon/image
  s_x = 1,           # Subplot x position
  s_y = 0.75,        # Subplot y position
  s_width = 0.6,     # Subplot width
  s_height = 0.6,    # Subplot height

  # Package name
  package = "packagename",
  p_x = 1,           # Package name x position
  p_y = 1.4,         # Package name y position
  p_size = 20,       # Font size
  p_family = "source",
  p_color = "#FFFFFF",

  # Hexagon
  h_fill = "#0054a6",   # Fill color
  h_color = "#003d7a",  # Border color
  h_size = 1.5,         # Border size

  # Output
  filename = "man/figures/logo.png",
  dpi = 300
)

# Create SVG version for better scaling
sticker(
  subplot = ~ plot.new(),
  s_x = 1, s_y = 0.75, s_width = 0.6, s_height = 0.6,
  package = "packagename",
  p_x = 1, p_y = 1.4, p_size = 20,
  p_family = "source", p_color = "#FFFFFF",
  h_fill = "#0054a6", h_color = "#003d7a", h_size = 1.5,
  filename = "man/figures/logo.svg"
)

# Resize PNG to standard dimensions
library(magick)
logo <- image_read("man/figures/logo.png")
logo <- image_resize(logo, "240x278!")
image_write(logo, "man/figures/logo.png")
```

Alternative: Use simple ggplot2-based logo:

```r
library(ggplot2)

p <- ggplot(data.frame(x = 1, y = 1), aes(x, y)) +
  geom_text(label = "PKG", size = 30, fontface = "bold", color = "white") +
  theme_void() +
  theme(plot.background = element_rect(fill = "#0054a6", color = NA))

hexSticker::sticker(
  p,
  package = "packagename",
  p_size = 20, p_color = "#FFFFFF",
  s_x = 1, s_y = 1, s_width = 1.3, s_height = 1,
  h_fill = "#0054a6", h_color = "#003d7a",
  filename = "man/figures/logo.png"
)
```

Run the script to generate `logo.png` and `logo.svg`.

Add logo to README:

```r
usethis::use_logo("man/figures/logo.png")
```

This adds the image to README.md and creates proper sizing.

### Step 3: Create GitHub Actions Workflow

Create `.github/workflows/pkgdown.yaml`:

```yaml
# Workflow derived from https://github.com/r-lib/actions/tree/v2/examples
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]
  release:
    types: [published]
  workflow_dispatch:

name: pkgdown

jobs:
  pkgdown:
    runs-on: ubuntu-latest
    # Only restrict concurrency for non-PR jobs
    concurrency:
      group: pkgdown-${{ github.event_name != 'pull_request' || github.run_id }}
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - uses: r-lib/actions/setup-pandoc@v2

      - uses: r-lib/actions/setup-r@v2
        with:
          use-public-rspm: true

      - uses: r-lib/actions/setup-r-dependencies@v2
        with:
          extra-packages: any::pkgdown, local::.
          needs: website

      - name: Build site
        run: pkgdown::build_site_github_pages(new_process = FALSE, install = FALSE)
        shell: Rscript {0}

      - name: Deploy to GitHub pages 🚀
        if: github.event_name != 'pull_request'
        uses: JamesIves/github-pages-deploy-action@v4.5.0
        with:
          clean: false
          branch: gh-pages
          folder: docs
```

This workflow:

- Builds the site on every push to main
- Deploys to gh-pages branch automatically
- Works with pull requests (builds but doesn't deploy)

### Step 4: Update DESCRIPTION and URLs

Ensure DESCRIPTION contains the URL:

```
URL: https://username.github.io/packagename/,
    https://github.com/username/packagename
BugReports: https://github.com/username/packagename/issues
```

The URL in DESCRIPTION must match `url:` in `_pkgdown.yml`.

### Step 5: Update .Rbuildignore

Add pkgdown-related files to `.Rbuildignore`:

```
^_pkgdown\.yml$
^docs$
^pkgdown$
^\.github$
^man/figures/generate_logo\.R$
```

Run:

```r
usethis::use_pkgdown()  # Automatically adds these
```

### Step 6: Create CITATION File

Create `inst/CITATION` for academic citations:

```r
bibentry(
  bibtype = "Manual",
  title = "packagename: Elegant Text Preprocessing",
  author = c(
    person("First", "Last", email = "email@example.com", role = c("aut", "cre"))
  ),
  year = "2024",
  note = "R package version 0.1.0",
  url = "https://username.github.io/packagename/"
)
```

Or use the simpler auto-generated format:

```r
usethis::use_citation()
```

This creates a template you can customize.

Users can then cite your package:

```r
citation("packagename")
```

### Step 7: Create NEWS.md

Create `NEWS.md` to track changes:

```markdown
# packagename (development version)

* Initial release
* Added core text cleaning functions
* Includes example datasets

# packagename 0.1.0

* First CRAN release
* Core features:
  - `clean_text()` for text preprocessing
  - `remove_urls()` for URL removal
  - `customer_data` example dataset
```

News entries appear on the website automatically.

Format:

- Use `#` for version headers
- Use `# packagename (development version)` for unreleased changes
- Use `* ` for bullet points
- Group related changes

### Step 8: Build Site Locally

Build and preview the website locally:

```r
# Build entire site
pkgdown::build_site()

# Preview in browser
pkgdown::preview_site()

# Build specific components
pkgdown::build_home()       # Just README
pkgdown::build_reference()  # Just function docs
pkgdown::build_articles()   # Just vignettes
pkgdown::build_news()       # Just NEWS
```

Check the `docs/` directory for the generated HTML.

Review:

- ✓ Logo appears in navbar
- ✓ Functions organized in logical sections
- ✓ Examples render correctly
- ✓ Vignettes display properly
- ✓ Light/dark mode toggle works
- ✓ Social links functional
- ✓ Search works

Fix any issues before deploying.

### Step 9: Push and Enable GitHub Pages

Commit and push all files:

```bash
git add _pkgdown.yml .github/ man/figures/logo.png inst/CITATION NEWS.md
git commit -m "Setup pkgdown website"
git push
```

Wait for the GitHub Action to complete (check Actions tab).

Enable GitHub Pages:

1. Go to repository Settings → Pages
2. Source: Deploy from a branch
3. Branch: `gh-pages` / `root`
4. Save

The site will be available at: `https://username.github.io/packagename/`

### Step 10: Verify Deployment

After the action completes:

1. Visit the URL to confirm site is live
2. Check that:
   - Logo displays correctly
   - Reference pages load
   - Vignettes are accessible
   - Search functions
   - Light/dark toggle works
   - Links are not broken

Run link checker:

```r
pkgdown::check_pkgdown()
```

### Final Checklist

Before completing:

- [ ] `_pkgdown.yml` created with organized reference sections
- [ ] Hex logo created (240x278 px PNG)
- [ ] Logo added to README
- [ ] `.github/workflows/pkgdown.yaml` workflow created
- [ ] URL in DESCRIPTION matches `_pkgdown.yml`
- [ ] `^_pkgdown\.yml$`, `^docs$`, `^pkgdown$` in .Rbuildignore
- [ ] `inst/CITATION` created
- [ ] `NEWS.md` created with version history
- [ ] `pkgdown::build_site()` runs without errors
- [ ] Site pushed to GitHub
- [ ] GitHub Pages enabled on gh-pages branch
- [ ] Site live and functional at URL

Report to user:

```
✓ Pkgdown website setup complete!

Created:
  - _pkgdown.yml (Bootstrap 5 with light/dark mode)
  - man/figures/logo.png (hex sticker)
  - .github/workflows/pkgdown.yaml (auto-deploy)
  - inst/CITATION (for academic citations)
  - NEWS.md (version tracking)

Website features:
  - Bootstrap 5 with light-switch toggle
  - Google Fonts (Source Sans Pro)
  - Organized reference (4 sections)
  - Social media links
  - Search functionality
  - Automatic deployment on push

Site URL: https://username.github.io/packagename/

Next steps:
  1. Push to GitHub: git push
  2. Enable GitHub Pages (Settings → Pages → gh-pages)
  3. Wait 2-3 minutes for deployment
  4. Visit your site!

To update: Just push changes, site rebuilds automatically.
```

## Example Interaction

User: "Setup a website for my 'cleantext' package"