Fun with R: A look at the Dutch voter
================

- [CBS and Voting data](#cbs-and-voting-data)
  - [Demographics data](#demographics-data)
  - [An explanation of the columns](#an-explanation-of-the-columns)
  - [Voting data](#voting-data)
- [Simple visualizations](#simple-visualizations)
  - [A first scatter plots](#a-first-scatter-plots)
  - [A prettier plot](#a-prettier-plot)
  - [Combining multiple data sources in a
    plot](#combining-multiple-data-sources-in-a-plot)
- [Plotting maps](#plotting-maps)
  - [Shape files](#shape-files)
  - [Combining shapes and demographic
    information](#combining-shapes-and-demographic-information)
- [Simple statistics](#simple-statistics)
  - [Correlation](#correlation)
  - [Regression](#regression)

<style>
.Info {
    background-color: rgb(204, 229, 255);
    border: 1px solid rgb(184, 218, 255);
    color: rgb(0, 64, 133);
    padding: 1em;
    margin: 1em;
}
.Info::before {
  font-weight: bold;
  content: "🛈 Information";
}
&#10;</style>

## CBS and Voting data

For this tutorial, we will use demographic data from CBS (the Dutch
statistics bureau) and voting results from the election board. Both of
these can be downloaded from the respective sources, but to make it easy
we use cleaned up versions from this repository.

Note: As you can see below, R can read `csv` files (a common format that
can also be imported or exported from e.g. Excel or SPSS) directly from
the internet!

### Demographics data

After running the code below with the `► Run Code` button, you will see
a *data frame* on the screen. Data frames are the main data object in R.

Generally, each row represents a unit of analysis or measurement (e.g. a
case or respondent), and each column represents a quantity of
information about that row, e.g. a measured or computed variable.

In this case, each row represents a Dutch municipality (*gemeente*),
with the columns being a selection of demographic information collected
from [CBS regional
figures](https://opendata.cbs.nl/statline/#/CBS/nl/dataset/70072ned/table?ts=1691399765879):

``` r
library(tidyverse)
demographics <- read_csv('https://raw.githubusercontent.com/vanatteveldt/ccslearnr/master/data/dutch_demographics.csv')
head(demographics)
```

Can you guess what the columns mean? Click ‘continue’ to get an
explanation of each column.

<div class="Info">

Note that you cannot break anything in these example code boxes!
Everytime you hit the ‘Run code’ button, it will start over from
scratch. Moreover, if you feel that you have messed up the code, you can
press the `↻ Start Over` button and the original code will run.

</div>

### An explanation of the columns

| Variable        | Explanation                                                                  |
|-----------------|------------------------------------------------------------------------------|
| gm              | Unique code for each municipality                                            |
| gemeente        | Unique name of each municipality                                             |
| v01_pop         | Total population                                                             |
| v57_density     | Population Density                                                           |
| v43_nl          | Percentage of population without a migration history                         |
| v122_disposable | Average disposable income per household                                      |
| v132_income     | Average standardized income per household                                    |
| v142_wealth     | Median household wealth                                                      |
| v153_uitkering  | Percentage of population receiving social benefits (excluding state pension) |
| c_65plus        | Percentage of population older than 65                                       |

### Voting data

Similar to above, we can load voting data per municipality. The code
below will load data for each party in each municipality in the 2023
provincial elections.

R also has powerful commands to clean up and restructure data. For
example, the code below shows the results for a single municipality
(Groningen), ordered by vote share. What happens if you remove the
`desc(..)` function from arrange, i.e. change `arrange(desc(votes))`
into `arrange(votes)`?

**Exercise**: Can you change the code to show the results for the
municipality of Amsterdam, ordered by party (starting from A)?

``` r
library(tidyverse)
results <- read_csv('https://raw.githubusercontent.com/vanatteveldt/ccslearnr/master/data/dutch_elections_2023ps.csv')
results |> 
  filter(gemeente == "Groningen") |> 
  arrange(desc(votes))
```

<div class="Info">

The code box above is an exercise with a single correct solution. You
need to change some code and click the `☑️Submit Answer` button to
submit your answer. Don’t worry about making mistakes, you can try as
often as you want and start over if needed. Some exercises (like this
one) also contains `💡 Hints` which you can click if you are stuck.
Also, if you submit an incorrect answer, R will do it’s best to pinpoint
where the problem is (but of course it’s only a computer, so it’s not
always very helpful…)

</div>

## Simple visualizations

Besides the ability to read and clean up data, R has a very powerful
visualization suite called `ggplot`. Although this can be hard to
master, there are many useful resources to learn more and it can be very
rewarding (and fun!) to make nice visualizations.

For this section, we will use the same data as above:

``` r
library(tidyverse)
results <- read_csv('https://raw.githubusercontent.com/vanatteveldt/ccslearnr/master/data/dutch_elections_2023ps.csv')
demographics <- read_csv('https://raw.githubusercontent.com/vanatteveldt/ccslearnr/master/data/dutch_demographics.csv')
```

### A first scatter plots

Let’s make a first minimal scatter plot. A scatter plot can be very
useful to see the relation between variables, for example between
population density and age: do older people live mostly in cities or in
more rural regions?

``` r
ggplot(demographics, 
       aes(x=v57_density, y=c_65plus)) + 
  geom_point()
```

So, it turns out that older people mostly live in less dense
municipalities. Have a quick look at the code above. The ggplot function
takes two arguments, the data (`demographics`) and an *aesthetic
mapping* (`aes(..)`). This mapping links columns in the the data frame
(e.g. `v57_density`) to graphical elements of the plot (e.g. the x
position of the scratter points). Finally, you add geometrical elements
to the plot, in this case a `geom_point` or scatter point.

What happens when you replace variables in the mapping with other
variables, e.g. `v43_nl` or `v132_income`? You can also try adding
another aesthetic, for example `fill=v43_nl` or `size=v01_pop`.

### A prettier plot

The plot above was informative, but not very pretty. The example below
shows a much richer plot command. It may look scary at first, but if you
look at the commands they make more sense than you would think. It also
shows quite well how ggplot works: You first construct a basic plot
consisting of data, aesthetics, and geoms. This generally produces a
decent plot, as ggplot is quite good at guessing what good scales etc.
are based on your data. However, you can add many optional elements to
the plot to change the appearance, from changing the color bars to
repositioning the legend. Take a look at the code below:

<div class="Info">

Note: the code box below contains multiple *comments*. Every line
starting with a `#` is called a comment, and is simply ignored by R so
can be used to explain your code.

</div>

``` r
# Basic ggplot 
#    - data (demographics)
#    - a mapping of density (y) against income (x), population size (dot size), and migration history (color),
#    - and a geom (in this case geom_point to create a scatter plot)
ggplot(demographics, 
       aes(x=v57_density, y=v132_income, size=v01_pop, color=100-v43_nl)) + 
  geom_point(alpha=.7) + 
# All optional elements to change appearance
  # Add nice titles for the plot and axes
  ggtitle("Differences between rural and urban municipalities") +
  xlab("Population density") + 
  ylab("% of people older than 65") +  
  # Change the scale for migration history to go from light to dark blue:
  scale_color_gradient(low="lightblue", high="darkblue") +
  # Use the scale command to tweak the legend;
  scale_size(breaks = c(5000, 50000, 500000), 
             labels = c("5.000", "50.000", "500.000")) + 
  # Use the guide command to specify titles and direction for the legend
  guides(size=guide_legend(title="Population", title.position="top", 
                           override.aes=list(color='darkblue')),
         color=guide_colorbar(title="% with Migration History",
                              title.position="top", 
                              direction = "horizontal")) + 
  # Use the 'classic' theme
  theme_classic() +
  # Tweak the theme to drop the background for legend keys
  theme(legend.key=element_blank()) 
```

In the code above, almost all lines are optional. What happens if you
remove e.g. the theme or guide information? And can you change the
colors used in the color scale?

<div class="Info">

In ggplot, many aspects of the appearance are controlled by *themes*,
that specify things like fonts, colors, and grid lines. Note that You
can also use other themes, for example `theme_linedraw` or `theme_void`.
You can also install new themes. For example, the `ggthemes` packages
contains a number of themes that mimic existing sources. Try using
`ggthemes::theme_economist` or `ggthemes::theme_tufte`.

</div>

### Combining multiple data sources in a plot

Often, we need to combine data from multiple sources. For example, it
could be interesting to combine the election results in `results` with
the demographic data. Fortunately, this is quite easy because they have
a common key variable (`gm`) on which to join them.

So, let’s see if we can plot the support for the farmer’s party BBB
against the population density:

``` r
bbb <- filter(results, party == "BBB")
data <- inner_join(bbb, demographics)
ggplot(data, aes(x=log10(v57_density), y=votes, color=party, size=v01_pop)) + 
  geom_point(alpha=.5) + 
  xlab("Population density (log scale)") + ylab("Relative support for party") +
  scale_color_manual(values=c("BBB"="green", "Anti-immigration"="blue"))+  scale_size(guide="none") + 
  ggtitle("Support for BBB and anti-immigration parties per municipality",
          "(Dutch 2023 provincial elections; note: size of point relative to logged municipality population)") + 
  theme_classic() + 
  theme(legend.position = "top", legend.title = element_blank()) 
```

## Plotting maps

Finally, to look at regional or national differences, it can be very
insightful to plot variables on a map.

### Shape files

To do this, R first needs to know the *shape* of the various regions
(such as the municipalities used above). We can download a cleaned up
Dutch ‘shapefile’ from the tutorial repository:

``` r
library(tidyverse)
library(sf)
shapes = read_rds("https://github.com/vanatteveldt/ccslearnr/raw/master/data/sf_nl.rds")
head(shapes)
```

Now, we can plot this map using a regular ggplot command, using the
`geom_sg` geometrical object, which can directly use the `geom` column
from the shapefile. We can also specify the fill color of regions, for
example using the province column:

``` r
ggplot(shapes) + geom_sf(aes(geometry=geom, fill=provincie))
```

### Combining shapes and demographic information

In the example above, we used only the spatial information to plot a map
of Dutch municipalities. Of course, it becomes much more interesting
(and relevant!) if we can plot sociologically relevant information.

For this end, we need to `join` the spatial and demographic information,
and then we can use other columns from the combined data set to color
the regions. For example, the code below plots the population density
per municipality:

``` r
inner_join(shapes, demographics) |> 
  ggplot() + 
  geom_sf(aes(geometry=geom, fill=v57_density)) + 
  scale_fill_gradient(low="white", high="red", guide="none") + 
  ggtitle("Population density per municipality") + 
  theme_void()
```

We could compare this with the vote share of e.g. the (farmer’s) party
BBB:

``` r
bbb = results |> filter(party == "BBB")  
inner_join(shapes, bbb) |> ggplot() + geom_sf(aes(geometry=geom, fill=votes)) +
  ggtitle("Support for BBB per municipality") +
  scale_fill_gradient(low="white", high="green",name='% Support') +
  theme_void() + theme(legend.position = "bottom")
```

What does this tell us about the target constituency of the BBB party?
Can you plot the same maps for other parties or variables? Can you
change the color scheme?

## Simple statistics

Of course, R is also (and even at it’s core) a statistical toolkit. In
this sub section, we will look at some simple statistical results.

### Correlation

First, let’s see how the demographics are correlated. The command `cor`
gives the bivariate correlations of all variables, quite useful for
quick exploration:

``` r
vars <- select(demographics, v01_pop:c_65plus)
cor(vars, use = "pairwise")
```

We can also plot this using the `ggcorrplot` package:

``` r
library(ggcorrplot)
vars <- select(demographics, v01_pop:c_65plus)
correlations <- cor(vars, use = "pairwise")
ggcorrplot(correlations, method="circle")
```

### Regression

Let’s see if we can predict/explain support for BBB from the demographic
information. For this, we combine the demographic and vote data as
above, and use the `lm` function to compute the linear regression model.
Finally, we use the `tab_model` function from the `sjPlot` package to
create a nice looking regression table:

``` r
library(sjPlot)
bbb = filter(results, party == "BBB")
data = inner_join(bbb, demographics)
m = lm(data, formula=votes ~ v57_density + c_65plus)
tab_model(m)
```

So, BBB’s vote share can be predicted from population density (the
denser the muncipality, the fewer votes), but not by the percentage of
people over 65.

**Exercise**: Change the code above predict the VVD’s vote share from
`c65_plus`, `v43_nl`, and `v132_income`. How do you interpret this? Do
the coefficients surprise you?

(Note: Be sure to rename the `bbb` data to `vvd` to avoid confusing R’s
code checking)
