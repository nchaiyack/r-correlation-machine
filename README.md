
# `run_correlations`: run lots of correlations against a single variable, with tidyverse-friendly syntax and lots of knobs (stratification, Bonferroni correction, test directionality, ...)

A very, very large and tidyverse-friendly R function.

**Motivation.** A common task that psychometricians do to evaluate the validity of
a measure, to is run a *lot* of correlations against other variables, in order to test "no duh"
relationships. The function `run_correlations` allows you to do so with the
absurd convenience of a "industrial-strength" tidyverse function. Fundamentally,
though, this just calls `stats::cor.test` many, many times.

Here is the simplest example, on `mtcars`, that computes the correlation between
`mpg` and `disp`, and `mpg` and `hp`:

```{r}
> mtcars %>% run_correlations(
+     mpg,
+     disp, hp
+ )
# A tibble: 2 × 9
  stratification_family predictor outcome method  n_pair    cor  p_value directionality       p_bonf
  <chr>                 <chr>     <chr>   <chr>    <int>  <dbl>    <dbl> <chr>                 <dbl>
1 unstratified          disp      mpg     pearson     32 -0.848 9.38e-10 two.sided           1.88e-9
2 unstratified          hp        mpg     pearson     32 -0.776 1.79e- 7 two.sided           3.58e-7
```

## Usage

```{r}
run_correlations <- function(
    data,
    outcome_var,
    
    # Predictor variables
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
    
    # Advanced options
    drop_na_strata = TRUE,
    debug.print = TRUE
) 
```

## Arguments

- `data`: A data frame, data frame extension (e.g. a tibble), or a lazy data frame.

- `outcome_var`: The name of the outcome variable for the correlation, as a string or symbol.

- `...`: `<tidy-select>` Predictors to run correlations with against the `outcome_var`.

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
  
  - Finally, note that all observation subgroups with less than 3 elements are
  dropped from the analysis, for compatibility with `stats::cor.test`.
    
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

## Simple examples

Simplest example: no stratification, two-sided tests, etc. Bonferroni correction
happens by default; you can turn it off with `bonferroni_correct = FALSE`.


```{r}
> mtcars %>% run_correlations(
+     mpg,
+     disp, hp, drat, wt, qsec
+ )
# A tibble: 5 × 9
  stratification_family predictor outcome method  n_pair    cor
  <chr>                 <chr>     <chr>   <chr>    <int>  <dbl>
1 unstratified          disp      mpg     pearson     32 -0.848
2 unstratified          drat      mpg     pearson     32  0.681
3 unstratified          hp        mpg     pearson     32 -0.776
4 unstratified          qsec      mpg     pearson     32  0.419
5 unstratified          wt        mpg     pearson     32 -0.868
# ℹ 3 more variables: p_value <dbl>, directionality <chr>,
#   p_bonf <dbl>
```

Add stratification; by default, don't consider nested strata.

```{r}
> mtcars %>% run_correlations(
+     mpg,
+     stratification_vars = c(cyl, gear),
+     disp, hp, drat, wt, qsec
+ )
# A tibble: 30 × 11
   stratification_family   cyl  gear predictor outcome method  n_pair    cor p_value directionality
   <chr>                 <dbl> <dbl> <chr>     <chr>   <chr>    <int>  <dbl>   <dbl> <chr>         
 1 cyl                       4    NA disp      mpg     pearson     11 -0.805 0.00278 two.sided     
 2 cyl                       4    NA drat      mpg     pearson     11  0.424 0.193   two.sided     
 3 cyl                       4    NA hp        mpg     pearson     11 -0.524 0.0984  two.sided     
 4 cyl                       4    NA qsec      mpg     pearson     11 -0.236 0.485   two.sided     
 5 cyl                       4    NA wt        mpg     pearson     11 -0.713 0.0137  two.sided     
 6 cyl                       6    NA disp      mpg     pearson      7  0.103 0.826   two.sided     
 7 cyl                       6    NA drat      mpg     pearson      7  0.115 0.807   two.sided     
 8 cyl                       6    NA hp        mpg     pearson      7 -0.127 0.786   two.sided     
 9 cyl                       6    NA qsec      mpg     pearson      7 -0.419 0.350   two.sided     
10 cyl                       6    NA wt        mpg     pearson      7 -0.682 0.0918  two.sided     
# ℹ 20 more rows
# ℹ 1 more variable: p_bonf <dbl>
# ℹ Use `print(n = ...)` to see more rows
```

You can run nest strata within each other by `stratification_mode = "crossed"`,
recalling that all resulting data subgroups with less than 3 observations are
not analyzed;

```{r}
> mtcars %>% run_correlations(
+     mpg,
+     stratification_vars = c(cyl, gear),
+     stratification_mode = "crossed",
+     disp, hp, drat, wt, qsec
+ )
# A tibble: 15 × 11
   stratification_family   cyl  gear predictor outcome method  n_pair     cor p_value directionality
   <chr>                 <dbl> <dbl> <chr>     <chr>   <chr>    <int>   <dbl>   <dbl> <chr>         
 1 cyl×gear                  4     4 disp      mpg     pearson      8 -0.818   0.0132 two.sided     
 2 cyl×gear                  4     4 drat      mpg     pearson      8  0.508   0.198  two.sided     
 3 cyl×gear                  4     4 hp        mpg     pearson      8 -0.760   0.0288 two.sided     
 4 cyl×gear                  4     4 qsec      mpg     pearson      8 -0.156   0.712  two.sided     
 5 cyl×gear                  4     4 wt        mpg     pearson      8 -0.732   0.0389 two.sided     
 6 cyl×gear                  6     4 disp      mpg     pearson      4 -0.930   0.0702 two.sided     
 7 cyl×gear                  6     4 drat      mpg     pearson      4 -0.930   0.0702 two.sided     
 8 cyl×gear                  6     4 hp        mpg     pearson      4 -0.930   0.0702 two.sided     
 9 cyl×gear                  6     4 qsec      mpg     pearson      4 -0.968   0.0323 two.sided     
10 cyl×gear                  6     4 wt        mpg     pearson      4 -0.900   0.100  two.sided     
11 cyl×gear                  8     3 disp      mpg     pearson     12 -0.535   0.0729 two.sided     
12 cyl×gear                  8     3 drat      mpg     pearson     12 -0.0190  0.953  two.sided     
13 cyl×gear                  8     3 hp        mpg     pearson     12 -0.507   0.0926 two.sided     
14 cyl×gear                  8     3 qsec      mpg     pearson     12 -0.105   0.746  two.sided     
15 cyl×gear                  8     3 wt        mpg     pearson     12 -0.675   0.0159 two.sided     
# ℹ 1 more variable: p_bonf <dbl>
```

You can optionally choose between undirectional and bidirectional alternate
hypotheses:

```{r}
> mtcars %>% run_correlations(
+     mpg,
+     disp, hp, drat,
+     directionality_map = list(disp = "less")
+ )
# A tibble: 3 × 9
  stratification_family predictor outcome method  n_pair    cor  p_value directionality       p_bonf
  <chr>                 <chr>     <chr>   <chr>    <int>  <dbl>    <dbl> <chr>                 <dbl>
1 unstratified          disp      mpg     pearson     32 -0.848 9.38e-10 less                2.81e-9
2 unstratified          drat      mpg     pearson     32  0.681 1.78e- 5 two.sided           5.33e-5
3 unstratified          hp        mpg     pearson     32 -0.776 1.79e- 7 two.sided           5.36e-7
```

On ordinal variables, we can also optionally conduct Spearman's rho and Kendall's tau;

```{r}
> mtcars %>% run_correlations(
+     mpg,
+     disp, carb, 
+     method_map = c(carb = "spearman")
+ )
Warning: There was 1 warning in `dplyr::summarise()`.
ℹ In argument: `p_value = `$`(...)`.
ℹ In group 1: `predictor = "carb"`.
Caused by warning in `cor.test.default()`:
! Cannot compute exact p-value with ties
ℹ Bonferroni correction for executed tests (m): 2
# A tibble: 2 × 9
  stratification_family predictor outcome method   n_pair    cor  p_value directionality      p_bonf
  <chr>                 <chr>     <chr>   <chr>     <int>  <dbl>    <dbl> <chr>                <dbl>
1 unstratified          carb      mpg     spearman     32 -0.657 4.34e- 5 two.sided          8.68e-5
2 unstratified          disp      mpg     pearson      32 -0.848 9.38e-10 two.sided          1.88e-9
```

Finally, as an advanced option, you can choose to only Bonferroni-correct within-strata correlations;

```{r}
> mtcars %>% run_correlations(
+     mpg,
+     stratification_vars = c(cyl, gear),
+     stratification_mode = "crossed",
+     disp, hp, drat, wt, qsec,
+     bonferroni_scope = "stratified_only"
+ )
# A tibble: 15 × 11
   stratification_family   cyl  gear predictor outcome method  n_pair     cor p_value directionality
   <chr>                 <dbl> <dbl> <chr>     <chr>   <chr>    <int>   <dbl>   <dbl> <chr>         
 1 cyl×gear                  4     4 disp      mpg     pearson      8 -0.818   0.0132 two.sided     
 2 cyl×gear                  4     4 drat      mpg     pearson      8  0.508   0.198  two.sided     
 3 cyl×gear                  4     4 hp        mpg     pearson      8 -0.760   0.0288 two.sided     
 4 cyl×gear                  4     4 qsec      mpg     pearson      8 -0.156   0.712  two.sided     
 5 cyl×gear                  4     4 wt        mpg     pearson      8 -0.732   0.0389 two.sided     
 6 cyl×gear                  6     4 disp      mpg     pearson      4 -0.930   0.0702 two.sided     
 7 cyl×gear                  6     4 drat      mpg     pearson      4 -0.930   0.0702 two.sided     
 8 cyl×gear                  6     4 hp        mpg     pearson      4 -0.930   0.0702 two.sided     
 9 cyl×gear                  6     4 qsec      mpg     pearson      4 -0.968   0.0323 two.sided     
10 cyl×gear                  6     4 wt        mpg     pearson      4 -0.900   0.100  two.sided     
11 cyl×gear                  8     3 disp      mpg     pearson     12 -0.535   0.0729 two.sided     
12 cyl×gear                  8     3 drat      mpg     pearson     12 -0.0190  0.953  two.sided     
13 cyl×gear                  8     3 hp        mpg     pearson     12 -0.507   0.0926 two.sided     
14 cyl×gear                  8     3 qsec      mpg     pearson     12 -0.105   0.746  two.sided     
15 cyl×gear                  8     3 wt        mpg     pearson     12 -0.675   0.0159 two.sided     
# ℹ 1 more variable: p_bonf <dbl>
```