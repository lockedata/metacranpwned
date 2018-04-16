
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Get email addresses of maintainers

``` r
library("magrittr")
```

Code adapted from [this blog
post](http://blog.revolutionanalytics.com/2018/03/the-most-prolific-package-maintainers-on-cran.html).

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
#> Parsed with column specification:
#> cols(
#>   Package = col_character(),
#>   email = col_character()
#> )
knitr::kable(emails[1:10,])
```

| Package     | email                               |
| :---------- | :---------------------------------- |
| A3          | <scottfr@berkeley.edu>              |
| abbyyR      | <gsood07@gmail.com>                 |
| abc         | <michael.blum@imag.fr>              |
| ABCanalysis | <lerch@mathematik.uni-marburg.de>   |
| abc.data    | <michael.blum@imag.fr>              |
| abcdeFBA    | <gangutalk@gmail.com>               |
| ABCoptim    | <g.vegayon@gmail.com>               |
| ABCp2       | <katie.duryea@gmail.com>            |
| ABC.RAP     | <a.alsaleh@hotmail.co.nz>           |
| abcrf       | <jean-michel.marin@umontpellier.fr> |

We have 12444 packages with 7173 unique email addresses. We do not have
to care about their uniqueness: since `HIBPwned` implements caching
inside an active R session via `memoise` duplicate emails do not mean
duplicate request\! :nail\_care:

# Pwned?

``` r
library("magrittr")
emails <- dplyr::group_by(emails, package) %>%
  dplyr::mutate(pwned = list(HIBPwned::account_breaches(email)[[1]]))

```

# Pasted?

``` r
emails <- dplyr::group_by(emails, package) %>%
  dplyr::mutate(pastes = list(HIBPwned::pastes(email)))

save(emails, file = "emails.RData")
```
