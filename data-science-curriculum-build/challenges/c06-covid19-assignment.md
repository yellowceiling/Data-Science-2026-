COVID-19
================
(Your name here)
2020-

- [Grading Rubric](#grading-rubric)
  - [Individual](#individual)
  - [Submission](#submission)
- [The Big Picture](#the-big-picture)
- [Get the Data](#get-the-data)
  - [Navigating the Census Bureau](#navigating-the-census-bureau)
    - [**q1** Load Table `B01003` into the following tibble. Make sure
      the column names are
      `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.](#q1-load-table-b01003-into-the-following-tibble-make-sure-the-column-names-are-id-geographic-area-name-estimatetotal-margin-of-errortotal)
  - [Automated Download of NYT Data](#automated-download-of-nyt-data)
    - [**q2** Visit the NYT GitHub repo and find the URL for the **raw**
      US County-level data. Assign that URL as a string to the variable
      below.](#q2-visit-the-nyt-github-repo-and-find-the-url-for-the-raw-us-county-level-data-assign-that-url-as-a-string-to-the-variable-below)
- [Join the Data](#join-the-data)
  - [**q3** Process the `id` column of `df_pop` to create a `fips`
    column.](#q3-process-the-id-column-of-df_pop-to-create-a-fips-column)
  - [**q4** Join `df_covid` with `df_q3` by the `fips` column. Use the
    proper type of join to preserve *only* the rows in
    `df_covid`.](#q4-join-df_covid-with-df_q3-by-the-fips-column-use-the-proper-type-of-join-to-preserve-only-the-rows-in-df_covid)
- [Analyze](#analyze)
  - [Normalize](#normalize)
    - [**q5** Use the `population` estimates in `df_data` to normalize
      `cases` and `deaths` to produce per 100,000 counts \[3\]. Store
      these values in the columns `cases_per100k` and
      `deaths_per100k`.](#q5-use-the-population-estimates-in-df_data-to-normalize-cases-and-deaths-to-produce-per-100000-counts-3-store-these-values-in-the-columns-cases_per100k-and-deaths_per100k)
  - [Guided EDA](#guided-eda)
    - [**q6** Compute some summaries](#q6-compute-some-summaries)
    - [**q7** Find and compare the top
      10](#q7-find-and-compare-the-top-10)
  - [Self-directed EDA](#self-directed-eda)
    - [**q8** Drive your own ship: You’ve just put together a very rich
      dataset; you now get to explore! Pick your own direction and
      generate at least one punchline figure to document an interesting
      finding. I give a couple tips & ideas
      below:](#q8-drive-your-own-ship-youve-just-put-together-a-very-rich-dataset-you-now-get-to-explore-pick-your-own-direction-and-generate-at-least-one-punchline-figure-to-document-an-interesting-finding-i-give-a-couple-tips--ideas-below)
    - [Ideas](#ideas)
    - [Aside: Some visualization
      tricks](#aside-some-visualization-tricks)
    - [Geographic exceptions](#geographic-exceptions)
- [Notes](#notes)

*Purpose*: In this challenge, you’ll learn how to navigate the U.S.
Census Bureau website, programmatically download data from the internet,
and perform a county-level population-weighted analysis of current
COVID-19 trends. This will give you the base for a very deep
investigation of COVID-19, which we’ll build upon for Project 1.

<!-- include-rubric -->

# Grading Rubric

<!-- -------------------------------------------------- -->

Unlike exercises, **challenges will be graded**. The following rubrics
define how you will be graded, both on an individual and team basis.

## Individual

<!-- ------------------------- -->

| Category | Needs Improvement | Satisfactory |
|----|----|----|
| Effort | Some task **q**’s left unattempted | All task **q**’s attempted |
| Observed | Did not document observations, or observations incorrect | Documented correct observations based on analysis |
| Supported | Some observations not clearly supported by analysis | All observations clearly supported by analysis (table, graph, etc.) |
| Assessed | Observations include claims not supported by the data, or reflect a level of certainty not warranted by the data | Observations are appropriately qualified by the quality & relevance of the data and (in)conclusiveness of the support |
| Specified | Uses the phrase “more data are necessary” without clarification | Any statement that “more data are necessary” specifies which *specific* data are needed to answer what *specific* question |
| Code Styled | Violations of the [style guide](https://style.tidyverse.org/) hinder readability | Code sufficiently close to the [style guide](https://style.tidyverse.org/) |

## Submission

<!-- ------------------------- -->

Make sure to commit both the challenge report (`report.md` file) and
supporting files (`report_files/` folder) when you are done! Then submit
a link to Canvas. **Your Challenge submission is not complete without
all files uploaded to GitHub.**

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.2.1     ✔ readr     2.2.0
    ## ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ## ✔ ggplot2   4.0.3     ✔ tibble    3.3.1
    ## ✔ lubridate 1.9.5     ✔ tidyr     1.3.2
    ## ✔ purrr     1.2.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

*Background*:
[COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019) is
the disease caused by the virus SARS-CoV-2. In 2020 it became a global
pandemic, leading to huge loss of life and tremendous disruption to
society. The New York Times (as of writing) publishes up-to-date data on
the progression of the pandemic across the United States—we will study
these data in this challenge.

*Optional Readings*: I’ve found this [ProPublica
piece](https://www.propublica.org/article/how-to-understand-covid-19-numbers)
on “How to understand COVID-19 numbers” to be very informative!

# The Big Picture

<!-- -------------------------------------------------- -->

We’re about to go through *a lot* of weird steps, so let’s first fix the
big picture firmly in mind:

We want to study COVID-19 in terms of data: both case counts (number of
infections) and deaths. We’re going to do a county-level analysis in
order to get a high-resolution view of the pandemic. Since US counties
can vary widely in terms of their population, we’ll need population
estimates in order to compute infection rates (think back to the
`Titanic` challenge).

That’s the high-level view; now let’s dig into the details.

# Get the Data

<!-- -------------------------------------------------- -->

1.  County-level population estimates (Census Bureau)
2.  County-level COVID-19 counts (New York Times)

## Navigating the Census Bureau

<!-- ------------------------- -->

**Steps**: Our objective is to find the 2018 American Community
Survey\[1\] (ACS) Total Population estimates, disaggregated by counties.
To check your results, this is Table `B01003`.

1.  Go to [data.census.gov](data.census.gov).
2.  Scroll down and click `View Tables`.
3.  Apply filters to find the ACS **Total Population** estimates,
    disaggregated by counties. I used the filters:

- `Topics > Populations and People > Counts, Estimates, and Projections > Population Total`
- `Geography > County > All counties in United States`

5.  Select the **Total Population** table and click the `Download`
    button to download the data; make sure to select the 2018 5-year
    estimates.
6.  Unzip and move the data to your `challenges/data` folder.

- Note that the data will have a crazy-long filename like
  `ACSDT5Y2018.B01003_data_with_overlays_2020-07-26T094857.csv`. That’s
  because metadata is stored in the filename, such as the year of the
  estimate (`Y2018`) and my access date (`2020-07-26`). **Your filename
  will vary based on when you download the data**, so make sure to copy
  the filename that corresponds to what you downloaded!

### **q1** Load Table `B01003` into the following tibble. Make sure the column names are `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.

*Hint*: You will need to use the `skip` keyword when loading these data!

``` r
## TASK: Load the census bureau data with the following tibble name.
#setwd("C:/Users/sjalinous/Data-Science-2026-/data-science-curriculum-build/challenges/ACSDT5Y2018.B01003_2026-10-15T105154")

# df_pop <-
#   read.csv("C:/Users/sjalinous/Data-Science-2026-/data-science-curriculum-build/challenges/ACSDT5Y2018.B01003_2026-10-15T105154/ACSDT5Y2018.B01003-Data.csv", skip = 1)
# 
# colnames(df_pop) <-
#   c("id", "Geographic Area Name", "Estimate!!Total", "Margin of Error!!Total")

filepath <- 
  "C:/Users/sjalinous/Data-Science-2026-/data-science-curriculum-build/challenges/ACSDT5Y2018.B01003_2026-10-15T105154/ACSDT5Y2018.B01003-Data.csv"

column_list <- 
  c("id", 
    "Geographic Area Name", 
    "Estimate!!Total", 
    "Margin of Error!!Total"
  )


df_pop <- 
  filepath %>%
  read_csv(
    skip = 2,
    col_names = column_list,
    col_select = column_list #banish column 5
  )
```

    ## Warning: Using an external vector in selections was deprecated in tidyselect 1.1.0.
    ## ℹ Please use `all_of()` or `any_of()` instead.
    ##   # Was:
    ##   data %>% select(column_list)
    ## 
    ##   # Now:
    ##   data %>% select(all_of(column_list))
    ## 
    ## See <https://tidyselect.r-lib.org/reference/faq-external-vector.html>.
    ## This warning is displayed once per session.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

    ## Rows: 3220 Columns: 4
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (3): id, Geographic Area Name, Margin of Error!!Total
    ## dbl (1): Estimate!!Total
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

*Note*: You can find information on 1-year, 3-year, and 5-year estimates
[here](https://www.census.gov/programs-surveys/acs/guidance/estimates.html).
The punchline is that 5-year estimates are more reliable but less
current.

## Automated Download of NYT Data

<!-- ------------------------- -->

ACS 5-year estimates don’t change all that often, but the COVID-19 data
are changing rapidly. To that end, it would be nice to be able to
*programmatically* download the most recent data for analysis; that way
we can update our analysis whenever we want simply by re-running our
notebook. This next problem will have you set up such a pipeline.

The New York Times is publishing up-to-date data on COVID-19 on
[GitHub](https://github.com/nytimes/covid-19-data).

### **q2** Visit the NYT [GitHub](https://github.com/nytimes/covid-19-data) repo and find the URL for the **raw** US County-level data. Assign that URL as a string to the variable below.

``` r
## TASK: Find the URL for the NYT covid-19 county-level data
url_counties <- "https://raw.githubusercontent.com/nytimes/covid-19-data/refs/heads/master/us-counties.csv"
```

Once you have the url, the following code will download a local copy of
the data, then load the data into R.

``` r
## NOTE: No need to change this; just execute
## Set the filename of the data to download
filename_nyt <- "./data/nyt_counties.csv"

## Download the data locally
curl::curl_download(
        url_counties,
        destfile = filename_nyt
      )

## Loads the downloaded csv
df_covid <- read_csv(filename_nyt)
```

    ## Rows: 2502832 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (3): county, state, fips
    ## dbl  (2): cases, deaths
    ## date (1): date
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#view(df_covid)
```

You can now re-run the chunk above (or the entire notebook) to pull the
most recent version of the data. Thus you can periodically re-run this
notebook to check in on the pandemic as it evolves.

*Note*: You should feel free to copy-paste the code above for your own
future projects!

# Join the Data

<!-- -------------------------------------------------- -->

To get a sense of our task, let’s take a glimpse at our two data
sources.

``` r
## NOTE: No need to change this; just execute
df_pop %>% glimpse
```

    ## Rows: 3,220
    ## Columns: 4
    ## $ id                       <chr> "0500000US01001", "0500000US01003", "0500000U…
    ## $ `Geographic Area Name`   <chr> "Autauga County, Alabama", "Baldwin County, A…
    ## $ `Estimate!!Total`        <dbl> 55200, 208107, 25782, 22527, 57645, 10352, 20…
    ## $ `Margin of Error!!Total` <chr> "*****", "*****", "*****", "*****", "*****", …

``` r
df_covid %>% glimpse
```

    ## Rows: 2,502,832
    ## Columns: 6
    ## $ date   <date> 2020-01-21, 2020-01-22, 2020-01-23, 2020-01-24, 2020-01-24, 20…
    ## $ county <chr> "Snohomish", "Snohomish", "Snohomish", "Cook", "Snohomish", "Or…
    ## $ state  <chr> "Washington", "Washington", "Washington", "Illinois", "Washingt…
    ## $ fips   <chr> "53061", "53061", "53061", "17031", "53061", "06059", "17031", …
    ## $ cases  <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
    ## $ deaths <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …

To join these datasets, we’ll need to use [FIPS county
codes](https://en.wikipedia.org/wiki/FIPS_county_code).\[2\] The last
`5` digits of the `id` column in `df_pop` is the FIPS county code, while
the NYT data `df_covid` already contains the `fips`.

### **q3** Process the `id` column of `df_pop` to create a `fips` column.

``` r
## TASK: Create a `fips` column by extracting the county code
# view(df_pop)
# df_pop

df_q3 <- 
  df_pop %>%
  mutate(fips = str_extract(id, "\\d{5}$"))
df_q3
```

    ## # A tibble: 3,220 × 5
    ##    id      `Geographic Area Name` `Estimate!!Total` Margin of Error!!Tot…¹ fips 
    ##    <chr>   <chr>                              <dbl> <chr>                  <chr>
    ##  1 050000… Autauga County, Alaba…             55200 *****                  01001
    ##  2 050000… Baldwin County, Alaba…            208107 *****                  01003
    ##  3 050000… Barbour County, Alaba…             25782 *****                  01005
    ##  4 050000… Bibb County, Alabama               22527 *****                  01007
    ##  5 050000… Blount County, Alabama             57645 *****                  01009
    ##  6 050000… Bullock County, Alaba…             10352 *****                  01011
    ##  7 050000… Butler County, Alabama             20025 *****                  01013
    ##  8 050000… Calhoun County, Alaba…            115098 *****                  01015
    ##  9 050000… Chambers County, Alab…             33826 *****                  01017
    ## 10 050000… Cherokee County, Alab…             25853 *****                  01019
    ## # ℹ 3,210 more rows
    ## # ℹ abbreviated name: ¹​`Margin of Error!!Total`

Use the following test to check your answer.

``` r
## NOTE: No need to change this
## Check known county
assertthat::assert_that(
              (df_q3 %>%
              filter(str_detect(`Geographic Area Name`, "Autauga County")) %>%
              pull(fips)) == "01001"
            )
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

### **q4** Join `df_covid` with `df_q3` by the `fips` column. Use the proper type of join to preserve *only* the rows in `df_covid`.

``` r
## TASK: Join df_covid and df_q3 by fips.
# view(df_covid)
# view(df_q3)

df_q4 <- 
  df_covid %>%
  left_join( #preserve the rows in df_covid
    df_q3,
    by = "fips" 
  )
```

Use the following test to check your answer.

``` r
## NOTE: No need to change this
if (!any(df_q4 %>% pull(fips) %>% str_detect(., "02105"), na.rm = TRUE)) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Your df_q4 contains a row for the Hoonah-Angoon Census Area (AK),",
    "which is not in df_covid. You used the incorrect join type.",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
if (any(df_q4 %>% pull(fips) %>% str_detect(., "78010"), na.rm = TRUE)) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Your df_q4 does not include St. Croix, US Virgin Islands,",
    "which is in df_covid. You used the incorrect join type.",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

For convenience, I down-select some columns and produce more convenient
column names.

``` r
## NOTE: No need to change; run this to produce a more convenient tibble
df_data <-
  df_q4 %>%
  select(
    date,
    county,
    state,
    fips,
    cases,
    deaths,
    population = `Estimate!!Total`
  )

df_data
```

    ## # A tibble: 2,502,832 × 7
    ##    date       county      state      fips  cases deaths population
    ##    <date>     <chr>       <chr>      <chr> <dbl>  <dbl>      <dbl>
    ##  1 2020-01-21 Snohomish   Washington 53061     1      0     786620
    ##  2 2020-01-22 Snohomish   Washington 53061     1      0     786620
    ##  3 2020-01-23 Snohomish   Washington 53061     1      0     786620
    ##  4 2020-01-24 Cook        Illinois   17031     1      0    5223719
    ##  5 2020-01-24 Snohomish   Washington 53061     1      0     786620
    ##  6 2020-01-25 Orange      California 06059     1      0    3164182
    ##  7 2020-01-25 Cook        Illinois   17031     1      0    5223719
    ##  8 2020-01-25 Snohomish   Washington 53061     1      0     786620
    ##  9 2020-01-26 Maricopa    Arizona    04013     1      0    4253913
    ## 10 2020-01-26 Los Angeles California 06037     1      0   10098052
    ## # ℹ 2,502,822 more rows

# Analyze

<!-- -------------------------------------------------- -->

Now that we’ve done the hard work of loading and wrangling the data, we
can finally start our analysis. Our first step will be to produce county
population-normalized cases and death counts. Then we will explore the
data.

## Normalize

<!-- ------------------------- -->

### **q5** Use the `population` estimates in `df_data` to normalize `cases` and `deaths` to produce per 100,000 counts \[3\]. Store these values in the columns `cases_per100k` and `deaths_per100k`.

``` r
## TASK: Normalize cases and deaths
df_normalized <-
  df_data %>%
  mutate(cases_per100k = ((cases/population) * 100000)) %>%
  mutate(deaths_per100k = (deaths/population) * 100000)

view(df_normalized)
```

You may use the following test to check your work.

``` r
## NOTE: No need to change this
## Check known county data
if (any(df_normalized %>% pull(date) %>% str_detect(., "2020-01-21"))) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Date 2020-01-21 not found; did you download the historical data (correct),",
    "or just the most recent data (incorrect)?",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
if (any(df_normalized %>% pull(date) %>% str_detect(., "2022-05-13"))) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Date 2022-05-13 not found; did you download the historical data (correct),",
    "or a single year's data (incorrect)?",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
## Check datatypes
assertthat::assert_that(is.numeric(df_normalized$cases))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$deaths))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$population))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$cases_per100k))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$deaths_per100k))
```

    ## [1] TRUE

``` r
## Check that normalization is correct
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(cases_per100k) - 0.127) < 1e-3
            )
```

    ## [1] TRUE

``` r
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(deaths_per100k) - 0) < 1e-3
            )
```

    ## [1] TRUE

``` r
print("Excellent!")
```

    ## [1] "Excellent!"

## Guided EDA

<!-- --------------------------->

Before turning you loose, let’s complete a couple guided EDA tasks.

### **q6** Compute some summaries

Compute the mean and standard deviation for `cases_per100k` and
`deaths_per100k`. *Make sure to carefully choose **which rows** to
include in your summaries,* and justify why!

``` r
## TASK: Compute mean and sd for cases_per100k and deaths_per100k
df_normalized %>%
  
  summarize(
    mean_cases = mean(cases_per100k, na.rm = TRUE),
    mean_deaths = mean(deaths_per100k, na.rm = TRUE),
    sd_cases = sd(cases_per100k, na.rm = TRUE),
    sd_deaths = sd(deaths_per100k, na.rm = TRUE)
  )
```

    ## # A tibble: 1 × 4
    ##   mean_cases mean_deaths sd_cases sd_deaths
    ##        <dbl>       <dbl>    <dbl>     <dbl>
    ## 1      9975.        174.    8449.      159.

- Which rows did you pick?
  - All of the rows except the one that has all of the NA values.
- Why?
  - The NA values mess up the calculations and cause the calculations to
    return nothing because it can not make sense of the data.

### **q7** Find and compare the top 10

Find the top 10 counties in terms of `cases_per100k`, and the top 10 in
terms of `deaths_per100k`. Report the population of each county along
with the per-100,000 counts. Compare the counts against the mean values
you found in q6. Note any observations.

``` r
## TASK: Find the top 10 max cases_per100k counties; report populations as well
df_normalized %>%
  arrange(desc(cases_per100k)) %>%
  distinct(county, .keep_all = TRUE) %>%
  top_n(10)
```

    ## Selecting by deaths_per100k

    ## # A tibble: 10 × 9
    ##    date       county           state fips  cases deaths population cases_per100k
    ##    <date>     <chr>            <chr> <chr> <dbl>  <dbl>      <dbl>         <dbl>
    ##  1 2022-05-12 Loving           Texas 48301   196      1        102       192157.
    ##  2 2022-05-13 Galax city       Virg… 51640  2561     78       6638        38581.
    ##  3 2022-05-12 Lamb             Texas 48279  4356    122      13262        32846.
    ##  4 2022-05-09 Robertson        Kent… 21201   672     21       2143        31358.
    ##  5 2022-05-13 Martinsville ci… Virg… 51690  3478    124      13101        26548.
    ##  6 2022-04-26 McMullen         Texas 48311   169      9        662        25529.
    ##  7 2022-03-26 Motley           Texas 48345   286     13       1156        24740.
    ##  8 2022-05-10 Emporia city     Virg… 51595  1187     55       5381        22059.
    ##  9 2022-05-11 Towns            Geor… 13281  2400    116      11417        21021.
    ## 10 2022-03-18 Jerauld          Sout… 46073   415     20       2029        20453.
    ## # ℹ 1 more variable: deaths_per100k <dbl>

``` r
df_normalized %>%
  arrange(desc(deaths_per100k)) %>%
  distinct(county, .keep_all = TRUE) %>%
  top_n(10)
```

    ## Selecting by deaths_per100k

    ## # A tibble: 10 × 9
    ##    date       county           state fips  cases deaths population cases_per100k
    ##    <date>     <chr>            <chr> <chr> <dbl>  <dbl>      <dbl>         <dbl>
    ##  1 2022-02-19 McMullen         Texas 48311   166      9        662        25076.
    ##  2 2022-04-27 Galax city       Virg… 51640  2551     78       6638        38430.
    ##  3 2022-03-10 Motley           Texas 48345   271     13       1156        23443.
    ##  4 2022-04-20 Hancock          Geor… 13141  1577     90       8535        18477.
    ##  5 2022-04-19 Emporia city     Virg… 51595  1169     55       5381        21725.
    ##  6 2022-04-27 Towns            Geor… 13281  2396    116      11417        20986.
    ##  7 2022-02-14 Jerauld          Sout… 46073   404     20       2029        19911.
    ##  8 2022-03-04 Loving           Texas 48301   165      1        102       161765.
    ##  9 2022-02-03 Robertson        Kent… 21201   570     21       2143        26598.
    ## 10 2022-05-05 Martinsville ci… Virg… 51690  3452    124      13101        26349.
    ## # ℹ 1 more variable: deaths_per100k <dbl>

``` r
## TASK: Find the top 10 deaths_per100k counties; report populations as well
```

``` r
df_normalized %>%
  arrange(desc(cases_per100k)) %>%
  distinct(county, .keep_all = TRUE) %>%
  filter(row_number() <= 10,)
```

    ## # A tibble: 10 × 9
    ##    date       county           state fips  cases deaths population cases_per100k
    ##    <date>     <chr>            <chr> <chr> <dbl>  <dbl>      <dbl>         <dbl>
    ##  1 2022-05-12 Loving           Texas 48301   196      1        102       192157.
    ##  2 2022-05-11 Chattahoochee    Geor… 13053  7486     22      10767        69527.
    ##  3 2022-05-11 Nome Census Area Alas… 02180  6245      5       9925        62922.
    ##  4 2022-05-11 Northwest Arcti… Alas… 02188  4837     13       7734        62542.
    ##  5 2022-05-13 Crowley          Colo… 08025  3347     30       5630        59449.
    ##  6 2022-05-11 Bethel Census A… Alas… 02050 10362     41      18040        57439.
    ##  7 2022-03-30 Dewey            Sout… 46041  3139     42       5779        54317.
    ##  8 2022-05-12 Dimmit           Texas 48127  5760     51      10663        54019.
    ##  9 2022-05-12 Jim Hogg         Texas 48247  2648     22       5282        50133.
    ## 10 2022-05-11 Kusilvak Census… Alas… 02158  4084     14       8198        49817.
    ## # ℹ 1 more variable: deaths_per100k <dbl>

``` r
df_normalized %>%
  arrange(desc(deaths_per100k)) %>%
  distinct(county, .keep_all = TRUE) %>%
  filter(row_number() <= 10,)
```

    ## # A tibble: 10 × 9
    ##    date       county           state fips  cases deaths population cases_per100k
    ##    <date>     <chr>            <chr> <chr> <dbl>  <dbl>      <dbl>         <dbl>
    ##  1 2022-02-19 McMullen         Texas 48311   166      9        662        25076.
    ##  2 2022-04-27 Galax city       Virg… 51640  2551     78       6638        38430.
    ##  3 2022-03-10 Motley           Texas 48345   271     13       1156        23443.
    ##  4 2022-04-20 Hancock          Geor… 13141  1577     90       8535        18477.
    ##  5 2022-04-19 Emporia city     Virg… 51595  1169     55       5381        21725.
    ##  6 2022-04-27 Towns            Geor… 13281  2396    116      11417        20986.
    ##  7 2022-02-14 Jerauld          Sout… 46073   404     20       2029        19911.
    ##  8 2022-03-04 Loving           Texas 48301   165      1        102       161765.
    ##  9 2022-02-03 Robertson        Kent… 21201   570     21       2143        26598.
    ## 10 2022-05-05 Martinsville ci… Virg… 51690  3452    124      13101        26349.
    ## # ℹ 1 more variable: deaths_per100k <dbl>

**Observations**:

- Because the number of cases in Loving is bigger than the population,
  it looks really interesting in comparison to everything else because
  it stands out on top despite having a relatively small amount of
  cases. All of the values seem to be a bit larger than what I am seeing
  outputted by the table? I don’t know why that is happening.
- When did these “largest values” occur?
  - Texas, Georgia, Alaska, Colorado, South Dakota

## Self-directed EDA

<!-- ------------------------- -->

### **q8** Drive your own ship: You’ve just put together a very rich dataset; you now get to explore! Pick your own direction and generate at least one punchline figure to document an interesting finding. I give a couple tips & ideas below:

### Ideas

<!-- ------------------------- -->

- Look for outliers.
- Try web searching for news stories in some of the outlier counties.
- Investigate relationships between county population and counts.
- Do a deep-dive on counties that are important to you (e.g. where you
  or your family live).
- Fix the *geographic exceptions* noted below to study New York City.
- Your own idea!

**DO YOUR OWN ANALYSIS HERE**

``` r
# p <- df_normalized %>%
#    ggplot(aes(date, cases)) +
#    geom_line(alpha = 0.4, color = "blue")
#  
# p <- p + geom_line(aes(date, deaths), alpha = 0.4, color = "red") + scale_y_log10()
# print(p)
# coeff <- 10
# df_normalized %>%
#   ggplot(aes(x=dates)) +
#   geom_line(y=cases, alpha = 0.4, color = "blue") +
#   geom_line(y=(deaths / coeff), alpha = 0.4, color = "red") + 
#   scale_y_continuous(
#     name = "First axis",
#     sec.axis = sec_axis(~.*coeff, name="Second axis")
#   )

df_normalized %>%
  filter(
    state == "Massachusetts", # Focus on Mass only
    !is.na(fips), # fct_reorder2 can choke with missing data
  ) %>%
  
  ggplot(
    aes(date, cases_per100k, color = fct_reorder2(county, date, cases_per100k))
  ) +
  geom_line(size = 0.6) +
  scale_y_log10() +
  theme_minimal() + 
  scale_color_discrete(name = "County") +
  labs(title = "Covid Cases in Massachusetts")
```

    ## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
    ## ℹ Please use `linewidth` instead.
    ## This warning is displayed once per session.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
df_normalized %>%
  filter(
    county == "Nantucket", # Focus on Nantucket only
    !is.na(fips), # fct_reorder2 can choke with missing data
  ) %>%
  
  ggplot(
    aes(date, cases_per100k, color = fct_reorder2(county, date, cases_per100k))
  ) +
  geom_line(size = 1) +
  scale_y_log10() +
  theme_minimal() + 
  scale_color_discrete(name = "County") +
  labs(title = "Covid cases in Nantucket")
```

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-2-2.png)<!-- -->

``` r
#had the slowest rise out of all of the counties yet got to be a county with one of the highest amounts of cases
nantucket_only <- 
  df_normalized %>% 
    filter(county == "Nantucket") %>%
    arrange(desc(cases))
nantucket_only <- #nantucket_only[, -which(names(nantucket_only) == "fips")]
  nantucket_only %>%
  select(-fips, -population)
head(nantucket_only)
```

    ## # A tibble: 6 × 7
    ##   date       county    state         cases deaths cases_per100k deaths_per100k
    ##   <date>     <chr>     <chr>         <dbl>  <dbl>         <dbl>          <dbl>
    ## 1 2022-05-13 Nantucket Massachusetts  3481      0        31358.              0
    ## 2 2022-05-12 Nantucket Massachusetts  3477      0        31322.              0
    ## 3 2022-05-11 Nantucket Massachusetts  3475      0        31303.              0
    ## 4 2022-05-10 Nantucket Massachusetts  3471      0        31267.              0
    ## 5 2022-05-09 Nantucket Massachusetts  3468      0        31240.              0
    ## 6 2022-05-06 Nantucket Massachusetts  3457      0        31141.              0

``` r
###
df_normalized %>%
  filter(
    state == "Massachusetts", # Focus on Mass only
    !is.na(fips), # fct_reorder2 can choke with missing data
  ) %>%
  
  ggplot(
    aes(date, deaths_per100k, color = fct_reorder2(county, date, deaths_per100k))
  ) +
  geom_line(size = 0.6) +
  scale_y_log10() +
  theme_minimal() + 
  scale_color_discrete(name = "County") +
  labs(title = "Deaths in Nantucket")
```

    ## Warning in scale_y_log10(): log-10 transformation introduced infinite values.

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-2-3.png)<!-- -->

``` r
nantucket_only2 <-
  df_normalized %>% 
  filter(county == "Nantucket") %>%
  arrange(desc(deaths)) %>%
  select(-fips, -population)
head(nantucket_only2)
```

    ## # A tibble: 6 × 7
    ##   date       county    state         cases deaths cases_per100k deaths_per100k
    ##   <date>     <chr>     <chr>         <dbl>  <dbl>         <dbl>          <dbl>
    ## 1 2021-06-13 Nantucket Massachusetts  1519      3        13683.           27.0
    ## 2 2020-11-06 Nantucket Massachusetts   191      2         1721.           18.0
    ## 3 2020-11-07 Nantucket Massachusetts   194      2         1748.           18.0
    ## 4 2020-11-08 Nantucket Massachusetts   202      2         1820.           18.0
    ## 5 2020-11-09 Nantucket Massachusetts   202      2         1820.           18.0
    ## 6 2020-11-10 Nantucket Massachusetts   212      2         1910.           18.0

Interesting findings:

- I looked into what was happening in Nantucket in July of 2020 to see
  if there was anything of note. According to an NBC article (<a
  href="https://www.nbcboston.com/news/local/nantucket-sees-increase-of-coronavirus-cases-with-arrival-of-summer-crowd/2165713/"
  class="uri">source</a>), there was a surge in Covid cases due to the
  summer months and people migrating to the island for vacation.
  Nantucket is already a small island with a small population with
  around 10,000 residents in 2022 (which corresponds to the year with
  highest amounts of covid cases). The slow increase in cases compared
  to all other counties was probably due to there not being many people
  coming onto the island and them being relatively isolated, which
  abruptly changed when people from the mainland came to the island
  during the summer months. Due to covid’s burst nature of spreading and
  the small population of Nantucket, it makes sense that it was one of
  the highest counties by 2022 in terms of cases because there are
  already fewer people to get sick in the first place and each person
  can spread it to a ton of others.

- The deaths graph is interesting because Nantucket has a really weird
  looking graph just because there were so few deaths at all it does not
  form a smooth line and instead looks really jagged. One potential
  cause I can think of for this is the fact that Nantucket is the
  wealthiest county in Massachusetts so it’s possible the lower death
  rates was due to people in Nantucket having better access to
  healthcare and treatment options.

### Aside: Some visualization tricks

<!-- ------------------------- -->

These data get a little busy, so it’s helpful to know a few `ggplot`
tricks to help with the visualization. Here’s an example focused on
Massachusetts.

``` r
# ## NOTE: No need to change this; just an example
# df_normalized %>%
#   filter(
#     state == "Massachusetts", # Focus on Mass only
#     !is.na(fips), # fct_reorder2 can choke with missing data
#   ) %>%
# 
#   ggplot(
#     aes(date, cases_per100k, color = fct_reorder2(county, date, cases_per100k))
#   ) +
#   geom_line() +
#   scale_y_log10(labels = scales::label_number_si()) +
#   scale_color_discrete(name = "County") +
#   theme_minimal() +
#   labs(
#     x = "Date",
#     y = "Cases (per 100,000 persons)"
#   )
```

*Tricks*:

- I use `fct_reorder2` to *re-order* the color labels such that the
  color in the legend on the right is ordered the same as the vertical
  order of rightmost points on the curves. This makes it easier to
  reference the legend.
- I manually set the `name` of the color scale in order to avoid
  reporting the `fct_reorder2` call.
- I use `scales::label_number_si` to make the vertical labels more
  readable.
- I use `theme_minimal()` to clean up the theme a bit.
- I use `labs()` to give manual labels.

### Geographic exceptions

<!-- ------------------------- -->

The NYT repo documents some [geographic
exceptions](https://github.com/nytimes/covid-19-data#geographic-exceptions);
the data for New York, Kings, Queens, Bronx and Richmond counties are
consolidated under “New York City” *without* a fips code. Thus the
normalized counts in `df_normalized` are `NA`. To fix this, you would
need to merge the population data from the New York City counties, and
manually normalize the data.

# Notes

<!-- -------------------------------------------------- -->

\[1\] The census used to have many, many questions, but the ACS was
created in 2010 to remove some questions and shorten the census. You can
learn more in [this wonderful visual
history](https://pudding.cool/2020/03/census-history/) of the census.

\[2\] FIPS stands for [Federal Information Processing
Standards](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standards);
these are computer standards issued by NIST for things such as
government data.

\[3\] Demographers often report statistics not in percentages (per 100
people), but rather in per 100,000 persons. This is [not always the
case](https://stats.stackexchange.com/questions/12810/why-do-demographers-give-rates-per-100-000-people)
though!
