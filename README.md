The R Encoding FAQ
================

# Introduction

The goal of this document is twofold: (1) collect current best practices
to solve text encoding related issues and (2) include links to further
information about text encoding.

# Encoding of text

# R’s text representation

# R Connections

# Printing text to the console

## Display width

# Issues when building packages

## Code files

## Unit tests

-   How do I test my code in a different locale?

## Manual pages

## Vignettes

# Graphics

# Issues on Windows

# Encoding related R functions and packages

-   `base::Encoding()`

-   `base::iconv()`

-   `base::iconvlist()`

-   `base::nchar()`

-   `base::l10n_info()`

-   [stringi package](https://cran.rstudio.com/web/packages/stringi/)

-   [utf8 package](https://cran.rstudio.com/web/packages/utf8)

-   [brio package](https://cran.rstudio.com/web/packages/brio)

-   [cli package](https://cran.rstudio.com/web/packages/cli/)

-   [Unicode package](https://cran.rstudio.com/web/packages/Unicode)

# Further Reading

Good general introductions to text encodings:

-   [The Absolute Minimum Every Software Developer Absolutely,
    Positively Must Know About Unicode and Character Sets (No
    Excuses!)](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/)

-   [What Every Programmer Absolutely, Positively Needs To Know About
    Encodings And Character Sets To Work With
    Text](https://kunststube.net/encoding/)

-   [UTF-8 Everywhere](http://utf8everywhere.org)

R documentation:

-   [Encodings for
    CHARSXPs](https://cran.r-project.org/doc/manuals/r-devel/R-ints.html#Encodings-for-CHARSXPs)
    section in [R
    internals](https://cran.r-project.org/doc/manuals/r-devel/R-ints.html)

-   [Writing portable packages / Encoding
    issues](https://cran.r-project.org/doc/manuals/R-exts.html#Encoding-issues)
    in [Writing R
    Extensions](https://cran.r-project.org/doc/manuals/R-exts.html)

-   [Encodings and
    R](https://developer.r-project.org/Encodings_and_R.html) – *Old and
    slightly outdated, it gives a good summary of what it took to change
    R’s internal encoding.*

Intros to various encodings:

-   [UTF-8 on Wikipedia](https://en.wikipedia.org/wiki/UTF-8)

-   [UTF-16 on Wikipedia](https://en.wikipedia.org/wiki/UTF-16)

-   [latin-1 (ISO/IEC 8859-1) on
    Wikipedia](https://en.wikipedia.org/wiki/ISO/IEC_8859-1)

About Unicode:

-   [The main Unicode homepage](https://home.unicode.org/)

-   [Unicode FAQs](https://unicode.org/faq/)
