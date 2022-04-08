
<!-- README.md is generated from README.Rmd. Please edit that file -->

# phsmethods

[![GitHub release (latest by
date)](https://img.shields.io/github/v/release/Public-Health-Scotland/phsmethods)](https://github.com/Public-Health-Scotland/phsmethods/releases/latest)
[![Build
Status](https://github.com/Public-Health-Scotland/phsmethods/workflows/R-CMD-check/badge.svg)](https://github.com/Public-Health-Scotland/phsmethods/actions)
[![codecov](https://codecov.io/gh/Public-Health-Scotland/phsmethods/branch/master/graph/badge.svg)](https://codecov.io/gh/Public-Health-Scotland/phsmethods)

`phsmethods` contains functions for commonly undertaken analytical tasks
in [Public Health Scotland
(PHS)](https://www.publichealthscotland.scot/):

-   `create_age_groups()` categorises ages into groups
-   `chi_check()` assesses the validity of a CHI number
-   `chi_pad()` adds a leading zero to nine-digit CHI numbers
-   `sex_from_chi()` extracts the sex of a person from a CHI number
-   `file_size()` returns the names and sizes of files in a directory
-   `extract_fin_year()` assigns a date to a financial year in the
    format `YYYY/YY`
-   `match_area()` converts geography codes into area names
-   `format_postcode()` formats improperly recorded postcodes
-   `qtr()`, `qtr_end()`, `qtr_next()` and `qtr_prev()` assign a date to
    a quarter
-   `age_calculate` calculates age between two dates
-   `dob_from_chi` extracts Date of Birth (DoB) from the CHI number
-   `age_from_chi` extracts age from the CHI number

`phsmethods` can be used on both the
[server](https://rstudio.nhsnss.scot.nhs.uk/) and desktop versions of
RStudio.

## Installation

To install `phsmethods`, the package `remotes` is required, and can be
installed with `install.packages("remotes")`.

You can then install `phsmethods` on RStudio server from GitHub with:

``` r
remotes::install_github("Public-Health-Scotland/phsmethods",
  upgrade = "never"
)
```

Network security settings may prevent `remotes::install_github()` from
working on RStudio desktop. If this is the case, `phsmethods` can be
installed by downloading the [zip of the
repository](https://github.com/Public-Health-Scotland/phsmethods/archive/master.zip)
and running the following code (replacing the section marked `<>`,
including the arrows themselves):

``` r
remotes::install_local("<FILEPATH OF ZIPPED FILE>/phsmethods-master.zip",
  upgrade = "never"
)
```

## Using phsmethods

Load `phsmethods` using `library()`:

``` r
library(phsmethods)
```

To see the documentation for any `phsmethods`’ functions, type
`?function_name` into the RStudio console after loading the package:

``` r
?extract_fin_year
?format_postcode
```

### create_age_groups

``` r
ages <- c(54, 7, 77, 1, 26, 101)

# By default create_age_groups goes in 5 year increments from 0 to 90+
create_age_groups(ages)
#> [1] "50-54" "5-9"   "75-79" "0-4"   "25-29" "90+"

# But these settings can be changed
create_age_groups(ages, from = 0, to = 80, by = 10)
#> [1] "50-59" "0-9"   "70-79" "0-9"   "20-29" "80+"
```

### chi_check

``` r
# chi_check returns specific feedback on why a CHI number might be invalid
library(dplyr)
chi_data <- tibble(chi = c("0101011237", "0101336489", "3213201234", "123456789", "12345678900", "010120123?", NA))
chi_data %>% 
  mutate(validity = chi_check(chi))
#> # A tibble: 7 × 2
#>   chi         validity                    
#>   <chr>       <chr>                       
#> 1 0101011237  Valid CHI                   
#> 2 0101336489  Valid CHI                   
#> 3 3213201234  Invalid date                
#> 4 123456789   Too few characters          
#> 5 12345678900 Too many characters         
#> 6 010120123?  Invalid character(s) present
#> 7 <NA>        Missing
```

### chi_pad

``` r
chi_numbers <- c("101011237", "101201234", "123223", "abcdefghi", "12345tuvw")
# Only nine-digit characters comprised exclusively of numeric digits are prefixed with a zero
chi_pad(chi_numbers)
#> [1] "0101011237" "0101201234" "123223"     "abcdefghi"  "12345tuvw"
```

### sex_from_chi

``` r
# By default it will check that the CHI is valid before extracting the sex.
library(dplyr)
chi_data <- tibble(chi = c("0101011237", "0101336489", "123456789", "12345678900", "010120123?", NA))

chi_data %>% 
  mutate(chi_sex = sex_from_chi(chi))
#> # A tibble: 6 × 2
#>   chi         chi_sex
#>   <chr>         <int>
#> 1 0101011237        1
#> 2 0101336489        2
#> 3 123456789        NA
#> 4 12345678900      NA
#> 5 010120123?       NA
#> 6 <NA>             NA

# Use custom values for male and female
chi_data %>% 
  mutate(chi_sex = sex_from_chi(chi, male_value = "M", female_value = "F"))
#> Using custom values: Male = M Female = F.
#> The return variable will be character.
#> # A tibble: 6 × 2
#>   chi         chi_sex
#>   <chr>       <chr>  
#> 1 0101011237  M      
#> 2 0101336489  F      
#> 3 123456789   <NA>   
#> 4 12345678900 <NA>   
#> 5 010120123?  <NA>   
#> 6 <NA>        <NA>

# Alternatively return the result as a factor (with labels 'Male' and 'Female')
chi_data %>% 
  mutate(chi_sex = sex_from_chi(chi, as_factor = TRUE))
#> # A tibble: 6 × 2
#>   chi         chi_sex
#>   <chr>       <fct>  
#> 1 0101011237  Male   
#> 2 0101336489  Female 
#> 3 123456789   <NA>   
#> 4 12345678900 <NA>   
#> 5 010120123?  <NA>   
#> 6 <NA>        <NA>
```

### file_size

``` r
# Names and sizes of all files in the tests/testthat/files folder
file_size(testthat::test_path("files"))
#> # A tibble: 8 × 2
#>   name             size       
#>   <chr>            <chr>      
#> 1 airquality.xls   Excel 26 KB
#> 2 bod.xlsx         Excel 5 KB 
#> 3 iris.csv         CSV 4 KB   
#> 4 mtcars.sav       SPSS 4 KB  
#> 5 plant-growth.rds RDS 316 B  
#> 6 puromycin.txt    Text 418 B 
#> 7 stackloss.fst    FST 897 B  
#> 8 swiss.tsv        TSV 1 KB

# Names and sizes of Excel files only in the tests/testthat/files folder
file_size(testthat::test_path("files"), pattern = "\\.xlsx?$")
#> # A tibble: 2 × 2
#>   name           size       
#>   <chr>          <chr>      
#> 1 airquality.xls Excel 26 KB
#> 2 bod.xlsx       Excel 5 KB
```

### extract_fin_year

``` r
dates <- lubridate::dmy(c(21012017, 04042017, 17112017))
extract_fin_year(dates)
#> [1] "2016/17" "2017/18" "2017/18"
```

### match_area

``` r
match_area("S13002781")
#> [1] "Ayr North"

geog_data <- tibble(code = c("S02000656", "S02001042", "S08000020", "S12000013", "S13002605"))
geog_data %>% 
  mutate(name = match_area(code))
#> # A tibble: 5 × 2
#>   code      name               
#>   <chr>     <chr>              
#> 1 S02000656 Govan and Linthouse
#> 2 S02001042 Peebles North      
#> 3 S08000020 Grampian           
#> 4 S12000013 Na h-Eileanan Siar 
#> 5 S13002605 Steòrnabhagh a Deas
```

### format_postcode

``` r
# The default is pc7 format
format_postcode("G26QE")
#> [1] "G2  6QE"

# But pc8 format can also be applied
format_postcode(c("KA89NB", "PA152TY"), format = "pc8")
#> [1] "KA8 9NB"  "PA15 2TY"

# postcode accounts for irregular spacing and lower case letters
postcode_data <- tibble(postcode = c("G 4 2 9 B A", "g207al", "Dg98bS", "DD37J    y"))
postcode_data %>% 
  mutate(postcode = format_postcode(postcode))
#> # A tibble: 4 × 1
#>   postcode
#>   <chr>   
#> 1 G42 9BA 
#> 2 G20 7AL 
#> 3 DG9 8BS 
#> 4 DD3 7JY
```

### qtr, qtr_end, qtr_next and qtr_prev

``` r
dates <- lubridate::dmy(c(26032012, 04052012, 23092012))

# qtr returns the current quarter and year
# The default is long format
qtr(dates)
#> [1] "January to March 2012"  "April to June 2012"     "July to September 2012"

# But short format can also be applied
qtr(dates, format = "short")
#> [1] "Jan-Mar 2012" "Apr-Jun 2012" "Jul-Sep 2012"


# qtr_end returns the last month in the quarter
qtr_end(dates)
#> [1] "March 2012"     "June 2012"      "September 2012"
qtr_end(dates, format = "short")
#> [1] "Mar 2012" "Jun 2012" "Sep 2012"


# qtr_next returns the next quarter
qtr_next(dates)
#> [1] "April to June 2012"       "July to September 2012"  
#> [3] "October to December 2012"
qtr_next(dates, format = "short")
#> [1] "Apr-Jun 2012" "Jul-Sep 2012" "Oct-Dec 2012"


# qtr_prev returns the previous quarter
qtr_prev(dates)
#> [1] "October to December 2011" "January to March 2012"   
#> [3] "April to June 2012"
qtr_prev(dates, format = "short")
#> [1] "Oct-Dec 2011" "Jan-Mar 2012" "Apr-Jun 2012"
```

### age_calculate

``` r
birth_date <- lubridate::ymd("2020-02-29")
end_date <- lubridate::ymd("2022-02-21")

# Change the argument of date_class can make a difference.
age_calculate(birth_date, end_date, round_down = FALSE, date_class = "period") * 365.25
#> [1] 723.0625
age_calculate(birth_date, end_date, round_down = FALSE, date_class = "duration") * 365.25
#> [1] 723

# If the start day is leap day (February 29th), age increases on 1st March every year. 
leap1 <- lubridate::ymd("2020-02-29")
leap2 <- lubridate::ymd("2022-02-28")

age_calculate(leap1, leap2, date_class = "period")
#> [1] 1
```

### dob_from_chi

``` r
dob_from_chi("0101336489")
#> [1] "1933-01-01"

library(tibble)
library(dplyr)
data <- tibble(chi = c(
 "0101336489",
 "0101405073",
 "0101625707"
), adm_date = as.Date(c(
 "1950-01-01",
 "2000-01-01",
 "2020-01-01"
)))

data %>%
 mutate(chi_dob = dob_from_chi(chi))
#> # A tibble: 3 × 3
#>   chi        adm_date   chi_dob   
#>   <chr>      <date>     <date>    
#> 1 0101336489 1950-01-01 1933-01-01
#> 2 0101405073 2000-01-01 1940-01-01
#> 3 0101625707 2020-01-01 1962-01-01

data %>%
 mutate(chi_dob = dob_from_chi(chi,
   min_date = as.Date("1930-01-01"),
   max_date = adm_date
 ))
#> # A tibble: 3 × 3
#>   chi        adm_date   chi_dob   
#>   <chr>      <date>     <date>    
#> 1 0101336489 1950-01-01 1933-01-01
#> 2 0101405073 2000-01-01 1940-01-01
#> 3 0101625707 2020-01-01 1962-01-01
```

### age_from_chi

``` r
age_from_chi("0101336489")
#> [1] 89

library(tibble)
library(dplyr)
data <- tibble(chi = c(
 "0101336489",
 "0101405073",
 "0101625707"
), dis_date = as.Date(c(
 "1950-01-01",
 "2000-01-01",
 "2020-01-01"
)))

data %>%
 mutate(chi_age = age_from_chi(chi))
#> # A tibble: 3 × 3
#>   chi        dis_date   chi_age
#>   <chr>      <date>       <dbl>
#> 1 0101336489 1950-01-01      89
#> 2 0101405073 2000-01-01      82
#> 3 0101625707 2020-01-01      60

data %>%
 mutate(chi_age = age_from_chi(chi, min_age = 18, max_age = 65))
#> 2 CHI numbers produced ambiguous dates and will be given NA for DoB, if possible try different values for min_date and/or max_date
#> # A tibble: 3 × 3
#>   chi        dis_date   chi_age
#>   <chr>      <date>       <dbl>
#> 1 0101336489 1950-01-01      NA
#> 2 0101405073 2000-01-01      NA
#> 3 0101625707 2020-01-01      60

data %>%
 mutate(chi_age = age_from_chi(chi,
   ref_date = dis_date
 ))
#> # A tibble: 3 × 3
#>   chi        dis_date   chi_age
#>   <chr>      <date>       <dbl>
#> 1 0101336489 1950-01-01      17
#> 2 0101405073 2000-01-01      60
#> 3 0101625707 2020-01-01      58
```

## Contributing to phsmethods

At present, the maintainer of this package is [Tina
Fu](https://github.com/Tina815).

This package is intended to be in continuous development and
contributions may be made by anyone within PHS. If you would like to
contribute, please first create an
[issue](https://github.com/Public-Health-Scotland/phsmethods/issues) on
GitHub and assign **both** of the package maintainers to it. This is to
ensure that no duplication of effort occurs in the case of multiple
people having the same idea. The package maintainers will discuss the
issue and get back to you as soon as possible.

While the most obvious and eye-catching (as well as intimidating) way of
contributing is by writing a function, this isn’t the only way to make a
useful contribution. Fixing typos in documentation, for example, isn’t
the most glamorous way to contribute, but is of great help to the
package maintainers. Please see this [blog post by Jim
Hester](https://www.tidyverse.org/blog/2017/08/contributing/) for more
information on getting started with contributing to open-source
software.

When contributing, please create a
[branch](https://github.com/Public-Health-Scotland/phsmethods/branches)
in this repository and carry out all work on it. Please ensure you have
linked RStudio to your GitHub account using `usethis::edit_git_config()`
prior to making your contribution. When you are ready for a review,
please create a [pull
request](https://github.com/Public-Health-Scotland/phsmethods/pulls) and
assign **both** of the package maintainers as reviewers. One or both of
them will conduct a review, provide feedback and, if necessary, request
changes before merging your branch.

Please be mindful of information governance when contributing to this
package. No data files (aside from publicly available and downloadable
datasets or unless explicitly approved), server connection details,
passwords or person identifiable or otherwise confidential information
should be included anywhere within this package or any other repository
(whether public or private) used within PHS. This includes within code
and code commentary. For more information on security when using git and
GitHub, and on using git and GitHub for version control more generally,
please see the [Transforming Publishing
Programme](https://www.isdscotland.org/Products-and-Services/Transforming-Publishing-Programme/)’s
[Git guide](https://Public-Health-Scotland.github.io/git-guide/) and
[GitHub
guidance](https://github.com/Public-Health-Scotland/GitHub-guidance).

Please feel free to add yourself to the ‘Authors’ section of the
`Description` file when contributing. As a rule of thumb, please assign
your role as author (`"aut"`) when writing an exported function, and as
contributor (`"ctb"`) for anything else.

`phsmethods` will, as much as possible, adhere to the [tidyverse style
guide](https://style.tidyverse.org/) and the [rOpenSci package
development guide](https://devguide.ropensci.org/). The most pertinent
points to take from these are:

-   All function names should be in lower case, with words separated by
    an underscore
-   Put a space after a comma, never before
-   Put a space before and after infix operators such as `<-`, `==` and
    `+`
-   Limit code to 80 characters per line
-   Function documentation should be generated using
    [`roxygen2`](https://github.com/r-lib/roxygen2)
-   All functions should be tested using
    [`testthat`](https://github.com/r-lib/testthat)
-   The package should always pass `devtools::check()`

It’s not necessary to have experience with GitHub or of building an R
package to contribute to `phsmethods`. If you wish to contribute code
then, as long as you can write an R function, the package maintainers
can assist with error handling, writing documentation, testing and other
aspects of package development. It is advised, however, to consult
[Hadley Wickham’s R Packages book](https://r-pkgs.org/) prior to making
a contribution. It may also be useful to consult the
[documentation](https://github.com/Public-Health-Scotland/phsmethods/tree/master/R)
and
[tests](https://github.com/Public-Health-Scotland/phsmethods/tree/master/tests/testthat)
of existing functions within this package as a point of reference.

Please note that this README may fail to ‘Knit’ at times as a result of
network security settings. This will likely be due to the badges for the
package’s release version, continuous integration status and test
coverage at the top of the document. You should only make edits to the
`.Rmd` version, you can then knit yourself, or a GitHub action should
run and knit it for you when you open a pull request.
