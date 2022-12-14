
<!-- README.md is generated from README.Rmd. Please edit that file -->

# mfa-dict

This repository hosts the words we have had to add to the English
pronunciation dictionary for the Montreal Forced Aligner
([Docs](https://montreal-forced-aligner.readthedocs.io/en/latest/index.html "Read The Docs page for the MFA"),
[GitHub](https://github.com/MontrealCorpusTools/Montreal-Forced-Aligner "MFA GitHub page").

## contents

Here are the contents of the custom dictionary, as an R data-frame,
where I have named unimportant columns `"ignoreX"`.

``` r
"english_us_arpa_extras.dict" |> 
  readr::read_tsv(
    col_names = c("word", "prob", "ignore0", "ignore1", "ignore2", "phones"),
    col_types = "cddddc"
  ) |> 
  print(n = Inf)
#> # A tibble: 6 × 6
#>   word     prob ignore0 ignore1 ignore2 phones            
#>   <chr>   <dbl>   <dbl>   <dbl>   <dbl> <chr>             
#> 1 hotdog   0.99    0.17       1       1 HH AA1 T D AO2 G  
#> 2 hotdogs  0.99    0.17       1       1 HH AA1 T D AO2 G Z
#> 3 standed  0.99    0.17       1       1 S T AE1 N D IH0 D 
#> 4 maked    0.99    0.17       1       1 M EY1 K T         
#> 5 dranked  0.99    0.17       1       1 D R AE1 NG K T    
#> 6 baggie   0.99    0.17       1       1 B AE1 G IY0
```

To use the dictionary, copy and paste the contents of file
`english_us_arpa_extras.dict` onto the end of the dictionary file used
by the MFA.

## updating/modifying the dictionary

I used the following to force an update to my English models.

    mfa model download dictionary english_us_arpa --ignore_cache
    mfa model download acoustic english_us_arpa --ignore_cache

So, for my Windows computer, the dictionary was saved to:

`C:/Users/Tristan/Documents/MFA/pretrained_models/dictionary/english_us_arpa.dict`

Then in R I can read in the MFA dictionary, our custom one, and combine
them and overwrite the MFA one.

``` r
# Using the convention that ~ points to a user's `Documents` folder on Windows
path_mfa_dict <- "~/MFA/pretrained_models/dictionary/english_us_arpa.dict"

current <- readr::read_tsv(
  path_mfa_dict,
  col_names = FALSE,
  col_types = "cddddc"
)

new <- readr::read_tsv(
  "english_us_arpa_extras.dict",
  col_names = FALSE,
  col_types = "cddddc"
)

# Add or overwrite rows in `current` with rows from `new`
dplyr::rows_upsert(current, new, by = c("X1", "X6")) |> 
  readr::write_tsv(path_mfa_dict, col_names = FALSE)
```

### the dictionary format

The format for the probabilistic pronunciation dictionaries is
\[described here\]\[mfa-pron-prob\]. Consider the word *the*:

    the 0.99    0.01    1.21    0.97    DH AH0
    the 0.01    0.12    1.87    0.84    DH AH1
    the 0.17    0.11    1.27    0.96    DH IY0

The docs tell us:

> The first float column is the probability of the pronunciation, the
> next float is the probability of silence following the pronunciation,
> and the final two floats are correction terms for preceding silence
> and non-silence. Given that each entry in a dictionary is independent
> and there is no way to encode information about the preceding context,
> the correction terms are calculated as how much more common was
> silence or non-silence compared to what we would expect factoring out
> the likelihood of silence from the previous word.

I am not sure what this means, but the first column seems to be the most
important (marginal) probability of pronunciation.
