---
layout: post
title: Enter the Tidyverse | Getting back into R through some basic data cleaning
excerpt_separator:  <!--more-->
---
So late last week, I dipped my toes into the Tidyverse ocean for the first time. If you've never heard of the <a href="https://www.Tidyverse.org/">Tidyverse</a>, let me quickly get you up to speed: it's a collection of packages designed for data science within the R ecosystem (if you don't know what R is, <a href="https://en.wikipedia.org/wiki/R_(programming_language)">it's a programming language heavily orientated towards statistical computing</a>). 

![Enter the Tidyverse](/assets/images/enter-the-tidyverse.png)

I've had a very love-hate relationship with R over the years. Generally, I loved what one could do with it, but I lacked the patience to get my head of the seemingly steep learning curve (<a href="http://r4stats.com/articles/why-r-is-hard-to-learn/">this post by Robert A Muenchen sums up many of the main issues I also encountered with R!</a>). Beyond the learning curve, the love-hate relationship manifested in two critical ways. 

<ul>
<li>Firstly, I often encountered frustrating issues getting R and R Studio to run correctly. I don't know how many times I stopped using it after hitting error messages when merely trying to <i>install</i> basic packages.</li>
<li>Secondly, when I did get it going, I wasn't using it for things that benefitted me often, but using it to practice machine learning techniques (like Bayesian classifications and regressions). These are all well and good, but it's not like I use these functions frequently. I can count the number of regressions I've done in the last seven years on the one hand.</li>
</ul>

For a long time, R wasn't something I needed to do my work. But things have changed over the last year as I find myself more and more engaged in doing a lot of data-science adjacent work. Being efficient and consistent when cleansing data suddenly is a crucial thing. 

I frequently require to cleanse data. Typically, I rely on tools like Excel, PowerQuery and Tableau Prep to do so. As any data person knows, data cleansing is the most time-consuming part of the job - but it's also incredibly crucial to get right. It's also sometimes mind-numbing and prone to errors. Anything that can reduce the time to accurately clean datasets is a desirable proposition.

Lately, I've been working with more extensive and more complex datasets and was getting frustrated by PowerQuery and Tableau Prep's performance. While I could set up cleansing workflows on these platforms quickly enough, I encountered issues when working with data with many millions of rows. Frequently, these products would demand colossal system resources and often crash. 

After joining a Slack channel with many enthusiastic R users (shoutout to runapp), I finally decided to give it a go again - this time focusing on using Tidyverse to do a bit of essential dataset cleansing. And now I'm thrilled I took the time to experiment. 

<h2>Cleansing some population data</h2>

To give you a quick example of how I've started my Tidyverse journey. I'm doing some work using some huge United Nations population text files. These data contain historical records of every country's population from 1950 to the present day plus forecasts up to the year 2100. For each of these years, the data is separated into individual age brackets. For example, I can use these data to see the projected number of 18 years old in Nigeria in 2100 if I wanted. 

So basically, these data contained [150 years] * [100 age groups] * [All countries plus some country groupings]. The result? 7,000,000 observation in two text files totalling over 500 megabytes. These files are <i>enormous</i>. My job was to join these files together, remove the unneeded rows and columns, aggregate the data into a specific age group (separate from the UN's usual groupings) and then output the data in a new file. 

While PowerQuery and Tableau Prep can process these files, the cleaning flow took a long time to design, an exceptionally long time to run and frequently crashed my reasonably high-end PC. These problems evaporated once I got my head around the essential cleansing functions available in R and Tidyverse. 

Here's some example code that quickly allows me to load two large text files, remove unneeded columns and then merge them. 

```
library(Tidyverse)
library(readr)
library(countrycode)

# Read in population data covering the years 2020 through 2050, skipping unneeded columns
pop_1950_2019 <- read_csv("PopulationFile1_1950_2019.csv", 
                            col_types = cols(LocID = col_skip(),
                                             AgeGrpSpan = col_skip(), 
                                             AgeGrpStart = col_skip(), MidPeriod = col_skip(), 
                                             PopFemale = col_skip(), PopMale = col_skip(), 
                                             VarID = col_skip(), Variant = col_skip()))

# Read in population data covering the years 2020 through 2050, skipping unneeded columns
pop_2020_2100 <- read_csv("PopulationFile2_2020_2100.csv", 
                             col_types = cols(LocID = col_skip(),
                                              AgeGrpSpan = col_skip(), 
                                              AgeGrpStart = col_skip(), MidPeriod = col_skip(), 
                                              PopFemale = col_skip(), PopMale = col_skip(), 
                                              VarID = col_skip(), Variant = col_skip()))

# Now bind these two dataframes together to form one big dataset with all population observations

pop_1950_2050 <- bind_rows(pop_2020_2100, pop_1950_2019)
```

The above code uses of the <b>readr</b> package to import the files and then <b>dplyr</b> to do the merging. Both these packages live in the Tidyverse

My next task was to create some custom age groupings by aggregating the data and drop all data outside the year range of 2000 to 2050. Boom, one line of code!

{% highlight r %}
# Filter out the unneeded years and age groups.  We only need the years 2000-2050 and age groups 20 to 41.
pop_2020_2050_20_41_yr_old <- pop_1950_2050 
  %>% filter(Time >= 2000 & Time <= 2050 & AgeGrp>= 20 & AgeGrp <= 41)
{% endhighlight %}

Next, I wanted to aggregate all my data into a new age group category "20 to 41" and summarise it by country and year and then output it to a new csv file for use in another application. Again, R/Tidy makes this easy. 

{% highlight r %}
# Aggregate the age group data by location (country) and time (year)
output_pop_2020_2050_20_41_yr_old <- group_by(pop_2020_2050_20_41_yr_old, Location, Time ) 
  %>% summarise(pop_2020_2050_20_41_yr_old= sum(PopTotal))
  write_excel_csv(output_pop_2020_2050_20_41_yr_old, "output_pop_2020_2050_20_41_yr_old.csv")
{% endhighlight %}

And viola! Basic data cleansing done. Now on the press of a button (assuming the source data structure does not change), I can get my data into the correct shape in seconds and even faster than the time it would take me to load up the source datasets in excel/powerquery/Tableau Prep and do the exact same thing. 

The above example is basic and scratching the surface of what is possible with R and Tidyverse but it's incredibly useful and saves hours/weeks of work over a long period. I suspect I'll continue to play in the Tidyverse space for some time going forward and hopefully I'll find the time to document some of the journey here. 
