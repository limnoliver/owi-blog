---
author: Lindsay R Carr
date: 2018-08-03
slug: beyond-basic-plotting
draft: True
title: Beyond Basic R - Plotting with ggplot2 and Custom Themes
type: post
categories: Data Science
image: static/beyond-basic-plotting/cowplotmulti-1.png
author_twitter: LindsayRCarr
author_github: lindsaycarr
 
 
author_staff: lindsay-r-carr
author_email: <lcarr@usgs.gov>

tags: 
  - R
  - Beyond Basic R
 
description: Resources for plotting, plus short examples for using ggplot2 for common use-cases and adding USGS style.
keywords:
  - R
  - Beyond Basic R
 
  - ggplot2
 
---
R can create almost any plot imaginable and as with most things in R if you don’t know where to start, try Google. The Introduction to R curriculum summarizes some of the most used plots, but cannot begin to expose people to the breadth of plot options that exist.There are existing resources that are great references for plotting in R:

In base R:

-   [Breakdown of how to create a plot](https://www.r-bloggers.com/how-to-plot-a-graph-in-r/) from R-bloggers
-   [Another blog breaking down basic plotting](https://flowingdata.com/2012/12/17/getting-started-with-charts-in-r/) from FlowingData
-   [Basic plots](https://www.cyclismo.org/tutorial/R/plotting.html) (histograms, boxplots, scatter plots, QQ plots) from University of Georgia
-   [Intermediate plots](https://www.cyclismo.org/tutorial/R/intermediatePlotting.html) (error bars, density plots, bar charts, multiple windows, saving to a file, etc) from University of Georgia

In ggplot2:

-   [ggplot2 homepage](http://ggplot2.tidyverse.org/)
-   [ggplot2 video tutorial](https://www.youtube.com/watch?v=rsG-GgR0aEY)
-   [Website with everything you want to know about ggplot2](http://r-statistics.co/Complete-Ggplot2-Tutorial-Part1-With-R-Code.html) by Selva Prabhakaran
-   [R graphics cookbook site](http://www.cookbook-r.com/Graphs/)
-   [ggplot2 cheatsheet](https://www.rstudio.com/wp-content/uploads/2015/03/ggplot2-cheatsheet.pdf)
-   [ggplot2 reference guide](http://ggplot2.tidyverse.org/reference/)

In the [Introduction to R](https://owi.usgs.gov/R/training-curriculum/intro-curriculum) class, we have switched to teaching ggplot2 because it works nicely with other tidyverse packages (dplyr, tidyr), and can create interesting and powerful graphics with little code. The following plotting examples will go through some additional features of ggplot2, and how to apply them.

Custom theme
------------

Producing a plot to look at your data is fairly straightforward; however, when you are ready to produce the plot for a publication or to share in some way, there is a lot more work. One convenience with ggplot2 are the use of built-in themes which can change plot aesthetics. Theme elements can be manually manipulated so that you can produce the perfect plot, but it does take a lot of work and Googling to figure out. Luckily for us, developers at the Lower Mississippi-Gulf Water Science Center shared their custom ggplot2 theme for USGS style guidelines. This does not guarantee perfect USGS style, but gets you pretty close and gives you the ability to edit, update, or amend specific pieces for your needs.

``` r
# custom USGS theme for plots
theme_USGS <-  function(base_size = 8){
  theme(
    plot.title = element_text (vjust = 3, size = 9,family="serif"), 
    panel.border = element_rect (colour = "black", fill = F, size = 0.1),
    panel.grid.major = element_blank (),
    panel.grid.minor = element_blank (),
    panel.background = element_rect (fill = "white"),
    legend.background = element_blank(),
    legend.justification=c(0, 0),
    legend.position = c(0.1, 0.7),
    legend.key = element_blank (),
    legend.title = element_blank (),
    legend.text = element_text (size = 8),
    axis.title.x = element_text (size = 9, family="serif"),
    axis.title.y = element_text (vjust = 1, angle = 90, size = 9, family="serif"),
    axis.text.x = element_text (size = 8, vjust = -0.25, colour = "black", 
                                family="serif", margin=margin(10,5,20,5,"pt")),
    axis.text.y = element_text (size = 8, hjust = 1, colour = "black", 
                                family="serif", margin=margin(5,10,10,5,"pt")),
    axis.ticks = element_line (colour = "black", size = 0.1),
    axis.ticks.length = unit(-0.25 , "cm")
  )
}
```

Here is an example of how to use that function (make sure you actually source the file or run the function’s code to have it available in your environment):

``` r
library(ggplot2)
library(dataRetrieval)
```

``` r
# load theme_USGS() from a file in your current working directory
source('theme_USGS.R')
```

``` r
wi_daily_q <- readNWISdv(siteNumbers = "03339500", parameterCd = "00060",
                         startDate = "2018-04-01", endDate = "2018-04-30")
wi_daily_q <- renameNWISColumns(wi_daily_q)

usgs_plot <- ggplot(wi_daily_q, aes(x=Date, y=Flow)) + 
  geom_point() + 
  theme_USGS()
usgs_plot
```

<img src='/static/beyond-basic-plotting/use_theme_plot-1.png'/ title='Simple flow timeseries for site 03339500.' alt='Hydrograph produced by ggplot2 with USGS-style.' class=''/>

Using `cowplot` to create multiple plots in one image
-----------------------------------------------------

There are a few other tricks with ggplot2 that can make it easy to get the plots you want. There’s a helper package called `cowplot` that has some nice wrapper functions for ggplot2 plots to have shared legends, put plots into a grid, annotate plots, and more. Below is some code that shows how to use some of these helpful `cowplot` functions to create a figure that has three plots and a shared title.

``` r
library(dataRetrieval)
library(dplyr) # for `rename`
library(tidyr) # for `gather`
library(ggplot2)
library(cowplot)
```

``` r
# load theme_USGS() from a file in your current working directory
source('theme_USGS.R')
```

``` r
wi_daily_wq <- readNWISdv(siteNumbers = "05430175", 
                          parameterCd = c("00060", "00530", "00631"),
                          startDate = "2017-08-01", endDate = "2017-08-31")
wi_daily_wq <- renameNWISColumns(wi_daily_wq)
wi_daily_wq <- rename(wi_daily_wq, TSS = `X_00530`, InorganicN = `X_00631`)

flow_timeseries <- ggplot(wi_daily_wq, aes(x=Date, y=Flow)) + 
  geom_point() + theme_USGS()

wi_daily_wq_long <- gather(wi_daily_wq, Nutrient, Nutrient_va, TSS, InorganicN)
nutrient_boxplot <- ggplot(wi_daily_wq_long, aes(x=Nutrient, y=Nutrient_va)) +
  geom_boxplot() + theme_USGS()

tss_flow_plot <- ggplot(wi_daily_wq, aes(x=Flow, y=TSS)) + 
  geom_point() + theme_USGS()

# Create Flow timeseries plot that spans the grid by making one plot_grid
#   and then nest it inside of a second. Also, include a title at the top 
#   for the whole figure. 
title <- ggdraw() + draw_label("Conditions for site 05430175", fontface='bold')
bottom_row <- plot_grid(nutrient_boxplot, tss_flow_plot, ncol = 2, labels = "AUTO")
plot_grid(title, bottom_row, flow_timeseries, nrow = 3, labels = c("", "", "C"),
          rel_heights = c(0.2, 1, 1))
```

<img src='/static/beyond-basic-plotting/cowplotmulti-1.png'/ title='Multi-plot figure generated using cowplot.' alt='Three plots in one figure: boxplot of inorganic N & TSS, TSS vs flow, and hydrograph.' class=''/>

Grouped boxplots in `ggplot2`
-----------------------------

The final plotting example we will show is how to create grouped boxplots, which has been a common question. This is one of those instances where factors are useful because the first step is to make your grouping variable a factor. Once it is a factor, ggplot2 will automatically understand that it should treat those as groups. Then, you can easily create grouped boxplots by setting the x aesthetic to the grouping variable column. The example below also shows off the function `case_when` from dplyr to create a new categorical column based on the values of another variable (in this case, using time to say whether the data was from the daytime or nighttime). Running this code assumes that you already have `theme_USGS()` in your environment.

``` r
library(dataRetrieval)
library(dplyr) # for `mutate` and `case_when`
library(ggplot2)

temp_q_data <- readNWISuv(siteNumbers = c("04026561", "04063700", "04082400", "05427927"),
                          parameterCd = c('00060', '00010'), 
                          startDate = "2018-06-01", endDate = "2018-06-03")
temp_q_data <- renameNWISColumns(temp_q_data)

# add an hour of day to create groups (daytime or nighttime)
temp_q_data <- temp_q_data %>% 
  mutate(site_no = factor(site_no)) %>% # grouping var should be a factor
  mutate(hourOfDay = as.numeric(format(dateTime, "%H"))) %>% 
  mutate(timeOfDay = case_when(
    hourOfDay < 20 & hourOfDay > 6 ~ "daytime",
    TRUE ~ "nighttime" # catchall for anything that doesn't fit above
  ))

# grouped boxplot
ggplot(temp_q_data, aes(x=site_no, y=Wtemp_Inst, fill=timeOfDay)) +
  geom_boxplot() +
  theme_USGS()
```

<img src='/static/beyond-basic-plotting/grouped_barplots-1.png'/ title='Grouped boxplot produced by ggplot2 with USGS style.' alt='Boxplots of water temperature for day and night grouped by USGS sites.' class=''/>
