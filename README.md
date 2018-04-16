
# How many CRAN package maintainers have been pwned?

The alternative title of this blog post is *`HIBPwned` version 0.1.7 has
been released\! W00t\!*. Steph Locke’s `HIBPwned` package utilises the
[HaveIBeenPwned.com API](https://haveibeenpwned.com/API/v2) to check
whether email addresses and/or user names have been present in a
publicly disclosed data breach. In other words, this package potentially
delivers bad news, but useful bad news\!

This release is mainly a maintenance release, with some cool code
changes invisible to the user. Wouldn’t it be a pity to echo the
[release
notes](https://github.com/lockedata/HIBPwned/blob/master/NEWS.md#hibpwned-017)
without a nifty use case? Another blog post will give more details about
the technical niftiness of the release, but here, let’s make you
curious\! How many CRAN package maintainers were pwned?

<!-- README.md is generated from README.Rmd. Please edit that file -->

# Get the email addresses of CRAN package maintainers

``` r
library("magrittr")
```

Data was gathered thanks to an adaptation of the code published in [this
blog post of David Smith’s about prolific package
maintainers](http://blog.revolutionanalytics.com/2018/03/the-most-prolific-package-maintainers-on-cran.html).
*We* are after the most endangered package maintainers on CRAN\!

The helper function below extracts the email address of a string such as
“Jane Doe <jane.doe@fakedomain.io>”. On top of using the `as.person`
conversion, this function also deals with a few particular use cases.

``` r
get_maintainer_email <- function(maintainer_string){
  if(inherits(maintainer_string, "data.frame")){
    maintainer_string <- maintainer_string$Maintainer[1]
  }
  
  if(maintainer_string != "ORPHANED"){
     maintainer_string <- stringr::str_replace_all(maintainer_string,
                                                '"', '')
     maintainer_string <- stringr::str_replace_all(maintainer_string,
                                                ',', '')
     # particular case!
     maintainer_string <- stringr::str_replace_all(maintainer_string,
                                                'Berlin School of Economics and Law', '')
    maintainer <- as.person(maintainer_string)
    maintainer$email
  }else{
    ""
  }
  
}
```

Here it is in action.

``` r
get_maintainer_email("Jane Doe <jane.doe@fakedomain.io>")
#> [1] "jane.doe@fakedomain.io"
```

The following code then gathers the email addresses of all CRAN package
maintainers.

``` r
tools::CRAN_package_db() %>%
  .[, c("Package", "Maintainer")] %>%
  tidyr::nest(Maintainer, .key = "Maintainer") %>%
  # get the email out of the maintainer
  dplyr::mutate(email = purrr::map_chr(Maintainer,
                                       get_maintainer_email)) %>%
  dplyr::select(- Maintainer) %>%
  # only keep the ones with email
  dplyr::filter(email != "") %>%
  # save result
  readr::write_csv(path = "data/all_packages.csv")
```

``` r
emails <- readr::read_csv("data/all_packages.csv")
```

We obtained 12444 packages with 7173 unique email addresses. We do not
have to care about their uniqueness: since `HIBPwned` implements caching
inside an active R session via
[`memoise`](https://github.com/r-lib/memoise) duplicate emails do not
mean duplicate request\! :nail\_care: Another aspect we users do not
need to care about is rate limiting: `HIBPwned` uses the nice
[`ratelimitr` package](https://github.com/tarakc02/ratelimitr) in order
to automatically pause R when needed.

# So, have they been pwned?

Thanks to setting the new `as_list` option to FALSE we get a data.frame
as output. Note that choosing this means we only get back accounts with
breaches. Depending on the analysis, we could supplement the original
`emails` data.frame with the information using `dplyr::left_join` for
instance.

``` r
pwned <- HIBPwned::account_breaches(emails$email,
                                    as_list = FALSE)
pwned <- unique(pwned)
```

There are 7173 unique CRAN maintainer emails, among which 6 i.e. 0% have
been pwned.

``` r
pwned %>%
  dplyr::count(account) %>%
  dplyr::summarise(median = median(n),
                   min = min(n),
                   max = max(n)) %>%
  knitr::kable()
```

| median | min | max |
| -----: | --: | --: |
|    2.5 |   1 |   7 |

There are 10 breaches. What were the most common breaches?

``` r
pwned %>%
  dplyr::group_by(Title, BreachDate) %>%
  dplyr::tally() %>%
  dplyr::arrange(desc(n)) %>%
  head(10) %>%
  knitr::kable()
```

| Title       | BreachDate | n |
| :---------- | :--------- | -: |
| LinkedIn    | 2012-05-05 | 5 |
| Dropbox     | 2012-07-01 | 4 |
| Last.fm     | 2012-03-22 | 3 |
| Adobe       | 2013-10-04 | 1 |
| Bitly       | 2014-05-08 | 1 |
| Dailymotion | 2016-10-20 | 1 |
| Disqus      | 2012-07-01 | 1 |
| GeekedIn    | 2016-08-15 | 1 |
| MDPI        | 2016-08-30 | 1 |
| MySpace     | 2008-07-01 | 1 |

# Pasted?

Keep as idea but do not implement here.
