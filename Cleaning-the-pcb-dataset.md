Cleaning the pcb dataset
================
Marc Roddis
2/9/2020

### Preliminary cleaning (creating pcb\_tib2)

By importing and viewing the given dataset `pcb.csv` we see that there
are 5056 observations of 18 variables. We begin by cleaning the dataset;
we first look for NA values; we see that some values are stated
explicitly as NA. Moreover, some values are stated as \(-99.99\),
\(-9.0000\) etc.; we interpret all such values as NA values. We
therefore replace all values that are less than \(-8\) with NA values,
in order to consistently denote all NA values as NA; we denote the
resulting tibble as `pcb_tib1`. The numbers of NA values for each CB28,
…, CB180 (listed in ascending numerical order of n for CBn) in
`pcb_tib1` are listed below.

We find that there are \((298, 474, 138, 97, 6, 28, 30)\) NA values
coded in this manner for
\((CB28, CB52, CB101, CB118, CB138, CB153, CB180)\), respectively.

    ## [1] 298

    ## [1] 474

    ## [1] 138

    ## [1] 97

    ## [1] 6

    ## [1] 28

    ## [1] 30

From viewing `pcb_tib1` we see that all observations (with only one
exception) which have an NA value for CB153 also have NA values for all
variables except CB138. So as a matter of convenience we remove these
sparse observations, which results in `pcb_tib2`, which has 5028
observations. We find that there are \((271, 447, 111, 70, 0, 0, 3)\) NA
values coded in this manner for
\((CB28, CB52, CB101, CB118, CB138, CB153, CB180)\), respectively. We do
indeed see that did remove all observations that consisted almost
completely of NA values. We will use `pcb_tib2` are the starting point
for the remainder of our analysis.

    ## [1] 271

    ## [1] 447

    ## [1] 111

    ## [1] 70

    ## [1] 0

    ## [1] 0

    ## [1] 3

#### Further cleaning (creating pcb\_tib3)

Starting from `pcb_tib2` we will now check the three observations for
CB180 that have NA values; we see that these observations have measured
values for at least five other PCB in each case. Checking for further NA
values, we now see that many values for CB28 are stated as either
\(0.0000\) or \(0\), which we will also interpret as NA values; let’s
see how many such values there are. Since `tib_CB28zero` has 360 rows,
we conclude that for CB28 there are 360 NA values that are encoded as
\(0.0000\) or \(0\).

Replacing all such encoded NA values with explicit NA values resulted in
`pcb_tib3`, which has \((631, 565, 125, 71, 0, 0, 44)\) NA values for
\((CB28, CB52, CB101, CB118, CB138, CB153, CB180)\), respectively.

    ## [1] 631

    ## [1] 565

    ## [1] 125

    ## [1] 71

    ## [1] 0

    ## [1] 0

    ## [1] 44

#### Visualisation of pcb\_tib3$CB138
