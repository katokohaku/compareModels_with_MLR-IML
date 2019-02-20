---
title: "measure performance and feature importances with DALEX + mlr"
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


# measure model performance

## simple

Empirical Cumulative Distribution Function (ecdf) of residual error is as default.


```r
mp <- model_performance(explainer.rf)
str(mp)
```

```
Classes 'model_performance_explainer' and 'data.frame':	9000 obs. of  4 variables:
 $ predicted: num  4187 3319 2744 2682 2904 ...
 $ observed : num  4644 3082 2498 2735 2781 ...
 $ diff     : num  -456.7 237.2 246.2 -53.2 123.3 ...
 $ label    : chr  "rf" "rf" "rf" "rf" ...
```

```r
plot(mp)
```

![](010_performance_and_importance_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
plot(mp, geom = "boxplot")
```

![](010_performance_and_importance_files/figure-html/unnamed-chunk-4-2.png)<!-- -->

## model comparison


```r
mps <- list()
for(model.name in model.labels){
  mps[[model.name]] <- model_performance(explainer[[model.name]])
}
plot(mps[["enet"]], 
     mps[["svm"]], 
     mps[["rf"]], 
     mps[["gbm"]])
```

![](010_performance_and_importance_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
plot(geom = "boxplot",
     mps[["enet"]], 
     mps[["svm"]], 
     mps[["rf"]], 
     mps[["gbm"]])
```

![](010_performance_and_importance_files/figure-html/unnamed-chunk-5-2.png)<!-- -->

# Variable importance

## simple

`type="raw"` is as default.


```r
var.imp <- variable_importance(explainer     = explainer.rf, 
                               loss_function = loss_root_mean_square)
plot(var.imp)
```

![](010_performance_and_importance_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
print(var.imp)
```

```
           variable dropout_loss label
1      _full_model_     293.7500    rf
2 construction.year     411.1187    rf
3          no.rooms     426.4472    rf
4             floor     444.6077    rf
5           surface     469.8710    rf
6          district     860.9642    rf
7        _baseline_    1108.6362    rf
```

## set baseline from fullmodel

For better comparison of the models we can hook the variabe importance at 0 using the type="difference", returns `drop_loss - drop_loss_full_model`.


```r
var.imp.diff <- variable_importance(explainer     = explainer.rf,
                                    loss_function = loss_root_mean_square,
                                    type          = "difference")

plot(var.imp.diff)
```

![](010_performance_and_importance_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

## model comparison



```r
vid <- list()
pvid <- list()
for(model.name in model.labels){
  vid[[model.name]] <- variable_importance(explainer     = explainer[[model.name]],
                                           loss_function = loss_root_mean_square,
                                           type          = "difference")
  pvid[[model.name]] <- plot(vid[[model.name]])
}

plot(vid[["enet"]], 
     vid[["svm"]], 
     vid[["rf"]], 
     vid[["gbm"]])
```

![](010_performance_and_importance_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
# gridExtra::grid.arrange(grobs = pvid)
```
