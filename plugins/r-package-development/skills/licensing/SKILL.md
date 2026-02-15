---
name: licensing
description: R package licensing including common licenses, compatibility matrix, proper license file setup, and handling bundled code
---

# R Package Licensing Guide

This skill covers everything about licensing R packages, from choosing the right license to properly handling bundled third-party code. Proper licensing is essential for CRAN submission and legal compliance.

## Rules

1. **Every package must have a license** specified in DESCRIPTION
2. **Use standard license identifiers** (MIT, GPL-3, Apache-2.0, etc.)
3. **MIT requires LICENSE file** with YEAR and COPYRIGHT HOLDER
4. **GPL-3 allows use in other GPL software** only
5. **Check license compatibility** before bundling code
6. **Copyright holders get cph role** in Authors@R
7. **Stack Overflow code is CC-BY-SA** (GPL-3 compatible only)
8. **Data can have different licenses** from code (e.g., CC0)
9. **Proprietary licenses need CRAN approval** before submission
10. **Bundle third-party code** only with compatible licenses

## Common Open Source Licenses

### MIT License

**Characteristics**:
- Very permissive
- Can be used in proprietary software
- Requires attribution
- No patent grant

**When to use**:
- You want maximum adoption
- Don't care if used in proprietary software
- Want simple, short license

**Setup**:
```r
usethis::use_mit_license()
# Or specify copyright holder
usethis::use_mit_license("Your Name")
```

**Result**:
```
# DESCRIPTION
License: MIT + file LICENSE

# LICENSE (DCF format - no newlines!)
YEAR: 2024
COPYRIGHT HOLDER: Your Name

# LICENSE.md (full text)
# MIT License
#
# Copyright (c) 2024 Your Name
# [full license text...]

# .Rbuildignore
^LICENSE\.md$
```

**Important**:
- LICENSE file is two-line DCF format (used by R)
- LICENSE.md has full text (for GitHub)
- LICENSE.md is excluded from built package

### GPL-3 (GNU General Public License v3)

**Characteristics**:
- Copyleft (derivative works must be GPL)
- Cannot be used in proprietary software
- Includes patent grant
- Source code must be provided

**When to use**:
- You want derivatives to remain open source
- Okay with restricting commercial use
- Philosophical commitment to free software

**Setup**:
```r
usethis::use_gpl_license(version = 3)
# Or
usethis::use_gpl_license(version = 2)  # For GPL-2
```

**Result**:
```
# DESCRIPTION
License: GPL (>= 3)

# Or for GPL-2 only
License: GPL-2
```

**GPL versions**:
- `GPL-2`: Version 2 only
- `GPL-3`: Version 3 only
- `GPL (>= 2)`: Version 2 or later (recommended)
- `GPL (>= 3)`: Version 3 or later

### Apache License 2.0

**Characteristics**:
- Permissive like MIT
- Explicit patent grant
- Requires preservation of notices
- Can be used in proprietary software

**When to use**:
- Want patent protection
- More formal than MIT
- Corporate environments prefer it

**Setup**:
```r
usethis::use_apache_license(version = 2)
```

**Result**:
```
# DESCRIPTION
License: Apache License (>= 2)
```

### LGPL (Lesser General Public License)

**Characteristics**:
- Copyleft for library itself
- Can be used in proprietary software (dynamically linked)
- Modifications to library must be LGPL

**When to use**:
- Want library improvements to be shared
- Okay with use in proprietary software

**Setup**:
```r
usethis::use_lgpl_license(version = 3)
```

**Result**:
```
# DESCRIPTION
License: LGPL (>= 3)
```

### AGPL (Affero General Public License)

**Characteristics**:
- Like GPL but covers network use
- If used in web service, source must be provided
- Strongest copyleft

**When to use**:
- Want to prevent SaaS loophole
- Ensure cloud/web services share code

**Setup**:
```r
usethis::use_agpl_license(version = 3)
```

**Result**:
```
# DESCRIPTION
License: AGPL (>= 3)
```

### Creative Commons Licenses

**For data and documentation**, not code:

**CC0 (Public Domain)**:
```
# DESCRIPTION (for data package)
License: CC0
```

**CC-BY 4.0** (Attribution required):
```
# DESCRIPTION
License: CC BY 4.0
```

**When to use**:
- CC0: Dedicate to public domain (datasets)
- CC-BY: Require attribution (datasets, documentation)

**Setup**:
```r
usethis::use_cc0_license()  # For data
usethis::use_ccby_license()  # For data with attribution
```

### BSD Licenses

**BSD-2-Clause** (similar to MIT):
```
# DESCRIPTION
License: BSD_2_clause + file LICENSE
```

**BSD-3-Clause** (adds non-endorsement clause):
```
# DESCRIPTION
License: BSD_3_clause + file LICENSE
```

**Setup**:
```r
# No usethis helper, manual setup
# Copy license text to LICENSE file
```

## License Compatibility Matrix

### Can Package A (license X) use Package B (license Y)?

|  A \ B  |  MIT  | Apache-2.0 | BSD | LGPL | GPL-2 | GPL-3 | AGPL-3 |
|---------|-------|------------|-----|------|-------|-------|--------|
| MIT     | ✓     | ✓          | ✓   | ✓    | ✓     | ✓     | ✓      |
| Apache  | ✓     | ✓          | ✓   | ✓    | ✗     | ✓     | ✓      |
| LGPL    | ✓     | ✓          | ✓   | ✓    | ✓     | ✓     | ✓      |
| GPL-2   | ✓     | ✗          | ✓   | ✓    | ✓     | ✗     | ✗      |
| GPL-3   | ✓     | ✓          | ✓   | ✓    | ✗     | ✓     | ✓      |
| AGPL-3  | ✓     | ✓          | ✓   | ✓    | ✗     | ✓     | ✓      |

✓ = Compatible (can use)
✗ = Incompatible (cannot use)

### Interpretation

**MIT/BSD packages can use anything** (permissive → any direction)

**GPL-3 packages cannot use GPL-2-only** code (version incompatibility)

**GPL-2 packages cannot use Apache-2.0** (patent clause conflicts)

**AGPL is GPL-3 compatible** (but stricter network copyleft)

### Practical Examples

```r
# Your package: MIT
# Can depend on: anything

# Your package: GPL-3
# Can depend on: MIT, Apache-2.0, BSD, LGPL, GPL-2+, GPL-3, AGPL-3
# Cannot depend on: GPL-2 only

# Your package: GPL-2
# Can depend on: MIT, BSD, LGPL, GPL-2+, GPL-2
# Cannot depend on: Apache-2.0, GPL-3 only, AGPL-3
```

## Proper License File Setup

### MIT License Files

**DESCRIPTION**:
```
License: MIT + file LICENSE
```

**LICENSE** (DCF format, auto-generated by usethis):
```
YEAR: 2024
COPYRIGHT HOLDER: Your Name
```

**LICENSE.md** (full text, for humans):
```markdown
# MIT License

Copyright (c) 2024 Your Name

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

**.Rbuildignore**:
```
^LICENSE\.md$
```

**Why this structure?**
- R packages use DCF LICENSE file (required by CRAN)
- LICENSE.md is for GitHub display (excluded from package)
- MIT + file LICENSE is standard CRAN format

### GPL License Files

**DESCRIPTION**:
```
License: GPL (>= 3)
```

**No LICENSE file needed** - R includes GPL text

**Optional LICENSE.md** for GitHub:
```markdown
# GPL-3 License

This package is licensed under GPL-3 or later.
See https://www.gnu.org/licenses/gpl-3.0.html
```

### Apache License Files

**DESCRIPTION**:
```
License: Apache License (>= 2)
```

**Optional LICENSE.md**:
```markdown
# Apache License 2.0

See https://www.apache.org/licenses/LICENSE-2.0
```

## Authors@R and Copyright

### Specifying Copyright Holders

```r
Authors@R: c(
    person("Jane", "Developer",
           email = "jane@example.com",
           role = c("aut", "cre"),
           comment = c(ORCID = "0000-0001-2345-6789")),
    person("John", "Contributor",
           role = "aut"),
    person("University Name",
           role = "cph"),  # Copyright holder
    person("Company Name",
           role = c("cph", "fnd"))  # Copyright holder and funder
)
```

**Roles**:
- `aut`: Author (wrote substantial code)
- `cre`: Creator/Maintainer (only one, must have email)
- `ctb`: Contributor (small contributions)
- `cph`: Copyright holder (legal owner)
- `fnd`: Funder
- `ths`: Thesis advisor

### When Institution Holds Copyright

If employed and work is done for institution:

```r
Authors@R: c(
    person("Your", "Name",
           email = "you@university.edu",
           role = c("aut", "cre"),
           comment = c(ORCID = "0000-0001-2345-6789")),
    person("University of Example",
           role = "cph")
)
```

LICENSE file:
```
YEAR: 2024
COPYRIGHT HOLDER: University of Example
```

### Multiple Copyright Holders

```r
Authors@R: c(
    person("First", "Author",
           role = c("aut", "cre", "cph"),
           email = "first@example.com"),
    person("Second", "Author",
           role = c("aut", "cph"))
)
```

LICENSE:
```
YEAR: 2024
COPYRIGHT HOLDER: First Author and Second Author
```

## Bundling Third-Party Code

### When You Include External Code

**Scenario**: You want to include JavaScript library, Python code, or code from another R package.

### License Compatibility Check

1. Check third-party code license
2. Verify compatibility with your license (see matrix above)
3. Preserve original copyright and license

### Example: Including MIT-licensed JavaScript

**Directory structure**:
```
inst/
  htmlwidgets/
    lib/
      d3/
        d3.min.js
        LICENSE
```

**Preserve original LICENSE**:
- Keep original LICENSE file with the code
- Or create LICENSE.note

**Update Authors@R**:
```r
Authors@R: c(
    person("Your", "Name",
           role = c("aut", "cre"),
           email = "you@example.com"),
    person("Mike", "Bostock",  # D3.js author
           role = "cph",
           comment = "D3.js library")
)
```

**Optional LICENSE.note**:
```
This package includes code from:

D3.js (https://d3js.org/)
Copyright (c) 2010-2024 Mike Bostock
Licensed under ISC License
```

### Example: Including GPL Code

If you include GPL code, **your package must be GPL**:

```r
# DESCRIPTION
License: GPL (>= 3)

Authors@R: c(
    person("Your", "Name",
           role = c("aut", "cre"),
           email = "you@example.com"),
    person("Original", "Author",
           role = "cph",
           comment = "Portions of C code from original-package")
)
```

**Important**: Cannot include GPL code in MIT package!

### Stack Overflow Code

Code from Stack Overflow is **CC-BY-SA 4.0** licensed.

**CC-BY-SA is GPL-3 compatible** but not MIT/Apache compatible.

**If using Stack Overflow code**:

Option 1: Make package GPL-3+
```r
# DESCRIPTION
License: GPL (>= 3)
```

Option 2: Rewrite the code yourself

Option 3: Get explicit permission from author to use under different license

**Attribution**:
```r
# In code comments
# Based on Stack Overflow answer by Username
# https://stackoverflow.com/a/12345678
# Licensed under CC-BY-SA 4.0
```

### CPP/C/Fortran Code

**Example: Including Eigen library (MPL-2.0)**:

```
src/
  eigenlib/
    Eigen/
      Core
      [other headers]
    COPYING  # Eigen's license

inst/
  COPYRIGHTS
```

COPYRIGHTS file:
```
This package includes Eigen library:
  Copyright (C) Eigen developers
  Licensed under MPL-2.0
  https://eigen.tuxfamily.org/
```

AUTHORS@R:
```r
person("Eigen developers",
       role = "cph",
       comment = "Eigen C++ library")
```

## Data Licenses

### Separating Code and Data Licenses

Code and data can have different licenses:

**For code with open data**:
```r
# DESCRIPTION
License: MIT + file LICENSE  # Code is MIT
```

**In data documentation**:
```r
#' Example Dataset
#'
#' @format A data frame with 1000 rows and 5 variables
#' @source \url{https://example.com/data}
#' @section License:
#' This dataset is released under CC0 (public domain).
#' Code in this package is MIT licensed.
"example_data"
```

### Common Data Licenses

**CC0** (Public Domain Dedication):
```r
#' @section License:
#' CC0 - dedicated to the public domain.
#' \url{https://creativecommons.org/publicdomain/zero/1.0/}
```

**CC-BY 4.0** (Attribution Required):
```r
#' @section License:
#' CC-BY 4.0 - Attribution required.
#' \url{https://creativecommons.org/licenses/by/4.0/}
```

**ODbL** (Open Database License):
```r
#' @section License:
#' Open Database License (ODbL) v1.0
#' \url{https://opendatacommons.org/licenses/odbl/}
```

### Data Citation

```r
#' Example Dataset from Study X
#'
#' @format Data frame with 500 observations
#' @source Original data from:
#'   Smith et al. (2020) Journal of Examples.
#'   DOI: 10.1234/example
#' @section License:
#'   Data licensed under CC-BY 4.0 by the original authors.
#'   Please cite the original publication when using this data.
"study_data"
```

## Proprietary Licenses

### When You Can't Open Source

Sometimes code must be proprietary (employer requirement, trade secrets).

**Setup**:
```r
usethis::use_proprietary_license("Your Company Name")
```

**Result**:
```
# DESCRIPTION
License: file LICENSE

# LICENSE
Proprietary

Do not distribute outside of Your Company Name.
```

**CRAN Submission**:
- Contact CRAN before submitting
- Usually rejected unless special arrangement
- Consider internal repository instead

### Limited Distribution

```
# LICENSE
Proprietary - Academic Use Only

This software is provided for academic research use only.
Commercial use requires a separate license.
Contact: licensing@university.edu
```

## Common Pitfalls

### 1. Missing LICENSE File for MIT

**Problem**:
```
DESCRIPTION: License: MIT
# But no LICENSE file
```

**Fix**:
```r
usethis::use_mit_license("Your Name")
```

### 2. Wrong LICENSE File Format

**Problem**:
```
# LICENSE (wrong - this is prose)
MIT License

Copyright (c) 2024 Your Name
...
```

**Fix**: Use DCF format:
```
YEAR: 2024
COPYRIGHT HOLDER: Your Name
```

Full text goes in LICENSE.md (excluded from build).

### 3. Including Incompatible Licenses

**Problem**: MIT package includes GPL-3 code

**Fix**: Either:
- Remove GPL code
- Change package to GPL-3
- Get permission to relicense

### 4. Forgetting Copyright Holder

**Problem**:
```r
Authors@R: person("Name", role = c("aut", "cre"))
# No cph for employer!
```

**Fix**:
```r
Authors@R: c(
    person("Name", role = c("aut", "cre")),
    person("Company", role = "cph")
)
```

### 5. Using Code Without Attribution

**Problem**: Copying code without preserving copyright

**Fix**: Always attribute:
```r
# Based on code from package X by Author Y
# Licensed under MIT
# https://github.com/author/package
```

### 6. CC Licenses for Code

**Problem**:
```
License: CC-BY 4.0  # For software code
```

**Fix**: Use CC licenses for data/documentation only. Use MIT/GPL/Apache for code.

### 7. GPL-2 Only Inflexibility

**Problem**:
```
License: GPL-2  # Only version 2
```

**Fix**: Use "or later" for flexibility:
```
License: GPL (>= 2)
```

### 8. LICENSE.md Not Excluded

**Problem**: LICENSE.md included in built package

**Fix**: Add to .Rbuildignore:
```
^LICENSE\.md$
```

### 9. Stack Overflow Code in MIT Package

**Problem**: Using SO code in permissive-licensed package

**Fix**:
- Switch to GPL-3
- Or rewrite the code
- Or get explicit permission

### 10. Unclear Data Provenance

**Problem**: Datasets with unknown licenses

**Fix**: Document data sources and licenses:
```r
#' @source Original data: \url{https://example.com}
#' @section License: CC0 public domain dedication
```

## License Decision Tree

```
Do you want others to share improvements?
├─ Yes → GPL-3 (strong copyleft)
│  └─ Network use matters? → AGPL-3
└─ No (permissive) → Continue

Do you need patent protection?
├─ Yes → Apache-2.0
└─ No → MIT (simplest)

Corporate/institutional setting?
├─ Yes → Apache-2.0 (explicit patent grant)
└─ No → MIT (simplest)

Data or documentation?
├─ Data → CC0 (public domain)
└─ Documentation → CC-BY 4.0
```

## Checking License Compliance

### Manual Check

```r
# List all dependencies
desc::desc_get_deps()

# Check each dependency's license
# On CRAN: https://cran.r-project.org/package=PACKAGE
# Look at DESCRIPTION file

# Verify compatibility with your license
```

### Automated Check

```r
# Install license checker
install.packages("renv")

# Check licenses
renv::dependencies() |>
  dplyr::select(Package) |>
  dplyr::distinct() |>
  dplyr::rowwise() |>
  dplyr::mutate(License = tryCatch(
    packageDescription(Package)$License,
    error = function(e) "Not installed"
  ))
```

## Resources

### Official Documentation

- [CRAN License List](https://cran.r-project.org/doc/manuals/r-release/R-exts.html#Licensing)
- [Choose a License](https://choosealicense.com/)
- [Open Source Initiative](https://opensource.org/licenses)

### License Texts

- MIT: https://opensource.org/licenses/MIT
- GPL-3: https://www.gnu.org/licenses/gpl-3.0.html
- Apache-2.0: https://www.apache.org/licenses/LICENSE-2.0
- CC0: https://creativecommons.org/publicdomain/zero/1.0/

### Getting Help

```r
# List available licenses in usethis
help(package = "usethis", topic = "licenses")

# Common functions
usethis::use_mit_license()
usethis::use_gpl_license()
usethis::use_apache_license()
usethis::use_cc0_license()
usethis::use_ccby_license()
usethis::use_proprietary_license()
```

Proper licensing protects both you and your users!
