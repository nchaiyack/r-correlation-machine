
# `run_correlations`: run lots of correlations against a single variable, with
tidyverse-friendly syntax and lots of knobs (stratification, Bonferroni correction, test directionality, ...)

A very, very large and tidyverse-friendly R function.

**Motivation.** A common task that psychometricians do to evaluate the validity of
a measure, to is run a *lot* of correlations against other variables, in order to test "no duh"
relationships. The function `run_correlations` allows you to do so with the
absurd convenience of a "industrial-strength" tidyverse function. Fundamentally,
though, this just calls `stats::cor.test` many, many times.

## Usage

```{r}
run_correlations <- function(
    data,
    outcome_var,
    ...,
    # Optional stratification
    stratification_vars = NULL,
    stratification_mode = c("separate", "crossed"),
    # Optional test parameter options
    default_directionality = c("two.sided", "less", "greater"),
    directionality_map = NULL, # (default, bidirectional)
    default_method = c("pearson", "kendall", "spearman"),
    method_map = NULL,
    use = "pairwise.complete.obs",
    # Optional Bonferroni correction
    bonferroni_correct = TRUE,
    bonferroni_scope = c("both", "stratified_only"),
    # Advanced options: how to deal with small/empty strata (or combinations of?)
    drop_na_strata = TRUE,
    # Diagnostic variables
    debug.print = TRUE
) 
```

## Arguments

- `data`: A data frame, data frame extension (e.g. a tibble), or a lazy data frame.

- `outcome_var`: The name of the outcome variable for the correlation, as a string or symbol.

- `...`: `<tidy-select>` Predictors to run correlations with against the `outcome_var`.

    # Optional stratification
    stratification_vars = NULL,
    stratification_mode = c("separate", "crossed"),
    # Optional test parameter options
    default_directionality = c("two.sided", "less", "greater"),
    directionality_map = NULL, # (default, bidirectional)
    default_method = c("pearson", "kendall", "spearman"),
    method_map = NULL,
    use = "pairwise.complete.obs",
    # Optional Bonferroni correction
    bonferroni_correct = TRUE,
    bonferroni_scope = c("both", "stratified_only"),
    # Advanced options
    drop_na_strata = TRUE,
    debug_print = TRUE
    
**Optional stratification**

- `stratification_vars`: `<tidy-select>` By default, `NULL`; otherwise, ordinal/
categorical variables to act as stratification variables, in the sense indicated
by `stratification_mode` (next).

- `stratification_mode`: `"separate"` by default; controls how the variables
selected by `stratification_vars` are used to generate subgroups of observations,
by variable level, to compute correlations over. The value can be:

  - `"separate"`: correlations computed over (a) over the whole dataset,
  and then (b) for each level of each stratification variable, over the resulting
  subset of observations;
  
  - `"crossed"`: correlations are computed over (a) the whole dataset, and then
  (b) for each combination of levels in stratification variables (i.e. each level in
  the n-way interaction of all stratification variables), over the resulting
  subset of observations.
    
**Optional test parameter options**

- `default_directionality`: `two.sided` (default). A string indicating the direction
of the alternative hypothesis when testing correlation signficance; one of `"two.sided"`,
`"less"`, or `"greater"`.

- `directionality_map`: optionally, a list of name-string pairs, where the name
is a predictor selected by `...`, and the value is either `"two.sided"`, `"less"`,
or `"greater"`. THis overrides the default hypothesis given above by `default_directionality`.

- `default_method`: `"pearson"` (default). A string indicating the type of correlation
coefficient to compute; one of `"pearson"`, `"kendall"`, or `"spearman"`.

- `method_map`: optionally, a list of name-string pairs, where the name is a
a predictor selected by `...`, and the value is either `"pearson"`, `"kendall"`,
or `"spearman"`, overriding the default method specified in `default_method`.

- `use`: `"pairwise.complete.obs"` (default). An optional string giving a method for computing correlatios
in the presence of missing values. This must be (an abbreviation of) one of the strings
`"everything"`, `"all.obs"`, `"complete.obs"`, `"na.or.complete"`, or `"pairwise.complete.obs"`.
    
**Optional Bonferroni correction**

- `bonferroni_correct`: `TRUE` (default), or a logical indicating whether to compute
Bonferroni-corrected p-values.

- `bonferroni_scope`: if `"both"`, consider non-stratified and stratified subgroup
correlations as one family for tests for Bonferroni correction purposes. If `"stratified_only"`,
only consider stratified subgroup correlations for Bonferroni correction purposes.

**Advanced options for stratification**

- `drop_na_strat`: `TRUE` (default), or a logical indicating whether NAs are considered
valid levels of stratification variables.

- `debug_print`: `TRUE` (default), or a logical indicating whether to print
diagnostic information about expansions of `tidy-select` statements for predictors
and stratification variables.


## Value

A dataframe containing the following columns:

- `stratification_family`: A character string indicating the variable, or crossed
set of variables, whose levels we compute correlations within.

- `...(stratification variables)...`: For each stratification variable specified when
calling `run_correlations`, the level of the predictor variable within which
the current correlation is being run.

- `predictor`: String: predictor variable of the correlation, as part of the group selected
by the `...` above.

- `outcome`: String: outcome variable of the correlation.

- `method`: String: type of correlation statistic computed.

- `n_pair`: Number of full pairs of observations the current correlation statistic
was computed on.

- `cor`: Numeric: the correlation statistic.

- `p_value`: Numeric: non-Bonferroni-corrected p-value.

- `directionality`: String: direction of the alternate hypothesis.

- `p_value`: (optional) Numeric: Bonferroni-corrected p-value, if `bonferroni_corrected == TRUE`.

## Examples

Simple examples:


```{r}
library(datasets)
# Simplest example: no stratification, two-sided tests, etc. Bonferroni correction
# happens by default; you can turn it off with `bonferroni_correct = FALSE`.
mtcars %>% run_correlations(
  mpg,
  disp, hp, drat, wt, qsec
)
# Add stratification; by default, don't consider nested strata;
mtcars %>% run_correlations(
  mpg,
  stratification_vars = c(cyl, gear),
  disp, hp, drat, wt, qsec
)
# You can run analyses by combinations of strata, and subgroups < 3 will be
# dropped;
mtcars %>% run_correlations(
  mpg,
  stratification_vars = c(cyl, gear),
  stratification_mode = "crossed",
  disp, hp, drat, wt, qsec
)
# You can optionally choose between bi- and unidirectional variables:
mtcars %>% run_correlations(
  mpg,
  disp, hp, drat,
  directionality_map = list(disp = "less")
)
# On ordinal variables, we can also optionally conduct Spearman's rho and Kendall's tau;
mtcars %>% run_correlations(
  mpg,
  disp, carb, 
  method_map = c(carb = "spearman")
)
```

Example usage of some of the more advanced options:

```{r}
# You can choose to only Bonferroni-correct within-strata correlations;
mtcars %>% run_correlations(
  mpg,
  stratification_vars = c(cyl, gear),
  stratification_mode = "crossed",
  disp, hp, drat, wt, qsec,
  bonferroni_scope = "stratified_only"
)
```