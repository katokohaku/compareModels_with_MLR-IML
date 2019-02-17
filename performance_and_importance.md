---
title: "measure performance and feature importances with DALEX + mlr"
author: "Satoshi Kato"
date: "2019/02/17"
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

regression task for Boston dataset.


```r
data("Boston", package  = "MASS")
Boston.task <- makeRegrTask(data = Boston, target = "medv")

model.regr.RF <- makeLearner("regr.randomForest") %>% 
  train(Boston.task)

# tuned.model <- readRDS("./tuned_models.RDS")
# model.names <- names(tuned.model)
# tuned.model %>% str(2)
```

# DALEX + mlr

according to:

https://rawgit.com/pbiecek/DALEX_docs/master/vignettes/DALEX_mlr.html

# The explain() function

## prepare customized predit()

For the models created by mlr package we have to provide custom predict function which takes two arguments: model and newdata and returns a numeric vector with predictions because function predict() from mlr returns not only predictions but an object with more information.


```r
predictMLR <- function(object, newdata) {
  pred <- predict(object, newdata=newdata)
  response <- pred$data$response
  return(response)
}
```

## build explainer


```r
library("DALEX")
Welcome to DALEX (version: 0.2.6).

Attaching package: 'DALEX'
The following object is masked from 'package:dplyr':

    explain

explainer <-  DALEX::explain(model = model.regr.RF,
                             data  = Boston, 
                             y     = Boston$medv,
                             predict_function = predictMLR)
```

# measure model performance

Empirical Cumulative Distribution Function (ecdf) of residual error is as default.


```r
mp <- model_performance(explainer)
str(mp)
Classes 'model_performance_explainer' and 'data.frame':	506 obs. of  4 variables:
 $ predicted: num  25.6 22.5 35.2 34.6 34.9 ...
 $ observed : num  24 21.6 34.7 33.4 36.2 28.7 22.9 27.1 16.5 18.9 ...
 $ diff     : num  1.592 0.855 0.461 1.158 -1.295 ...
 $ label    : chr  "WrappedModel" "WrappedModel" "WrappedModel" "WrappedModel" ...
plot(mp)
```

![](performance_and_importance_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
plot(mp, geom = "boxplot")
```

![](performance_and_importance_files/figure-html/unnamed-chunk-3-2.png)<!-- -->

# Variable importance

## simple

`type="raw"` is as default.


```r
var.imp <- variable_importance(explainer     = explainer, 
                               loss_function = loss_root_mean_square)
plot(var.imp)
```

![](performance_and_importance_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
print(var.imp)
       variable dropout_loss        label
1  _full_model_     1.471415 WrappedModel
2          medv     1.471415 WrappedModel
3            zn     1.522595 WrappedModel
4          chas     1.548049 WrappedModel
5           rad     1.611237 WrappedModel
6         black     1.768830 WrappedModel
7           age     1.941574 WrappedModel
8           tax     2.037273 WrappedModel
9         indus     2.096454 WrappedModel
10      ptratio     2.347358 WrappedModel
11         crim     2.494592 WrappedModel
12          nox     2.573429 WrappedModel
13          dis     2.578693 WrappedModel
14           rm     4.904256 WrappedModel
15        lstat     6.387729 WrappedModel
16   _baseline_    12.638205 WrappedModel
```

## set baseline from fullmodel

For better comparison of the models we can hook the variabe importance at 0 using the type="difference", returns `drop_loss - drop_loss_full_model`.

```r
var.imp.diff <- variable_importance(explainer     = explainer,
                                    loss_function = loss_root_mean_square,
                                    type          = "difference")

plot(var.imp.diff)
```

![](performance_and_importance_files/figure-html/unnamed-chunk-5-1.png)<!-- -->
## model comparison

TBD
