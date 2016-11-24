---
layout: post
title: "Exploring the EPL since 2001"
---

* TOC
{:toc}

## Introduction

This is a weekend project, that will evolve a bit more with an R shiny app in
near future. As of today September 9th 2016 we are just 3 weeks in the new season,
so I decided to look into some historic data (from 2001), and observe how the league
has changed since. Back in 2001 teams like Leicester and Swansea were deep in the
English football ranks but times have changed. I also hope to see some of the big
money movements that have since occurred (Chelsea and Man City).

I will be using a set of R function I wrote that scrape espnfc.com to get my stats.
Note that these are under development and can fail frequently.

What the league looks like as of today,

~~~r
epl_now <- getCurrentLeagueStandings() # get latest league standings in dataframe
kable(epl_now)
~~~

| position|team                 | played| points|
|--------:|:--------------------|------:|------:|
|        1|Manchester City      |      3|      9|
|        2|Chelsea              |      3|      9|
|        3|Manchester United    |      3|      9|
|        4|Everton              |      3|      7|
|        5|Hull City            |      3|      6|
|        6|Middlesbrough        |      3|      5|
|        7|Tottenham Hotspur    |      3|      5|
|        8|Arsenal              |      3|      4|
|        9|Leicester City       |      3|      4|
|       10|West Bromwich Albion |      3|      4|
|       11|Liverpool            |      3|      4|
|       12|West Ham United      |      3|      3|
|       13|Burnley              |      3|      3|
|       14|Swansea City         |      3|      3|
|       15|Southampton          |      3|      2|
|       16|Sunderland           |      3|      1|
|       17|Crystal Palace       |      3|      1|
|       18|Watford              |      3|      1|
|       19|AFC Bournemouth      |      3|      1|
|       20|Stoke City           |      3|      1|


## Point Distributions

What is most interesting to me is the distribution of points. One would expect to
see some symmetry, few teams get a lot points, and a few get barely any, but the majority
of teams should be somewhere in the middle. We will use density estimation to see
the way the points are distributed,

<iframe width="100%" height="600px" frameborder="0" seamless="seamless" scrolling="no" src="/assets/post_assets/epl-transformations/epl_years_density_plotly.html"> </iframe>

The above is an interactive density estimation of the end of year points. Click
on league years to show and hide corresponding distribution.

![EPL All points Distribution](/assets/post_assets/epl-transformations/epl_all_point_density.png)

Each of the colored distributions rarely diverge too much from the parent 2001-2015
distribution above. The dashed lines indicate the average minimum for relegation and
European Championship positions.

## Relegation Positions

The EPL relegates the bottom three teams (position 18, 19, 20). Here is a list
of all the teams that have been relegated since 2001, some multiple times.

|Team                    | Number of Times Relegated| Average Points in EPL|
|:-----------------------|-----:|----------:|
|Birmingham City         |     3|   36.00000|
|Norwich City            |     3|   33.33333|
|West Bromwich Albion    |     3|   29.33333|
|Burnley                 |     2|   31.50000|
|Derby County            |     2|   20.50000|
|Hull City               |     2|   32.50000|
|Leicester City          |     2|   30.50000|
|Newcastle United        |     2|   35.50000|
|Queens Park Rangers     |     2|   27.50000|
|Reading                 |     2|   32.00000|
|Sunderland              |     2|   17.00000|
|West Ham United         |     2|   37.50000|
|Wolverhampton Wanderers |     2|   29.00000|
|Aston Villa             |     1|   17.00000|
|Blackburn Rovers        |     1|   31.00000|
|Blackpool               |     1|   39.00000|
|Bolton Wanderers        |     1|   36.00000|
|Cardiff City            |     1|   30.00000|
|Charlton Athletic       |     1|   34.00000|
|Crystal Palace          |     1|   33.00000|
|Fulham                  |     1|   32.00000|
|Ipswich Town            |     1|   36.00000|
|Leeds United            |     1|   33.00000|
|Middlesbrough           |     1|   32.00000|
|Portsmouth              |     1|   19.00000|
|Sheffield United        |     1|   38.00000|
|Southampton             |     1|   32.00000|
|Watford                 |     1|   28.00000|
|Wigan Athletic          |     1|   36.00000|


## European League positions

The EPL qualifies the top 4 teams to the Champions League, things are much less
hectic here.

|team              | Number of Times Qualified| Average Points in EPL| Best Position|
|:-----------------|-----:|----------:|-------------:|
|Arsenal           |    15|   75.93333|             1|
|Manchester United |    13|   82.61538|             1|
|Chelsea           |    12|   82.00000|             1|
|Liverpool         |     7|   76.57143|             2|
|Manchester City   |     6|   78.16667|             1|
|Tottenham Hotspur |     3|   69.66667|             3|
|Newcastle United  |     2|   70.00000|             3|
|Everton           |     1|   61.00000|             4|
|Leicester City    |     1|   81.00000|             1|
