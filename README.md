
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Get email addresses of maintainers

``` r
set.seed(123)
packages <- sample(as.character(available.packages(contriburl = contrib.url("https://cran.rstudio.com/"))[,1]), size = 1000)

get_maintainer_email <- function(package){
  Sys.sleep(1)
  metadata <- crandb::package(package)
  maintainer <- as.person(metadata$Maintainer)
  
  if(maintainer$family != "ORPHANED"){
    email <- maintainer$email[1]
    name <- paste(maintainer$given, maintainer$family)
    tibble::tibble(package = package,
                   maintainer = name,
                   email = email)
  }else{
    NULL
  }
  
  
}

emails <- purrr::map_df(packages,
                        get_maintainer_email)
```

Since `HIBPwned` implements caching inside an active R session via
`memoise` I don’t need to care about duplicate emails\! :nail\_care:

# Pwned?

``` r
library("magrittr")
emails <- dplyr::group_by(emails, package) %>%
  dplyr::mutate(pwned = list(HIBPwned::account_breaches(email)[[1]]))
```

# Pasted?

``` r
emails <- dplyr::group_by(emails, package) %>%
  dplyr::mutate(pastes = list(HIBPwned::pastes(email)[[1]]))

save(emails, file = "emails.RData")
```
