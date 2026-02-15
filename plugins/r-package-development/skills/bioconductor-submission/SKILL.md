---
name: bioconductor-submission
description: Bioconductor-specific package submission requirements, BiocCheck validation, version numbering, and review process beyond standard CRAN requirements
---

# Bioconductor Submission Guide

This skill covers submitting packages to Bioconductor, which has additional requirements beyond CRAN. Bioconductor is the primary repository for computational biology and bioinformatics R packages.

## Rules

1. **Version scheme**: Use x.y.z where y is even for release, odd for devel
2. **biocViews required**: Must include appropriate biocViews terms in DESCRIPTION
3. **BiocCheck must pass**: Run BiocCheck::BiocCheck() and fix all errors/warnings
4. **Vignettes must run**: No eval=FALSE for all chunks; use real data
5. **S4 classes preferred**: Use S4 for complex data structures
6. **BiocStyle for vignettes**: Use BiocStyle package for consistent formatting
7. **Use BiocManager::install()**: Not install.packages() in documentation
8. **Submit via GitHub issue**: To Bioconductor/Contributions repository
9. **Active maintenance required**: Must respond to build reports and issues
10. **Follow Bioconductor guidelines**: Read package guidelines thoroughly

## Bioconductor vs CRAN

### When to Submit to Bioconductor

Submit to Bioconductor if your package:
- Analyzes genomic data (sequences, annotations, variants)
- Analyzes high-throughput biological data (microarrays, RNA-seq, proteomics)
- Provides infrastructure for biological data types
- Integrates with existing Bioconductor packages
- Uses Bioconductor data structures (SummarizedExperiment, GenomicRanges, etc.)

### When to Submit to CRAN

Submit to CRAN if your package:
- Provides general statistical methods
- Doesn't specifically work with biological data
- Doesn't depend on Bioconductor packages
- Is a general-purpose tool that happens to be useful for biology

**Note**: Packages can be on both, but start with one.

## Version Numbering Scheme

Bioconductor uses a specific version numbering system:

### Version Format: x.y.z

- **x** (major): Rarely changes; major redesign
- **y** (minor): Even for release, odd for devel
- **z** (patch): Bug fixes and minor updates

### Examples

```
# Initial development
0.99.0  -> Start here for new packages
0.99.1  -> Bug fixes during review
0.99.2  -> More fixes

# First release (Bioconductor assigns this)
1.0.0   -> First Bioc release version

# Development continues
1.1.0   -> Devel version after 1.0.0 release

# Next release cycle
1.2.0   -> Next Bioc release (even y)
1.3.0   -> Devel version after 1.2.0

# Bug fixes
1.2.1   -> Bug fix for release
1.3.1   -> Bug fix for devel
```

### Version Management

```r
# Start new package
Version: 0.99.0

# During review, bump z for fixes
Version: 0.99.1
Version: 0.99.2

# After acceptance, Bioconductor core sets
Version: 1.0.0  # For next release

# You continue development
Version: 1.1.0  # Devel version
```

**Critical**: Don't manually set version to 1.0.0 - Bioconductor does this.

## DESCRIPTION File Requirements

### Complete Bioconductor DESCRIPTION

```
Package: MyBiocPackage
Title: Analysis of Single-Cell RNA Sequencing Data
Version: 0.99.0
Authors@R: c(
    person("First", "Last",
           email = "email@institution.edu",
           role = c("aut", "cre"),
           comment = c(ORCID = "0000-0001-2345-6789")),
    person("Second", "Author",
           role = "aut",
           comment = c(ORCID = "0000-0001-2345-6780"))
    )
Description: Provides methods for analyzing single-cell RNA sequencing
    data. Implements novel clustering algorithms and visualization
    techniques. Integrates with Bioconductor infrastructure including
    SingleCellExperiment and other core data structures.
License: Artistic-2.0
Encoding: UTF-8
LazyData: false
Depends:
    R (>= 4.4.0)
Imports:
    BiocGenerics,
    S4Vectors,
    SummarizedExperiment,
    SingleCellExperiment,
    methods,
    stats,
    graphics
Suggests:
    BiocStyle,
    knitr,
    rmarkdown,
    testthat (>= 3.0.0),
    scRNAseq
biocViews: Software, SingleCell, RNASeq, Clustering, Visualization,
    DimensionReduction, GeneExpression
VignetteBuilder: knitr
RoxygenNote: 7.3.0
Roxygen: list(markdown = TRUE)
URL: https://github.com/username/MyBiocPackage
BugReports: https://github.com/username/MyBiocPackage/issues
```

### biocViews Requirements

Every Bioconductor package must have appropriate biocViews terms:

```r
# Find appropriate terms
BiocManager::install("biocViews")
library(biocViews)

# Browse available terms
data(biocViewsVocab)
biocViewsVocab

# Common top-level categories
# Software - computational tools
# AnnotationData - annotation packages
# ExperimentData - example/experiment data packages
# Workflow - workflow packages
```

### Common biocViews Terms

**Software packages**:
```
biocViews: Software, RNASeq, GeneExpression, Transcriptomics,
    DifferentialExpression, Sequencing, Coverage, Alignment
```

**By technology**:
- RNASeq, ChIPSeq, ATACSeq, DNASeq, MethylSeq
- Microarray, Proteomics, Metabolomics
- SingleCell, SpatialData, MultiChannel

**By analysis type**:
- DifferentialExpression, Clustering, Classification
- Normalization, Preprocessing, QualityControl
- Visualization, Annotation, GenomeAnnotation

**By data type**:
- GeneExpression, Epigenetics, StructuralVariation
- CopyNumberVariation, SNP, Transcriptomics

### License Requirements

Bioconductor prefers open-source licenses:
- **Artistic-2.0** (recommended for Bioconductor)
- GPL (>= 2)
- LGPL
- MIT
- BSD

```r
# Set license
usethis::use_mit_license()
# Or manually in DESCRIPTION
License: Artistic-2.0
```

## BiocCheck Validation

### Installing BiocCheck

```r
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install("BiocCheck")
```

### Running BiocCheck

```r
# Basic check
BiocCheck::BiocCheck()

# More detailed
BiocCheck::BiocCheck(".", new.package = TRUE)

# For package updates
BiocCheck::BiocCheck(".", new.package = FALSE)
```

### Understanding BiocCheck Output

BiocCheck reports three levels:
- **ERROR**: Must fix (will prevent acceptance)
- **WARNING**: Should fix (reviewers will ask)
- **NOTE**: Consider fixing (good practice)

### Common BiocCheck Issues and Fixes

#### 1. Version Number Issues

**Problem**:
```
ERROR: Version number must be 0.99.0 for new packages
```

**Fix**:
```
# In DESCRIPTION
Version: 0.99.0
```

#### 2. biocViews Missing

**Problem**:
```
ERROR: Package must have biocViews
```

**Fix**:
```
# Add to DESCRIPTION
biocViews: Software, Sequencing, RNASeq
```

#### 3. Vignette eval=FALSE

**Problem**:
```
WARNING: Vignette chunks should not use eval=FALSE
```

**Fix**: Make vignettes actually run:
```r
# Bad
```{r, eval=FALSE}
result <- analyze_data(data)
```

# Good - provide real example data
```{r}
data("example_dataset")
result <- analyze_data(example_dataset)
```
```

#### 4. Non-BiocStyle Vignette

**Problem**:
```
NOTE: Consider using BiocStyle package
```

**Fix**:
```yaml
---
title: "Package Vignette"
author: "Your Name"
output: BiocStyle::html_document
vignette: >
  %\VignetteIndexEntry{Package Vignette}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---

```{r style, echo=FALSE, results='asis'}
BiocStyle::markdown()
```
```

#### 5. install.packages() in Documentation

**Problem**:
```
WARNING: Use BiocManager::install() not install.packages()
```

**Fix**:
```r
# Bad
#' Install with: install.packages("MyPackage")

# Good
#' Install with: BiocManager::install("MyPackage")
```

#### 6. Long Line Lengths

**Problem**:
```
NOTE: Lines should be <= 80 characters
```

**Fix**: Break long lines:
```r
# Use styler
styler::style_pkg()

# Or manually
result <- my_function(
    argument1 = value1,
    argument2 = value2,
    argument3 = value3
)
```

#### 7. Missing NEWS File

**Problem**:
```
NOTE: Consider adding NEWS file
```

**Fix**:
```r
usethis::use_news_md()
```

Content:
```markdown
# MyBiocPackage 0.99.0

* Initial Bioconductor submission
* Implements core functionality for X
* Includes vignette demonstrating Y
```

#### 8. Package Size Too Large

**Problem**:
```
WARNING: Package size is X MB
```

**Fix**:
- Remove large example data
- Create separate data package
- Compress data files
- Use external data with ExperimentHub

#### 9. Using .Rbuildignore Incorrectly

**Problem**:
```
WARNING: Don't use .Rbuildignore excessively
```

**Fix**: Only ignore truly unnecessary files:
```
^.*\.Rproj$
^\.Rproj\.user$
^\.github$
^_pkgdown\.yml$
^docs$
^pkgdown$
```

#### 10. Missing runnable examples

**Problem**:
```
ERROR: All exported functions must have runnable examples
```

**Fix**:
```r
#' My Function
#'
#' @examples
#' # Load example data
#' data("example_data")
#'
#' # Run analysis
#' result <- my_function(example_data)
#'
#' @export
my_function <- function(x) {
    # implementation
}
```

## S4 Classes and Methods

Bioconductor strongly encourages S4 for complex data structures:

### Defining S4 Classes

```r
#' MyData Class
#'
#' @slot counts matrix of counts
#' @slot metadata data.frame of sample metadata
#' @slot features data.frame of feature metadata
#'
#' @export
setClass("MyData",
    slots = c(
        counts = "matrix",
        metadata = "data.frame",
        features = "data.frame"
    )
)
```

### S4 Constructor

```r
#' Create MyData Object
#'
#' @param counts matrix of counts
#' @param metadata data.frame of metadata
#' @param features data.frame of features
#'
#' @return MyData object
#'
#' @examples
#' counts <- matrix(rpois(100, 10), nrow=10)
#' metadata <- data.frame(sample=paste0("S", 1:10))
#' features <- data.frame(gene=paste0("G", 1:10))
#' obj <- MyData(counts, metadata, features)
#'
#' @export
MyData <- function(counts, metadata, features) {
    new("MyData",
        counts = counts,
        metadata = metadata,
        features = features
    )
}
```

### S4 Methods

```r
#' @export
setGeneric("getCounts", function(x) standardGeneric("getCounts"))

#' @export
setMethod("getCounts", "MyData", function(x) x@counts)

#' @export
setMethod("show", "MyData", function(object) {
    cat("MyData object\n")
    cat("  Samples:", ncol(object@counts), "\n")
    cat("  Features:", nrow(object@counts), "\n")
})
```

## Vignette Requirements

### BiocStyle Vignette Template

```r
---
title: "Introduction to MyBiocPackage"
author:
  - name: Your Name
    affiliation: Institution
    email: email@institution.edu
date: "`r Sys.Date()`"
output:
  BiocStyle::html_document:
    toc: true
    toc_depth: 2
vignette: >
  %\VignetteIndexEntry{Introduction to MyBiocPackage}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE, warning = FALSE, message = FALSE)
```

```{r style, echo=FALSE, results='asis'}
BiocStyle::markdown()
```

# Introduction

This vignette demonstrates the basic usage of `r Biocpkg("MyBiocPackage")`.

# Installation

```{r install, eval=FALSE}
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install("MyBiocPackage")
```

# Quick Start

```{r quickstart}
library(MyBiocPackage)

# Load example data
data("example_data")

# Run analysis
result <- analyze(example_data)
```

# Session Info

```{r sessionInfo}
sessionInfo()
```
```

### Using Example Data

Create example data for vignettes:

```r
# In R/data.R
#' Example Single-Cell Dataset
#'
#' @description
#' A subset of single-cell RNA-seq data for demonstration.
#'
#' @format A SingleCellExperiment object with 100 cells and 500 genes
#' @examples
#' data("example_sce")
#' example_sce
"example_sce"

# Create and save data
# In data-raw/example_sce.R
library(SingleCellExperiment)
set.seed(123)

counts <- matrix(rpois(50000, 5), nrow=500, ncol=100)
rownames(counts) <- paste0("Gene", 1:500)
colnames(counts) <- paste0("Cell", 1:100)

example_sce <- SingleCellExperiment(
    assays = list(counts = counts)
)

usethis::use_data(example_sce, overwrite = TRUE)
```

## Submission Process

### 1. Pre-Submission Checklist

- [ ] Package passes `R CMD check` with no errors, warnings, or notes
- [ ] Package passes `BiocCheck::BiocCheck()` with no errors
- [ ] Version is 0.99.0 (for new packages)
- [ ] biocViews terms are appropriate and specific
- [ ] All vignette chunks run (no eval=FALSE)
- [ ] S4 classes used for complex data structures
- [ ] Vignette uses BiocStyle
- [ ] Examples use BiocManager::install()
- [ ] Package has runnable examples for all exported functions
- [ ] NEWS file exists and is up-to-date
- [ ] Package is on GitHub
- [ ] README.md is informative
- [ ] Package size is reasonable (<5 MB preferred)

### 2. GitHub Repository Setup

```r
# Ensure package is on GitHub
usethis::use_github()

# Add required files
usethis::use_readme_md()
usethis::use_news_md()

# Add GitHub Actions
usethis::use_github_action("check-bioc")
```

### 3. Open Submission Issue

1. Go to https://github.com/Bioconductor/Contributions/issues/new/choose
2. Choose "Submit a new package"
3. Fill in the template:

```markdown
## Package Name

MyBiocPackage

## Submitter Name

Your Name

## Submitter Email

your.email@institution.edu

## Package Source

https://github.com/username/MyBiocPackage

## Confirmation

- [x] I understand that Bioconductor has a 6-month release cycle
- [x] I will maintain this package
- [x] I have read the package guidelines
- [x] My package passes BiocCheck

## Additional Information

This package provides methods for analyzing single-cell RNA-seq data.
It implements novel algorithms for clustering and visualization.
```

### 4. Review Process

Timeline:
1. **Automated checks** (immediate): GitHub Actions runs checks
2. **Initial review** (1-2 weeks): Reviewer assigned
3. **Review iterations** (varies): Address reviewer comments
4. **Acceptance** (after all issues resolved): Merged to Bioconductor

### 5. Responding to Reviews

Reviewers will comment on your issue. For each comment:

1. Make requested changes
2. Commit and push to GitHub
3. Reply to the specific comment with what you changed
4. Tag reviewer when all comments addressed

Example response:
```markdown
@reviewer-name I've addressed this by:
- Adding more detailed documentation to `my_function()`
- Including an additional example
- Fixing the typo in the vignette

Commit: abc1234
```

### 6. Common Review Comments

**Documentation**:
- "Please add more details to the vignette"
- "Function X needs a more detailed description"
- "Add references to the methods section"

**Code quality**:
- "Consider using S4 classes for this data structure"
- "Use vapply instead of sapply"
- "Avoid using deprecated functions from package Y"

**Examples**:
- "Provide a complete, runnable example"
- "Example should demonstrate the main use case"
- "Add more comments to explain the workflow"

**Dependencies**:
- "Package X should be in Imports, not Suggests"
- "Do you really need all these dependencies?"
- "Consider using the Bioconductor version of package Y"

### 7. Post-Acceptance

After acceptance:
```r
# Bioconductor will:
# 1. Fork your repository to their GitHub
# 2. Set up build servers
# 3. Assign version 1.0.0 for next release
# 4. Add to Bioconductor package list

# You continue development:
# 1. Push changes to YOUR GitHub repository
# 2. Changes propagate to Bioconductor every 24 hours
# 3. Monitor build reports
# 4. Respond to user issues
```

## Package Types

### Software Packages

Most common type:
- Provides tools for analysis
- Uses existing Bioconductor infrastructure
- biocViews: Software, ...

### Annotation Packages

Provide biological annotations:
- Gene annotations
- Pathway databases
- Organism databases
- biocViews: AnnotationData, ...

### Experiment Data Packages

Provide example or published datasets:
- Must be >2 MB
- Separate from software package
- biocViews: ExperimentData, ...

### Workflow Packages

Step-by-step analysis workflows:
- Complete analysis examples
- Use multiple Bioconductor packages
- biocViews: Workflow, ...

## Data Packages

### Creating ExperimentData Package

```r
# Structure
MyExperimentData/
  DESCRIPTION
  R/
    data.R
  data/
    dataset1.rda
    dataset2.rda
  vignettes/
    overview.Rmd
  man/
    dataset1.Rd
```

DESCRIPTION:
```
Package: MyExperimentData
Title: Example Data for MyBiocPackage
Version: 0.99.0
biocViews: ExperimentData, RNASeqData, SingleCellData
License: Artistic-2.0
```

### Using ExperimentHub

For very large data (>50 MB):

```r
# Create metadata
library(ExperimentHub)

# In inst/extdata/metadata.csv
# See ExperimentHub documentation

# Users download with:
library(ExperimentHub)
eh <- ExperimentHub()
mydata <- eh[["EH12345"]]
```

## Common Pitfalls

### 1. Wrong Version Number

**Problem**: Using 1.0.0 for initial submission

**Fix**: Always use 0.99.0 for new packages

### 2. Non-Running Vignettes

**Problem**: Vignette chunks use eval=FALSE

**Fix**: Provide real example data and make vignettes run

### 3. Missing biocViews

**Problem**: Forgot to add biocViews to DESCRIPTION

**Fix**: Add appropriate terms from biocViews vocabulary

### 4. Using CRAN-style Installation

**Problem**: Documentation says install.packages()

**Fix**: Use BiocManager::install() everywhere

### 5. S3 When S4 Needed

**Problem**: Using S3 for complex data structures

**Fix**: Use S4 classes for anything non-trivial

### 6. Not Maintaining Package

**Problem**: Ignoring build reports after acceptance

**Fix**: Check build reports daily, fix issues promptly

### 7. Too Many Dependencies

**Problem**: Importing 30+ packages

**Fix**: Only import what you actually need

### 8. Large Package Size

**Problem**: Including large datasets in main package

**Fix**: Create separate ExperimentData package

### 9. Slow Vignettes

**Problem**: Vignette takes 10+ minutes to build

**Fix**: Use smaller example datasets, cache results

### 10. Ignoring Reviewer Comments

**Problem**: Not addressing all comments

**Fix**: Address every single comment, ask for clarification if needed

## Additional Resources

### Documentation

- [Bioconductor Package Guidelines](https://contributions.bioconductor.org/index.html)
- [BiocCheck Documentation](https://bioconductor.org/packages/BiocCheck/)
- [Package Submission Tracker](https://github.com/Bioconductor/Contributions/issues)

### Tools

```r
# Development tools
BiocManager::install("BiocCheck")
BiocManager::install("BiocStyle")

# Check package
BiocCheck::BiocCheck()

# Test locally
BiocManager::valid()
```

### Getting Help

- Bioconductor Slack: https://bioc-community.herokuapp.com/
- Support site: https://support.bioconductor.org/
- Developer mailing list: bioc-devel@r-project.org

### Key Differences from CRAN

| Feature | CRAN | Bioconductor |
|---------|------|--------------|
| Version scheme | x.y.z (any) | y even=release, odd=devel |
| Submission | Web form | GitHub issue |
| Review | Automated mostly | Human reviewer |
| Build cycle | On submission | Daily |
| Release cycle | Anytime | Every 6 months |
| Dependencies | Any CRAN package | Bioc or CRAN |
| Data structures | Any | S4 preferred |
| Vignette style | Any | BiocStyle preferred |

This guide should help you successfully submit your package to Bioconductor!
