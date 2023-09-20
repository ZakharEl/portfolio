Births from 2000 to 2014
================

Data for this example is taken from [this
repository](https://github.com/fivethirtyeight/data/blob/master/births/US_births_2000-2014_SSA.csv).
This example uses sqlite 3 but explains what to change to use other
methods like Azure SQL Database (used by SQL Server Management Studio).
Really only one line in the following R code block needs to be changed
to use other methods. This github flavored markdown file is generated
from [this rmarkdown file](rmarkdown-and-sql-built-births-display.rmd).

# Connecting to the SQL Server

``` r
library('DBI')
library('reshape')
con <- dbConnect(RSQLite::SQLite(), dbname = "rmarkdown-and-sql-built-births-display.db")
birth_data <- read.csv('https://raw.githubusercontent.com/fivethirtyeight/data/master/births/US_births_2000-2014_SSA.csv')
melted_birth_data <- melt(birth_data, id = "year", measure = "births")
birth_data_by_year <- as.data.frame(cast(melted_birth_data, year~variable, sum))
dbWriteTable(con, "births2000_2014", birth_data_by_year, overwrite = TRUE)
```

For this example to work one must have DBI, reshape and RSQLite R
packages installed. One also needs to have RSQLite for one’s OS
installed too. If one is using a different SQL database type then one
needs to have that installed on their OS and have the appropriately DBI
supported R package for that type installed as well ([see this for
supported DBI backend packages](https://dbi.r-dbi.org/)). One would also
have to have the dbConnect call of line 3 of the above R code block
changed to the call appropriate for the given database type.

## Example with Azure SQL Database

Change line 3 of the above R code block to the following:

``` r
con <- dbConnect(AzureKusto::AzureKusto(), server="https://azure-database-url.net",
database="desired-database", tenantid="appropriate-azure-tenantid")
```

. Of course AzureKusto R package must be installed as well.

# Displaying Table with SQL Code Block

``` sql
SELECT * FROM births2000_2014;
```

<div class="knitsql-table">

| year |  births |
|:-----|--------:|
| 2000 | 4149598 |
| 2001 | 4110963 |
| 2002 | 4099313 |
| 2003 | 4163060 |
| 2004 | 4186863 |
| 2005 | 4211941 |
| 2006 | 4335154 |
| 2007 | 4380784 |
| 2008 | 4310737 |
| 2009 | 4190991 |

Displaying records 1 - 10

</div>

# Disconnecting from the SQL Server

``` r
dbDisconnect(con)
```
