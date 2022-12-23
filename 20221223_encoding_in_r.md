2022-12-23 Character encoding in R (windows)
================
Pep Porrà
2022-12-23

## Goal

Show examples of how character encoding works in R

## Reference

This analysis is based on the document \[Encoding character strings in
R\]
(<https://rstudio-pubs-static.s3.amazonaws.com/279354_f552c4c41852439f910ad620763960b6.html>).
I also used additional information from [R4DS 2ed,
WIP](https://r4ds.hadley.nz/strings.html)

## Steps

### Platform

``` r
Sys.info()[1:5]
```

    ##           sysname           release           version          nodename 
    ##         "Windows"          "10 x64"     "build 22000" "DESKTOP-392MM2S" 
    ##           machine 
    ##          "x86-64"

### System info

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
    ## loaded via a namespace (and not attached):
    ##  [1] compiler_4.1.1  magrittr_2.0.1  fastmap_1.1.0   cli_3.0.1      
    ##  [5] tools_4.1.1     htmltools_0.5.2 rstudioapi_0.13 yaml_2.2.1     
    ##  [9] stringi_1.7.5   rmarkdown_2.11  knitr_1.36      stringr_1.4.0  
    ## [13] xfun_0.31       digest_0.6.28   rlang_1.0.3     evaluate_0.14

Language is set to English in the .Rprofile file with the line
`invisible(Sys.setlocale(locale = "English"))`. I revert it to my
windows OS language with `Sys.setlocale()`

``` r
getOption("encoding")
```

    ## [1] "native.enc"

### Localization

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

``` r
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

Therefore, by default I’m using “Latin-1” encoding.

### Encoding functions

Three functions related to encoding in R base: `Encoding()`,
`enc2native()` and `enc2utf8()`

``` r
x_latin1 <- "Els calçots creixen millor quan 'El Niño' és molt fort"
```

``` r
Encoding(x_latin1)
```

    ## [1] "latin1"

``` r
x_utf8 <- enc2utf8(x_latin1)
Encoding(x_utf8)
```

    ## [1] "UTF-8"

### Convert to raw

``` r
charToRaw(x_latin1)
```

    ##  [1] 45 6c 73 20 63 61 6c e7 6f 74 73 20 63 72 65 69 78 65 6e 20 6d 69 6c 6c 6f
    ## [26] 72 20 71 75 61 6e 20 27 45 6c 20 4e 69 f1 6f 27 20 e9 73 20 6d 6f 6c 74 20
    ## [51] 66 6f 72 74

``` r
# split a string into characters
chars_in_str <- function(x){
  strsplit(x, "") |> 
    unlist()
}

# return the byte/s for each character assuming the string has the correct encoding
string_to_code <- function(y){
  sapply(chars_in_str(y), 
    function(x) paste(charToRaw(x), collapse = ":")
  )
}
```

``` r
string_to_code(x_latin1)
```

    ##    E    l    s         c    a    l    ç    o    t    s         c    r    e    i 
    ## "45" "6c" "73" "20" "63" "61" "6c" "e7" "6f" "74" "73" "20" "63" "72" "65" "69" 
    ##    x    e    n         m    i    l    l    o    r         q    u    a    n      
    ## "78" "65" "6e" "20" "6d" "69" "6c" "6c" "6f" "72" "20" "71" "75" "61" "6e" "20" 
    ##    '    E    l         N    i    ñ    o    '         é    s         m    o    l 
    ## "27" "45" "6c" "20" "4e" "69" "f1" "6f" "27" "20" "e9" "73" "20" "6d" "6f" "6c" 
    ##    t         f    o    r    t 
    ## "74" "20" "66" "6f" "72" "74"

``` r
string_to_code(x_utf8)
```

    ##       E       l       s               c       a       l       ç       o       t 
    ##    "45"    "6c"    "73"    "20"    "63"    "61"    "6c" "c3:a7"    "6f"    "74" 
    ##       s               c       r       e       i       x       e       n         
    ##    "73"    "20"    "63"    "72"    "65"    "69"    "78"    "65"    "6e"    "20" 
    ##       m       i       l       l       o       r               q       u       a 
    ##    "6d"    "69"    "6c"    "6c"    "6f"    "72"    "20"    "71"    "75"    "61" 
    ##       n               '       E       l               N       i       ñ       o 
    ##    "6e"    "20"    "27"    "45"    "6c"    "20"    "4e"    "69" "c3:b1"    "6f" 
    ##       '               é       s               m       o       l       t         
    ##    "27"    "20" "c3:a9"    "73"    "20"    "6d"    "6f"    "6c"    "74"    "20" 
    ##       f       o       r       t 
    ##    "66"    "6f"    "72"    "74"

``` r
z <- c("–", "—")
```

``` r
Encoding(z)
```

    ## [1] "latin1" "latin1"

``` r
string_to_code("—")
```

    ##    — 
    ## "97"
