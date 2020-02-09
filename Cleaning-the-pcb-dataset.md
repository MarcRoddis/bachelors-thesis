Cleaning the pcb dataset
================
Marc Roddis
2/9/2020

### Cleaning the pcb dataset

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
observations. The corresponding numbers of NA values in `pcb_tib2` for
our seven PCBs of interest are listed below. We do indeed see that we
did remove all observations that consisted almost completely of NA
values. We will use `pcb_tib2` are the starting point for the remainder
of our analysis.

    ## [1] 271

    ## [1] 447

    ## [1] 111

    ## [1] 70

    ## [1] 0

    ## [1] 0

    ## [1] 3
