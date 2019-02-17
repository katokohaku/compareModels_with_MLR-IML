---
title: "iml package with mlr"
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
Boston.task = makeRegrTask(data = Boston, target = "medv")

models <- c("lasso", "svm", "rf", "xgb")

tuned.model <- readRDS("./tuned_models.RDS")
tuned.model %>% str(2)
List of 4
 $ lasso:List of 8
  ..$ learner      :List of 14
  .. ..- attr(*, "class")= chr [1:4] "regr.glmnet" "RLearnerRegr" "RLearner" "Learner"
  ..$ learner.model:List of 12
  .. ..- attr(*, "class")= chr [1:2] "elnet" "glmnet"
  .. ..- attr(*, "mlr.train.info")=List of 5
  .. .. ..- attr(*, "class")= chr "FixDataInfo"
  ..$ task.desc    :List of 9
  .. ..- attr(*, "class")= chr [1:3] "RegrTaskDesc" "SupervisedTaskDesc" "TaskDesc"
  ..$ subset       : int [1:506] 1 2 3 4 5 6 7 8 9 10 ...
  ..$ features     : chr [1:13] "crim" "zn" "indus" "chas" ...
  ..$ factor.levels: Named list()
  ..$ time         : num 0.002
  ..$ dump         : NULL
  ..- attr(*, "class")= chr "WrappedModel"
 $ svm  :List of 8
  ..$ learner      :List of 14
  .. ..- attr(*, "class")= chr [1:4] "regr.ksvm" "RLearnerRegr" "RLearner" "Learner"
  ..$ learner.model:Formal class 'ksvm' [package "kernlab"] with 24 slots
  ..$ task.desc    :List of 9
  .. ..- attr(*, "class")= chr [1:3] "RegrTaskDesc" "SupervisedTaskDesc" "TaskDesc"
  ..$ subset       : int [1:506] 1 2 3 4 5 6 7 8 9 10 ...
  ..$ features     : chr [1:13] "crim" "zn" "indus" "chas" ...
  ..$ factor.levels: Named list()
  ..$ time         : num 0.12
  ..$ dump         : NULL
  ..- attr(*, "class")= chr "WrappedModel"
 $ rf   :List of 8
  ..$ learner      :List of 14
  .. ..- attr(*, "class")= chr [1:4] "regr.randomForest" "RLearnerRegr" "RLearner" "Learner"
  ..$ learner.model:List of 17
  .. ..- attr(*, "class")= chr "randomForest"
  ..$ task.desc    :List of 9
  .. ..- attr(*, "class")= chr [1:3] "RegrTaskDesc" "SupervisedTaskDesc" "TaskDesc"
  ..$ subset       : int [1:506] 1 2 3 4 5 6 7 8 9 10 ...
  ..$ features     : chr [1:13] "crim" "zn" "indus" "chas" ...
  ..$ factor.levels: Named list()
  ..$ time         : num 0.182
  ..$ dump         : NULL
  ..- attr(*, "class")= chr "WrappedModel"
 $ xgb  :List of 8
  ..$ learner      :List of 14
  .. ..- attr(*, "class")= chr [1:4] "regr.xgboost" "RLearnerRegr" "RLearner" "Learner"
  ..$ learner.model:List of 9
  .. ..- attr(*, "class")= chr "xgb.Booster"
  ..$ task.desc    :List of 9
  .. ..- attr(*, "class")= chr [1:3] "RegrTaskDesc" "SupervisedTaskDesc" "TaskDesc"
  ..$ subset       : int [1:506] 1 2 3 4 5 6 7 8 9 10 ...
  ..$ features     : chr [1:13] "crim" "zn" "indus" "chas" ...
  ..$ factor.levels: Named list()
  ..$ time         : num 0.614
  ..$ dump         : NULL
  ..- attr(*, "class")= chr "WrappedModel"
```

# iml + mlr

according to:

https://www.r-bloggers.com/interpretable-machine-learning-with-iml-and-mlr/



```r
library("iml")
X = Boston[which(names(Boston) != "medv")]

predictor <- Predictor$new(tuned.model[["svm"]], data = X, y = Boston$medv)
```

# Feature importance

We can measure how important each feature was for the predictions with FeatureImp. The feature importance measure works by shuffling each feature and measuring how much the performance drops. For this regression task we choose to measure the loss in performance with the mean absolute error (ÅemaeÅf); another choice would be the mean squared error (ÅemseÅf).


```r
imp = FeatureImp$new(predictor, loss = "mae")
plot(imp)
```

![](iml_mlr_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r

imp %>% str
Classes 'FeatureImp', 'InterpretationMethod', 'R6' <FeatureImp>
  Inherits from: <InterpretationMethod>
  Public:
    clone: function (deep = FALSE) 
    compare: ratio
    initialize: function (predictor, loss, compare = "ratio", n.repetitions = 5, 
    loss: function (actual, predicted) 
    n.repetitions: 5
    original.error: 1.60205214365129
    plot: function (...) 
    predictor: Predictor, R6
    print: function () 
    results: data.frame
  Private:
    aggregate: function () 
    combine.aggregations: function (agg, dat) 
    dataDesign: NULL
    dataSample: data.table, data.frame
    feature.names: NULL
    finished: TRUE
    flush: function () 
    generatePlot: function (sort = TRUE, ...) 
    get.parallel.fct: function (parallel = FALSE) 
    getData: function (...) 
    intervene: function () 
    loss_string: mae
    multiClass: FALSE
    parallel: FALSE
    plotData: gg, ggplot
    predictResults: data.frame
    printParameters: function () 
    q: function (pred) 
    qResults: NULL
    run: function (n) 
    run.prediction: function (dataDesign) 
    sampler: Data, R6
    set_loss: function (loss) 
    weightSamples: function ()  
```
# Partial dependence

Besides learning which features were important, we are interested in how the features influence the predicted outcome. The Partial class implements partial dependence plots and individual conditional expectation curves. Each individual line represents the predictions (y-axis) for one data point when we change one of the features (e.g. ÅelstatÅf on the x-axis). The highlighted line is the point-wise average of the individual lines and equals the partial dependence plot. The marks on the x-axis indicates the distribution of the ÅelstatÅf feature, showing how relevant a region is for interpretation (little or no points mean that we should not over-interpret this region).


```r
pdp.obj = Partial$new(predictor, feature = "lstat")
plot(pdp.obj)
```

![](iml_mlr_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
pdp.obj$set.feature("rm")
pdp.obj$center(min(Boston$rm))
plot(pdp.obj)
```

![](iml_mlr_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

Surrogate model
Another way to make the models more interpretable is to replace the black box with a simpler model ??? a decision tree. We take the predictions of the black box model (in our case the random forest) and train a decision tree on the original features and the predicted outcome.
The plot shows the terminal nodes of the fitted tree.
The maxdepth parameter controls how deep the tree can grow and therefore how interpretable it is.



```r
tree = TreeSurrogate$new(predictor, maxdepth = 2)
plot(tree)
```

![](iml_mlr_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

# Explain single predictions with a local model

Global surrogate model can improve the understanding of the global model behaviour.
We can also fit a model locally to understand an individual prediction better. The local model fitted by LocalModel is a linear regression model and the data points are weighted by how close they are to the data point for wich we want to explain the prediction.


```r

lime.explain = LocalModel$new(predictor, x.interest = X[1,])
Loading required package: glmnet
Loading required package: Matrix

Attaching package: 'Matrix'
The following object is masked from 'package:tidyr':

    expand
Loading required package: foreach

Attaching package: 'foreach'
The following objects are masked from 'package:purrr':

    accumulate, when
Loaded glmnet 2.0-16
Loading required package: gower
lime.explain$results
              beta x.recoded    effect x.original feature feature.value
rm       4.3808881     6.575 28.804339      6.575      rm      rm=6.575
ptratio -0.5473573    15.300 -8.374566       15.3 ptratio  ptratio=15.3
lstat   -0.4197536     4.980 -2.090373       4.98   lstat    lstat=4.98
```


```r
plot(lime.explain)
```

![](iml_mlr_files/figure-html/unnamed-chunk-7-1.png)<!-- -->


# Explain single predictions with game theory

An alternative for explaining individual predictions is a method from coalitional game theory named Shapley value.
Assume that for one data point, the feature values play a game together, in which they get the prediction as a payout. The Shapley value tells us how to fairly distribute the payout among the feature values.


```r

shapley = Shapley$new(predictor, x.interest = X[1,])
plot(shapley)
```

![](iml_mlr_files/figure-html/unnamed-chunk-8-1.png)<!-- -->


The results in data.frame form can be extracted like this:


```r
results = shapley$results
head(results)
  feature         phi    phi.var feature.value
1    crim  0.29255560  0.7858053  crim=0.00632
2      zn -0.62692400  1.2805874         zn=18
3   indus -1.63013191 11.8363462    indus=2.31
4    chas -0.02189959  2.3488339        chas=0
5     nox  0.25791505  5.9126934     nox=0.538
6      rm  1.82456312 21.6518296      rm=6.575
```

