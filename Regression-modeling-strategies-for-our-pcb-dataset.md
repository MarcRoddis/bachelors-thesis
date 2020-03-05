Imputation methods from RMS book First attempt
================
Marc Roddis
2/16/2020

### Preliminary studies of censored data

We create `pcb_tib1`, `pcb_tib2`, and `pcb_tib3` using the same code as
we used in “Preliminary studies of censored data”. We then create
`pcbtib_I1`, which will serve as the starting point for our imputation
studies; this tibble has all missing values coded as “NA”, all censored
values C substituted with abs(C)/sqrt(2), and all concentrations
substituted with their log-values. We will perform imputation of NA
values of `pcbtib_I1` using methods from Chapter 3 of Harrell’s book.

Harrell (page 48) says “In general, observations should only be
discarded if the MCAR assumption is justified, there is a rarely missing
predictor of overriding importance that cannot be reliably imputed from
other information, or if the fraction of observations excluded is very
small and the original sample size is large. Even then, there is no
advantage of such deletion other than saving analyst time. If a
predictor is MAR but its missingness depends on Y , casewise deletion is
biased.” In the context of this study, we know that the LOQ used to
censor pcb concentrations is associated with `FPRC`, so these censored
values are not censored completely at random (CCAR). We should first
investigate missing values to establish whether they are MCAR or not so
that we can decide whether or not to delete these values in accordance
with Harrell’s guidance.

We will first perform `aregImpute()` on `pcbtib_I1` (following page 56).
Link to aregImpute documentation
<https://www.rdocumentation.org/packages/Hmisc/versions/4.3-1/topics/aregImpute>

I spent 6 hours trying to implement imputation in R but without success;
the resulting dataset still contains NA values.
