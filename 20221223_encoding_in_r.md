2022-12-23 Character encoding in R (windows)
================
Pep Porrà
2022-12-23

``` r
library(Unicode)
```

    ## Warning: package 'Unicode' was built under R version 4.1.3

## Goal

Show examples of how character encoding works in R

## Reference

This analysis is based on:

-   [Encoding character strings in
    R](https://rstudio-pubs-static.s3.amazonaws.com/279354_f552c4c41852439f910ad620763960b6.html)
-   [R4DS (2ed), WIP](https://r4ds.hadley.nz/strings.html)
-   [Encoding
    introduction](https://www.w3.org/International/questions/qa-what-is-encoding)

## Steps

### Platform

First, I show which platform (operating system and computer) I’m working
on

``` r
Sys.info()[1:5]
```

    ##           sysname           release           version          nodename 
    ##         "Windows"          "10 x64"     "build 22000" "DESKTOP-392MM2S" 
    ##           machine 
    ##          "x86-64"

### System info

More extended info about my system, including the locale setup

``` r
sessionInfo()
```

    ## R version 4.1.1 (2021-08-10)
    ## Platform: x86_64-w64-mingw32/x64 (64-bit)
    ## Running under: Windows 10 x64 (build 22000)
    ## 
    ## Matrix products: default
    ## 
    ## locale:
    ## [1] LC_COLLATE=English_United States.1252 
    ## [2] LC_CTYPE=English_United States.1252   
    ## [3] LC_MONETARY=English_United States.1252
    ## [4] LC_NUMERIC=C                          
    ## [5] LC_TIME=English_United States.1252    
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] Unicode_15.0.0-1
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] compiler_4.1.1  magrittr_2.0.1  fastmap_1.1.0   cli_3.0.1      
    ##  [5] tools_4.1.1     htmltools_0.5.2 rstudioapi_0.13 yaml_2.2.1     
    ##  [9] stringi_1.7.5   rmarkdown_2.11  knitr_1.36      stringr_1.4.0  
    ## [13] xfun_0.31       digest_0.6.28   rlang_1.0.3     evaluate_0.14

Language is set to English in my .Rprofile file with the line
`invisible(Sys.setlocale(locale = "English"))`. I revert it to my
windows OS language with `Sys.setlocale()`. I’ve created function
`locale_details()` to print locale info differently.

``` r
locales_detail <- function(){
  info <- Sys.getlocale() |> 
    strsplit(";") |> 
    unlist() |> 
    strsplit("=") 
  v <- sapply(info, \(x) x[2])
  names(v) <- sapply(info, \(x) x[1])
  return(v)
}
```

``` r
locales_detail()
```

    ##                   LC_COLLATE                     LC_CTYPE 
    ## "English_United States.1252" "English_United States.1252" 
    ##                  LC_MONETARY                   LC_NUMERIC 
    ## "English_United States.1252"                          "C" 
    ##                      LC_TIME 
    ## "English_United States.1252"

``` r
invisible(Sys.setlocale())
locales_detail()
```

    ##           LC_COLLATE             LC_CTYPE          LC_MONETARY 
    ## "Spanish_Spain.1252" "Spanish_Spain.1252" "Spanish_Spain.1252" 
    ##           LC_NUMERIC              LC_TIME 
    ##                  "C" "Spanish_Spain.1252"

By default, the global option “encoding” is set to “native.enc”

``` r
getOption("encoding")
```

    ## [1] "native.enc"

### Localization

Function `l10n_info()` gives additional localization information. Last
two (codepage and system.codepage) only apply to Windows systems

``` r
l10n_info()
```

    ## $MBCS
    ## [1] FALSE
    ## 
    ## $`UTF-8`
    ## [1] FALSE
    ## 
    ## $`Latin-1`
    ## [1] TRUE
    ## 
    ## $codepage
    ## [1] 1252
    ## 
    ## $system.codepage
    ## [1] 1252

With function `Sys.localeconv()` get details of the numerical and
monetary represetantions in the current locale

``` r
Sys.localeconv()
```

    ##     decimal_point     thousands_sep          grouping   int_curr_symbol 
    ##               "."                ""                ""             "EUR" 
    ##   currency_symbol mon_decimal_point mon_thousands_sep      mon_grouping 
    ##               "\200"               ","               "."            "\003" 
    ##     positive_sign     negative_sign   int_frac_digits       frac_digits 
    ##                ""               "-"               "2"               "2" 
    ##     p_cs_precedes    p_sep_by_space     n_cs_precedes    n_sep_by_space 
    ##               "0"               "1"               "0"               "1" 
    ##       p_sign_posn       n_sign_posn 
    ##               "1"               "1"

As I’m using Spanish Language, currency symbol is euro €. If locale is
set to “English” then locale currency is USD.

``` r
old <- Sys.setlocale(locale = "English")
Sys.localeconv()
```

    ##     decimal_point     thousands_sep          grouping   int_curr_symbol 
    ##               "."                ""                ""             "USD" 
    ##   currency_symbol mon_decimal_point mon_thousands_sep      mon_grouping 
    ##               "$"               "."               ","            "\003" 
    ##     positive_sign     negative_sign   int_frac_digits       frac_digits 
    ##                ""               "-"               "2"               "2" 
    ##     p_cs_precedes    p_sep_by_space     n_cs_precedes    n_sep_by_space 
    ##               "1"               "0"               "1"               "0" 
    ##       p_sign_posn       n_sign_posn 
    ##               "3"               "0"

``` r
invisible(Sys.setlocale())
```

### Encoding functions

Three functions related to encoding in R base: `Encoding()`,
`enc2native()` and `enc2utf8()`

I’m showing how a short sentence in encode both in ‘latin-1’ and ‘UTF-8’

``` r
x_latin1 <- "Calçots més\n'El Niño'\n"
```

``` r
Encoding(x_latin1)
```

    ## [1] "latin1"

``` r
print(x_latin1)
```

    ## [1] "Calçots més\n'El Niño'\n"

``` r
cat(x_latin1)
```

    ## Calçots més
    ## 'El Niño'

``` r
enc2native(x_latin1) |> 
  Encoding()
```

    ## [1] "latin1"

WE can encode the sentence in ’UTF-8) with function `enc2utf8()`

``` r
x_utf8 <- enc2utf8(x_latin1)
Encoding(x_utf8)
```

    ## [1] "UTF-8"

``` r
print(x_utf8)
```

    ## [1] "Calçots més\n'El Niño'\n"

``` r
cat(x_utf8)
```

    ## Calçots més
    ## 'El Niño'

So, enoding is completely trasparent to the user is you read a string
encoded in a given system using the same system. For strings with a
defined encoding, `print()` function uses the right encoding.

### Convert to raw

Encoding system determines the specific bytes values that represent that
string. Representation in bytes of a string is turn character into raw
class. For instance, Bytes representing the sentence used before encoded
in latin-1 reads

``` r
x_bytes <- charToRaw(x_latin1)
x_bytes
```

    ##  [1] 43 61 6c e7 6f 74 73 20 6d e9 73 0a 27 45 6c 20 4e 69 f1 6f 27 0a

``` r
class(x_bytes[1])
```

    ## [1] "raw"

Bytes (in hexadecimal notation) can be turn into integers

``` r
as.integer(x_bytes)
```

    ##  [1]  67  97 108 231 111 116 115  32 109 233 115  10  39  69 108  32  78 105 241
    ## [20] 111  39  10

``` r
as.hexmode(as.character(x_bytes[1]))
```

    ## [1] "43"

Function `as.hexmode()` turns strings into hexmode class objects

``` r
as.hexmode("0xe9")
```

    ## [1] "e9"

``` r
as.hexmode("e9")
```

    ## [1] "e9"

``` r
print("\xe9")
```

    ## [1] "é"

``` r
print(rawToChar(as.raw("0xe9")))
```

    ## [1] "é"

We create a function that shows the bytes representation of each
character in a given string (that have a defined encoding)

``` r
# split a string into characters
chars_in_str <- function(x){
  strsplit(x, "") |> 
    unlist()
}

# return the byte/s for each character assuming the string has the correct encoding
string_to_code <- function(y){
  sapply(chars_in_str(y), 
    function(x) paste(charToRaw(x), collapse = "")
  )
}
```

``` r
string_to_code(x_latin1)
```

    ##    C    a    l    ç    o    t    s         m    é    s   \n    '    E    l      
    ## "43" "61" "6c" "e7" "6f" "74" "73" "20" "6d" "e9" "73" "0a" "27" "45" "6c" "20" 
    ##    N    i    ñ    o    '   \n 
    ## "4e" "69" "f1" "6f" "27" "0a"

when encoding is “latin-1”, note that each character corresponds to one
byte. Therefore, the length of the vector of characters returned by
function `string_to_code()` will be the same as the length of the
string:

``` r
nchar(x_latin1)
```

    ## [1] 22

``` r
length(string_to_code(x_latin1))
```

    ## [1] 22

``` r
length(charToRaw(x_latin1))
```

    ## [1] 22

when using ‘UTF-8’ encoding, some characters are represented using two
bytes

``` r
string_to_code(x_utf8)
```

    ##      C      a      l      ç      o      t      s             m      é      s 
    ##   "43"   "61"   "6c" "c3a7"   "6f"   "74"   "73"   "20"   "6d" "c3a9"   "73" 
    ##     \n      '      E      l             N      i      ñ      o      '     \n 
    ##   "0a"   "27"   "45"   "6c"   "20"   "4e"   "69" "c3b1"   "6f"   "27"   "0a"

Therefore, character “é” is represented differently depending on the
encoding used.

In “latin-1”, “é” is “” while in “UTF-8” reads “”

``` r
nchar(x_utf8)
```

    ## [1] 22

``` r
length(string_to_code(x_utf8))
```

    ## [1] 22

``` r
length(charToRaw(x_utf8))
```

    ## [1] 25

``` r
print("latin-1 Encoding")
```

    ## [1] "latin-1 Encoding"

``` r
c_latin1 <- "é"
cat("Character:", c_latin1, "\n")
```

    ## Character: é

``` r
cat("Encoding:", Encoding(c_latin1), "\n")
```

    ## Encoding: latin1

``` r
cat("Bytes representing the character:", charToRaw(c_latin1), "\n")
```

    ## Bytes representing the character: e9

``` r
cat("Class of the Raw bytes:", class(charToRaw(c_latin1)), "\n") 
```

    ## Class of the Raw bytes: raw

``` r
cat("Bytes as hexadecimals:")
```

    ## Bytes as hexadecimals:

``` r
print(as.hexmode(as.character(charToRaw(c_latin1))))
```

    ## [1] "e9"

``` r
cat("Class of hexadecimals:", class(as.hexmode(as.character(charToRaw(c_latin1)))), "\n")
```

    ## Class of hexadecimals: hexmode

``` r
int_eq <- as.integer(charToRaw(c_latin1))
cat("Integer value:", int_eq, "\n")
```

    ## Integer value: 233

``` r
cat("Unicode code for the character: ")
```

    ## Unicode code for the character:

``` r
print(Unicode::as.u_char(int_eq))
```

    ## [1] U+00E9

``` r
cat("Unicode label for the character:", Unicode::u_char_label(int_eq), "\n")
```

    ## Unicode label for the character: LATIN SMALL LETTER E WITH ACUTE

``` r
print("UTF-8 Encoding")
```

    ## [1] "UTF-8 Encoding"

``` r
c_latin1 <- "é"
c_utf8 <- enc2utf8(c_latin1)
cat("Character:", c_utf8, "\n")
```

    ## Character: é

``` r
cat("Encoding:", Encoding(c_utf8), "\n")
```

    ## Encoding: UTF-8

``` r
cat("Bytes representing the character:", charToRaw(c_utf8), "\n")
```

    ## Bytes representing the character: c3 a9

``` r
cat("Class of the Raw bytes:", class(charToRaw(c_utf8)), "\n") 
```

    ## Class of the Raw bytes: raw

``` r
cat("Bytes as hexadecimals:")
```

    ## Bytes as hexadecimals:

``` r
print(as.hexmode(as.character(charToRaw(c_utf8))))
```

    ## [1] "c3" "a9"

``` r
cat("Class of hexadecimals:", class(as.hexmode(as.character(charToRaw(c_utf8)))), "\n")
```

    ## Class of hexadecimals: hexmode

``` r
int_eq <- utf8ToInt(c_utf8)
cat("Integer value:", int_eq, "\n")
```

    ## Integer value: 233

``` r
cat("Unicode code for the character:")
```

    ## Unicode code for the character:

``` r
print(Unicode::as.u_char(int_eq))
```

    ## [1] U+00E9

``` r
cat("Unicode label for the character:", Unicode::u_char_label(int_eq), "\n")
```

    ## Unicode label for the character: LATIN SMALL LETTER E WITH ACUTE

Unicode code can be indicated with “\u”

``` r
readLines(textConnection("\u00e9"))
```

    ## [1] "é"

``` r
readLines(textConnection(rawToChar(as.raw("0x00E9"))))
```

    ## [1] "é"

``` r
readLines(textConnection("\u00e9", encoding = "UTF-8"), encoding = "UTF-8")
```

    ## [1] "é"

``` r
readLines(textConnection(x_utf8, encoding = "UTF-8"), encoding = "UTF-8") |> 
  {`[[`}(1) |> 
  string_to_code()
```

    ##      C      a      l      ç      o      t      s             m      é      s 
    ##   "43"   "61"   "6c" "c3a7"   "6f"   "74"   "73"   "20"   "6d" "c3a9"   "73"

Reading a string encoded in UTF-8 in latin-1 generates issues with some
characters

``` r
readLines(textConnection(x_latin1, encoding = "UTF-8")) |> 
  {`[[`}(1) |> 
  try(string_to_code())
```

    ## [1] "CalÃ§ots mÃ©s"

And the opposite is also true, reading in UTF-8 a text encoded in
latin-1, generates errors

``` r
readLines(textConnection(x_latin1), encoding = "UTF-8") |> 
  {`[[`}(1) |> 
  try(string_to_code())
```

    ## [1] "Cal<e7>ots m<e9>s"

``` r
readLines("file_latin1.txt", encoding = "latin-1")
```

    ## [1] "Calçots més" "'El Niño'"

``` r
readLines("file_utf8.txt", encoding = "UTF-8")
```

    ## [1] "Calçots més" "'El Niño'"

If the encoding is not the right one, some characters are not read
correctly

``` r
readLines("file_latin1.txt", encoding = "UTF-8")
```

    ## [1] "Cal<e7>ots m<e9>s" "'El Ni<f1>o'"

``` r
readLines("file_utf8.txt")
```

    ## [1] "CalÃ§ots mÃ©s" "'El NiÃ±o'"

### Print a sequence of Unicode characters

``` r
c_ex <- c("\u00B5", "\U00BB")
```

``` r
print(c_ex)
```

    ## [1] "µ" "»"

``` r
print(c_ex)
```

    ## [1] "µ" "»"

``` r
code_c <- sapply(c_ex, utf8ToInt)
print("unicode name")
```

    ## [1] "unicode name"

``` r
Unicode::u_char_name(code_c)
```

    ## [1] "MICRO SIGN"                                
    ## [2] "RIGHT-POINTING DOUBLE ANGLE QUOTATION MARK"

``` r
print("unicode label")
```

    ## [1] "unicode label"

``` r
Unicode::u_char_label(code_c)
```

    ## [1] "MICRO SIGN"                                
    ## [2] "RIGHT-POINTING DOUBLE ANGLE QUOTATION MARK"

``` r
class(c_ex)
```

    ## [1] "character"

``` r
sapply(c_ex, charToRaw)
```

    ##       µ  »
    ## [1,] c2 c2
    ## [2,] b5 bb

``` r
Encoding(c_ex)
```

    ## [1] "UTF-8" "UTF-8"

``` r
sapply(c_ex, utf8ToInt)
```

    ##   µ   » 
    ## 181 187

Turn a UTF-8 sequence into characters

``` r
int_series <- 1024+0:9
cat(int_series, "\n")
```

    ## 1024 1025 1026 1027 1028 1029 1030 1031 1032 1033

``` r
char_series <- intToUtf8(int_series)
cat("Bytes as hexadecimals:", "\n" )
```

    ## Bytes as hexadecimals:

``` r
string_to_code(char_series)
```

    ## <U+0400> <U+0401> <U+0402> <U+0403> <U+0404> <U+0405> <U+0406> <U+0407> <U+0408> <U+0409> 
    ## "d080" "d081" "d082" "d083" "d084" "d085" "d086" "d087" "d088" "d089"

``` r
cat("word: ", char_series, "\n")
```

    ## word:  <U+0400><U+0401><U+0402><U+0403><U+0404><U+0405><U+0406><U+0407><U+0408><U+0409>

``` r
cat("# of characters: ", nchar(char_series), "\n")
```

    ## # of characters:  10

``` r
cat("# of bytes: ", nchar(char_series, type = "bytes"), "\n")
```

    ## # of bytes:  20

``` r
cat("width: ", nchar(char_series, type = "width"), "\n")
```

    ## width:  10

### Alternative minus sign

``` r
z <- c("+","-","–", "—", "\u2212", "\ufe63", "\uff0d")
print(z)
```

    ## [1] "+"  "-"  "–"  "—"  "-"  "<U+FE63>" "-"

``` r
Encoding(z)
```

    ## [1] "unknown" "unknown" "latin1"  "latin1"  "UTF-8"   "UTF-8"   "UTF-8"

``` r
z_utf8 <- sapply(z, function(s) {
  if(Encoding(s) == "UTF-8") {
    return(s)
  } else {
    
    return(enc2utf8(s))
  }
})
```

``` r
print(z_utf8)
```

    ##    +    -    –    —    - <U+FE63>    - 
    ##  "+"  "-"  "–"  "—"  "-" "<U+FE63>"  "-"

``` r
Encoding(z_utf8)
```

    ## [1] "unknown" "unknown" "UTF-8"   "UTF-8"   "UTF-8"   "UTF-8"   "UTF-8"

``` r
cat("UTF-8 Encoding\n---------\n")
```

    ## UTF-8 Encoding
    ## ---------

``` r
sapply(z_utf8, string_to_code)
```

    ##      +.+      -.-      –.–      —.—      -.- <U+FE63>.<U+FE63>      -.- 
    ##     "2b"     "2d" "e28093" "e28094" "e28892" "efb9a3" "efbc8d"

``` r
cat("UTF-8 into integers\n---------\n")
```

    ## UTF-8 into integers
    ## ---------

``` r
sapply(z_utf8, utf8ToInt)
```

    ##     +     -     –     —     - <U+FE63>     - 
    ##    43    45  8211  8212  8722 65123 65293

``` r
cat("---------\nChar to raw\n")
```

    ## ---------
    ## Char to raw

``` r
sapply(z_utf8, charToRaw)
```

    ## $`+`
    ## [1] 2b
    ## 
    ## $`-`
    ## [1] 2d
    ## 
    ## $`–`
    ## [1] e2 80 93
    ## 
    ## $`—`
    ## [1] e2 80 94
    ## 
    ## $`-`
    ## [1] e2 88 92
    ## 
    ## $`<U+FE63>`
    ## [1] ef b9 a3
    ## 
    ## $`-`
    ## [1] ef bc 8d

``` r
cat("---------\nUnicode labels\n")
```

    ## ---------
    ## Unicode labels

``` r
sapply(z_utf8, function(x) Unicode::u_char_label(utf8ToInt(x)))
```

    ##                        +                        -                        – 
    ##              "PLUS SIGN"           "HYPHEN-MINUS"                "EN DASH" 
    ##                        —                        -                 <U+FE63> 
    ##                "EM DASH"             "MINUS SIGN"     "SMALL HYPHEN-MINUS" 
    ##                        - 
    ## "FULLWIDTH HYPHEN-MINUS"

``` r
cat("Latin-1 Encoding\n---------\n")
```

    ## Latin-1 Encoding
    ## ---------

``` r
Encoding(z)
```

    ## [1] "unknown" "unknown" "latin1"  "latin1"  "UTF-8"   "UTF-8"   "UTF-8"

``` r
sapply(z, string_to_code)
```

    ##      +.+      -.-      –.–      —.—      -.- <U+FE63>.<U+FE63>      -.- 
    ##     "2b"     "2d"     "96"     "97" "e28892" "efb9a3" "efbc8d"

``` r
cat("---------\nChar to raw\n")
```

    ## ---------
    ## Char to raw

``` r
sapply(z, charToRaw)
```

    ## $`+`
    ## [1] 2b
    ## 
    ## $`-`
    ## [1] 2d
    ## 
    ## $`–`
    ## [1] 96
    ## 
    ## $`—`
    ## [1] 97
    ## 
    ## $`-`
    ## [1] e2 88 92
    ## 
    ## $`<U+FE63>`
    ## [1] ef b9 a3
    ## 
    ## $`-`
    ## [1] ef bc 8d
