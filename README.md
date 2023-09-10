
# rbcb <img src="man/figures/logo.png" align="right" width="120" />

<!-- badges: start -->

[![](https://www.r-pkg.org/badges/version/rbcb)](https://cran.r-project.org/package=rbcb)
[![](http://cranlogs.r-pkg.org/badges/last-month/rbcb)](https://cran.r-project.org/package=rbcb)
[![R-CMD-check](https://github.com/wilsonfreitas/rbcb/actions/workflows/check-standard.yaml/badge.svg)](https://github.com/wilsonfreitas/rbcb/actions/workflows/check-standard.yaml)
[![Codecov test
coverage](https://codecov.io/gh/wilsonfreitas/rbcb/branch/master/graph/badge.svg)](https://app.codecov.io/gh/wilsonfreitas/rbcb?branch=master)
<!-- badges: end -->

An interface to structure the information provided by the [Brazilian
Central Bank](https://www.bcb.gov.br). This package interfaces the
[Brazilian Central Bank web services](https://dadosabertos.bcb.gov.br)
to provide data already formatted into R’s data structures.

## Install

From CRAN:

``` r
install.packages("rbcb")
```

From github using remotes:

``` r
remotes::install_github('wilsonfreitas/rbcb')
```

## Features

-   [rbcb](#rbcb)
    -   [Install](#install)
    -   [Features](#features)
        -   [Usage](#usage)
            -   [The `get_series` function](#the-get_series-function)
            -   [Market expectations](#market-expectations)
            -   [OLINDA API for currency
                rates](#olinda-api-for-currency-rates)
            -   [Currency rates](#currency-rates)
            -   [Cross currency rates](#cross-currency-rates)

### Usage

Load the package:

``` r
library(rbcb)
```

#### The `get_series` function

<a name="single-series"></a> Download the series by calling
`rbcb::get_series` and pass the time series code is as the first
argument. For example, let’s download the USDBRL time series which code
is `1`.

``` r
rbcb::get_series(c(USDBRL = 1))
#> # A tibble: 9,434 x 2
#>    date       USDBRL
#>    <date>      <dbl>
#>  1 1984-11-28   2828
#>  2 1984-11-29   2828
#>  3 1984-11-30   2881
#>  4 1984-12-03   2881
#>  5 1984-12-04   2881
#>  6 1984-12-05   2923
#>  7 1984-12-06   2923
#>  8 1984-12-07   2923
#>  9 1984-12-10   2965
#> 10 1984-12-11   2965
#> # ... with 9,424 more rows
```

Note that this series starts at 1984 and has approximately 8000 rows.
Also note that you can name the downloaded series by passing a named
`vector` in the `code` argument. To download recent values you should
use the argument `last = N`, see below.

<a name="tibble-objects"></a>

``` r
rbcb::get_series(c(USDBRL = 1), last = 10)
#> # A tibble: 10 x 2
#>    date       USDBRL
#>    <date>      <dbl>
#>  1 2022-07-12   5.41
#>  2 2022-07-13   5.40
#>  3 2022-07-14   5.46
#>  4 2022-07-15   5.40
#>  5 2022-07-18   5.37
#>  6 2022-07-19   5.39
#>  7 2022-07-20   5.43
#>  8 2022-07-21   5.48
#>  9 2022-07-22   5.45
#> 10 2022-07-25   5.41
```

<a name="download-types"></a> The series can be downloaded in many
different types: `tibble`, `xts`, `ts` or `data.frame`, but the default
is `tibble`. See the next example where the Brazilian Broad Consumer
Price Index (IPCA) is downloaded as `xts` object.

<a name="xts-objects"></a>

``` r
rbcb::get_series(c(IPCA = 433), last = 12, as = "xts")
#>            IPCA
#> 2021-07-01 0.96
#> 2021-08-01 0.87
#> 2021-09-01 1.16
#> 2021-10-01 1.25
#> 2021-11-01 0.95
#> 2021-12-01 0.73
#> 2022-01-01 0.54
#> 2022-02-01 1.01
#> 2022-03-01 1.62
#> 2022-04-01 1.06
#> 2022-05-01 0.47
#> 2022-06-01 0.67
```

or as a `ts` object.

<a name="ts-objects"></a>

``` r
rbcb::get_series(c(IPCA = 433), last = 12, as = "ts")
#>       Jan  Feb  Mar  Apr  May  Jun  Jul  Aug  Sep  Oct  Nov  Dec
#> 2021                               0.96 0.87 1.16 1.25 0.95 0.73
#> 2022 0.54 1.01 1.62 1.06 0.47 0.67
```

<a name="multiple-series"></a> Multiple series can be downloaded at once
by passing a named vector with the series codes. The return is a named
list with the downloaded series.

``` r
rbcb::get_series(c(IPCA = 433, IGPM = 189), last = 12, as = "ts")
#> $IPCA
#>       Jan  Feb  Mar  Apr  May  Jun  Jul  Aug  Sep  Oct  Nov  Dec
#> 2021                               0.96 0.87 1.16 1.25 0.95 0.73
#> 2022 0.54 1.01 1.62 1.06 0.47 0.67                              
#> 
#> $IGPM
#>        Jan   Feb   Mar   Apr   May   Jun   Jul   Aug   Sep   Oct   Nov   Dec
#> 2021                                      0.78  0.66 -0.64  0.64  0.02  0.87
#> 2022  1.82  1.83  1.74  1.41  0.52  0.59
```

#### Market expectations

<a name="market-expectations"></a> The function
`get_market_expectations` returns market expectations discussed in the
Focus Report that summarizes the statistics calculated from expectations
collected from market practitioners.

The first argument `type` accepts the following values:

-   `annual`: annual expectations
-   `quarterly`: quarterly expectations
-   `monthly`: monthly expectations
-   `top5s-monthly`: monthly expectations for top 5 indicators
-   `top5s-annual`: annual expectations for top 5 indicators
-   `inflation-12-months`: inflation expectations for the next 12 months
-   `institutions`: market expectations informed by financial
    institutions

The example below shows how to download IPCA’s monthly expectations.

``` r
rbcb::get_market_expectations("monthly", "IPCA", end_date = "2018-01-31", `$top` = 5)
#> # A tibble: 5 x 10
#>   Indicador Data       DataReferencia Media Mediana DesvioPadrao Minimo Maximo numeroRespondentes baseCalculo
#>   <chr>     <date>     <chr>          <dbl>   <dbl>        <dbl>  <dbl>  <dbl>              <int>       <int>
#> 1 IPCA      2018-01-31 06/2019         0.21    0.2          0.07   0.13   0.36                 14           1
#> 2 IPCA      2018-01-31 06/2019         0.2     0.2          0.1   -0.3    0.36                 43           0
#> 3 IPCA      2018-01-31 05/2019         0.31    0.29         0.06   0.22   0.43                 19           1
#> 4 IPCA      2018-01-31 05/2019         0.31    0.3          0.06   0.15   0.45                 55           0
#> 5 IPCA      2018-01-31 04/2019         0.38    0.39         0.1    0.16   0.61                 20           1
```

#### OLINDA API for currency rates

<a name="olinda-currency-rates"></a> Use currency functions to download
currency rates from the BCB OLINDA API.

``` r
olinda_list_currencies()
#>    symbol                     name currency_type
#> 1     AUD        Dólar australiano             B
#> 2     CAD          Dólar canadense             A
#> 3     CHF             Franco suíço             A
#> 4     DKK       Coroa dinamarquesa             A
#> 5     EUR                     Euro             B
#> 6     GBP          Libra Esterlina             B
#> 7     JPY                     Iene             A
#> 8     NOK         Coroa norueguesa             A
#> 9     SEK              Coroa sueca             A
#> 10    USD Dólar dos Estados Unidos             A
```

Use `olinda_get_currency` function to download data from specific
currency by the currency symbol.

``` r
olinda_get_currency("USD", "2017-03-01", "2017-03-03")
#> # A tibble: 13 x 3
#>    datetime              bid   ask
#>    <dttm>              <dbl> <dbl>
#>  1 2017-03-01 14:37:41  3.10  3.10
#>  2 2017-03-01 15:37:01  3.10  3.10
#>  3 2017-03-01 15:37:01  3.10  3.10
#>  4 2017-03-02 10:04:33  3.11  3.11
#>  5 2017-03-02 11:07:36  3.10  3.10
#>  6 2017-03-02 12:10:41  3.12  3.12
#>  7 2017-03-02 13:06:27  3.12  3.12
#>  8 2017-03-02 13:06:27  3.11  3.11
#>  9 2017-03-03 10:10:38  3.13  3.13
#> 10 2017-03-03 11:10:48  3.13  3.13
#> 11 2017-03-03 12:07:35  3.14  3.14
#> 12 2017-03-03 13:07:10  3.14  3.14
#> 13 2017-03-03 13:07:10  3.14  3.14
```

The rates come quoted in BRL, so 3.10 is worth 1 USD in BRL.

**Parity values**

Type A currencies have parity values quoted in USD (1 CURRENCY in USD).

``` r
olinda_get_currency("CAD", "2017-03-01", "2017-03-01")
#> # A tibble: 3 x 3
#>   datetime              bid   ask
#>   <dttm>              <dbl> <dbl>
#> 1 2017-03-01 14:37:41  2.32  2.32
#> 2 2017-03-01 15:37:01  2.32  2.32
#> 3 2017-03-01 15:37:01  2.32  2.32
```

``` r
olinda_get_currency("CAD", "2017-03-01", "2017-03-01", parity = TRUE)
#> # A tibble: 3 x 3
#>   datetime              bid   ask
#>   <dttm>              <dbl> <dbl>
#> 1 2017-03-01 14:37:41  1.33  1.33
#> 2 2017-03-01 15:37:01  1.33  1.33
#> 3 2017-03-01 15:37:01  1.33  1.33
```

Type B currencies have parity values as 1 USD in CURRENCY, see AUD, for
example.

``` r
olinda_get_currency("AUD", "2017-03-01", "2017-03-01")
#> # A tibble: 3 x 3
#>   datetime              bid   ask
#>   <dttm>              <dbl> <dbl>
#> 1 2017-03-01 14:37:41  2.38  2.38
#> 2 2017-03-01 15:37:01  2.38  2.38
#> 3 2017-03-01 15:37:01  2.38  2.38
```

``` r
olinda_get_currency("AUD", "2017-03-01", "2017-03-01", parity = TRUE)
#> # A tibble: 3 x 3
#>   datetime              bid   ask
#>   <dttm>              <dbl> <dbl>
#> 1 2017-03-01 14:37:41 0.768 0.768
#> 2 2017-03-01 15:37:01 0.767 0.768
#> 3 2017-03-01 15:37:01 0.767 0.768
```

#### Currency rates

<a name="currency-rates"></a> Use currency functions to download
currency rates from the BCB web site.

``` r
rbcb::get_currency("USD", "2017-03-01", "2017-03-10")
#> # A tibble: 8 x 3
#>   date         bid   ask
#>   <date>     <dbl> <dbl>
#> 1 2017-03-01  3.10  3.10
#> 2 2017-03-02  3.11  3.11
#> 3 2017-03-03  3.14  3.14
#> 4 2017-03-06  3.11  3.11
#> 5 2017-03-07  3.12  3.12
#> 6 2017-03-08  3.15  3.15
#> 7 2017-03-09  3.17  3.17
#> 8 2017-03-10  3.16  3.16
```

The rates come quoted in BRL, so 3.0970 is worth 1 USD in BRL.

All currency time series have an attribute called `symbol` that stores
its own currency name.

``` r
attr(rbcb::get_currency("USD", "2017-03-01", "2017-03-10"), "symbol")
#> [1] "USD"
```

Trying another currency.

``` r
get_currency("JPY", "2017-03-01", "2017-03-10") |> Ask()
#> # A tibble: 8 x 2
#>   date          JPY
#>   <date>      <dbl>
#> 1 2017-03-01 0.0273
#> 2 2017-03-02 0.0272
#> 3 2017-03-03 0.0274
#> 4 2017-03-06 0.0274
#> 5 2017-03-07 0.0274
#> 6 2017-03-08 0.0274
#> 7 2017-03-09 0.0276
#> 8 2017-03-10 0.0275
```

To see the avaliable currencies call `list_currencies`.

``` r
rbcb::list_currencies()
#> # A tibble: 218 x 5
#>    name                   code symbol country_name          country_code
#>    <chr>                 <dbl> <chr>  <chr>                        <dbl>
#>  1 AFEGANE AFEGANIST         5 AFN    AFEGANISTAO                    132
#>  2 RANDE/AFRICA SUL        785 ZAR    AFRICA DO SUL                 7560
#>  3 LEK ALBANIA REP         490 ALL    ALBANIA, REPUBLICA DA          175
#>  4 EURO                    978 EUR    ALEMANHA                       230
#>  5 KWANZA/ANGOLA           635 AOA    ANGOLA                         400
#>  6 DOLAR CARIBE ORIENTAL   215 XCD    ANGUILLA                       418
#>  7 DOLAR CARIBE ORIENTAL   215 XCD    ANTIGUA E BARBUDA              434
#>  8 RIAL/ARAB SAUDITA       820 SAR    ARABIA SAUDITA                 531
#>  9 DINAR ARGELINO           95 DZD    ARGELIA                        590
#> 10 PESO ARGENTINO          706 ARS    ARGENTINA                      639
#> # ... with 208 more rows
```

There are 216 currencies available.

#### Cross currency rates

<a name="cross-currency-rates"></a> The API provides a matrix with the
relations between exchange rates, this is the matrix of cross currency
rates. This is a square matrix with the all exchange rates between all
currencies.

``` r
x <- rbcb::get_currency_cross_rates("2017-03-10")
dim(x)
#> [1] 156 156
```

``` r
# Since there are many currencies it is interesting to subset the matrix.
cr <- c("USD", "BRL", "EUR", "CAD")
x[cr, cr]
#>           USD    BRL       EUR       CAD
#> USD 1.0000000 3.1623 0.9380896 1.3465764
#> BRL 0.3162255 1.0000 0.2966479 0.4258218
#> EUR 1.0659963 3.3710 1.0000000 1.4354454
#> CAD 0.7426240 2.3484 0.6966479 1.0000000
```

The rates are quoted by its columns labels, so the numbers in the BRL
column are worth one currency unit in BRL.
