---
title: "measure variable responces of categorical feature with DALEX + mlr"
author: "Satoshi Kato"
date: "2019/02/21"
output:
  html_document:
    fig_caption: yes
    pandoc_args:
    - --from
    - markdown+autolink_bare_uris+tex_math_single_backslash-implicit_figures
    toc: yes
    toc_depth: 4
    keep_md: yes
  word_document:
    toc: yes
    toc_depth: 3
  pdf_document:
    toc: yes
    toc_depth: 3
editor_options: 
  chunk_output_type: inline
---



# read mlr models

regression task for apartments dataset.


```r
tuned.model <- readRDS("./tuned_models.RDS")
# tuned.model %>% str(2)
```


# iml + mlr

according to:

https://www.r-bloggers.com/interpretable-machine-learning-with-iml-and-mlr/


# build predictor

## simple


```r
require("iml")
# X = Boston[which(names(Boston) != "medv")]
require(DALEX)
```

```
Loading required package: DALEX
```

```
Welcome to DALEX (version: 0.2.6).
```

```

Attaching package: 'DALEX'
```

```
The following object is masked from 'package:dplyr':

    explain
```

```r
data("apartmentsTest", package = "DALEX")
X = apartmentsTest %>% select(-m2.price)
Y = apartmentsTest$m2.price

predictor.rf <- Predictor$new(tuned.model[["rf"]], data = X, y = Y)
```


## multiple predictor


```r
model.labels <- names(tuned.model)
predictor    <- list()

for(model.name in model.labels){
  predictor[[model.name]] <- Predictor$new(tuned.model[[model.name]], data = X, y = Y)
}
```


# Measure interactions
 
 We can also measure how strongly features interact with each other. The interaction measure regards how much of the variance of f(x) is explained by the interaction. The measure is between 0 (no interaction) and 1 (= 100% of variance of f(x) due to interactions). For each feature, we measure how much they interact with any other feature:


```r
interact.rf <- Interaction$new(predictor.rf)
plot(interact.rf)
```

![](040_feature_interaction_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

We can also specify a feature and measure all itÅfs 2-way interactions with all other features:


```r
interact.2way.rf <- Interaction$new(predictor.rf, feature = "surface")

plot(interact.2way.rf)
```

![](040_feature_interaction_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

You can also plot the feature effects for all features at once:


```r
effs.a <- FeatureEffects$new(predictor.rf, method="ale")
plot(effs.a)
```

![](040_feature_interaction_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
effs.p <- FeatureEffects$new(predictor.rf, method="pdp+ice")
plot(effs.p)
```

![](040_feature_interaction_files/figure-html/unnamed-chunk-5-2.png)<!-- -->


