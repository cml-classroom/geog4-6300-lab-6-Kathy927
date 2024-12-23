Geog6300: Lab 6
================

## Regression

``` r
library(sf)
library(tidyverse)
library(tmap)
library(car)
library(lmtest)
```

**Overview:** This lab focuses on regression techniques. You’ll be
analyzing the association of various physical and climatological
characteristics in Australia with observations of several animals
recorded on the citizen science app iNaturalist.

\###Data and research questions###

Let’s import the dataset.

``` r
lab6_data<-st_read("data/aus_climate_inat.gpkg")
```

    ## Reading layer `aus_climate_inat' from data source 
    ##   `C:\Users\oseik\OneDrive - University of Georgia\Desktop\geog4-6300-lab-6-Kathy927\data\aus_climate_inat.gpkg' 
    ##   using driver `GPKG'
    ## Simple feature collection with 716 features and 22 fields
    ## Geometry type: POLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 113.875 ymin: -43.38632 xmax: 153.375 ymax: -11.92074
    ## Geodetic CRS:  WGS 84 (CRS84)

``` r
write.csv(lab6_data, file = "lab6_data.csv", row.names = FALSE)
```

The dataset for this lab is a 1 decimal degree hexagon grid that has
aggregate statistics for a number of variables:

- ndvi: NDVI/vegetation index values from Landsat data (via Google Earth
  Engine). These values range from -1 to 1, with higher values
  indicating more vegetation.
- maxtemp_00/20_med: Median maximum temperature (C) in 2000 or 2020
  (data from SILO/Queensland government)
- mintemp_00/20_med: Median minimum temperature (C) in 2020 or 2020
  (data from SILO/Queensland government)
- rain_00/20_sum: Total rainfall (mm) in 2000 or 2020 (data from
  SILO/Queensland government)
- pop_00/20: Total population in 2000 or 2020 (data from NASA’s Gridded
  Population of the World)
- water_00/20_pct: Percentage of land covered by water at some point
  during the year in 2000 or 2020
- elev_med: Median elevation (meters) (data from the Shuttle Radar
  Topography Mission/NASA)

There are also observation counts from iNaturalist for several
distinctively Australian animal species: the central bearded dragon, the
common emu, the red kangaroo, the agile wallaby, the laughing
kookaburra, the wombat, the koala, and the platypus.

Our primary research question is how the climatological/physical
variables in our dataset are predictive of the NDVI value. We will build
models for 2020 as well as the change from 2000 to 2020. The second is
referred to as a “first difference” model and can sometimes be more
useful for identifying causal mechanisms.

\###Part 1: Analysis of 2020 data###

We will start by looking at data for 2020.

**Question 1** *Create histograms for NDVI, max temp., min temp., rain,
and population, and water in 2020 as well as elevation. Based on these
graphs, assess the normality of these variables.*

``` r
variables <- c("ndvi_20_med","maxtemp_20_med","mintemp_20_med","rain_20_sum","pop_20","water_20_pct","elev_med")

for (var in variables) {
  hist(lab6_data[[var]],
  main = paste("Histogram of", var))
}
```

![](Lab-6_regression_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->![](Lab-6_regression_files/figure-gfm/unnamed-chunk-3-2.png)<!-- -->![](Lab-6_regression_files/figure-gfm/unnamed-chunk-3-3.png)<!-- -->![](Lab-6_regression_files/figure-gfm/unnamed-chunk-3-4.png)<!-- -->![](Lab-6_regression_files/figure-gfm/unnamed-chunk-3-5.png)<!-- -->![](Lab-6_regression_files/figure-gfm/unnamed-chunk-3-6.png)<!-- -->![](Lab-6_regression_files/figure-gfm/unnamed-chunk-3-7.png)<!-- -->

{Overall, the distributions of these variables deviate from normality,
showing varying degrees of skewness. NDVI, rainfall, population, and
water coverage are positively skewed, while min/max temperatures are
negatively skewed.}

**Question 2** *Use tmap to map these same variables using Jenks natural
breaks as the classification method. For an extra challenge, use
`tmap_arrange` to plot all maps in a single figure.*

``` r
map <- list()

for(var in variables) {
  map[[var]] <- tm_shape(lab6_data)+
    tm_polygons(var,style = "jenks", title = var)+
    tm_layout(legend.position = c("left","bottom"))
}

tmap_arrange(map, ncol = 2)
```

    ## Variable(s) "elev_med" contains positive and negative values, so midpoint is set to 0. Set midpoint = NA to show the full spectrum of the color palette.

![](Lab-6_regression_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

**Question 3** *Based on the maps from question 3, summarise major
patterns you see in the spatial distribution of these data from any of
your variables of interest. How do they appear to be associated with the
NDVI variable?*

{Higher NDVI values are concentrated along the eastern and southeastern
parts of Australia. This spatial pattern aligns with higher values of
rainfall (rain_20_sum), population (pop_20), and elevation (elev_med),
which are also concentrated in these regions. These overlaps suggest a
positive relationship between NDVI and these variables.}

**Question 4** *Create univariate models for each of the variables
listed in question 1, with NDVI in 2020 as the dependent variable. Print
a summary of each model. Write a summary of those results that indicates
the direction, magnitude, and significance for each model coefficient.*

``` r
variables1 <- c("maxtemp_20_med","mintemp_20_med","rain_20_sum","pop_20","water_20_pct","elev_med")

uni_model <- list()

for(var in variables1) {
  uni_model[[var]] <-
    lm(as.formula(paste("ndvi_20_med ~", var)), data = lab6_data)
}

for (var in variables1) {
  cat(var)
print(summary(uni_model[[var]]))}
```

    ## maxtemp_20_med
    ## Call:
    ## lm(formula = as.formula(paste("ndvi_20_med ~", var)), data = lab6_data)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.41874 -0.07657 -0.01927  0.06833  0.36382 
    ## 
    ## Coefficients:
    ##                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)     0.6612389  0.0294372   22.46   <2e-16 ***
    ## maxtemp_20_med -0.0130902  0.0009601  -13.63   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.1251 on 714 degrees of freedom
    ## Multiple R-squared:  0.2066, Adjusted R-squared:  0.2055 
    ## F-statistic: 185.9 on 1 and 714 DF,  p-value: < 2.2e-16
    ## 
    ## mintemp_20_med
    ## Call:
    ## lm(formula = as.formula(paste("ndvi_20_med ~", var)), data = lab6_data)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.36375 -0.08418 -0.03047  0.06972  0.40383 
    ## 
    ## Coefficients:
    ##                 Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)     0.464461   0.018997   24.45   <2e-16 ***
    ## mintemp_20_med -0.012282   0.001131  -10.86   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.1301 on 714 degrees of freedom
    ## Multiple R-squared:  0.1418, Adjusted R-squared:  0.1406 
    ## F-statistic:   118 on 1 and 714 DF,  p-value: < 2.2e-16
    ## 
    ## rain_20_sum
    ## Call:
    ## lm(formula = as.formula(paste("ndvi_20_med ~", var)), data = lab6_data)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.56681 -0.04753 -0.01210  0.04599  0.30930 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 1.303e-01  7.060e-03   18.45   <2e-16 ***
    ## rain_20_sum 9.124e-07  3.953e-08   23.08   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.1063 on 714 degrees of freedom
    ## Multiple R-squared:  0.4273, Adjusted R-squared:  0.4265 
    ## F-statistic: 532.6 on 1 and 714 DF,  p-value: < 2.2e-16
    ## 
    ## pop_20
    ## Call:
    ## lm(formula = as.formula(paste("ndvi_20_med ~", var)), data = lab6_data)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.47003 -0.07883 -0.03949  0.06384  0.48974 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 2.552e-01  5.013e-03  50.902   <2e-16 ***
    ## pop_20      1.500e-06  1.500e-07   9.998   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.1316 on 714 degrees of freedom
    ## Multiple R-squared:  0.1228, Adjusted R-squared:  0.1216 
    ## F-statistic: 99.97 on 1 and 714 DF,  p-value: < 2.2e-16
    ## 
    ## water_20_pct
    ## Call:
    ## lm(formula = as.formula(paste("ndvi_20_med ~", var)), data = lab6_data)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.26898 -0.08838 -0.04838  0.06871  0.50911 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   0.268988   0.006287  42.781   <2e-16 ***
    ## water_20_pct -0.178263   0.154480  -1.154    0.249    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.1403 on 714 degrees of freedom
    ## Multiple R-squared:  0.001862,   Adjusted R-squared:  0.0004636 
    ## F-statistic: 1.332 on 1 and 714 DF,  p-value: 0.2489
    ## 
    ## elev_med
    ## Call:
    ## lm(formula = as.formula(paste("ndvi_20_med ~", var)), data = lab6_data)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.27082 -0.09585 -0.04270  0.07954  0.44272 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 2.138e-01  9.741e-03  21.952  < 2e-16 ***
    ## elev_med    1.787e-04  2.895e-05   6.171 1.14e-09 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.1369 on 714 degrees of freedom
    ## Multiple R-squared:  0.05064,    Adjusted R-squared:  0.04931 
    ## F-statistic: 38.08 on 1 and 714 DF,  p-value: 1.136e-09

{For maxtemp, the coefficient is -0.0131, which means that for every 1°C
increase in maximum temperature, NDVI decreases by 0.0131. The p-value
(\<0.05) indicates that this relationship is statistically significant.

For mintemp, the coefficient is -0.0123, indicating that for every 1°C
increase in minimum temperature, NDVI decreases by 0.0123. This
relationship is also statistically significant, with a p-value (\<0.05).

For rainfall, the coefficient is 9.124 × 10⁻⁷, meaning that for every 1
mm increase in rainfall, NDVI increases by a very small amount. This
relationship is highly significant, with a p-value (\<0.05).

For water coverage, the coefficient is -0.1783, meaning that for every
1% increase in water coverage, NDVI decreases by 0.1783. However, this
relationship is not statistically significant, with a p-value (0.249)
greater than 0.05.

For population, the coefficient is 1.500 × 10⁻⁶, indicating that for
every 1 additional person in the population (or per 1,000, 100,000,
etc., depending on the scale), NDVI increases by a very small amount.
This relationship is statistically significant, with a p-value (\<0.05)

For elevation, the coefficient is 0.0001787, indicating that for every 1
meter increase in elevation, NDVI increases by 0.0001787. This
relationship is statistically significant, with a p-value (\<0.05).}

**Question 5** *Create a multivariate regression model with the
variables of interest, choosing EITHER max or min temperature (but not
both) You may also choose to leave out any variables that were
insignificant in Q4. Use the univariate models as your guide. Call the
results.*

``` r
#Code goes here
model_all <- lm(ndvi_20_med~mintemp_20_med+rain_20_sum+pop_20+elev_med, data = lab6_data)
summary(model_all)
```

    ## 
    ## Call:
    ## lm(formula = ndvi_20_med ~ mintemp_20_med + rain_20_sum + pop_20 + 
    ##     elev_med, data = lab6_data)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.49488 -0.02773  0.00412  0.03928  0.25115 
    ## 
    ## Coefficients:
    ##                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)     3.208e-01  1.413e-02  22.707  < 2e-16 ***
    ## mintemp_20_med -1.391e-02  7.675e-04 -18.121  < 2e-16 ***
    ## rain_20_sum     9.420e-07  3.276e-08  28.756  < 2e-16 ***
    ## pop_20          2.424e-07  1.032e-07   2.349   0.0191 *  
    ## elev_med        1.028e-04  1.774e-05   5.797 1.02e-08 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.0832 on 711 degrees of freedom
    ## Multiple R-squared:  0.6507, Adjusted R-squared:  0.6487 
    ## F-statistic: 331.1 on 4 and 711 DF,  p-value: < 2.2e-16

**Question 6** *Summarize the results of the multivariate model. What
are the direction, magnitude, and significance of each coefficient? How
did it change from the univariate models you created in Q4 (if at all)?
What do the R2 and F-statistic values tell you about overall model fit?*

{Overall, the relationship between these variables and NDVI are
statistically significant (p\<0.05):

For every 1°C increase in minimum temperature, NDVI decreases by 0.0139.
The coefficient remained similar to the univariate model.

For every 1 mm increase in rainfall, NDVI increases by slightly
(0.0000009420). This coefficient is very similar to that of the
univariate model in Q4.

There is a positive relationship between population and NDVI. The
coefficient is smaller compared to the univariate model, suggesting that
other variables may explain some of the variation previously attributed
to population.

For every 1 meter increase in elevation, NDVI increases by 0.0001028.
The coefficient is slightly smaller compared to the univariate model.

R-square (0.6507): The model explains 65.07% of the variance in NDVI,
which seems like a good fit.

F-Statistic (331.1, p\< 0.05) The F-statistic tests whether the overall
model is significant. Base of the f-stat value and p value, we can
conclude that the model as a whole is statistically significant.}

**Question 7** *Use a histogram and a map to assess the normality of
residuals and any spatial autocorrelation. Summarise any notable
patterns that you see.*

``` r
lab6_data$residuals<-residuals(model_all)

ggplot(lab6_data,aes(x=residuals))+
  geom_histogram(fill= "black")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Lab-6_regression_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
tm_shape(lab6_data)+
  tm_dots("residuals",size = 0.2)
```

    ## Variable(s) "residuals" contains positive and negative values, so midpoint is set to 0. Set midpoint = NA to show the full spectrum of the color palette.

![](Lab-6_regression_files/figure-gfm/unnamed-chunk-7-2.png)<!-- -->

{From the histogram, there are a few outliers in the tails (more on the
left side). However, the majority of residuals are tightly clustered
near zero. This suggests that the residuals are relatively normally
distributed From the map, the residuals are mostly close to zero
(indicated by light green), suggesting that the model performs well
across much of the area. However, at the periphery, there are noticeable
clusters of negative residuals (red and orange). These spatial patterns
suggest the presence of spatial autocorrelation, indicating that the
residuals are not randomly distributed.}

**Question 8** *Assess any issues with multicollinearity or
heteroskedastity in this model using the techniques shown in class. Run
the appropriate tests and explain what their results show you.*

``` r
vif(model_all)
```

    ## mintemp_20_med    rain_20_sum         pop_20       elev_med 
    ##       1.127167       1.121045       1.183240       1.015657

``` r
bptest(model_all)
```

    ## 
    ##  studentized Breusch-Pagan test
    ## 
    ## data:  model_all
    ## BP = 120.07, df = 4, p-value < 2.2e-16

{The VIF results for the all variables are below 2, which indicates no
significant multicollinearity in this model.

For the Breusch-Pagan test, since the p-value is extremely small (\<
2.2e-16), we reject the null hypothesis that the residuals have constant
variance. This indicates heteroskedasticity in the model, meaning the
variability of the residuals changes across observations. This can make
some parts of this model less reliable for drawing conclusions.}

**Question 9** *How would you summarise the results of this model in a
sentence or two? In addition, looking at the full model and your
diagnostics, do you feel this is a model that provides meaningful
results? Explain your answer.*

{Based on the r-square value, 65% of the variations in NDVI can be
explained by this model, which suggest that it provides meaningful
results. Also, all predictor variables are statistically significant and
the F-statistic is highly significant (p \< 2.2e-16), confirming that
the model as a whole is meaningful.}

**Disclosure of assistance:** *Besides class materials, what other
sources of assistance did you use while completing this lab? These can
include input from classmates, relevant material identified through web
searches (e.g., Stack Overflow), or assistance from ChatGPT or other AI
tools. How did these sources support your own learning in completing
this lab?*

{For this lab exercise, I used ChatGPT to understand how to create a
loop for generating maps and to troubleshoot error codes i got while
running new functions.}

**Lab reflection:** *How do you feel about the work you did on this lab?
Was it easy, moderate, or hard? What were the biggest things you learned
by completing it?*

{I feel confident about the work I completed for this lab. Overall, I
would rate it as moderate in difficulty because it took some time to
complete it. The biggest things I learned were how to create loops for
efficient plotting and modeling and interpreting regression output
meaningfully.}

**Challenge question**

\#Option 1 Create a first difference model. To do that, subtract the
values in 2000 from the values in 2020 for each variable for which that
is appropriate. Then create a new model similar to the one you created
in question 5, but using these new variables showing the *change in
values* over time. Call the results of the model, and interpret the
results in the same ways you did above. Also chart and map the residuals
to assess model error. Finally, write a short section that summarises
what, if anything, this model tells you.

``` r
difference<- lab6_data %>%
  mutate(ndvi_diff = ndvi_20_med-ndvi_00_med,
         maxtemp_diff = maxtemp_20_med - maxtemp_00_med,
         mintemp_diff = mintemp_20_med - mintemp_00_med,
         rain_diff = rain_20_sum - rain_00_sum,
         pop_diff = pop_20 - pop_00,
         water_diff = water_20_pct - water_00_pct)

difference_select <- difference %>%
  select("ndvi_diff","maxtemp_diff","mintemp_diff","rain_diff","pop_diff","water_diff")

modeldiff<- lm(ndvi_diff~maxtemp_diff+mintemp_diff+rain_diff+pop_diff+water_diff,data= difference_select)
summary(modeldiff)
```

    ## 
    ## Call:
    ## lm(formula = ndvi_diff ~ maxtemp_diff + mintemp_diff + rain_diff + 
    ##     pop_diff + water_diff, data = difference_select)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.56948 -0.02068  0.00079  0.02271  0.45407 
    ## 
    ## Coefficients:
    ##                Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  -2.152e-02  4.291e-03  -5.016 6.69e-07 ***
    ## maxtemp_diff  2.142e-03  2.489e-03   0.861     0.39    
    ## mintemp_diff -1.266e-02  2.809e-03  -4.506 7.74e-06 ***
    ## rain_diff     2.180e-07  3.878e-08   5.622 2.71e-08 ***
    ## pop_diff     -1.424e-07  2.868e-07  -0.496     0.62    
    ## water_diff   -1.959e-02  8.618e-02  -0.227     0.82    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.06052 on 710 degrees of freedom
    ## Multiple R-squared:  0.1061, Adjusted R-squared:  0.09985 
    ## F-statistic: 16.86 on 5 and 710 DF,  p-value: 9.076e-16

``` r
difference_select$residuals <- residuals(modeldiff)

ggplot(difference_select, aes(x= residuals))+
  geom_histogram(fill = "blue")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Lab-6_regression_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
tm_shape(difference_select)+
  tm_dots("residuals",size = 0.2)
```

    ## Variable(s) "residuals" contains positive and negative values, so midpoint is set to 0. Set midpoint = NA to show the full spectrum of the color palette.

![](Lab-6_regression_files/figure-gfm/unnamed-chunk-9-2.png)<!-- -->
{The R-squared value indicates that the model explains about 10.6% of
the variation in changes in NDVI. Compared to the other models run, this
is not the best for providing meaningful results, even though the
F-statistic (16.86, p \< 0.001) confirms that the model as a whole is
statistically significant. Regarding predictors, only the variables,
mintemp_diff and rain_diff, are statistically significant.

From the histogram, there are a few outliers in the tails. However, the
majority of residuals are tightly clustered near zero. This suggests
that the residuals are relatively normally distributed From the map, the
residuals are mostly close to zero (indicated by light green and
yellow), suggesting that the model performs well across much of the
area.There is also a cluster of larger residuals in the southeast
region, which suggest that the model performs poorly in that area.}

\#Option 2 The animal data included in this dataset is an example of
count data, and usually we would use a Poisson or similar model for that
purpose. Let’s try it with regular OLS regression though. Create two
regression models to assess how the counts of two different animals
(say, koalas and emus) are associated with at least three of the
environmental/climatological variables given above. Be sure to use the
same independent variables in each model. Interpret the results of each
model and then explain the importance of any differences in the model
coefficients between them, focusing on direction, magnitude, and
significance.
