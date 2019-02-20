---
title: "measure variable responces of continuous feature with DALEX + mlr"
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

## ceteris paribus plot

```r
install.packages("ceterisParibus", dependencies = TRUE)
```


```r
library("ceterisParibus")
```

```
Loading required package: gower
```

```r
single.instance <- apartmentsTest[7, ]
profile_rf <- ceteris_paribus(explainer.rf, 
                              observations = single.instance)
plot(profile_rf, color = "blue")
```

![](020_variable_responce_continuous_files/figure-html/ceteris_paribus-1.png)<!-- -->

## ICE plot with neghbor instances

we highlight residuals with red intervals. Residuals here are relatively small what suggest that around the point of interest the fit is relatively good.

Blue point stands for the point of interests. Red points are the other.obs. The blue curve is the Ceteris Paribus profile for the blue observation. The red intervals are residuals - differences between the true label and the model response. The grey profiles are Ceteris Paribus profiles for other.obs.


```r
target.feature = "construction.year"
```


```r
other.obs <- sample_n(apartmentsTest, 10)

profile.rf.neig  <- ceteris_paribus(explainer.rf,  
                                    observations = other.obs, 
                                    y = other.obs$m2.price)

cp.neigh <- plot(
  profile.rf.neig, 
  selected_variables = target.feature,
  show_profiles = TRUE, 
  show_observations = TRUE,
  color = "black", 
  alpha = 0.2 
  # show_residuals = TRUE,
  # size_residuals = 2,
  # color_residuals = "red"
) +
  ceteris_paribus_layer(
    show_observations = TRUE,
    profile_rf,
    selected_variables = target.feature,
    size = 1, alpha = 1, color = "blue") 

cp.neigh
```

![](020_variable_responce_continuous_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
cp.neigh +
  ceteris_paribus_layer(
    profile.rf.neig, 
    selected_variables = target.feature,
    aggregate_profiles = mean,
    show_observations = FALSE,
    size = 2, alpha = 1, color = "red")
```

![](020_variable_responce_continuous_files/figure-html/unnamed-chunk-6-1.png)<!-- -->


```r
apartmentsTest.sub <- apartmentsTest %>% sample_n(500)
profile.rf.sub  <- ceteris_paribus(explainer.rf,  
                                   observations = apartmentsTest.sub, 
                                   y = apartmentsTest.sub$m2.price)

plot(profile.rf.sub, 
     selected_variables = target.feature,
     show_residuals = FALSE,
     show_observations = TRUE,
     size = 1, alpha = 0.1) +
  ceteris_paribus_layer(
    profile.rf.sub, 
    selected_variables = target.feature,
    aggregate_profiles = mean,
    show_observations = FALSE,
    size = 2, alpha = 1, color = "red")
```

![](020_variable_responce_continuous_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

## Partial Dependence Plot

according to: 

https://pbiecek.github.io/DALEX_docs/5-1-cetParSingleObseSingleModel.html

### single model


```r
pdp  <- variable_response(explainer.rf, variable =  target.feature, type = "pdp")

plot(pdp)
```

![](020_variable_responce_continuous_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

### model comparison


```r
PDPs <- list()
for(model.name in model.labels){
  PDPs[[model.name]] <- variable_response(explainer[[model.name]],
                                         variable =  target.feature,
                                         type = "pdp")
}
plot.pdps <- plot(PDPs[["enet"]], 
                  PDPs[["svm"]], 
                  PDPs[["rf"]], 
                  PDPs[["gbm"]]) + 
  ggtitle("PD plot")

plot.pdps
```

![](020_variable_responce_continuous_files/figure-html/unnamed-chunk-9-1.png)<!-- -->



## Acumulated Local Effects plot

```r
ale  <- variable_response(explainer.rf, variable =  target.feature, type = "ale")

plot(ale)
```

![](020_variable_responce_continuous_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

### model comparison


```r
ALEs <- list()
for(model.name in model.labels){
  ALEs[[model.name]] <- variable_response(explainer[[model.name]],
                                         variable =  target.feature,
                                         type = "ale")
}
plot.ales <- plot(ALEs[["enet"]], 
                  ALEs[["svm"]], 
                  ALEs[["rf"]], 
                  ALEs[["gbm"]]) + 
  ggtitle("ALE plot")

plot.ales
```

![](020_variable_responce_continuous_files/figure-html/unnamed-chunk-11-1.png)<!-- -->


```r
gridExtra::grid.arrange(plot.pdps, plot.ales, ncol=2)
```

![](020_variable_responce_continuous_files/figure-html/unnamed-chunk-12-1.png)<!-- -->


