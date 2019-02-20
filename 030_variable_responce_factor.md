---
title: "measure variable responces of categorical feature with DALEX + mlr"
author: "Satoshi Kato"
date: "2019/02/20"
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

# DALEX + mlr

according to:

https://rawgit.com/pbiecek/DALEX_docs/master/vignettes/DALEX_mlr.html

# The explain() function

## custom predict()

For the models created by mlr package we have to provide custom predict function which takes two arguments: model and newdata and returns a numeric vector with predictions because function predict() from mlr returns not only predictions but an object with more information.


```r
predictMLR <- function(object, newdata) {
  pred <- predict(object, newdata=newdata)
  response <- pred$data$response
  return(response)
}
```

# build explainer

## simple


```r
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

model.regr.rf <- tuned.model[["rf"]]

explainer <- list()
explainer.rf <-  explain(model = model.regr.rf, 
                         label = "rf",
                         data  = apartmentsTest %>% select(-m2.price), 
                         y     = apartmentsTest$m2.price,
                         predict_function = predictMLR)
```

## multiple explainers


```r
model.labels <- names(tuned.model)
model.regr   <- list()

for(model.name in model.labels){
  model.regr[[model.name]] <- tuned.model[[model.name]]
  
  explainer[[model.name]] <- explain(model = model.regr[[model.name]], 
                                     label = model.name,
                                     data  = apartmentsTest %>% select(-m2.price), 
                                     y     = apartmentsTest$m2.price,
                                     predict_function = predictMLR)
}
```

# Variable response
Explainers presented in this section are designed to better understand the relation between a variable and model output.

For more details of methods desribed in this section see Variable response section in DALEX docs.


## Merging Path Plots

The package ICEbox does not work for factor variables, while the pdp package returns plots that are hard to interpret.

An interesting tool that helps to understand what happens with factor variables is the factorMerger package. See (Sitko and Biecek 2017Sitko, Agnieszka, and Przemyslaw Biecek. 2017. FactorMerger: Hierarchical Algorithm for Post-Hoc Testing. https://github.com/MI2DataLab/factorMerger.).

Merging Path Plot is a method for exploration of a relation between a categorical variable and model outcome.

Function variable_response() with the parameter type = "factor" calls factorMerger::mergeFactors() function.


```r
mpp <- DALEX::variable_response(explainer.rf, variable =  "district", type = "factor")

plot(mpp)
```

```
Scale for 'x' is already present. Adding another scale for 'x', which
will replace the existing scale.
```

![](030_variable_responce_factor_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

## model comparison


```r
MPPs <- pMPPs <- list()
for(model.name in model.labels){
  MPPs[[model.name]] <- variable_response(explainer[[model.name]],
                                          variable =  "district",
                                          type = "factor")
}

table(apartmentsTest$district) %>% data.frame()
```

```
          Var1 Freq
1       Bemowo  896
2      Bielany  894
3      Mokotow  868
4       Ochota  909
5        Praga  971
6  Srodmiescie  924
7        Ursus  920
8      Ursynow  864
9         Wola  892
10    Zoliborz  862
```

```r
for(model.name in model.labels){
  plot(MPPs[[model.name]]) %>% print()
}
```

```
Scale for 'x' is already present. Adding another scale for 'x', which
will replace the existing scale.
Scale for 'x' is already present. Adding another scale for 'x', which
will replace the existing scale.
```

![](030_variable_responce_factor_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```
Scale for 'x' is already present. Adding another scale for 'x', which
will replace the existing scale.
```

![](030_variable_responce_factor_files/figure-html/unnamed-chunk-5-2.png)<!-- -->

```
Scale for 'x' is already present. Adding another scale for 'x', which
will replace the existing scale.
```

![](030_variable_responce_factor_files/figure-html/unnamed-chunk-5-3.png)<!-- -->![](030_variable_responce_factor_files/figure-html/unnamed-chunk-5-4.png)<!-- -->


