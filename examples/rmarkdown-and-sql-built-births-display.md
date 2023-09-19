Births from 2000 to 2014
================

Data for this example is taken from [this
repository](https://github.com/fivethirtyeight/data/blob/master/births/US_births_2000-2014_SSA.csv).
This example uses sqlite 3 but explains what to change to use other
methods like Azure SQL Database (used by SQL Server Management Studio).
Really only one line in the following R code block needs to be changed
to use other methods.

# Connecting to the SQL Server

``` r
library('DBI')
con <- dbConnect(RSQLite::SQLite(), dbname = "rmarkdown-and-sql-built-births-display.db")
birth_data <- read.csv('https://raw.githubusercontent.com/fivethirtyeight/data/master/births/US_births_2000-2014_SSA.csv')
dbWriteTable(con, "births2000_2014", birth_data, overwrite = TRUE)
```

# Displaying Table in SQL Code Block

``` sql
SELECT year, month, date_of_month, births FROM births2000_2014
WHERE births > 9000;
```

<div class="knitsql-table">

| year | month | date_of_month | births |
|-----:|------:|--------------:|-------:|
| 2000 |     1 |             1 |   9083 |
| 2000 |     1 |             3 |  11363 |
| 2000 |     1 |             4 |  13032 |
| 2000 |     1 |             5 |  12558 |
| 2000 |     1 |             6 |  12466 |
| 2000 |     1 |             7 |  12516 |
| 2000 |     1 |            10 |  11668 |
| 2000 |     1 |            11 |  12611 |
| 2000 |     1 |            12 |  12398 |
| 2000 |     1 |            13 |  11815 |

Displaying records 1 - 10

</div>

\#Disconnecting from the SQL Server

``` r
dbDisconnect(con)
```
