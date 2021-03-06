---
output: hugodown::hugo_document

slug: parsnip-0-1-2
title: parsnip 0.1.2
date: 2020-07-09
author: Max Kuhn
description: >
    A new version of parsnip bring improvements in how predictors are handled and
    a few other neat features.  

photo:
  url: https://unsplash.com/photos/kJxinkriuB4
  author: Kevin Lanceplaine

# one of: "deep-dive", "learn", "package", "programming", or "other"
categories: [package] 
tags: [parsnip, tidymodels,]
---

We're happy to announce the release of [parsnip](https://parsnip.tidymodels.org/) 0.1.2. parsnip is a unified tidy interface to many modeling techniques. 

You can install it from CRAN with:


```r
install.packages("parsnip")
```

You can see a full list of changes in the [release notes](https://parsnip.tidymodels.org/news/index.html). I'll highlight the big changes here. The primary improvement is related to the brand new versions of the [hardhat](https://hardhat.tidymodels.org/news/index.html) and [workflows](https://workflows.tidymodels.org/news/index.html) packages. 
 



## Predictor encoding consistency

Normally, when you give a formula to an R modeling function, the standard `model.matrix()` machinery converts factor predictors to a set of binary indicator columns. However, there are a few notable exceptions: 

* Tree-based models, such as CART, C5.0, and random forests, don't require binary indicator variables since their splitting methods can create groups of categories. When using a tree-based model function like `ranger::ranger()`, the formula does _not_ create indicators; the factor predictors are left as factors.  

* Naive Bayes models would rather have the predictors in their native format as well, so that the predictors' conditional distributions are estimated using discrete probability distributions. 

* Multi-level models, such as mixed models or Bayesian hierarchical models, would prefer that the columns that are associated with random effects (e.g. subject) remain factors. 

When `parsnip::fit()` is used with a modeling function that takes a formula, the formula is directly passed to the underlying model function (without processing the data). The resulting model is the same as what the underlying model would have produced. For example: 

```r
# Using older versions of: 
library(parsnip)
library(ranger)
library(modeldata)

data(penguins)
penguins <- na.omit(penguins)

rf_spec <-
  rand_forest() %>%
  set_engine("ranger", seed = 1221) %>%
  set_mode("regression")

rf_spec %>%
  fit(body_mass_g ~ species + island, data = penguins)
```

```
## parsnip model object
## 
## Fit time:  24ms 
## Ranger result
## 
## Call:
##  ranger::ranger(formula = formula, data = data, seed = ~1221, num.threads = 1, verbose = FALSE) 
## 
## Type:                             Regression 
## Number of trees:                  500 
## Sample size:                      333 
## Number of independent variables:  2 
## Mtry:                             1 
## Target node size:                 5 
## Variable importance mode:         none 
## Splitrule:                        variance 
## OOB prediction error (MSE):       224771.2 
## R squared (OOB):                  0.6533301
```

```r
ranger(body_mass_g ~ species + island, data = penguins, seed = 1221)
```

```
## Ranger result
## 
## Call:
##  ranger(body_mass_g ~ species + island, data = penguins, seed = 1221) 
## 
## Type:                             Regression 
## Number of trees:                  500 
## Sample size:                      333 
## Number of independent variables:  2 
## Mtry:                             1 
## Target node size:                 5 
## Variable importance mode:         none 
## Splitrule:                        variance 
## OOB prediction error (MSE):       224771.2 
## R squared (OOB):                  0.6533301
```

(Note the `Number of independent variables:  2`). 

However, the workflows package _does_ process the data before giving it to the modeling function. In this case using the previous version of these packages, indicators were produced when a formula was used. As a result, instead of two predictor columns, the `species` variable was expanded and five predictor columns are given to the model: 


```r
library(workflows)
library(hardhat)

rf_wflow <-
  workflow() %>%
  add_model(rf_spec) %>%
  add_formula(body_mass_g ~ species + island)

rf_wflow %>%
  fit(data = penguins)
```

```
## ══ Workflow [trained] ══════════════════════════════════════════════════════════════════════
## Preprocessor: Formula
## Model: rand_forest()
## 
## ── Preprocessor ────────────────────────────────────────────────────────────────────────────
## body_mass_g ~ species + island
## 
## ── Model ───────────────────────────────────────────────────────────────────────────────────
## Ranger result
## 
## Call:
##  ranger::ranger(formula = formula, data = data, seed = ~1221, num.threads = 1, verbose = FALSE) 
## 
## Type:                             Regression 
## Number of trees:                  500 
## Sample size:                      333 
## Number of independent variables:  5 
## Mtry:                             2 
## Target node size:                 5 
## Variable importance mode:         none 
## Splitrule:                        variance 
## OOB prediction error (MSE):       215619.5 
## R squared (OOB):                  0.6674451
```

Not only was the inconsistency of these two interfaces (parsnip vs. workflows) a problem, but ranger is very persnickety about column names and some indicator columns would result in errors (see, for example, [this issue](https://github.com/tidymodels/tune/issues/151)).

The new set of hardhat/workflows/parsnip versions now fixes this behavior. In parsnip, each model/engine combination has a recommended set of predictor encoding methods attached to them (including a "leave my data alone" option). These are designed to be consistent with what the underlying model function expects so that there are no inconsistencies. 

You can override these new default encoding methods by using a recipe (instead of a formula) or by using a hardhat `blueprint`. 

## One-hot encodings

A full one-hot-encoding method is now available via parsnip using a contrast function. This would generate the full set of indicators for _each factor predictor_. Using `model.matrix(~ 0 + factor, data)` _kind of_ does this, but only for the first factor: 


```r
library(tidymodels)
unique_levels <- 
  penguins %>% 
  select(species, island) %>% 
  distinct()

levels(unique_levels$species)
```

```
## [1] "Adelie"    "Chinstrap" "Gentoo"
```

```r
levels(unique_levels$island)
```

```
## [1] "Biscoe"    "Dream"     "Torgersen"
```

```r
model.matrix(~ 0 + species + island, data = unique_levels)
```

```
##   speciesAdelie speciesChinstrap speciesGentoo islandDream islandTorgersen
## 1             1                0             0           0               1
## 2             1                0             0           0               0
## 3             1                0             0           1               0
## 4             0                0             1           0               0
## 5             0                1             0           1               0
## attr(,"assign")
## [1] 1 1 1 2 2
## attr(,"contrasts")
## attr(,"contrasts")$species
## [1] "contr.treatment"
## 
## attr(,"contrasts")$island
## [1] "contr.treatment"
```

Notice that there are three indicators for `species` but two for `island`.

parsnip now has a contrast function that produces the whole set: 



```r
old_contr <- options("contrasts")$contrasts
new_contr <- old_contr
new_contr["unordered"] <- "contr_one_hot"
options(contrasts = new_contr)

model.matrix(~ species + island, data = unique_levels)
```

```
##   (Intercept) speciesAdelie speciesChinstrap speciesGentoo islandBiscoe
## 1           1             1                0             0            0
## 2           1             1                0             0            1
## 3           1             1                0             0            0
## 4           1             0                0             1            1
## 5           1             0                1             0            0
##   islandDream islandTorgersen
## 1           0               1
## 2           0               0
## 3           1               0
## 4           0               0
## 5           1               0
## attr(,"assign")
## [1] 0 1 1 1 2 2 2
## attr(,"contrasts")
## attr(,"contrasts")$species
## [1] "contr_one_hot"
## 
## attr(,"contrasts")$island
## [1] "contr_one_hot"
```

```r
# return to original options
options(contrasts = old_contr)
```

Now, removing the intercept does not change the nature of the indicator columns.

## Better call objects

In the output above when the the ranger model was fit via parsnip, the model formula in the ranger object what not the same as what we gave to `parsnip::fit()`: 

```
## parsnip model object
## 
## Fit time:  24ms 
## Ranger result
## 
## Call:
##  ranger::ranger(formula = formula, data = data, seed = ~1221, num.threads = 1, verbose = FALSE) 
```

In the new version of parsnip, if you use `parsnip::fit()` and the underlying model uses a formula, the formula is preserved. Here's an example using CART trees:


```r
cart_spec <-
  decision_tree() %>%
  set_engine("rpart") %>%
  set_mode("regression")

cart_fit <- 
  cart_spec %>%
  fit(body_mass_g ~ species + island, data = penguins)

cart_fit
```

```
## parsnip model object
## 
## Fit time:  2ms 
## n=342 (2 observations deleted due to missingness)
## 
## node), split, n, deviance, yval
##       * denotes terminal node
## 
## 1) root 342 219307700 4201.754  
##   2) species=Adelie,Chinstrap 219  41488530 3710.731 *
##   3) species=Gentoo 123  31004250 5076.016 *
```

```r
cart_fit$fit$call
```

```
## rpart::rpart(formula = body_mass_g ~ species + island, data = data)
```

The call still uses `data` instead of `penguins`. To get the right `data` name, there is a `repair_call()` function that can be used to get the exact data set name:


```r
cart_fit <- repair_call(cart_fit, data = penguins)
cart_fit$fit$call
```

```
## rpart::rpart(formula = body_mass_g ~ species + island, data = penguins)
```

There are some R packages that use the object's formula in other functions. For example, the new [`ggparty`](https://github.com/martin-borkovec/ggparty) package has some pretty cool methods for plotting tree-based models. To use them, the model must be converted into a `party` object and this requires a proper call object. Now that we have one, we can do the conversion from an `rpart` object to a `party` object:  


```r
library(partykit)
cart_party <- as.party(cart_fit$fit)
cart_party
```

```
## 
## Model formula:
## body_mass_g ~ species + island
## 
## Fitted party:
## [1] root
## |   [2] species in Adelie, Chinstrap: 3710.731 (n = 219, err = 41488533.1)
## |   [3] species in Gentoo: 5076.016 (n = 123, err = 31004248.0)
## 
## Number of inner nodes:    1
## Number of terminal nodes: 2
```

This can be used with `ggparty`:


```r
library(ggparty)
ggparty(cart_party) +
  geom_edge() +
  geom_edge_label() +
  geom_node_splitvar() +
  geom_node_plot(
    gglist = list(geom_histogram(aes(x = body_mass_g), bins = 15, col = "white"))
  )
```

<img src="figure/ggparty-1.svg" title="plot of chunk ggparty" alt="plot of chunk ggparty" width="60%" style="display: block; margin: auto;" />

## Other new features

 * A new main argument was added to [`boost_tree()`](https://parsnip.tidymodels.org/reference/boost_tree.html) called `stop_iter` for early stopping. The `xgb_train()` function gained arguments for early stopping and a percentage of data to leave out for a validation set. 

* The `predict()` method for `model_fit`s now checks to see if required modeling packages are installed. The packages are loaded (but not attached). 

 * The function `req_pkgs()` is a user interface to determining the required packages. 

## Acknowledgements

Many thanks to Julia Silge and Davis Vaughan for their patience and insights when discussing strategies for predictor encodings. 

Also, we want to thank everyone who contributed changes or issues since the last release: [&#x0040;cgoo4](https://github.com/cgoo4),  [&#x0040;Deleetdk](https://github.com/Deleetdk), [&#x0040;EllaKaye](https://github.com/EllaKaye), [&#x0040;EmilHvitfeldt](https://github.com/EmilHvitfeldt), [&#x0040;enixam](https://github.com/enixam), [&#x0040;FranciscoPalomares](https://github.com/FranciscoPalomares), [&#x0040;FrieseWoudloper](https://github.com/FrieseWoudloper), [&#x0040;hadley](https://github.com/hadley), [&#x0040;jtelleriar](https://github.com/jtelleriar), [&#x0040;kylegilde](https://github.com/kylegilde), [&#x0040;markbneal](https://github.com/markbneal), [&#x0040;markhwhiteii](https://github.com/markhwhiteii), [&#x0040;mdancho84](https://github.com/mdancho84), [&#x0040;oude-gao](https://github.com/oude-gao), [&#x0040;ouzor](https://github.com/ouzor), [&#x0040;RichardPilbery](https://github.com/RichardPilbery), [&#x0040;rorynolan](https://github.com/rorynolan), [&#x0040;simonpcouch](https://github.com/simonpcouch), [&#x0040;StefanBRas](https://github.com/StefanBRas), [&#x0040;stevenpawley](https://github.com/stevenpawley), [&#x0040;ThomasWolf0701](https://github.com/ThomasWolf0701), and [&#x0040;UnclAlDeveloper](https://github.com/UnclAlDeveloper).
 
