The R Encoding FAQ
================

- [Introduction](#introduction)
- [Contributing](#contributing)
- [Encoding of text](#encoding-of-text)
- [R’s text representation](#rs-text-representation)
- [R Connections and encodings](#r-connections-and-encodings)
- [Printing text to the console](#printing-text-to-the-console)
  - [Display width](#display-width)
- [Issues when building packages](#issues-when-building-packages)
  - [Code files](#code-files)
  - [Symbols](#symbols)
- [Unit tests](#unit-tests)
  - [Manual pages](#manual-pages)
  - [Vignettes](#vignettes)
- [Graphics](#graphics)
- [How-to](#how-to)
  - [How to query the current native
    encoding?](#how-to-query-the-current-native-encoding)
  - [How is the `encoding` argument and option
    used?](#how-is-the-encoding-argument-and-option-used)
  - [How to read lines from UTF-8
    files?](#how-to-read-lines-from-utf-8-files)
  - [How to write lines to UTF-8
    files?](#how-to-write-lines-to-utf-8-files)
  - [How to convert text between
    encodings?](#how-to-convert-text-between-encodings)
  - [How to guess the encoding of
    text?](#how-to-guess-the-encoding-of-text)
  - [How to check if a string is
    UTF-8?](#how-to-check-if-a-string-is-utf-8)
  - [How to download web pages in the right
    encoding?](#how-to-download-web-pages-in-the-right-encoding)
  - [Are RDS files encoding-safe?](#are-rds-files-encoding-safe)
  - [How to `parse()` UTF-8 text into UTF-8
    code?](#how-to-parse-utf-8-text-into-utf-8-code)
  - [How to `deparse()` UTF-8 code into UTF-8
    text?](#how-to-deparse-utf-8-code-into-utf-8-text)
  - [How to include UTF-8 characters in a package
    `DESCRIPTION`](#how-to-include-utf-8-characters-in-a-package-description)
  - [How to get a package `DESCRIPTION` in
    UTF-8?](#how-to-get-a-package-description-in-utf-8)
  - [How to use UTF-8 file names on
    Windows?](#how-to-use-utf-8-file-names-on-windows)
  - [How to capture UTF-8 output?](#how-to-capture-utf-8-output)
  - [How to test non-ASCII output?](#how-to-test-non-ascii-output)
  - [How to avoid non-ASCII characters in the
    manual?](#how-to-avoid-non-ascii-characters-in-the-manual)
  - [How to include non-ASCII characters in PDF
    vignettes?](#how-to-include-non-ascii-characters-in-pdf-vignettes)
- [Encoding related R functions and
  packages](#encoding-related-r-functions-and-packages)
- [Known encoding issues in packages and
  functions](#known-encoding-issues-in-packages-and-functions)
- [Tips for debugging encoding
  issues](#tips-for-debugging-encoding-issues)
- [Text transformers](#text-transformers)
- [Changes in R 4.1.0](#changes-in-r-410)
- [Changes in R 4.2.0](#changes-in-r-420)
- [Further Reading](#further-reading)

# Introduction

The goal of this document is twofold: (1) collect current best practices
to solve text encoding related issues and (2) include links to further
information about text encoding.

# Contributing

Encoding of text is a large topic, and to make this FAQ useful, we need
your contributions. There are a variety of ways to contribute:

- Found a typo a broken link or some example code is not working for
  you? Please open an issue about it. Or even better, consider fixing it
  in a pull request.
- Something is missing? There is a new (or old) package of function that
  should be mentioned here? Please open an issue about it.
- Some information is wrong or something is not explained well? Please
  open an issue about it. Or even better, fix it in a pull request!
- You found an answer useful? Please tweet about it and link to this
  FAQ.

Your contributions are most appreciated.

# Encoding of text

Good general introductions to text encodings:

- [The Absolute Minimum Every Software Developer Absolutely, Positively
  Must Know About Unicode and Character Sets (No
  Excuses!)](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/)

- [What Every Programmer Absolutely, Positively Needs To Know About
  Encodings And Character Sets To Work With
  Text](https://kunststube.net/encoding/)

# R’s text representation

See the R Internals manual:

<https://cran.r-project.org/doc/manuals/r-devel/R-ints.html#Encodings-for-CHARSXPs>

# R Connections and encodings

Some notes about connections.

- Connections assume that the input is in the encoding specified by
  `getOption("encoding")`. They convert text to the native encoding.
- Functions that *create* connections have an `encoding` argument, that
  overrides `getOption("encoding")`.
- If you tell a connection that the input is already in the native
  encoding, then it will not convert the input. If the input is in fact
  *not* in the native encoding, then it is up to the user to mark it
  accordingly, e.g. as UTF-8.

# Printing text to the console

R converts strings to the native encoding when printing them to the
screen. This is important to know when debugging encoding problems: two
strings that print the same way may have a different internal
representation:

``` r
s1 <- "\xfc"
Encoding(s1) <- "latin1"
s2 <- iconv(s1, "latin1", "UTF-8")
s1
#> [1] "ü"
s2
#> [1] "ü"
```

``` r
s1 == s2
#> [1] TRUE
identical(s1, s2)
#> [1] TRUE
testthat::expect_equal(s1, s2)
testthat::expect_identical(s1, s2)
```

``` r
Encoding(s1)
#> [1] "latin1"
Encoding(s2)
#> [1] "UTF-8"
```

``` r
charToRaw(s1)
#> [1] fc
charToRaw(s2)
#> [1] c3 bc
```

## Display width

Some Unicode glyphs, e.g. typical emojis, are *wide*. They are supposed
to take up two characters in a monospace font. The set of wide glyphs
(just like the set of glyphs in general) has been changing in different
Unicode versions.

Up to version 4.0.3 R followed Unicode 8 (released in 2015) in terms of
character width, so `nchar(..., type = "width")` miscalculated the width
of many Asian characters.

R 4.0.4 follows Unicode 12.1.

R 4.1.0 - R 4.3.3 follow Unicode 13.0.

Current R-devel (will be R 4.4.0 in about a month) still uses Unicode
13.0.

To correctly calculate the display width of Unicode text, across various
R versions, you can use the utf8 package, or `cli::ansi_nchar()`.

Note that some terminals, and also RStudio follow an older Unicode
version of the standard, and they also calculate the display width
incorrectly, typically printing characters on top of each other.

# Issues when building packages

See ‘Writing R Extensions’ about including UTF-8 characters in packages:
<https://cran.r-project.org/doc/manuals/R-exts.html#Encoding-issues>

## Code files

I suggest that you keep your source files in ASCII, except for comments,
which you can write in UTF-8 if you like. (But not necessarily in
roxygen2 comments, which will go in the manual, see below!)

If you need a string containing non-ASCII characters, construct it with
`\uxxxx` or `\U{xxxxxx}` escape sequences, which yields a UTF-8-encoded
string.

``` r
one_and_a_half <- "\u0031\u00BD"
one_and_a_half
#> [1] "1½"
Encoding(one_and_a_half)
#> [1] "UTF-8"
charToRaw(one_and_a_half)
#> [1] 31 c2 bd
```

Here are some handy ways to find the Unicode code points for an existing
string:

- Copy and paste into the [Unicode character
  inspector](https://apps.timwhitlock.info/unicode/inspect).
* `sprintf("\\u%04X", utf8ToInt(x))`
- *Do we have other suggestions?*

If you need to include non-UTF-8 non-ASCII characters, e.g. for testing,
then include them via raw data, or via `\xxx` escape sequences. E.g. to
include a latin1 string, you can do either of these:

``` r
t1 <- "t\xfck\xf6rf\xfar\xf3g\xe9p"
Encoding(t1) <- "latin1"
t2 <- rawToChar(as.raw(c(
  0x74, 0xfc, 0x6b, 0xf6, 0x72, 0x66, 0xfa, 0x72, 
  0xf3, 0x67, 0xe9, 0x70)))
Encoding(t2) <- "latin1"
t1 == t2
#> [1] TRUE
identical(charToRaw(t1), charToRaw(t2))
#> [1] TRUE
```

## Symbols

Symbols must be in the native encoding, so best practice is to only use
ASCII symbols.

This is not completely equivalent to using ASCII code files, because
names, e.g. in a named list, or the row names in a data frame are also
symbols in R. So do not use non-ACSII names. (Another reason to avoid
row names in data frames.)

A typical mistake is to assign file names as list or row names when
working with files. While it might be a convenient representation, it is
a bad idea because of the possible encoding errors.

Similarly, always use ASCII column names.

# Unit tests

Unit test files are code files, so the same rule applies to them: keep
them ASCII. See the ‘How-to’ section below on specific testing related
tasks.

## Manual pages

You can use *some* UTF-8 characters in manual pages, if you include
`\endoding{UTF-8}` in the manual page, or you declared `Encoding: UTF-8`
in the `DESCRIPTION` file of the package.

Unfortunately these do not include the characters typically used in the
output of tibble and cli. The problem is with the PDF manual and LaTeX,
so you can conditionally include UTF-8 in the HTML output with the `\if`
Rd command.

If your PDF manual does not build because of a non-supported UTF-8
character has sneaked in somewhere, `tools::showNonASCIIfile()` can show
where exactly they are.

## Vignettes

TODO

# Graphics

TODO

# How-to

## How to query the current native encoding?

R uses the native encoding extensively. Some examples:

- Symbols (i.e. variable names, names in a named list, column and row
  names, etc.) are always in the native encoding.
- Strings marked with `"unknown"` encoding are assumed to be in this
  encoding.
- Output connections assume that the output is in this encoding.
- Input connection convert the input into this encoding.
- `enc2native()` encodes its input into this encoding.
- Printing to the screen (via `cat()`, `print()`, `writeLines()`, etc.)
  re-encodes strings into this encoding.

It is surprisingly tricky to query the current native encoding,
especially on older R versions. Here are a number of things to try:

1.  Call `l10n_info()`. If the native encoding is UTF-8 or latin1 you’ll
    see that in the output:

    ``` r
    ❯ l10n_info()
    $MBCS
    [1] TRUE

    $`UTF-8`
    [1] TRUE

    $`Latin-1`
    [1] FALSE
    ```

2.  On Windows, you’ll also see the code page:

    ``` r
    ❯ l10n_info()
    $MBCS
    [1] FALSE

    $`UTF-8`
    [1] FALSE

    $`Latin-1`
    [1] TRUE

    $codepage
    [1] 1252

    $system.codepage
    [1] 1252
    ```

    Append the codepage number to `"CP"` to get a string that you can
    use in `iconv()`. E.g. in this case it would be `"CP1252"`.

3.  On Unix, from R 4.1 the name of the encoding is also included in the
    answer. This is useful on non-UTF-8 systems:

    ``` r
    ❯ l10n_info()
    $MBCS
    [1] FALSE

    $`UTF-8`
    [1] FALSE

    $`Latin-1`
    [1] FALSE

    $codeset
    [1] "ISO-8859-15"
    ```

4.  On older R versions this field is not included. On R 3.5 and above
    you can use the following trick to find the encoding:

    ``` r
    ❯ rawToChar(serialize(NULL, NULL, ascii = TRUE, version = 3))
    [1] "A\n3\n262148\n197888\n5\nUTF-8\n254\n"
    ```

    Line number 6 (i.e. after the fifth `\n`) in the output is the name
    of the encoding. On R 4.0.x you can also save an RDS file and then
    use the `infoRDS()` function on it to see the current native
    encoding.

5.  Otherwise can call `Sys.getlocale()` and parsing its output will
    probably give you an encoding name that works in `iconv()`:

    ``` r
    ❯ Sys.getlocale("LC_CTYPE")
    [1] "en_US.iso885915"
    ```

## How is the `encoding` argument and option used?

In general functions use the `encoding` argument two ways:

- For some functions it specifies the encoding of the input.
- For others it specifies how output should be marked.

For example

``` r
file("input.txt", encoding = "UTF-8")
```

tells `file()` that `input.txt` is in `UTF-8`. On the other hand

``` r
scan("input2.txt", what = "", encoding = "UTF-8")
```

means that the output of `scan()` will be *marked* as having UTF-8
encoding. (`scan()` also has a `fileEncoding` argument, which specifies
the encoding of the input file.

TODO: the weirdness of `readLines()`.

## How to read lines from UTF-8 files?

### With the brio package

The simplest option is to use the brio package, if you can allow it:

``` r
brio::read_lines("file")
```

(brio also has a `read_file()` function if you want to read in the whole
file into a string.)

brio does not currently check if the file is valid in UTF-8. See below
how to do that.

### With base R only

The xfun package has a nice function that reads UTF-8 files with base R
only: `xfun::read_utf8()`. It currently reads like this, after some
simplification:

``` r
read_utf8 <- function (path) {
  opts <- options(encoding = "native.enc")
  on.exit(options(opts), add = TRUE)
  x <- readLines(path, encoding = "UTF-8", warn = FALSE)
  x
}
```

(The original function also checks that the returned lines are valid
UTF-8.)

Explanation and notes:

- `path` must be a file name, or an un-opened connection, or a
  connection that was opened in the native encoding (i.e. with
  `encoding = "native.enc"`). Otherwise `read_utf8()` might (silently)
  return lines in the wrong encoding.
- If `readLines()` gets a file name, then it opens a connection to it in
  `UTF-8`, so the file is not re-encoded and it also marks the result as
  `UTF-8`.
- For extra safety, you can add a check that `path` is a file name.
- As far as I can tell, there is no R API to query the encoding of a
  connection.

TODO: does this work correctly with non-ASCII file names?

## How to write lines to UTF-8 files?

### With the brio package

Call `brio::write_file()` to write a character vector to a file. Notes:

1.  `brio::write_file()` converts the input character vector to UTF-8.
    It uses the marked encoding for this, so if that is not correct,
    then the conversion won’t be correct, either.
2.  Will handle UTF-8 file names correctly.
3.  `brio::write_file()` cannot write to connections currently, because
    with already opened connections there is no way to tell their
    encoding.

### With base R

Kevin Ushey’s excellent [blog
post](https://kevinushey.github.io/blog/2018/02/21/string-encoding-and-r/)
has an excellent detailed derivation of this base R function:

``` r
write_utf8 <- function(text, path) {
  # step 1: ensure our text is utf8 encoded
  utf8 <- enc2utf8(text)
  upath <- enc2utf8(path)
  
  # step 2: create a connection with 'native' encoding
  # this signals to R that translation before writing
  # to the connection should be skipped
  con <- file(upath, open = "w+", encoding = "native.enc")
  on.exit(close(con), add = TRUE)
  
  # step 3: write to the connection with 'useBytes = TRUE',
  # telling R to skip translation to the native encoding
  writeLines(utf8, con = con, useBytes = TRUE)
}
```

(This is a slightly more robust version than the one in the blog post.)

TODO: does this work correctly with non-ASCII file names?

## How to convert text between encodings?

The `iconv()` base R function can convert between encodings. It ignores
the `Encoding()` markings of the input completely. It uses `""` as the
synonym for the native encoding.

## How to guess the encoding of text?

The `stringi::stri_enc_detect()` function tries to detect the encoding
of a string. This is mostly guessing of course.

## How to check if a string is UTF-8?

Not all bytes and bytes sequences are valid in UTF-8. There are a few
alternatives to check if a string is valid:

- `stringi::stri_enc_isutf8()`

- `utf8::utf8_valid()`

- The following trick using only base R:

  ``` r
  ! is.na(iconv(x, "UTF-8", "UTF-8"))
  ```

  `iconv()` returns `NA` for the elements that are invalid in the input
  encoding.

## How to download web pages in the right encoding?

TODO

## Are RDS files encoding-safe?

No, in general they are not, but the situation is quite good. If you
save an RDS file that contains text in some encoding, it is safe to read
that file on a different computer (or the same computer with different
settings), if at least one these conditions hold:

- all text is either UTF-8 and latin1 encoded, and they are also marked
  as such, or
- both computers (or settings) have the same native encoding,
- the RDS file has version 3, and the loading platform can represent all
  characters in the RDS file. This usually holds if the loading platform
  is UTF-8.

Note that from RDS version 3 the strings in the native encoding are
re-encoded to the current native encoding when the RDS file is loaded.

## How to `parse()` UTF-8 text into UTF-8 code?

On R 3.6 and before we need to create a connection to create code with
UTF-8 strings, from an UTF-8 character vector. It goes like this:

``` r
safe_parse <- function(text) {
  text <- enc2utf8(text)
  Encoding(text) <- "unknown"
  con <- textConnection(text)
  on.exit(close(con), add = TRUE)
  eval(parse(con, encoding = "UTF-8"))
}
```

- By marking the text as `unknown` we make sure that `textConnection()`
  will not convert it.
- The `encoding` argument of `parse()` marks the output as `UTF-8`.
- Note that when `parse()` parses from a connection it will lose the
  source references. You can use the `srcfile` argument of `parse()` to
  add them back.

## How to `deparse()` UTF-8 code into UTF-8 text?

TODO

## How to include UTF-8 characters in a package `DESCRIPTION`

Some fields in `DESCRIPTION` may contain non-ASCII characters. Set the
`Encoding` field to `UTF-8`, to use UTF-8 characters in these:

    Encoding: UTF-8

## How to get a package `DESCRIPTION` in UTF-8?

The desc package always converts `DESCRIPTION` files to `UTF-8`.

## How to use UTF-8 file names on Windows?

You can use functions or a package that already handles UTF-8 file
names, e.g. brio or processx. If you need to open a file from C code,
you need to do the following:

- Convert the file name to UTF-8 just before passing it to C. In generic
  code `enc2utf8()` will do.
- In the C code, on Windows, convert the UTF-8 path to UTF-16 using the
  `MultiByteToWideChar()` Windows API function.
- Use the `_wfopen()` Windows API function to open the file.

Here is an example from the brio package:
<https://github.com/r-lib/brio/blob/2cf72bb77ad55c758b4a140112916ddc23f00b59/src/brio.c#L9>

## How to capture UTF-8 output?

This is only possible on UTF-8 systems, and there is no portable way
that works on Windows or other non-UTF-8 platforms. If the native
encoding is UTF-8, then `sink()` and `capture.output()` will create text
in UTF-8.

Capturing UTF-8 output on Windows (or a Unix system that does not
support a UTF-8 locale) is not currently possible. (Well, on Windows
there is a very dirty trick, but it involves changing an internal R
flag, and running the code in a subprocess.)

### Are testthat snapshot tests encoding-safe?

Somewhat. testthat 3 by default turn off non-ASCII output from cli (and
thus tibble, etc.). So your snapshots that use cli facilities will be in
ASCII, which is safe to use anywhere.

If you non-ASCII output from other sources, then hopefully there is a
switch to turn them off. As far as I can tell, rlang uses cli to produce
non-ASCII output, so it should be fine.

## How to test non-ASCII output?

If you need to test non-ASCII output produced by the cli package, then

- use snapshot tests,
- call `testthat::local_reproducible_output(unicode = TRUE)` at the
  beginning of your test,
- record your snapshots on a UTF-8 platform.

Your tests will not run on non-UTF-8 platforms. It is not currently
possible to record non-ASCII snapshot tests on non-UTF-8 platforms.

## How to avoid non-ASCII characters in the manual?

If you accidentally included some non-ASCII characters in the manual,
e.g. because you included some UTF-8 output from cli or tibble/pillar,
then you can set `options(cli.unicode = FALSE)` at the right place to
avoid them.

The `tools::showNonASCIIfile()` function helps finding non-ASCII
characters in a package.

## How to include non-ASCII characters in PDF vignettes?

TODO

# Encoding related R functions and packages

- `base::charToRaw()`

- `base::Encoding()` and `` base::`Encoding<-` ``

- `base::iconv()`

- `base::iconvlist()`

- `base::nchar()`

- `base::l10n_info()`

- [stringi package](https://cran.rstudio.com/web/packages/stringi/)

- [utf8 package](https://cran.rstudio.com/web/packages/utf8)

- [brio package](https://cran.rstudio.com/web/packages/brio)

- [cli package](https://cran.rstudio.com/web/packages/cli/)

- [Unicode package](https://cran.rstudio.com/web/packages/Unicode)

# Known encoding issues in packages and functions

- `yaml::write_yaml` crashes on Windows on latin1 encoded strings:
  <https://github.com/viking/r-yaml/issues/90>

# Tips for debugging encoding issues

- `charToRaw()` is your best friend.

- Don’t forget, if they print the same, if they are `identical()`, they
  can still be in a different encoding. `charToRaw()` is your best
  friend.

- `testthat::CheckReporter` saves a `testthat-problems.rds` file, if
  there were any test failures. You can get this file from win-builder,
  R-hub, etc. The file is a version 2 RDS file, so no encoding
  conversion will be done by `readRDS()`.

- Don’t trust any function that processes text. Some functions keep the
  encoding of the input, some convert to the native encoding, some
  convert to UTF-8. Some convert to UTF-8, without marking the output as
  UTF-8. Different R versions do different re-encodings.

- Typing in a string on the console is not the same as `parse()`-ing the
  same code from a file. The console is always assumed to provide text
  in the native encoding, package files are typically assumed UTF-8.
  (This is why it is important to use `\uxxxx` escape sequences for
  UTF-8 text.)

# Text transformers

- `print()`

- `format()`

- `normalizePath()`

- `basename()`

- `encodeString()`

# Changes in R 4.1.0

TODO

# Changes in R 4.2.0

TODO

# Further Reading

Good general introductions to text encodings:

- [The Absolute Minimum Every Software Developer Absolutely, Positively
  Must Know About Unicode and Character Sets (No
  Excuses!)](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/)

- [What Every Programmer Absolutely, Positively Needs To Know About
  Encodings And Character Sets To Work With
  Text](https://kunststube.net/encoding/)

- [UTF-8 Everywhere](http://utf8everywhere.org)

R documentation:

- [Encodings for
  CHARSXPs](https://cran.r-project.org/doc/manuals/r-devel/R-ints.html#Encodings-for-CHARSXPs)
  section in [R
  internals](https://cran.r-project.org/doc/manuals/r-devel/R-ints.html)

- [Writing portable packages / Encoding
  issues](https://cran.r-project.org/doc/manuals/R-exts.html#Encoding-issues)
  in [Writing R
  Extensions](https://cran.r-project.org/doc/manuals/R-exts.html)

- [Encodings and
  R](https://developer.r-project.org/Encodings_and_R.html) – *Old and
  slightly outdated, it gives a good summary of what it took to change
  R’s internal encoding.*

Intros to various encodings:

- [UTF-8 on Wikipedia](https://en.wikipedia.org/wiki/UTF-8)

- [UTF-16 on Wikipedia](https://en.wikipedia.org/wiki/UTF-16)

- [latin-1 (ISO/IEC 8859-1) on
  Wikipedia](https://en.wikipedia.org/wiki/ISO/IEC_8859-1)

About Unicode:

- [The main Unicode homepage](https://home.unicode.org/)

- [Unicode FAQs](https://unicode.org/faq/)
