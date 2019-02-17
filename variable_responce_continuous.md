---
title: "measure variable responces of continuous feature with DALEX + mlr"
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

explainer.rf <-  DALEX::explain(model = model.regr.RF,
                             data  = Boston, 
                             y     = Boston$medv,
                             predict_function = predictMLR)
```

# Variable response
Explainers presented in this section are designed to better understand the relation between a variable and model output.

For more details of methods desribed in this section see Variable response section in DALEX docs.

## ceteris paribus plot


```r
library("ceterisParibus")
Loading required package: gower
single.instance <- Boston[1, ]
profile_rf <- ceteris_paribus(explainer.rf, observations = single.instance)
plot(profile_rf)#, selected_variables = "dis")
```

![](variable_responce_continuous_files/figure-html/ceteris_paribus-1.png)<!-- -->

## ICE plot with neghbor instances

we highlight residuals with red intervals. Residuals here are relatively small what suggest that around the point of interest the fit is relatively good.

Blue point stands for the point of interests. Red points are the neighbours. The blue curve is the Ceteris Paribus profile for the blue observation. The red intervals are residuals - differences between the true label and the model response. The grey profiles are Ceteris Paribus profiles for neighbours.


```r
neighbours <- select_neighbours(Boston, observation = single.instance, n = 10)
profile.rf.neig  <- ceteris_paribus(explainer.rf,  
                                    observations = neighbours, 
                                    y = neighbours$medv)
target.feature = "dis"
cp.neigh <- plot(profile.rf.neig, 
     selected_variables = target.feature,
     show_residuals = TRUE,
     size_residuals = 2,
     color_residuals = "red", 
     show_observations = FALSE) +
  ceteris_paribus_layer(
    profile_rf,
    selected_variables = target.feature,
    size = 3, alpha = 1, color = "blue") 

cp.neigh
```

![](variable_responce_continuous_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
cp.neigh +
  ceteris_paribus_layer(
    profile.rf.neig, 
    selected_variables = target.feature,
    aggregate_profiles = mean,
    show_observations = FALSE,
    size = 3, alpha = 1, color = "green")
```

![](variable_responce_continuous_files/figure-html/unnamed-chunk-4-1.png)<!-- -->


```r
profile.rf.all  <- ceteris_paribus(explainer.rf,  
                                    observations = Boston, 
                                    y = Boston$medv)
target.feature = "dis"
plot(profile.rf.all, 
     selected_variables = target.feature,
     show_residuals = FALSE,
     show_observations = TRUE,
     size = 1, alpha = 0.3) +
  ceteris_paribus_layer(
    profile.rf.all, 
    selected_variables = target.feature,
    aggregate_profiles = mean,
    show_observations = FALSE,
    size = 2, alpha = 1, color = "green")
```

![](variable_responce_continuous_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

## Partial Dependence Plot

according to: 

https://pbiecek.github.io/DALEX_docs/5-1-cetParSingleObseSingleModel.html



```r
pdp  <- variable_response(explainer.rf, variable =  target.feature, type = "pdp")

plot(pdp)
```

![](variable_responce_continuous_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

## Acumulated Local Effects plot

```r
ale  <- variable_response(explainer.rf, variable =  target.feature, type = "ale")

plot(ale)
```

![](variable_responce_continuous_files/figure-html/unnamed-chunk-7-1.png)<!-- -->


