---
name: hex-logo
description: Create professional hexagonal package logos using hexSticker with Google Fonts, proper dimensions, and automated generation scripts
---

# Hexagonal Package Logo Creation

This skill covers creating professional hexagonal package logos using the hexSticker package. Hex logos are the standard branding for R packages, featured on package websites, GitHub repos, and CRAN pages.

## Rules

1. **Standard dimensions**: 5.08cm x 4.39cm (2 x 1.73 inches) for consistency
2. **Use hexSticker package** for proper proportions and formatting
3. **Generate with script** in man/figures/generate_logo.R (reproducible)
4. **Exclude script from build** via .Rbuildignore
5. **Output both PNG and SVG** formats (PNG for README, SVG for print)
6. **Use Google Fonts** with showtext for custom typography
7. **Match package identity** with appropriate colors and imagery
8. **Keep it simple** - logos should work at small sizes
9. **Add to README** with proper alignment and alt text
10. **Version control logos** (commit PNG/SVG, not temporary files)

## Installation and Setup

### Required Packages

```r
# Install hexSticker and dependencies
install.packages("hexSticker")
install.packages("showtext")  # For Google Fonts
install.packages("ggplot2")   # For custom plots
install.packages("magick")    # For image manipulation

# Load packages
library(hexSticker)
library(showtext)
library(ggplot2)
```

## Standard Hex Dimensions

### Official Specifications

- **Width**: 5.08 cm (2 inches)
- **Height**: 4.39 cm (1.73 inches)
- **Aspect ratio**: ~1.15:1
- **DPI**: 300 for print, 72 for web (hexSticker uses default appropriate for each)

These are set automatically by hexSticker.

## Basic Hex Sticker Creation

### Minimal Example (Text Only)

```r
library(hexSticker)

sticker(
  # Required: subplot (can be empty plot)
  subplot = ~ plot.new(),
  # Package name
  package = "mypackage",
  # Package name styling
  p_size = 20,           # Font size
  p_color = "#FFFFFF",   # White text
  # Hex styling
  h_fill = "#0054AD",    # Fill color
  h_color = "#000000",   # Border color
  # Output
  filename = "man/figures/logo.png"
)
```

### With Image Subplot

```r
library(hexSticker)

# Using an existing image
img_path <- "man/figures/icon.png"

sticker(
  subplot = img_path,
  s_x = 1,               # Center horizontally
  s_y = 0.75,            # Position vertically
  s_width = 0.6,         # Image width
  s_height = 0.6,        # Image height
  package = "mypackage",
  p_size = 20,
  p_y = 1.45,            # Position below image
  p_color = "#FFFFFF",
  h_fill = "#0054AD",
  h_color = "#000000",
  filename = "man/figures/logo.png"
)
```

### With ggplot2 Subplot

```r
library(hexSticker)
library(ggplot2)

# Create a plot
p <- ggplot(mtcars, aes(mpg, wt)) +
  geom_point(color = "white", size = 2) +
  theme_void() +
  theme_transparent()

# Create sticker
sticker(
  subplot = p,
  s_x = 1,
  s_y = 0.85,
  s_width = 1.3,
  s_height = 1,
  package = "mypackage",
  p_size = 20,
  p_y = 1.5,
  p_color = "#FFFFFF",
  h_fill = "#0054AD",
  h_color = "#FFFFFF",
  h_size = 1.5,
  filename = "man/figures/logo.png",
  dpi = 300
)
```

## Google Fonts Integration

### Using showtext for Custom Fonts

```r
library(hexSticker)
library(showtext)

# Add Google Fonts
font_add_google("Fira Sans", "firasans")
font_add_google("Fira Code", "firacode")

# Enable showtext
showtext_auto()

# Create sticker with custom font
sticker(
  subplot = ~ plot.new(),
  package = "mypackage",
  p_size = 24,
  p_family = "firasans",       # Use Google Font
  p_fontface = "bold",
  p_color = "#FFFFFF",
  h_fill = "#0054AD",
  h_color = "#000000",
  filename = "man/figures/logo.png",
  dpi = 300
)
```

### Popular Font Combinations

```r
# Modern and clean
font_add_google("Inter", "inter")
font_add_google("JetBrains Mono", "jetbrains")

# Professional
font_add_google("Roboto", "roboto")
font_add_google("Roboto Slab", "robotoslab")

# Friendly
font_add_google("Nunito", "nunito")
font_add_google("Quicksand", "quicksand")

# Technical
font_add_google("Fira Sans", "firasans")
font_add_google("Fira Code", "firacode")

# Elegant
font_add_google("Lato", "lato")
font_add_google("Merriweather", "merriweather")

# Bold/Impact
font_add_google("Montserrat", "montserrat")
font_add_google("Bebas Neue", "bebas")
```

## Complete Generation Script Template

### man/figures/generate_logo.R

```r
#!/usr/bin/env Rscript

# Hex sticker generation script for mypackage
# Run this script to regenerate the package logo

# Load required packages
library(hexSticker)
library(showtext)
library(ggplot2)

# Configuration
PACKAGE_NAME <- "mypackage"
OUTPUT_DIR <- "man/figures"

# Ensure output directory exists
if (!dir.exists(OUTPUT_DIR)) {
  dir.create(OUTPUT_DIR, recursive = TRUE)
}

# Add and configure fonts
font_add_google("Fira Sans", "firasans")
showtext_auto()

# Color scheme (adjust to match your package theme)
FILL_COLOR <- "#0054AD"      # Primary brand color
BORDER_COLOR <- "#000000"    # Black border
TEXT_COLOR <- "#FFFFFF"      # White text
ACCENT_COLOR <- "#FFA500"    # Orange accent

# Option 1: Text-only logo
# Uncomment to use
# sticker(
#   subplot = ~ plot.new(),
#   package = PACKAGE_NAME,
#   p_size = 24,
#   p_family = "firasans",
#   p_fontface = "bold",
#   p_color = TEXT_COLOR,
#   p_y = 1,
#   h_fill = FILL_COLOR,
#   h_color = BORDER_COLOR,
#   h_size = 1.5,
#   filename = file.path(OUTPUT_DIR, "logo.png"),
#   dpi = 300
# )

# Option 2: Logo with ggplot2 visualization
# Create a representative plot
set.seed(123)
plot_data <- data.frame(
  x = rnorm(100),
  y = rnorm(100)
)

p <- ggplot(plot_data, aes(x, y)) +
  geom_point(color = TEXT_COLOR, alpha = 0.6, size = 1.5) +
  geom_density_2d(color = ACCENT_COLOR, size = 0.8) +
  theme_void() +
  theme_transparent() +
  theme(
    legend.position = "none",
    panel.background = element_rect(fill = "transparent", color = NA),
    plot.background = element_rect(fill = "transparent", color = NA)
  )

# Create sticker with plot
sticker(
  subplot = p,
  s_x = 1,
  s_y = 0.85,
  s_width = 1.4,
  s_height = 1.1,
  package = PACKAGE_NAME,
  p_size = 20,
  p_family = "firasans",
  p_fontface = "bold",
  p_color = TEXT_COLOR,
  p_y = 1.5,
  h_fill = FILL_COLOR,
  h_color = BORDER_COLOR,
  h_size = 1.5,
  url = "github.com/username/mypackage",
  u_size = 3.5,
  u_color = TEXT_COLOR,
  u_family = "firasans",
  filename = file.path(OUTPUT_DIR, "logo.png"),
  dpi = 300
)

# Also generate SVG version
sticker(
  subplot = p,
  s_x = 1,
  s_y = 0.85,
  s_width = 1.4,
  s_height = 1.1,
  package = PACKAGE_NAME,
  p_size = 20,
  p_family = "firasans",
  p_fontface = "bold",
  p_color = TEXT_COLOR,
  p_y = 1.5,
  h_fill = FILL_COLOR,
  h_color = BORDER_COLOR,
  h_size = 1.5,
  url = "github.com/username/mypackage",
  u_size = 3.5,
  u_color = TEXT_COLOR,
  u_family = "firasans",
  filename = file.path(OUTPUT_DIR, "logo.svg")
)

message("Logo generated successfully:")
message("  PNG: ", file.path(OUTPUT_DIR, "logo.png"))
message("  SVG: ", file.path(OUTPUT_DIR, "logo.svg"))
```

### Make Script Executable

```bash
chmod +x man/figures/generate_logo.R
```

### Exclude from Package Build

Add to `.Rbuildignore`:

```
^man/figures/generate_logo\.R$
```

Or use usethis:

```r
usethis::use_build_ignore("man/figures/generate_logo.R")
```

## Comprehensive Parameter Reference

### sticker() Parameters

#### Subplot Parameters

- **subplot**: Image path, ggplot object, or expression (required)
- **s_x**: Horizontal position (default: 1, center)
- **s_y**: Vertical position (default: 0.75)
- **s_width**: Width scaling (default: 0.4)
- **s_height**: Height scaling (default: 0.4)

#### Package Name Parameters

- **package**: Package name string
- **p_x**: Horizontal position (default: 1, center)
- **p_y**: Vertical position (default: 1.4, bottom)
- **p_size**: Font size (default: 8)
- **p_color**: Text color (default: "#FFFFFF")
- **p_family**: Font family (default: "sans")
- **p_fontface**: Font face: "plain", "bold", "italic", "bold.italic"

#### Hex Parameters

- **h_fill**: Fill color (default: "#1881C2")
- **h_color**: Border color (default: "#87B13F")
- **h_size**: Border width (default: 1.2)

#### Spotlight/White Space Parameters

- **spotlight**: Add spotlight effect (default: FALSE)
- **l_x**: Spotlight horizontal position
- **l_y**: Spotlight vertical position
- **l_width**: Spotlight width
- **l_height**: Spotlight height
- **l_alpha**: Spotlight transparency (0-1)

#### URL Parameters

- **url**: URL text to display at bottom
- **u_x**: URL horizontal position (default: 1)
- **u_y**: URL vertical position (default: 0.08)
- **u_size**: URL font size (default: 1.5)
- **u_color**: URL text color (default: "black")
- **u_family**: URL font family
- **u_angle**: URL text angle (default: 30)

#### Output Parameters

- **filename**: Output file path (required)
- **dpi**: Resolution for PNG (default: 300)
- **asp**: Aspect ratio override (usually omit)

## Color Scheme Guidelines

### Choosing Colors

**Brand colors**: Match your organization/project colors

**Contrast**: Ensure text is readable on background

**Color theory**:
- **Complementary**: Opposite on color wheel (high contrast)
- **Analogous**: Adjacent on color wheel (harmonious)
- **Triadic**: Three evenly spaced colors (balanced)

### Color Palette Examples

```r
# Tidyverse style (blue and green)
h_fill = "#1881C2"
h_color = "#87B13F"
p_color = "#FFFFFF"

# Professional blue
h_fill = "#0054AD"
h_color = "#003366"
p_color = "#FFFFFF"

# Bold red
h_fill = "#DC3545"
h_color = "#8B0000"
p_color = "#FFFFFF"

# Nature green
h_fill = "#28A745"
h_color = "#155724"
p_color = "#FFFFFF"

# Modern purple
h_fill = "#6F42C1"
h_color = "#3D1F6B"
p_color = "#FFFFFF"

# Warm orange
h_fill = "#FD7E14"
h_color = "#A04100"
p_color = "#FFFFFF"

# Monochrome
h_fill = "#212529"
h_color = "#000000"
p_color = "#FFFFFF"

# Light theme (dark border)
h_fill = "#F8F9FA"
h_color = "#000000"
p_color = "#212529"
```

### Accessibility

Ensure sufficient contrast (WCAG AA standard: 4.5:1 for normal text):

```r
# Good contrast examples
h_fill = "#0054AD"; p_color = "#FFFFFF"  # ✓ High contrast
h_fill = "#FFFFFF"; p_color = "#000000"  # ✓ High contrast

# Poor contrast (avoid)
h_fill = "#FFD700"; p_color = "#FFFFFF"  # ✗ Low contrast
h_fill = "#87CEEB"; p_color = "#FFFFFF"  # ✗ Low contrast
```

## Advanced Examples

### Logo with Custom Icon

```r
library(hexSticker)
library(showtext)
library(magick)

# Load and prepare icon
icon <- image_read("man/figures/custom_icon.png")
icon <- image_transparent(icon, "white")  # Make background transparent
icon <- image_trim(icon)  # Remove whitespace
image_write(icon, "man/figures/icon_processed.png")

# Add font
font_add_google("Fira Sans", "firasans")
showtext_auto()

# Create sticker
sticker(
  subplot = "man/figures/icon_processed.png",
  s_x = 1,
  s_y = 0.85,
  s_width = 0.5,
  s_height = 0.5,
  package = "mypackage",
  p_size = 22,
  p_family = "firasans",
  p_fontface = "bold",
  p_color = "#FFFFFF",
  p_y = 1.45,
  h_fill = "#0054AD",
  h_color = "#003366",
  h_size = 1.5,
  url = "mypackage.com",
  u_size = 3,
  u_color = "#FFFFFF",
  filename = "man/figures/logo.png",
  dpi = 300
)
```

### Logo with Spotlight Effect

```r
library(hexSticker)

sticker(
  subplot = ~ plot.new(),
  package = "mypackage",
  p_size = 24,
  p_color = "#FFFFFF",
  p_y = 1,
  h_fill = "#0054AD",
  h_color = "#000000",
  spotlight = TRUE,
  l_x = 1,
  l_y = 0.8,
  l_width = 3,
  l_height = 3,
  l_alpha = 0.3,
  filename = "man/figures/logo.png"
)
```

### Statistical Visualization Logo

```r
library(hexSticker)
library(ggplot2)
library(showtext)

# Font setup
font_add_google("Roboto", "roboto")
showtext_auto()

# Create visualization
set.seed(42)
df <- data.frame(
  x = rep(1:10, each = 10),
  y = rep(1:10, times = 10),
  z = rnorm(100)
)

p <- ggplot(df, aes(x, y, fill = z)) +
  geom_tile() +
  scale_fill_gradient2(
    low = "#0054AD",
    mid = "#FFFFFF",
    high = "#DC3545",
    midpoint = 0
  ) +
  theme_void() +
  theme_transparent() +
  theme(legend.position = "none")

# Create sticker
sticker(
  subplot = p,
  s_x = 1,
  s_y = 0.8,
  s_width = 1.2,
  s_height = 1,
  package = "heatmapR",
  p_size = 18,
  p_family = "roboto",
  p_fontface = "bold",
  p_color = "#000000",
  p_y = 1.55,
  h_fill = "#F8F9FA",
  h_color = "#000000",
  h_size = 1.5,
  filename = "man/figures/logo.png",
  dpi = 300
)
```

### Network Visualization Logo

```r
library(hexSticker)
library(ggplot2)
library(ggraph)
library(igraph)
library(showtext)

# Font setup
font_add_google("Montserrat", "montserrat")
showtext_auto()

# Create network
set.seed(123)
g <- erdos.renyi.game(15, 0.3)

p <- ggraph(g, layout = "fr") +
  geom_edge_link(color = "white", alpha = 0.5, width = 0.5) +
  geom_node_point(color = "#FFA500", size = 3) +
  theme_void() +
  theme_transparent()

# Create sticker
sticker(
  subplot = p,
  s_x = 1,
  s_y = 0.85,
  s_width = 1.3,
  s_height = 1.1,
  package = "netanalysis",
  p_size = 16,
  p_family = "montserrat",
  p_fontface = "bold",
  p_color = "#FFFFFF",
  p_y = 1.5,
  h_fill = "#0054AD",
  h_color = "#000000",
  h_size = 1.5,
  filename = "man/figures/logo.png",
  dpi = 300
)
```

## Adding Logo to README

### Standard README Header

```markdown
# mypackage <img src="man/figures/logo.png" align="right" height="139" alt="" />

<!-- badges: start -->
[![R-CMD-check](https://github.com/username/mypackage/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/username/mypackage/actions/workflows/R-CMD-check.yaml)
<!-- badges: end -->

Brief description of your package.

## Installation

You can install the development version of mypackage from GitHub:

```r
# install.packages("devtools")
devtools::install_github("username/mypackage")
```
```

**Height specification**: 139 pixels is standard (1/5 of typical hex height)

### Alternative: Center Logo

```markdown
<div align="center">
  <img src="man/figures/logo.png" width="200" alt="mypackage logo">
  <h1>mypackage</h1>
  <p>Brief tagline</p>
</div>
```

## Common Pitfalls

### 1. Logo Not Showing on GitHub

**Problem**: Logo displays locally but not on GitHub

**Fix**:
- Commit logo files: `git add man/figures/logo.png`
- Use relative path: `man/figures/logo.png` (not absolute)
- Check file exists in repository

### 2. Poor Text Readability

**Problem**: Package name is hard to read at small sizes

**Fix**:
- Increase font size: `p_size = 24` or higher
- Use bold font: `p_fontface = "bold"`
- Ensure high contrast between text and background
- Avoid decorative fonts for package name

### 3. Image Subplot Off-Center

**Problem**: Icon or plot is positioned incorrectly

**Fix**: Adjust position parameters:
```r
s_x = 1     # Center horizontally (0.5-1.5 range)
s_y = 0.85  # Adjust vertical position (0.5-1.5 range)
```

Experiment with values to find optimal placement.

### 4. Fonts Not Rendering

**Problem**: Google Fonts don't appear in output

**Fix**:
```r
# Ensure showtext is loaded and enabled
library(showtext)
showtext_auto()

# Add font BEFORE creating sticker
font_add_google("Font Name", "fontid")

# Use correct font family ID
p_family = "fontid"  # Not "Font Name"
```

### 5. SVG Export Issues

**Problem**: SVG file has rendering problems

**Fix**:
- Some complex ggplot2 features don't export well to SVG
- Simplify plot (reduce transparency, gradients)
- Or stick with PNG for complex visualizations

### 6. Large File Size

**Problem**: PNG file is >500 KB

**Fix**:
```r
# After creating logo, optimize
library(magick)
logo <- image_read("man/figures/logo.png")
logo <- image_resize(logo, "1024x")  # Resize if too large
logo <- image_write(logo, "man/figures/logo.png", quality = 85)
```

### 7. Colors Look Different on Web

**Problem**: Colors appear different on different screens

**Fix**:
- Use web-safe colors
- Test on multiple devices
- Use hex color codes consistently
- Avoid very saturated colors

### 8. Logo Dimensions Wrong

**Problem**: Logo is squashed or stretched

**Fix**: Don't override aspect ratio - hexSticker handles this automatically. Remove any `asp` parameter.

### 9. URL Text Unreadable

**Problem**: URL at bottom is too small or angled

**Fix**:
```r
u_size = 4      # Increase size
u_angle = 0     # Make horizontal
u_color = "#FFFFFF"  # Ensure contrast
```

### 10. Subplot Cuts Off

**Problem**: Image or plot is clipped at edges

**Fix**:
```r
s_width = 0.5   # Reduce width
s_height = 0.5  # Reduce height
# Add padding in original image/plot
```

## Automation and Reproducibility

### Makefile Integration

Create `Makefile`:

```makefile
.PHONY: logo clean

logo:
	Rscript man/figures/generate_logo.R

clean:
	rm -f man/figures/logo.png man/figures/logo.svg
```

Usage:
```bash
make logo   # Generate logo
make clean  # Remove generated files
```

### GitHub Actions

Add to `.github/workflows/logo.yaml`:

```yaml
name: Update Logo

on:
  push:
    paths:
      - 'man/figures/generate_logo.R'

jobs:
  update-logo:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: r-lib/actions/setup-r@v2
      - name: Install dependencies
        run: |
          install.packages(c("hexSticker", "showtext", "ggplot2"))
        shell: Rscript {0}
      - name: Generate logo
        run: Rscript man/figures/generate_logo.R
      - name: Commit logo
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add man/figures/logo.png man/figures/logo.svg
          git commit -m "Update logo" || echo "No changes"
          git push
```

## Logo Variations

### Different Output Sizes

```r
# Standard size (for README)
sticker(..., filename = "man/figures/logo.png", dpi = 300)

# Large size (for print)
sticker(..., filename = "man/figures/logo_large.png", dpi = 600)

# Thumbnail (for website favicon)
library(magick)
img <- image_read("man/figures/logo.png")
img_thumb <- image_resize(img, "256x256")
image_write(img_thumb, "man/figures/logo_256.png")
```

### White Border Version

```r
# Black background version
sticker(..., h_fill = "#000000", p_color = "#FFFFFF")

# White background version
sticker(..., h_fill = "#FFFFFF", p_color = "#000000", h_color = "#000000")
```

Creating a professional hex logo enhances your package's visibility and brand recognition!
