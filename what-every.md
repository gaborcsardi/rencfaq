What every R developer must know about encodings
================

-   [Why are encodings so hard (in R)?](#why-are-encodings-so-hard-in-r)
-   [Important encodings (for the R
    developer)](#important-encodings-for-the-r-developer)
    -   [UTF-8](#utf-8)
    -   [ASCII](#ascii)
    -   [latin1 (and CP1252)](#latin1-and-cp1252)
    -   [UTF-16](#utf-16)
-   [How R stores text data](#how-r-stores-text-data)
-   [What encoding should I use?](#what-encoding-should-i-use)
-   [How-to?](#how-to)
-   [Debugging encoding issues](#debugging-encoding-issues)
    -   [Common issues](#common-issues)
    -   [Tips](#tips)

## Why are encodings so hard (in R)?

-   Impossible to interpret a piece of text without external
    information.
-   R’s internal encoding is different on different platforms, which
    make is hard to write (and test!) portable code, and to transfer
    data.
-   R’s functions do (seemingly) random encoding conversions.

## Important encodings (for the R developer)

### UTF-8

#### What is Unicode?

Unicode is an information technology standard for the consistent
encoding, representation, and handling of text expressed in most of the
world’s writing systems.

#### Numbered characters

-   First 128 is the same as ASCII. Then the rest of the 143,859
    characters (Unicode 13.0)

-   1,112,064 possible characters. (Original plan was much-much more.)

-   You can create text that encodes Unicode characters with `\u` and
    `\U` escapes:

    ``` r
    "Just a normal string with \u00fc is \u2713 \U{1F60A}"
    ```

        ## [1] "Just a normal string with ü is ✓ 😊"

-   R always encodes `\u` and `\U` in UTF-8.

-   Some are non-printing: zero width non-joiner, right-to-left mark,
    left-to-right mark, etc.

-   Some are combining:

``` r
"person + dark skin tone: \U1F9D1 \U1F3FF"
```

    ## [1] "person + dark skin tone: 🧑 🏿"

``` r
"person + dark skin tone: \U1F9D1\U1F3FF"
```

    ## [1] "person + dark skin tone: 🧑🏿"

#### UTF-8 encoding

-   Encodes all Unicode characters.

-   Variable length encoding: between 1 and 4 bytes. Think about the
    implications of this!

-   Includes ASCII, in the same exact encoding.

-   Again, R always encodes `\u` and `\U` in UTF-8, use it for UTF-8
    string literals.

-   <https://en.wikipedia.org/wiki/UTF-8#Encoding>

### ASCII

-   Ancient. 128 characters, stored on one byte, the highest bit is
    zero.

### latin1 (and CP1252)

-   Extension of ASCII, by defining the characters when the high bit is
    one.

-   CP1252 is an a modified latin1, it substitutes some control
    characters with printable ones.

-   Usually this is the default native encoding of R in the US and
    Western Europe.

-   Covers most Western European special characters:
    <https://en.wikipedia.org/wiki/ISO/IEC_8859-1#Modern_languages_with_complete_coverage>

### UTF-16

-   Encodes all of Unicode.

-   Each Unicode character is two or four bytes.

-   Internal encoding of Windows, Javascript, (older) Python.

-   We only need to deal with it in R when communicating with the
    Windows API. Most commonly this means passing file paths to Windows.

-   R cannot represent this in a string, because it has embedded zeros.

-   Best is to convert from/to UTF-8 immediately before passing to or
    getting it from Windows. (This usually happens in C code.)

## How R stores text data

-   Zero-terminated bytes. (Hence, no UTF-16 possible.)

    ``` r
    charToRaw("hello world!")
    ```

        ##  [1] 68 65 6c 6c 6f 20 77 6f 72 6c 64 21

-   R often assumes that strings are in the *native* encoding. E.g.
    symbols have to be in the native encoding, connections convert to
    the native encoding, so does printing, etc.

-   The native encoding can be different on different platforms. *Yes,
    this is the biggest challenge when writing portable R code that
    deals with strings.*

-   Use `l10n_info()` to query the native encoding. (But see the FAQ for
    the real answer.)

    ``` r
    l10n_info()
    ```

        ## $MBCS
        ## [1] TRUE
        ## 
        ## $`UTF-8`
        ## [1] TRUE
        ## 
        ## $`Latin-1`
        ## [1] FALSE

-   It is possible to *declare* the encoding of a string (not character
    vector). (!)

-   But alas… fun with `Encoding()` :

    ``` r
    x <- "\xfb"
    x
    ```

        ## [1] "\xfb"

    ``` r
    charToRaw(x)
    ```

        ## [1] fb

    ``` r
    Encoding(x) <- "latin1"
    x
    ```

        ## [1] "û"

    ``` r
    y <- "\xfb"
    Encoding(y) <- "latin2"
    y
    ```

        ## [1] "\xfb"

    ``` r
    Encoding(y)
    ```

        ## [1] "unknown"

-   The encoding information is only stored on two bits. So four values
    are possible:

    -   `UTF-8`

    -   `latin1`

    -   `bytes`

    -   `unknown`

-   If the native encoding is not UTF-8 or latin1, native strings are
    marked as `unknown`. But `unkown` means different things on
    different platforms.

## What encoding should I use?

UTF-8. Only use something else if you really have to. Convert to UTF-8
as soon as possible. Convert to something else as late as possible.

## How-to?

-   How to convert to UTF-8?

-   How to check if a string is UTF-8?

-   How to read a file in UTF-8?

-   How to write a file in UTF-8?

## Debugging encoding issues

### Common issues

#### Don’t let the printing fool you.

R converts strings to the native encoding when printing them to the
screen. This is important to know when debugging encoding problems: two
strings that print the same way may have a different internal
representation:

``` r
s1 <- "\xfc"
Encoding(s1) <- "latin1"
s2 <- iconv(s1, "latin1", "UTF-8")
s1
```

    ## [1] "ü"

``` r
s2
```

    ## [1] "ü"

``` r
s1 == s2
```

    ## [1] TRUE

``` r
identical(s1, s2)
```

    ## [1] TRUE

``` r
testthat::expect_equal(s1, s2)
testthat::expect_identical(s1, s2)
```

``` r
Encoding(s1)
```

    ## [1] "latin1"

``` r
Encoding(s2)
```

    ## [1] "UTF-8"

``` r
charToRaw(s1)
```

    ## [1] fc

``` r
charToRaw(s2)
```

    ## [1] c3 bc

#### Beware the silent conversions

R functions changing the encoding silently. All functions that transform
text are suspicious. Older R versions are usually worse:

``` r
> x <- "ü"
> Encoding(x)
[1] "latin1"
> charToRaw(x)
[1] fc
```

``` r
> ux <- enc2utf8(x)
> Encoding(ux)
[1] "UTF-8"
> charToRaw(ux)
[1] c3 bc
```

``` r
> nux <- normalizePath(ux, mustWork = FALSE)
> nux
[1] "C:\\Users\\Gabor\\works\\processx\\ü"
> Encoding(nux)
[1] "unknown"
> charToRaw(nux)
 [1] 43 3a 5c 55 73 65 72 73 5c 47
[11] 61 62 6f 72 5c 77 6f 72 6b 73 
[21] 5c 70 72 6f 63 65 73 73 78 5c 
[31] fc
```

``` r
> basename(nux)
[1] "ü"
> Encoding(basename(nux))
[1] "UTF-8"
> charToRaw(basename(nux))
[1] c3 bc
```

#### Display width is off everywhere

Aligning text with *wide* Unicode characters is hard.

### Tips

-   `charToRaw()` is your best friend.

-   Don’t forget, if they print the same, if they are `identical()` ,
    they can still be in a different encoding. `charToRaw()` is your
    best friend.

-   `testthat::CheckReporter` saves a `testthat-problems.rds` file, if
    there were any test failures. You can get this file from
    win-builder, R-hub, etc. The file is a version 2 RDS file, so no
    encoding conversion will be done by `readRDS()`.

-   Don’t trust any function that processes text. Some functions keep
    the encoding of the input, some convert to the native encoding, some
    convert to UTF-8. Some convert to UTF-8, without marking the output
    as UTF-8. Different R versions do different re-encodings.

-   Typing in a string on the console is not the same as `parse()`-ing
    the same code from a file. The console is always assumed to provide
    text in the native encoding, package files are typically assumed
    UTF-8. (This is why it is important to use `\uxxxx` escape sequences
    for UTF-8 text.)
