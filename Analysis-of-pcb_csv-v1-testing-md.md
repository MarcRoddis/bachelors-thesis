Analysis of pcb\_csv v1 testing md
================
Marc Roddis
8/2/2020

### Preliminary exploratory data analysis

We will first glimpse the pcb\_df data frame.

The pcb\_df dataset has 5056 observations of 18 variables. The number of
censored data values and NA values in this dataset is:

CB28 has 2265 censored values and 150 NA values.

CB52 has 1348 censored values and 356 NA values.

CB101 has 321 censored values and 103 NA values.

CB118 has 52 censored values and 81 NA values.

CB153 has 6 censored values and 22 NA values.

CB138 has 17 censored values and 0 NA values.

CB180 has 578 censored values and 23 NA values.

CB153 and CB138 have the lowest proportion of censored values so let’s
start with these variables. The smallest positive value in the dataset
is 0.0040 for CB153 and 0.0030 for CB138. We will begin by replacing all
negative values and NA values with these respective values, and then
calculating the mean for these two variables. We denote these mutated
variables as fabd\_CB153 and fabd\_CB138 see that the means are: 0.103
for CB138 and 0.140 for CB153.

We will now fit a linear model for x= fabd\_CB138 and y=fabd\_CB153. We
see that the association between these variables is highly significant
(p-value \< 2e-16) and that the correlation is strong \((R^2=0.9)\).

    ## 
    ## Call:
    ## lm(formula = fabd_CB153$CB153 ~ fabd_CB138$CB138)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1.33478 -0.00963 -0.00075  0.00685  0.58582 
    ## 
    ## Coefficients:
    ##                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)      0.002977   0.001185   2.512    0.012 *  
    ## fabd_CB138$CB138 1.329490   0.006994 190.077   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.06688 on 5054 degrees of freedom
    ## Multiple R-squared:  0.8773, Adjusted R-squared:  0.8773 
    ## F-statistic: 3.613e+04 on 1 and 5054 DF,  p-value: < 2.2e-16

We will use substitution methods of increasing sophistication. We will
begin by subsituting negative values of CB138 and CB with LOQ/2. We will
then use CB138 and our linear model to predict and substitute for the NA
values of CB153. We will then continue this process to predict and
substitute values for CB118 and so on.

#### Overview of rationale for some ideas to potentially implement

Concentrations of families of organic pollutants (e.g. PCBs) are often
below the LOQ, so can we use associations between such concentrations to
predict values below the LOQ. Such predicted values could be
substituted; this would follow the same methodology as has been used
previously, the new idea is simply to substitute predicted values
instead of the arbitrary LOQ/sqrt(2) value. The rationale for this is
that such predictions would be evidence-based (even though such
predicted values have uncertainty) and also different which should mean
that the variance is more realistic than using arbitrary values which
results in artificially low variance (ref. Helsel).

More explicitly and concretely, my work could begin with simulation
studies in which 2D (X,Y) scatter plots are created with different
values of R^2 (strength of association), beta (slope parameter),
proportions of censored values. For each such simulation, the data with
X\>LOQ and Y\>LOQ can be used to estimate beta, then the data X\>LOQ
together with beta would be used to predict Y\<LOQ values. These
predictions would then be validated and evaluated by comparison against
the simulated data. This family of simulations would illustrate the
space “space of goodness” of (R^2, beta, proportion of LOQ) triples
which allow for useful (good) predictions. The next step would be to
look for examples from experimental data that live within the “space of
goodness” and make predictions for those examples. Substitution of
predicted versus arbitrary values can then be evaluated and validated
using (for example: 10-fold cross-validation). Finally, my outcomes
could be evaluated against Anton’s outcomes to see whether the
conclusions from our distinct approaches are concordant, discordant or
neither.

#### Things for Marc to learn

Read Ch 11-12 of Helsel’s book.

Gain basic skills with Git, Github etc.; data wrangling with R (learn
tidyverse etc.)

Learn how to simulate data in a suitable way.

Perform a literature search to find relevant examples of simulation
studies. Can I use Helsel (page 5) to guide me?

Begin by performing the simplest possible simulation study I can design.

Learn how to present my work at the required standard (using R markdown
with calls to external scripts for generation of figures etc.)

#### Deleted parts

For CB28 there are 2265 censored values and 150 NA values. This means
there are \(5056-2265-150=2641\) measured values. The proportion of
censored values (excluding NA values) is 0.553. We will next select
appropriate methods from Helsel’s book for the analysis of the PCB data
from pcb.csv. The proportion of censored values for each PCB is given
below:
