Analysis of censored PCB data using regression imputation (main report)
gv7
================
Marc Roddis
6/20/2020

## Introduction

The Swedish National Monitoring Programme for Contaminants (SNMPC) in
freshwater biota has various goals and large scope (citation needed).
Our main goal in this study was to explore the viability of alternative
methodologies for the processing of censored data and to compare these
alternatives with the methodology used by SNMPC. At the outset, we
limited the scope of our study by choosing to focus on the estimation of
long-term time trends for the concentration of polychlorinated biphenyls
(PCBs) in biological samples. Our main idea was that since PCBs have
similar chemical and physical properties their concentrations may be
correlated such that censored measurements can be substituted using
regression imputation, which could then be used to draw better
conclusions compared with the methodology used by SNMPC.

Our study began with a large dataset `pcb.csv`, which has 5056
observations of 18 variables; these variables included: measured
concentrations of seven PCBs (CB28, CB53, CB101, CB118, CB138, CB153,
CB180); year (1984-2017); an ID for each observation; and nine other
variables such as species and age. We first performed exploratory data
analysis on this dataset to allow us to focus on the most important and
relevant observations and variables for our purpose. We then performed
simple computations on that data to obtain reasonable values for the two
fixed parameters and three variables for each of our simulation-based
studies; finally, a time-trend was fitted from each such study, which
was compared with the corresponding time-trend reported by SNMPC. We
then evaluated and summarised our findings.

## Exploratory data analysis (EDA)

### The goals of our EDA

Our EDA had five main goals:

1.  To identify censored, missing or bad observations, and calculate the
    proportions of such values in the dataset `pcb.csv`.

2.  To check the viability of our main idea by quantifying the degree
    and strength of association between PCB concentrations.

3.  To check that the pcb concentrations in our dataset have approximate
    log-normal distributions.

4.  To identify which PCBs to focus on for further study.

5.  To identify confounding variables for both the associations of PCB
    concentrations with one another and with time. We wanted to use as
    many observations as possible whilst keeping manageable scope.

#### Characteristics of censored, missing or bad observations in the dataset `pcb.csv`

To address goal 1 (see above) we imported and viewed the given dataset
`pcb.csv` and saw that there are 5056 observations of 18 variables. We
also saw that NA values are coded in various ways so we first replaced
all such values explicitly with `NA`. Concretely, this meant that all NA
values coded as \(-99.99\), \(-9.0000\), \(0\) or \(0.0000\) were
substituted for `NA`.

We then saw that there were only 6 `NA` values for CB138 and only 28
`NA` values for CB153, whereas there were many more NA values in the
data for the other pcb concentrations. Moreover, all observations (with
only one exception) which have an NA value for CB153 also have NA values
for all variables except CB138. So we removed these 28 sparse
observations, which gave the reduced dataset `pcb_tib3`, which has
\((631, 565, 125, 71, 0, 0, 44)\) NA values for (CB28, CB52, CB101,
CB118, CB138, CB153, CB180), respectively.

The CB138 variable of the `pcb_tib3` dataset has 11 censored
observations. For every one of these observations the only uncensored
value is for CB153, whereas the values for the other 6 PCBs are given by
one of two negative values, whereby CB138 and CB180 have the same value
as one another, and the four values for CB28, CB52, CB101 and CB118 are
equal to one another. So we chose to remove these 11 sparse
observations, which resulted in `pcb_tib4`, which has 5017 observations.
Our motivation is that `pcb_tib4` has no censored values and no missing
values for both CB138 and CB153; this allows us to perform preliminary
linear regression analysis to determine the strength of association
between CB138 and CB153 and fulfil goal 2.

To address goal 3, we first viewed histograms of the pcb concentrations
CB138 and CB153 (shown below); we see that these distributions each have
a large left-skew.

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk6-1.png)<!-- -->

Histograms (not shown) of the logarithm (with bases: 2, \(e\), 10, 100
and 1000 respectively) of the CB153 data, showed that the shape of the
distribution does not change noticeably when we change the base of the
logarithm. We will therefore use base \(e\) (as is standard practice)
throughout the remainder of this report. Histograms for \(log(CB138)\)
and \(log(CB153)\) are displayed below. We see that each of these
distributions still has some (but much less) left-skew and that each
loosely approximates the shape of a normal distribution. We will
therefore make the working assumption from now on that the data for each
of our seven PCBs of interest has a log-normal distribution.

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk8-1.png)<!-- -->

#### The degree and strength of association between PCB concentrations

Since the data for CB153 was most complete in the original dataset
`pcb.csv` we will view CB153 as the predictor variable (denoted by x and
shown on the horizontal axis) throughout our analysis. We will use
“response variables” to denote the variables that we make predictions
for, “response variables” (denoted by y and shown on the vertical axis).
We first display a scatter plot for \(y=CB138\) versus \(x=CB153\); the
second scatter plot shows \(y=log(CB138)\) versus \(x=log(CB153)\).

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk9-1.png)<!-- -->

Linear models corresponding to the two previous scatter plots were
fitted. The Adjusted R-squared value for the model with \(y=log(CB138)\)
and \(x=log(CB153)\) was \(0.957\), whereas the corresponding value was
\(0.931\) for the model without logarithms. This indicates stronger
correlation between the logarithms of the PCB concentrations than
between the PCB concentrations themselves. So from now on, we will use
logarithm-transformed data for all our linear regression analyses.

This preliminary regression analysis demonstrates the feasibility of our
approach: to use the fact that PCB concentrations are strongly
correlated to make predictions for censored values and for missing
values.

#### Confounding variables for the association between x=CB138 and y=CB153

We begin with `pcb_tib4` which was created as described in my document
“Cleaning the pcb dataset”. We will first mutate `pcb_tib4` (and
denote this mutated tibble as `pcb_tib4m`) so that it contains the
logarithms of the all pcb concentrations instead of the concentrations
themselves. In this section, we will explore the effect of five
variables (location `LOC`, species `SPECIES`, age `ALDR`, year `YEAR`,
percentage fat `FPRC`) on the association between x=CB138 and y=CB153.
For each variable, we will look for evidence of confounding from:

1.  The appearance of the scatter plot.

2.  The adjusted R-squared value and the slope coefficient
    `cb138_beta_cb153` for the fitted linear model.

##### Exploring the variable location `LOC`

We first get an overview by showing a scatter plot of all observations,
colour coded by location (this is a colour coded version of the previous
plot). We see that there are too many locations (27, to be precise) to
display clearly in a single plot.

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk12b-1.png)<!-- -->

We then created and viewed 27 separate scatter plots, each showing the
observations from a single location. Every one of these plots showed
strong (adjusted \(R^2\) \> 0.9) positive correlation between the
concentrations of CB138 and CB153.

The four plots shown below illustrate that the data from different
locations showed associations of various (albeit rather similar)
strengths. In addition the plot for Fladen shows possible clustering,
which is explored in the next sub-section.

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk12d-1.png)<!-- -->

##### Observations from Fladen show clustering by `SPECIES`

We saw that the scatter plot of observations from Fladen showed a
curious disjointed appearance. We then created and viewed a series of
plots in which the points on the plot were colour coded according to
values of the other variables. These colour-coded plots showed no
noticeable pattern of interest, with one exception; colour-coding by
species resulted in the plot shown below which appears to show two
distinct clusters for the two species present at this location. This
leads us to believe that species could be a confounding variable with
respect to the association between CB138 and CB153.

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk13-1.png)<!-- -->

We show below a scatter plot of all 5017 observations from `pcb_tib4m`,
colour coded by species (this is a colour coded version of the first
scatter plot in this EDA section of this report). We augment this
qualitative view with a more quantitative analysis in the next section.

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk14-1.png)<!-- -->

##### Association between CB138 and CB153 concentration: Confounding by `SPECIES`

We will now create sub-tibbles by filtering by species, and display the
corresponding scatter plot from each. The plots below each show all of
the observations from a single species.

We will now fit five linear models for y=CB153 and x=CB138, one for each
of the five species present in the `pcb.csv` dataset to explore our
conjecture that species is a confounding variable for the association
between x and y and to see whether we obtain higher R-squared values
from these five models. The adjusted R-squared values are (0.911, 0.933,
0.940, 0.971, 0.964) and the slope coefficients are (0.926, 0.904,
1.061, 0.844, 0.892) for (Blue mussel, Cod, Eelpout, Herring, Perch)
respectively.

##### Looking for evidence of confounding from location `LOC`

We will now focus solely on observations from herring. We find that
there are six locations that each have over 30 observations for herring.
We will now fit six linear models for y=CB153 and x=CB138, one for
observations from herring from each these six locations. The adjusted
R-squared values are (0.956, 0.953, 0.960, 0.915, 0.953, 0.950) and the
slope coefficients are (0.875, 0.831, 0.914, 0.868, 0.831, 0.948) for
(Ängskärsklubb, Fladen, Landsort, Utlängan, Utlängan (spring),
Väderöarna) respectively. The R-squared values are all within the
interval \[0.95, 0.96\] except for Utlängan (0.915). These results do
not show clear evidence of confounding by location and more detailed
analysis is omitted because it lies outside the scope of this report.

##### Looking for evidence of confounding from age `ALDR`

We will now proceed with data for herring from Landsort by first
displaying a scatter plot grouped by age `ALDR`. No confounding or
clustering by age can be discerned from this plot. Two linear models
were then fitted: one for the 169 fish aged 2-3 and one for the 254 fish
aged 5-7. The adjusted R-squared values are (0.894, 0.922) and the slope
coefficients are (0.893, 0.859) for (2-3 years old, 4-7 years old)
respectively; these values are quite similar so we will not view `ALDR`
as a confounding variable at this stage, so we will not filter by
`ALDR`.

##### Looking for evidence of confounding from age `ALDR`

We will now explore grouping by `YEAR`. The scatter plot shown below is
grouped by `YEAR` and shows distinct clustering, so we will next explore
filtering by `YEAR`.

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk105b-1.png)<!-- -->

We will begin by fitting two linear models: one for the 208 observations
from the 20th century and one for the 215 observations from the 21st
century. The adjusted R-squared values are (0.928, 0.954) and the slope
coefficients are (0.993, 0.997) for (20th century, 21st century)
respectively. The slope coefficients are remarkably similar for the two
centuries, so we will not filter by `YEAR`.

##### Looking for evidence of confounding from fat percentage `FPRC`

We will now explore whether we should filter by fat percentage `FPRC`.
The adjusted R-squared values are (0.956, 0.929, 0.931, 0.967) and the
slope coefficients are (0.879, 0.815, 0.763, 0.964) for fat percentages
that are (LOW, MEDIUM, HIGH, VHIGH) respectively. Although these slope
coefficients do show some variation, these values decrease then
increase, so there is no clear pattern. Moreover the adjusted R-squared
values are all in the fairly narrow interval \[0.929, 0.967\], so there
is no clear evidence for filtering the dataset by `FPRC` so we will not
do this.

#### Conclusions from our EDA

From our study of confounding variables, we conclude that we should view
`SPECIES` as a confounding variable because the scatter plot for herring
from Fladen showed distinct clusters for “Cod” and “Herring” and the
slope coefficient for a linear model fitted to observations from eelpout
was 26 % larger than the corresponding value from herring.

A possible conclusion indicated by weak evidence is that location `LOC`
could also be a weak confounder; the scatter plots for different
locations did show some differences of appearance. However, we did try
fitting some linear models for a few different locations and saw little
difference in slope coefficients. We therefore decided that it is not
necessary to view `LOC` as a confounder for the analysis that follows.
Moreover, our goal is to not exclude observations unless there is solid
evidence that this is advantageous.

To remove potential confounding by species, we will use observations
from herring exclusively for all our subsequent analyses. The dataset
`pcb_tib4h` (3269 observations) was obtained from the original dataset
`pcb.csv` (5056 observations) by removing: 28 observations that have NA
values for all variables except CB138; all 11 observations for which the
CB138 value was censored; all observations except those for herring.

The resulting `pcb_tib4h` dataset has (471, 295, 39, 16, 0, 0, 30) NA
values and (1163, 408, 9, 3, 0, 0, 310) censored values for (CB28, CB52,
CB101, CB118, CB138, CB153, CB180) respectively. So for CB28 and CB52
the number of NA values and censored values were both highest and second
highest respectively, whereas there were no such values for both CB138
and CB153. For this reason, we created the smaller dataset `pcb_tib4hs`
from `pcb_tib4h` by removing the variables CB101, CB118, CB180.

There are only 94 observations prior to the year 1989 whereas there are
3175 observations between 1989 and 2017. For this reason, we created the
smaller dataset `pcb_tib4hsy` from `pcb_tib4hs` by removing the
observations from years prior to 1989, and setting 1989 as “year zero”.

We will use `pcb_tib4hsy` as our starting point for the analyses
reported in the next section.

## The design of our simulation-based studies: High-level description

We begin with dataset `pcb_tib4hsy`, So we will use the complete data
for CB138 and CB153 and the strong correlations between pcb
concentrations to impute values for the observations with censored
and/or missing values for CB28 and/or CB52.

We will perform simulations studies in which we investigate the effect
of the three variables `sd`, `beta` and `LOD`: `sd` represents the
standard deviation of the pcb concentrations that are being imputed
(CB28 or CB52), `beta` represents the slope of the linear regression
line when \(x=YEAR\) and either \(y=CB28\) or \(y=CB52\), `LOD`
represents the level of detection, which is the threshold value used to
determine whether an observation gets censored or not.

Our simulation studies will inform us as to how robust or sensitive our
imputations are to the values of these three variables. Ideally such
sensitivity would be as low as possible because then the imputations
would be as generally applicable as possible.

We will use fixed values for the parameters `sd_cb` and `beta_cb`:
`sd_cb` represents the standard deviation of CB153 which we hold fixed
at the value calculated from the dataset because the data for CB153 is
complete, `beta_cb` represents the slope of the linear regression line
when x=CB153 and either y=CB28 or y=CB52 .

The estimation of appropriate values for these fixed and variable
parameters is described in the following sections.

### Estimation of appropriate values for the fixed parameters beta\_cb and sd\_cb

We will find estimates for `beta_cb` and `sd_cb` from various approaches
and compare these estimates.

We will use three main approaches for dealing with censored
observations, which we denote as \(C_i\):

1.  Substitution of \(C_i\) with \(abs(C_i)/\sqrt(2)\) for all i.

2.  Omission of \(C_i\) for all i.

3.  Keeping all \(C_i\) and using the cenreg() method to fit models,
    which provide estimates for `beta_cb` and `sd_cb`.

For approach 1 we will use two “sub-approaches” for dealing with the
missing values, which we denote as \(M_i\):

1a. Omission of \(M_i\) for all i.

1b. Use multiple imputation (MI) to impute all missing values,
substitute every \(M_i\) with its imputed value, fit a linear model to
the resulting “completed” dataset, and obtain parameter estimates from
the fitted model.

We will first calculate parameter estimates directly as stated above
without calculating annual means for pcb concentrations. We will then
perform the same calculation except that we will use annual means. We
will then attempt to evaluate whether or not it would be more
appropriate for us to use annual means in our subsequent work.

#### Approach 1 (Substitution of \(C_i\) with \(abs(C_i)/\sqrt(2)\) for all i)

We will first create our the dataset `vB1` by substitution of \(C_i\)
with \(abs(C_i)/\sqrt(2)\) for all i, re-coding all missing values as
`NA`, and substituting every concentration value with the natural
logarithm of its value. We will then use two approaches for the
estimation of beta\_cb and sd\_cb:

1a. Estimation from linear models fitted to `vB1` with omission of
missing values.

1b. Use of multiple imputation (MI). To do this, we start with `vB1` and
then create `completed_vB1` by replacing missing values with values
obtained by MI. We then perform estimation from linear models fitted to
`completed_vB1`. Our methodology is based on Van Buuren’s book “Flexible
Imputation of Missing Data”.

#### Estimation of beta\_cb and sd\_cb from linear models fitted by omission of missing values (approach 1a)

The most basic approach is to use `na.action = na.omit` in `lm()` to
perform listwise deletion. A drawback of this approach is loss of
information, for example we get “631 observations deleted due to
missingness” for `CB28 ~ CB153` or 954 deleted for `CB28 ~ CB52`. “If
the data are MCAR, listwise deletion produces unbiased estimates of
means, variances and regression weights. Under MCAR, listwise deletion
produces standard errors and significance levels that are correct for
the reduced subset of data, but that are often larger relative to all
available data. A disadvantage of listwise deletion is that it is
potentially wasteful. \[…\] If the data are not MCAR, listwise deletion
can severely bias estimates of means, regression coefficients and
correlations.” However, “There are cases in which listwise deletion can
provide better estimates than even the most sophisticated procedures.”
(see Section 2.6). Moreover, “Little and Rubin (2002) argue that it is
difficult to formulate rules of thumb since the consequences of using
listwise deletion depend on more than the missing data rate alone.”

Therefore despite its stated drawbacks, we begin by fitting linear
models by omission of missing values. We obtain the values (-0.054,
-0.061, -0.061, -0.046) for (beta\_cb28, beta\_cb52, beta\_cb138,
beta\_cb153) respectively, and (0.41, 0.33, 0.29, 0.19) for the
corresponding adjusted R-squared values. We will later compare the
parameters obtained from the simple approach with those obtained by
imputation. Note that we are using beta\_cb153 to guard against
erroneous results since we know that the values obtained here and by MI
later must be equal because vB1h$CB153 contains no missing values. The
adjusted R-squared value for the fitted linear model for \(x=YEAR\),
\(y=CB28\) was 0.41.

A linear model was also fitted for \(x=CB153\), \(y=CB28\); the slope
cb28\_beta\_cb153 of the regression line was 0.56 and the adjusted
R-squared was 0.54.

    ## 
    ## Call:
    ## lm(formula = CB28 ~ CB153, data = vB1hsy, na.action = na.omit)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1.91816 -0.27954  0.00096  0.26099  2.15472 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -3.98295    0.02684 -148.40   <2e-16 ***
    ## CB153        0.55006    0.00941   58.45   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.4539 on 2741 degrees of freedom
    ##   (432 observations deleted due to missingness)
    ## Multiple R-squared:  0.5549, Adjusted R-squared:  0.5547 
    ## F-statistic:  3417 on 1 and 2741 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = CB52 ~ CB153, data = vB1hsy, na.action = na.omit)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -3.1266 -0.3329  0.0311  0.3673  3.3047 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -2.73979    0.03388  -80.86   <2e-16 ***
    ## CB153        0.74011    0.01212   61.05   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.5989 on 2878 degrees of freedom
    ##   (295 observations deleted due to missingness)
    ## Multiple R-squared:  0.5643, Adjusted R-squared:  0.5642 
    ## F-statistic:  3728 on 1 and 2878 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = CB28 ~ YEAR, data = vB1hsy, na.action = na.omit)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -2.08362 -0.39213 -0.05294  0.32705  2.96476 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -4.73914    0.02033 -233.10   <2e-16 ***
    ## YEAR        -0.05136    0.00124  -41.41   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.5336 on 2741 degrees of freedom
    ##   (432 observations deleted due to missingness)
    ## Multiple R-squared:  0.3849, Adjusted R-squared:  0.3847 
    ## F-statistic:  1715 on 1 and 2741 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = CB52 ~ YEAR, data = vB1hsy, na.action = na.omit)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -2.35077 -0.53350 -0.02876  0.48844  2.95387 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -3.909716   0.026148 -149.52   <2e-16 ***
    ## YEAR        -0.059105   0.001662  -35.56   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.7562 on 2878 degrees of freedom
    ##   (295 observations deleted due to missingness)
    ## Multiple R-squared:  0.3053, Adjusted R-squared:  0.305 
    ## F-statistic:  1265 on 1 and 2878 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = CB138 ~ YEAR, data = vB1hsy, na.action = na.omit)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -3.3238 -0.5878 -0.0232  0.5499  2.8151 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -2.116634   0.026348  -80.33   <2e-16 ***
    ## YEAR        -0.061440   0.001712  -35.89   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.8292 on 3173 degrees of freedom
    ## Multiple R-squared:  0.2887, Adjusted R-squared:  0.2885 
    ## F-statistic:  1288 on 1 and 3173 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = CB153 ~ YEAR, data = vB1hsy, na.action = na.omit)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -3.1947 -0.6043 -0.0242  0.5587  3.0264 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -2.046541   0.026533  -77.13   <2e-16 ***
    ## YEAR        -0.046707   0.001724  -27.09   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.835 on 3173 degrees of freedom
    ## Multiple R-squared:  0.1879, Adjusted R-squared:  0.1876 
    ## F-statistic: 734.1 on 1 and 3173 DF,  p-value: < 2.2e-16

#### Estimation of beta\_cb and sd\_cb from multiple imputation (MI) of missing values (approach 1b)

##### Why MI was chosen

We will not use pairwise deletion since it is not generally applicable
and falls outside the scope of this study, “Pairwise deletion should
only be used if the procedure that follows it is specifically designed
to take deletion into account.” We will instead focus on using various
functions from the `mice` package for performing imputation in various
ways.

We will not use mean imputation since “Mean imputation is a fast and
simple fix for the missing data. However, it will underestimate the
variance, disturb the relations between variables, bias almost any
estimate other than the mean and bias the estimate of the mean when data
are not MCAR. Mean imputation should perhaps only be used as a rapid fix
when a handful of values are missing, and it should be avoided in
general.”

Regression imputation was used in our earlier report “Preliminary
studies of censored data”. However, the scatter plots showed that the
imputed data lay perfectly on the regression line. Ad hoc addition of
noise gave realistic looking scatter plots, however this report aims to
use theory-based rather than ad hoc approaches whenever possible, so we
will not explore regression imputation any further in this report.

The mice package allows us to perform theory based “Stochastic
regression imputation” (see Section 3.2), which is a potential areas for
further study later in this report. However, this method also has the
clear drawback that it can generate implausible values such as negative
values.

Based on what we have learnt so far, we view the other methods given by
van Buuren on page 16 as outside the scope of our study. We will choose
Multiple Imputation (MI) following the main recommendations from van
Buuren’s book.

##### Creation of the dataset `completed_vB1` by MI using the `mice` algorithm

For our second attempt at MI, we first the `quickpred()` function (see
vBed2p169 and
<https://www.rdocumentation.org/packages/mice/versions/3.8.0/topics/quickpred>)
and then perform multiple imputation using `mice()` and fill in the
missing values with `complete()`. The output below first compares CB28
from `vB1h` (which has 471 missing values) with CB28 from
`completed_vB1` (which has no missing values). Then linear models with
all significant predictors are fitted for each PCB concentration from
the `completed_vB1` dataset. There are approximately 10 significant
predictors for the fitted model for each PCB concentration. Although
trends in PCB concentration with time have been the main focus of
reports based on datasets similar to this one, `YEAR` is not even
significant for every PCB.

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk203-1.png)<!-- -->![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk203-2.png)<!-- -->

    ## 
    ## Call:
    ## lm(formula = CB28 ~ CB153, data = completed_vB1hsy)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1.98910 -0.29660 -0.00829  0.27339  2.12732 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -3.909495   0.025534 -153.11   <2e-16 ***
    ## CB153        0.563798   0.009118   61.84   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.4759 on 3173 degrees of freedom
    ## Multiple R-squared:  0.5465, Adjusted R-squared:  0.5463 
    ## F-statistic:  3824 on 1 and 3173 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = CB52 ~ CB153, data = completed_vB1hsy)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -3.1259 -0.3400  0.0285  0.3645  3.2835 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -2.74143    0.03225  -84.99   <2e-16 ***
    ## CB153        0.73484    0.01152   63.80   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.6011 on 3173 degrees of freedom
    ## Multiple R-squared:  0.562,  Adjusted R-squared:  0.5618 
    ## F-statistic:  4071 on 1 and 3173 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = CB28 ~ YEAR, data = completed_vB1hsy)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.3088 -0.3994 -0.0540  0.3379  2.9657 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -4.738329   0.017434 -271.78   <2e-16 ***
    ## YEAR        -0.051789   0.001133  -45.72   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.5487 on 3173 degrees of freedom
    ## Multiple R-squared:  0.3972, Adjusted R-squared:  0.397 
    ## F-statistic:  2090 on 1 and 3173 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = CB52 ~ YEAR, data = completed_vB1hsy)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.3194 -0.5282 -0.0228  0.4954  2.9765 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -3.944888   0.024147 -163.37   <2e-16 ***
    ## YEAR        -0.057853   0.001569  -36.88   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.7599 on 3173 degrees of freedom
    ## Multiple R-squared:    0.3,  Adjusted R-squared:  0.2998 
    ## F-statistic:  1360 on 1 and 3173 DF,  p-value: < 2.2e-16

My main motivation for fitting and viewing all the above linear model
sumaries was to see whether the number of predictors was associated with
the number of imputed values because I considered such an association to
be plausible since this was clearly the case for linear regression
imputation. However, the summary output above shows that there is no
such clear association, so this seems to be show an advantage of
multiple imputation over regression imputation. This is consistent with
(vB page 128) “it may seem that imputation would artificially strengthen
the relations of the complete data model, which would be clearly
undesirable. If done properly however, this is not the case.”

##### Estimation of beta\_cb and sd\_cb from linear models fitted to dataset `completed_vB1hs`

We used MI to replace NA values of CB28 with imputed values which
allowed us to calculate the standard deviation of (the logarithm of)
CB28 from the completed dataset. We obtained the values (-0.052, -0.047,
0.71, 0.93) for (beta\_cb28, beta\_cb153, sd\_cb28, sd\_cb153)
respectively; these values are used to choose baseline parameter value
for our simulation studies as described below. We also see that the
value beta\_cb28=-0.052 we obtained by MI is very similar to the value
beta\_cb28=-0.054 that we obtained initially by simple omission of
missing values. We did indeed get identical values for beta\_cb153 by
both methods, so our MI analysis passed this simple quality control.

Although we have compared parameters obtained by MI with those obtained
by omission and we have performed a simple check, we have not evaluated
the quality of our MI analysis. To do so, we could perform a comparison
with two main alternatives: Joint Modeling (JM) and Fully Conditional
Specification (FCS) and compare the outcomes in relation to van Buuren’s
conclusion (page 121) “For general missing data patterns, both JM and
FCS approaches can be used to impute multivariate missing data. JM is
the model of choice if the data conform to the modeling assumptions
because it has better theoretical properties.The FCS approach is much
more flexible and allows for imputations close to the data. Lee and
Carlin (2010) provide a comparison between both perspectives.” Such an
evaluation however, is outside the scope of this report. Moreover since
the values of beta\_cb28 obtained by MI and by omission were so similar,
we would likely also obtain similar values using JM and FCS, which would
make it difficult to distinguish between such alternative methodologies.

#### Approach 2A

We show below a scatter plot and fitted linear model summary for CB28
versus YEAR from `CB28_filtered4mhs`; this dataset was created from
`pcb_tib4mhs` by removal of all censored observations \(C_i\). The
fitted linear model for \(x=YEAR\), \(y=CB28\) has na.omit as the
na.action by default; for this model, the slope coefficient is -0.045;
SE = 0.0018; p-value \< 2e-16; Adjusted R-squared = 0.29.

    ## 
    ## Call:
    ## lm(formula = CB28_filtered4hsy$CB28 ~ CB28_filtered4hsy$YEAR)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -2.22538 -0.37220 -0.03511  0.35324  2.84068 
    ## 
    ## Coefficients:
    ##                         Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)            -4.650411   0.023167 -200.74   <2e-16 ***
    ## CB28_filtered4hsy$YEAR -0.042518   0.001786  -23.81   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.5504 on 1578 degrees of freedom
    ## Multiple R-squared:  0.2643, Adjusted R-squared:  0.2638 
    ## F-statistic: 566.9 on 1 and 1578 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = CB28_filtered4hsy$CB28 ~ CB28_filtered4hsy$CB153)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -2.45125 -0.28071 -0.01627  0.28326  2.23815 
    ## 
    ## Coefficients:
    ##                         Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)             -5.44958    0.01804 -302.02   <2e-16 ***
    ## CB28_filtered4hsy$CB153  2.48984    0.08670   28.72   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.52 on 1578 degrees of freedom
    ## Multiple R-squared:  0.3433, Adjusted R-squared:  0.3429 
    ## F-statistic: 824.8 on 1 and 1578 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = CB52_filtered4hsy$CB52 ~ CB52_filtered4hsy$YEAR)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -2.40338 -0.47208 -0.04393  0.43311  2.84537 
    ## 
    ## Coefficients:
    ##                         Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)            -3.881063   0.023869 -162.60   <2e-16 ***
    ## CB52_filtered4hsy$YEAR -0.051120   0.001609  -31.78   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.6741 on 2470 degrees of freedom
    ## Multiple R-squared:  0.2902, Adjusted R-squared:  0.2899 
    ## F-statistic:  1010 on 1 and 2470 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = CB52_filtered4hsy$CB52 ~ CB52_filtered4hsy$YEAR)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -2.40338 -0.47208 -0.04393  0.43311  2.84537 
    ## 
    ## Coefficients:
    ##                         Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)            -3.881063   0.023869 -162.60   <2e-16 ***
    ## CB52_filtered4hsy$YEAR -0.051120   0.001609  -31.78   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.6741 on 2470 degrees of freedom
    ## Multiple R-squared:  0.2902, Adjusted R-squared:  0.2899 
    ## F-statistic:  1010 on 1 and 2470 DF,  p-value: < 2.2e-16

#### Approach 2B

Approach 2B gave no additional insights and gave parameters estimates
that were similar to those from the other approaches so further details
are omitted for the sake of brevity.

#### Approach 3: Use of the cenreg() method

In this approach we used the cenreg() method to fit a linear model for
dataset `pcb_tib4hs` without removing or substituting censored values,
instead the information contained in the censored values is used to
obtain the fitted model `tib4hs_cenreg_28` for \(x=YEAR\), \(y=CB28\),
which has coefficients (0.010255, -0.000353) for (intercept,
beta\_CB28).

    ## Loading required package: survival

    ## 
    ## Attaching package: 'NADA'

    ## The following object is masked from 'package:stats':
    ## 
    ##     cor

    ##                             Value Std. Error      z         p
    ## (Intercept)              0.010255   2.07e-04   49.5  0.00e+00
    ## pcb_tib4hsy_CB28CI$YEAR -0.000353   1.27e-05  -27.8 3.22e-170
    ## Log(scale)              -5.218502   1.36e-02 -385.0  0.00e+00
    ## 
    ## Scale = 0.00542 
    ## 
    ## Gaussian distribution
    ## Loglik(model)= 3868.1   Loglik(intercept only)= 3527.2 
    ## Loglik-r:  0.4691553 
    ## 
    ## Chisq= 681.9 on 1 degrees of freedom, p= 0 
    ## Number of Newton-Raphson Iterations: 4 
    ## n = 2743

    ##                             Value Std. Error      z         p
    ## (Intercept)              0.026002   5.17e-04   50.3  0.00e+00
    ## pcb_tib4hsy_CB52CI$YEAR -0.000911   3.29e-05  -27.7 7.68e-169
    ## Log(scale)              -4.204058   1.32e-02 -318.3  0.00e+00
    ## 
    ## Scale = 0.0149 
    ## 
    ## Gaussian distribution
    ## Loglik(model)= 5769.1   Loglik(intercept only)= 5429.9 
    ## Loglik-r:  0.4581218 
    ## 
    ## Chisq= 678.43 on 1 degrees of freedom, p= 0 
    ## Number of Newton-Raphson Iterations: 4 
    ## n = 2880

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk207b-1.png)<!-- -->![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk207b-2.png)<!-- -->![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk207b-3.png)<!-- -->![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk207b-4.png)<!-- -->

We will now create model `tib4hsy_CB28LMpredSUB1` by substituting the
CB28 censored values with values predicted by the regression equation of
the model `tib4hs_cenreg_28` and fitting a linear model to the resulting
dataset. This model gives adjusted R-squared = 0.56, beta\_CB28 =
-0.075413. This adjusted R-squared value is slightly higher than we got
previously (without using cenreg() ). However, this is as we expected
because substitution using predicted values has given plotted points
that lie on the regression line so the increase in R-squared is at the
expense of unrealistic alignment of the substituted points. See our
document “Preliminary studies of censored data” to see how such
alignment can be eliminated by the addition of noise.

    ## 
    ## Call:
    ## lm(formula = pcb_tib4hsy_CB28LMpredSUB1$CB28 ~ pcb_tib4hsy_CB28LMpredSUB1$YEAR)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -2.30210 -0.28844  0.09674  0.20275  3.15016 
    ## 
    ## Coefficients:
    ##                                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                     -4.376328   0.021043 -207.97   <2e-16 ***
    ## pcb_tib4hsy_CB28LMpredSUB1$YEAR -0.075413   0.001283  -58.76   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.5523 on 2741 degrees of freedom
    ## Multiple R-squared:  0.5574, Adjusted R-squared:  0.5573 
    ## F-statistic:  3452 on 1 and 2741 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = pcb_tib4hsy_CB28LMpredSUB1$CB28 ~ pcb_tib4hsy_CB28LMpredSUB1$CB153)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -3.8440 -0.2959  0.1433  0.4422  2.4087 
    ## 
    ## Coefficients:
    ##                                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                      -5.77583    0.01797 -321.35   <2e-16 ***
    ## pcb_tib4hsy_CB28LMpredSUB1$CB153  3.13048    0.10845   28.86   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.727 on 2741 degrees of freedom
    ## Multiple R-squared:  0.2331, Adjusted R-squared:  0.2328 
    ## F-statistic: 833.2 on 1 and 2741 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = pcb_tib4hsy_CB52LMpredSUB1$CB52 ~ pcb_tib4hsy_CB52LMpredSUB1$YEAR)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -2.44623 -0.49887 -0.01257  0.50842  2.93745 
    ## 
    ## Coefficients:
    ##                                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                     -3.780291   0.025891 -146.00   <2e-16 ***
    ## pcb_tib4hsy_CB52LMpredSUB1$YEAR -0.070409   0.001646  -42.78   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.7488 on 2878 degrees of freedom
    ## Multiple R-squared:  0.3887, Adjusted R-squared:  0.3885 
    ## F-statistic:  1830 on 1 and 2878 DF,  p-value: < 2.2e-16

    ## 
    ## Call:
    ## lm(formula = pcb_tib4hsy_CB52LMpredSUB1$CB52 ~ pcb_tib4hsy_CB52LMpredSUB1$CB153)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -6.6748 -0.3407  0.0366  0.4691  2.2520 
    ## 
    ## Coefficients:
    ##                                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                      -5.20360    0.01839 -282.91   <2e-16 ***
    ## pcb_tib4hsy_CB52LMpredSUB1$CB153  4.39927    0.10588   41.55   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.7572 on 2878 degrees of freedom
    ## Multiple R-squared:  0.3749, Adjusted R-squared:  0.3747 
    ## F-statistic:  1726 on 1 and 2878 DF,  p-value: < 2.2e-16

We will now exclude both the observations for which the value of CB28 is
missing and those for which it is censored and fit a linear model and
compare with our previous results. The resulting fitted model has
coefficients (-4.650412 , -0.042518) and Adjusted-R-squared = 0.26.

    ## 
    ## Call:
    ## lm(formula = pcb_tib4hsy_CB28lmEXCLUDE$CB28 ~ pcb_tib4hsy_CB28lmEXCLUDE$YEAR)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -2.22538 -0.37220 -0.03515  0.35323  2.84071 
    ## 
    ## Coefficients:
    ##                                 Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                    -4.650412   0.023167 -200.74   <2e-16 ***
    ## pcb_tib4hsy_CB28lmEXCLUDE$YEAR -0.042518   0.001786  -23.81   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.5504 on 1578 degrees of freedom
    ## Multiple R-squared:  0.2643, Adjusted R-squared:  0.2638 
    ## F-statistic: 566.9 on 1 and 1578 DF,  p-value: < 2.2e-16

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk209-1.png)<!-- -->

In summary, three main approaches were tried: exclusion; substitution
using regression imputation; use of cenreg(). For each approach a model
was fitted, we will use the adjusted R-squared and likelihood-r values
to compare the goodness of fit for these models. The adjusted R-squared
values that resulted from models fitted by approaches 1a and 2A were
\(0.41\) and \(0.29\) respectively. The likelihood-r value obtained from
approach 3 was \(0.54\); since \(0.54^2=0.29\) we can say that the
models from approaches 2A and 3 fitted equally well, whereas the model
from approach 1a fit best of all.

The coefficients of the fitted cenreg() model were used to impute
missing values for CB28. A linear model was fit to the resulting
“completed” data set; the adjusted R-squared value for this model was
\(0.45\), which was the best fit of all the approaches we tried.

IS THIS STILL TRUE? The adjusted-R-squared values were reported for each
case and found to have similar values when exclusion and regression
imputation were used. However, for the cenreg() method, the value of
likelihood-r was reported and its squared value was found to be lower
than the adjusted R-squared values from the other approaches. Weaker
association after using cenreg() was not found in our document
“Preliminary studies of censored data”; our preliminary explanation is
to attribute this relative failure here to the larger proportion of
censored data for the CB28 variable than for the variables used in our
previous document. WAS IT EVER TRUE?

#### What effect (if any) does the use of annual (geometric) mean pcb concentrations have on our parameter estimates?

We will again use dataset `pcb_tib4h` as our starting point. The PCB
concentration data is incomplete for 1984, so we exclude this data from
1984, which results in 1987 being the first year. So we will remove all
observation prior to 1987 to create the reduced dataset
`pcb_tib4h_post87`, and we will set 1987 as “year zero”.

Since we are using the logarithms of the pcb concentrations we will use
the geometric means of these log-concentrations; for every year we
denote such a mean as the annual mean for that year. We will study the
effect of using such means by repeating the key parts of the above
approaches except that we will use the annual means instead of the
log-concentrations. We begin by displaying such data below.

### Scatter plots and linear model summaries for annual pcb concentrations after LOQ/sqrt(2) substitution

The scatter plots for CB28, CB52, CB138, CB153 respectively for this
approach are shown below.

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk212-1.png)<!-- -->

Summary output for fitted models for x=YEAR or x=CB153, and y=CB28 or
y=CB52 for this approach are shown below.

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_sqrt2_sub$CB28 ~ tib4hsy_year_sqrt2_sub$YEAR)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0031752 -0.0017559 -0.0006331  0.0008379  0.0078538 
    ## 
    ## Coefficients:
    ##                               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                  1.005e-02  9.535e-04  10.536 4.58e-11 ***
    ## tib4hsy_year_sqrt2_sub$YEAR -3.452e-04  5.846e-05  -5.904 2.72e-06 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.002634 on 27 degrees of freedom
    ## Multiple R-squared:  0.5635, Adjusted R-squared:  0.5474 
    ## F-statistic: 34.86 on 1 and 27 DF,  p-value: 2.724e-06

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_sqrt2_sub$CB52 ~ tib4hsy_year_sqrt2_sub$YEAR)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0053350 -0.0031217 -0.0009685  0.0023847  0.0100118 
    ## 
    ## Coefficients:
    ##                               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                  1.944e-02  1.439e-03  13.514 1.56e-13 ***
    ## tib4hsy_year_sqrt2_sub$YEAR -6.567e-04  8.822e-05  -7.443 5.25e-08 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.003975 on 27 degrees of freedom
    ## Multiple R-squared:  0.6723, Adjusted R-squared:  0.6602 
    ## F-statistic:  55.4 on 1 and 27 DF,  p-value: 5.248e-08

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_sqrt2_sub$CB28 ~ tib4hsy_year_sqrt2_sub$CB153)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0029466 -0.0012621 -0.0001325  0.0004256  0.0084535 
    ## 
    ## Coefficients:
    ##                               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                  -0.001859   0.001056  -1.760   0.0897 .  
    ## tib4hsy_year_sqrt2_sub$CB153  0.096789   0.013210   7.327    7e-08 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.002306 on 27 degrees of freedom
    ## Multiple R-squared:  0.6654, Adjusted R-squared:  0.653 
    ## F-statistic: 53.69 on 1 and 27 DF,  p-value: 6.999e-08

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_sqrt2_sub$CB52 ~ tib4hsy_year_sqrt2_sub$CB153)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0043544 -0.0018821 -0.0001388  0.0011245  0.0110239 
    ## 
    ## Coefficients:
    ##                               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                  -0.003156   0.001463  -2.158     0.04 *  
    ## tib4hsy_year_sqrt2_sub$CB153  0.183494   0.018298  10.028 1.34e-10 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.003195 on 27 degrees of freedom
    ## Multiple R-squared:  0.7883, Adjusted R-squared:  0.7805 
    ## F-statistic: 100.6 on 1 and 27 DF,  p-value: 1.336e-10

We see that the adjusted R-squared values are higher for the averaged
data than for the data itself.

#### Dataset creation, scatter plots and linear model summaries for pcb concentrations after exclusion of censored values

The `tib4h_year_remove_cens` dataset was created from our starting
dataset `pcb_tib4h` by removing observations from before 1987, setting
1987 as the zero year, removing observations with censored values,
substituting log-concentrations with annual (geometric) mean
log-concentrations. The four scatter plots for x=YEAR or x=CB153, and
y=CB28 or y=CB52 from this created dataset are shown below.

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk216-1.png)<!-- -->

Linear models were fitted for \(x=YEAR\), \(y=CBn\) where n denotes 28,
52, 138, 153 respectively. The values of (beta\_cb28, beta\_cb52,
beta\_cb138, beta\_cb153) given by the model are (-3.9e-04, -7.5e-04,
-0.0040, -0.0037), and the adjusted R-squared values are (0.57, 0.67,
0.71, 0.69). As before, we see that the adjusted R-squared values are
higher for the averaged data than the corresponding values (0.41, 0.33,
0.29, 0.19) we reported previously from applying the same methodology to
the corresponding unaveraged values.

A linear model was also fitted for \(x=CB153\), \(y=CB28\); the slope
cb28\_beta\_cb153 of the regression line was 0.073 and the adjusted
R-squared was 0.36.

A linear model was also fitted for \(x=CB153\), \(y=CB28\); the slope
cb28\_beta\_cb153 of the regression line was 0.56 and the adjusted
R-squared was 0.54.

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_remove_cens$CB28 ~ tib4hsy_year_remove_cens$YEAR)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0040373 -0.0008353 -0.0001769  0.0007668  0.0071752 
    ## 
    ## Coefficients:
    ##                                 Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                    1.072e-02  8.842e-04  12.129 1.94e-12 ***
    ## tib4hsy_year_remove_cens$YEAR -3.146e-04  5.421e-05  -5.803 3.57e-06 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.002443 on 27 degrees of freedom
    ## Multiple R-squared:  0.555,  Adjusted R-squared:  0.5385 
    ## F-statistic: 33.67 on 1 and 27 DF,  p-value: 3.565e-06

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_remove_cens$CB52 ~ tib4hsy_year_remove_cens$YEAR)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0042907 -0.0021298 -0.0005387  0.0013133  0.0096637 
    ## 
    ## Coefficients:
    ##                                 Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                    1.974e-02  1.307e-03  15.102 1.09e-14 ***
    ## tib4hsy_year_remove_cens$YEAR -6.065e-04  8.016e-05  -7.566 3.87e-08 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.003612 on 27 degrees of freedom
    ## Multiple R-squared:  0.6795, Adjusted R-squared:  0.6677 
    ## F-statistic: 57.25 on 1 and 27 DF,  p-value: 3.868e-08

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_remove_cens$CB28 ~ tib4hsy_year_remove_cens$CB153)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0032982 -0.0010561 -0.0002193  0.0005397  0.0079695 
    ## 
    ## Coefficients:
    ##                                 Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                    0.0002891  0.0010944   0.264    0.794    
    ## tib4hsy_year_remove_cens$CB153 0.0825464  0.0136911   6.029 1.96e-06 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.00239 on 27 degrees of freedom
    ## Multiple R-squared:  0.5738, Adjusted R-squared:  0.558 
    ## F-statistic: 36.35 on 1 and 27 DF,  p-value: 1.959e-06

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_remove_cens$CB52 ~ tib4hsy_year_remove_cens$CB153)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0039670 -0.0015581 -0.0000611  0.0008204  0.0107374 
    ## 
    ## Coefficients:
    ##                                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                    -0.0009638  0.0013846  -0.696    0.492    
    ## tib4hsy_year_remove_cens$CB153  0.1671779  0.0173211   9.652 3.02e-10 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.003024 on 27 degrees of freedom
    ## Multiple R-squared:  0.7753, Adjusted R-squared:  0.767 
    ## F-statistic: 93.16 on 1 and 27 DF,  p-value: 3.021e-10

#### Preliminary summary NEEDS TO BE UPDATED FROM HERE ONWARDS

The adjusted R-squared values for CB28:CB180 for the fitted model after
substitution using LOQ/sqrt(2) were (0.55, 0.64, 0.65, 0.65, 0.72, 0.75,
0.72). The corresponding values after exclusion of censored data were
(0.53, 0.64, 0.68, 0.64, 0.72, 0.75, 0.71). We see that the values in
these two vectors are very similar but much higher than the
corresponding values for non-averaged data (see our previous document).

#### Using LOQ/sqrt(2) substituted censored values for data from 1989 onwards

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk221-1.png)<!-- -->

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_sqrt2_sub_start1989$CB28 ~ tib4hsy_year_sqrt2_sub_start1989$YEAR)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0031752 -0.0017559 -0.0006331  0.0008379  0.0078538 
    ## 
    ## Coefficients:
    ##                                         Estimate Std. Error t value Pr(>|t|)
    ## (Intercept)                            1.005e-02  9.535e-04  10.536 4.58e-11
    ## tib4hsy_year_sqrt2_sub_start1989$YEAR -3.452e-04  5.846e-05  -5.904 2.72e-06
    ##                                          
    ## (Intercept)                           ***
    ## tib4hsy_year_sqrt2_sub_start1989$YEAR ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.002634 on 27 degrees of freedom
    ## Multiple R-squared:  0.5635, Adjusted R-squared:  0.5474 
    ## F-statistic: 34.86 on 1 and 27 DF,  p-value: 2.724e-06

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_sqrt2_sub_start1989$CB52 ~ tib4hsy_year_sqrt2_sub_start1989$YEAR)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0053350 -0.0031217 -0.0009685  0.0023847  0.0100118 
    ## 
    ## Coefficients:
    ##                                         Estimate Std. Error t value Pr(>|t|)
    ## (Intercept)                            1.944e-02  1.439e-03  13.514 1.56e-13
    ## tib4hsy_year_sqrt2_sub_start1989$YEAR -6.567e-04  8.822e-05  -7.443 5.25e-08
    ##                                          
    ## (Intercept)                           ***
    ## tib4hsy_year_sqrt2_sub_start1989$YEAR ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.003975 on 27 degrees of freedom
    ## Multiple R-squared:  0.6723, Adjusted R-squared:  0.6602 
    ## F-statistic:  55.4 on 1 and 27 DF,  p-value: 5.248e-08

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_sqrt2_sub_start1989$CB28 ~ tib4hsy_year_sqrt2_sub_start1989$CB153)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0029466 -0.0012621 -0.0001325  0.0004256  0.0084535 
    ## 
    ## Coefficients:
    ##                                         Estimate Std. Error t value Pr(>|t|)
    ## (Intercept)                            -0.001859   0.001056  -1.760   0.0897
    ## tib4hsy_year_sqrt2_sub_start1989$CB153  0.096789   0.013210   7.327    7e-08
    ##                                           
    ## (Intercept)                            .  
    ## tib4hsy_year_sqrt2_sub_start1989$CB153 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.002306 on 27 degrees of freedom
    ## Multiple R-squared:  0.6654, Adjusted R-squared:  0.653 
    ## F-statistic: 53.69 on 1 and 27 DF,  p-value: 6.999e-08

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_sqrt2_sub_start1989$CB28 ~ tib4hsy_year_sqrt2_sub_start1989$CB153)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0029466 -0.0012621 -0.0001325  0.0004256  0.0084535 
    ## 
    ## Coefficients:
    ##                                         Estimate Std. Error t value Pr(>|t|)
    ## (Intercept)                            -0.001859   0.001056  -1.760   0.0897
    ## tib4hsy_year_sqrt2_sub_start1989$CB153  0.096789   0.013210   7.327    7e-08
    ##                                           
    ## (Intercept)                            .  
    ## tib4hsy_year_sqrt2_sub_start1989$CB153 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.002306 on 27 degrees of freedom
    ## Multiple R-squared:  0.6654, Adjusted R-squared:  0.653 
    ## F-statistic: 53.69 on 1 and 27 DF,  p-value: 6.999e-08

#### Using exclusion of censored values for data from 1989 onwards

![](Analysis-of-censored-PCB-data-using-regression-imputation--main-report--gv7_files/figure-gfm/chunk225-1.png)<!-- -->

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_remove_cens_from1989$CB28 ~ tib4hsy_year_remove_cens_from1989$YEAR)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0040373 -0.0008353 -0.0001769  0.0007668  0.0071752 
    ## 
    ## Coefficients:
    ##                                          Estimate Std. Error t value Pr(>|t|)
    ## (Intercept)                             1.072e-02  8.842e-04  12.129 1.94e-12
    ## tib4hsy_year_remove_cens_from1989$YEAR -3.146e-04  5.421e-05  -5.803 3.57e-06
    ##                                           
    ## (Intercept)                            ***
    ## tib4hsy_year_remove_cens_from1989$YEAR ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.002443 on 27 degrees of freedom
    ## Multiple R-squared:  0.555,  Adjusted R-squared:  0.5385 
    ## F-statistic: 33.67 on 1 and 27 DF,  p-value: 3.565e-06

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_remove_cens_from1989$CB52 ~ tib4hsy_year_remove_cens_from1989$YEAR)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0042907 -0.0021298 -0.0005387  0.0013133  0.0096637 
    ## 
    ## Coefficients:
    ##                                          Estimate Std. Error t value Pr(>|t|)
    ## (Intercept)                             1.974e-02  1.307e-03  15.102 1.09e-14
    ## tib4hsy_year_remove_cens_from1989$YEAR -6.065e-04  8.016e-05  -7.566 3.87e-08
    ##                                           
    ## (Intercept)                            ***
    ## tib4hsy_year_remove_cens_from1989$YEAR ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.003612 on 27 degrees of freedom
    ## Multiple R-squared:  0.6795, Adjusted R-squared:  0.6677 
    ## F-statistic: 57.25 on 1 and 27 DF,  p-value: 3.868e-08

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_remove_cens_from1989$CB28 ~ tib4hsy_year_remove_cens_from1989$CB153)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0032982 -0.0010561 -0.0002193  0.0005397  0.0079695 
    ## 
    ## Coefficients:
    ##                                          Estimate Std. Error t value Pr(>|t|)
    ## (Intercept)                             0.0002891  0.0010944   0.264    0.794
    ## tib4hsy_year_remove_cens_from1989$CB153 0.0825464  0.0136911   6.029 1.96e-06
    ##                                            
    ## (Intercept)                                
    ## tib4hsy_year_remove_cens_from1989$CB153 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.00239 on 27 degrees of freedom
    ## Multiple R-squared:  0.5738, Adjusted R-squared:  0.558 
    ## F-statistic: 36.35 on 1 and 27 DF,  p-value: 1.959e-06

    ## 
    ## Call:
    ## lm(formula = tib4hsy_year_remove_cens_from1989$CB28 ~ tib4hsy_year_remove_cens_from1989$CB153)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -0.0032982 -0.0010561 -0.0002193  0.0005397  0.0079695 
    ## 
    ## Coefficients:
    ##                                          Estimate Std. Error t value Pr(>|t|)
    ## (Intercept)                             0.0002891  0.0010944   0.264    0.794
    ## tib4hsy_year_remove_cens_from1989$CB153 0.0825464  0.0136911   6.029 1.96e-06
    ##                                            
    ## (Intercept)                                
    ## tib4hsy_year_remove_cens_from1989$CB153 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.00239 on 27 degrees of freedom
    ## Multiple R-squared:  0.5738, Adjusted R-squared:  0.558 
    ## F-statistic: 36.35 on 1 and 27 DF,  p-value: 1.959e-06

#### Intermediate (second) summary

As stated previously: starting from 1987 the adjusted R-squared values
for CB28:CB180 for the fitted model after substitution using LOQ/sqrt(2)
were (0.55, 0.64, 0.65, 0.65, 0.72, 0.75, 0.72). The corresponding
values after exclusion of censored data were (0.53, 0.64, 0.68, 0.64,
0.72, 0.75, 0.71).

We have now also found that: starting from 1989 the adjusted R-squared
values for CB28:CB180 for the fitted model after substitution using
LOQ/sqrt(2) were (0.54, 0.62, 0.67, 0.73, 0.86, 0.90, 0.83). The
corresponding values after exclusion of censored data were (0.52, 0.62,
0.71, 0.72, 0.86, 0.90, 0.86).

We see that the adjusted R-squared values for CB118:CB180 were higher
when starting from 1989 than from 1987. However, the values from
“exclusion” or “substitution by LOQ/sqrt(2)” of censored values were
very similar.

We should remain alert to the fact that all of the above related to
fitting linear models so it may not be relevant if such models are
unsuitable.

### Presentation of our chosen design

We obtained the values (-0.055, -0.046, 0.74, 0.92) for (beta\_cb28,
beta\_cb153, sd\_cb28, sd\_cb153) respectively. The values for
(beta\_cb153, sd\_cb153) will be held fixed at (-0.046, 0.92). The three
values for `beta_cb28` we will use in our simulations are (-0.04,
-0.055, -0.07); for `sd_cb28` we will use the values (0.50, 0.75, 1.00).

For `LOD` we will use values that correspond to proportions of censored
observations of 0.1, 0.3 and 0.5 respectively. These values correspond
to the CB28 values at the 10th, 30th and 50th percentiles respectively.

## Implementation of our simulation-based studies

### High-level description of our design

To be written.

### Studies in which `sd` is varied

To be written.

### Studies in which `beta` is varied

To be written.

### Studies in which `LOD` is varied

To be written.

## Evaluation of our results and comparison with SNMPC results

To be written.

## Summary

To be written.

## References, appendices etc

To be written.
