---
title: "tune several models with mlr"
author: "Satoshi Kato"
date: "2019/02/22"
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



# install packages


```r
install.packages(c("mlr"))
install.packages(c("glmnet", "kernlab", "randomForest", "gbm"))

install.packages(c("DALEX", "iml"), dependencies = TRUE)
```


# Data

```r
data(apartments, package = "DALEX")
data(apartmentsTest, package = "DALEX")

apartments %>% str()
```

```
'data.frame':	1000 obs. of  6 variables:
 $ m2.price         : num  5897 1818 3643 3517 3013 ...
 $ construction.year: num  1953 1992 1937 1995 1992 ...
 $ surface          : num  25 143 56 93 144 61 127 105 145 112 ...
 $ floor            : int  3 9 1 7 6 6 8 8 6 9 ...
 $ no.rooms         : num  1 5 2 3 5 2 5 4 6 4 ...
 $ district         : Factor w/ 10 levels "Bemowo","Bielany",..: 6 2 5 4 3 6 3 7 6 6 ...
```

# setup mlr task, tune and resample.

regression task for `apartments` dataset.



```r
task <- makeRegrTask(id = "ap", data = apartments, target = "m2.price")
task %>% print()
```

```
Supervised task: ap
Type: regr
Target: m2.price
Observations: 1000
Features:
   numerics     factors     ordered functionals 
          4           1           0           0 
Missings: FALSE
Has weights: FALSE
Has blocking: FALSE
Has coordinates: FALSE
```

```r
tune.ctrl <- makeTuneControlRandom()
tune.ctrl %>% print()
```

```
Tune control: TuneControlRandom
Same resampling instance: TRUE
Imputation value: <worst>
Start: <NULL>
Budget: 100
Tune threshold: FALSE
Further arguments: maxit=100
```

```r
res.desc <- makeResampleDesc("CV", iters = 2)
res.desc %>% print()
```

```
Resample description: cross-validation with 2 iterations.
Predict: test
Stratification: FALSE
```

# choose model 


```r
listLearners(task) %>%
  select(class, short.name, package)
```

```
             class  short.name     package
1 regr.bartMachine bartmachine bartMachine
2       regr.bcart       bcart         tgp
3        regr.brnn        brnn        brnn
4        regr.btgp        btgp         tgp
5     regr.btgpllm     btgpllm         tgp
6        regr.btlm        btlm         tgp
... (#rows: 41, #cols: 3)
```

# Model setup 


```r
learner <- NULL
par.set <- NULL
```

## linear regression with penalty (elastincnet)


```r
# getLearnerParamSet(makeLearner("regr.glmnet"))
learner[["enet"]]<- makeLearner("regr.glmnet")
par.set[["enet"]] <- makeParamSet(
  makeNumericParam("alpha", lower = 0, upper = 1),
  makeNumericParam("s",     lower = 1, upper = 10^3))
```

## Support vector machine (SVM) 


```r
# getLearnerParamSet(makeLearner("regr.ksvm"))
learner[["svm"]]  <- makeLearner("regr.ksvm", kernel = "rbfdot")
par.set[["svm"]]  <- makeParamSet(
  makeNumericParam("C",     lower = -3, upper = 3, trafo = function(x) 10^x),
  makeNumericParam("sigma", lower = -3, upper = 3, trafo = function(x) 10^x))
```

## random forest (RF)


```r
# getLearnerParamSet(makeLearner("regr.randomForest"))
learner[["rf"]] <- makeLearner("regr.randomForest")
par.set[["rf"]] <- makeParamSet(
  makeIntegerParam("ntree", lower=50, upper=1000))
```

## Gradient Boosting Machine (GBM)


```r
# getLearnerParamSet(makeLearner("regr.gbm"))
learner[["gbm"]] <- makeLearner("regr.gbm")
par.set[["gbm"]] <- makeParamSet(
  makeIntegerParam("n.trees",           lower = 3L, upper = 50L),
  makeIntegerParam("interaction.depth", lower = 3L, upper = 20L))
```


```r
learner %>% print()
```

```
$enet
Learner regr.glmnet from package glmnet
Type: regr
Name: GLM with Lasso or Elasticnet Regularization; Short name: glmnet
Class: regr.glmnet
Properties: numerics,factors,ordered,weights
Predict-Type: response
Hyperparameters: s=0.01


$svm
Learner regr.ksvm from package kernlab
Type: regr
Name: Support Vector Machines; Short name: ksvm
Class: regr.ksvm
Properties: numerics,factors
Predict-Type: response
Hyperparameters: fit=FALSE,kernel=rbfdot


$rf
Learner regr.randomForest from package randomForest
Type: regr
Name: Random Forest; Short name: rf
Class: regr.randomForest
Properties: numerics,factors,ordered,se,oobpreds,featimp
Predict-Type: response
Hyperparameters: 


$gbm
Learner regr.gbm from package gbm
Type: regr
Name: Gradient Boosting Machine; Short name: gbm
Class: regr.gbm
Properties: missings,numerics,factors,weights,featimp
Predict-Type: response
Hyperparameters: distribution=gaussian,keep.data=FALSE
```

```r
par.set %>% print()
```

```
$enet
         Type len Def     Constr Req Tunable Trafo
alpha numeric   -   -     0 to 1   -    TRUE     -
s     numeric   -   - 1 to 1e+03   -    TRUE     -

$svm
         Type len Def  Constr Req Tunable Trafo
C     numeric   -   - -3 to 3   -    TRUE     Y
sigma numeric   -   - -3 to 3   -    TRUE     Y

$rf
         Type len Def      Constr Req Tunable Trafo
ntree integer   -   - 50 to 1e+03   -    TRUE     -

$gbm
                     Type len Def  Constr Req Tunable Trafo
n.trees           integer   -   - 3 to 50   -    TRUE     -
interaction.depth integer   -   - 3 to 20   -    TRUE     -
```

# tune model


```r
model.labels <- names(learner)

tuned.par.set <- NULL

for(model.name in model.labels) {
  
  # print(model.name)
  tuned.par.set[[model.name]] <- tuneParams(
    learner[[model.name]], 
    task = task, 
    resampling = res.desc,
    par.set = par.set[[model.name]],
    control = tune.ctrl)
}

tuned.par.set %>% print()
```

```
$enet
Tune result:
Op. pars: alpha=0.835; s=9.46
mse.test.mean=79193.6978653

$svm
Tune result:
Op. pars: C=38.7; sigma=0.0124
mse.test.mean=21675.5159546

$rf
Tune result:
Op. pars: ntree=66
mse.test.mean=92316.4520008

$gbm
Tune result:
Op. pars: n.trees=50; interaction.depth=11
mse.test.mean=13497.7322391
```

## Create a new model using tuned hyperparameters


```r
tuned.learner <- list()

for(model.name in model.labels) {
  
  # print(model.name)
  tuned.learner[[model.name]] <- setHyperPars(
    learner = learner[[model.name]],
    par.vals = tuned.par.set[[model.name]]$x
  )

}

tuned.learner %>% print()
```

```
$enet
Learner regr.glmnet from package glmnet
Type: regr
Name: GLM with Lasso or Elasticnet Regularization; Short name: glmnet
Class: regr.glmnet
Properties: numerics,factors,ordered,weights
Predict-Type: response
Hyperparameters: s=9.46,alpha=0.835


$svm
Learner regr.ksvm from package kernlab
Type: regr
Name: Support Vector Machines; Short name: ksvm
Class: regr.ksvm
Properties: numerics,factors
Predict-Type: response
Hyperparameters: fit=FALSE,kernel=rbfdot,C=38.7,sigma=0.0124


$rf
Learner regr.randomForest from package randomForest
Type: regr
Name: Random Forest; Short name: rf
Class: regr.randomForest
Properties: numerics,factors,ordered,se,oobpreds,featimp
Predict-Type: response
Hyperparameters: ntree=66


$gbm
Learner regr.gbm from package gbm
Type: regr
Name: Gradient Boosting Machine; Short name: gbm
Class: regr.gbm
Properties: missings,numerics,factors,weights,featimp
Predict-Type: response
Hyperparameters: distribution=gaussian,keep.data=FALSE,n.trees=50,interaction.depth=11
```

## Re-train parameters using tuned hyperparameters (and full training set)
  

```r
model.labels <- names(learner)

tuned.model   <- NULL

for(model.name in model.labels) {
  tuned.model[[model.name]] <- train(tuned.learner[[model.name]], task)
}

tuned.model %>% print()
```

```
$enet
Model for learner.id=regr.glmnet; learner.class=regr.glmnet
Trained on: task.id = ap; obs = 1000; features = 5
Hyperparameters: s=9.46,alpha=0.835

$svm
Model for learner.id=regr.ksvm; learner.class=regr.ksvm
Trained on: task.id = ap; obs = 1000; features = 5
Hyperparameters: fit=FALSE,kernel=rbfdot,C=38.7,sigma=0.0124

$rf
Model for learner.id=regr.randomForest; learner.class=regr.randomForest
Trained on: task.id = ap; obs = 1000; features = 5
Hyperparameters: ntree=66

$gbm
Model for learner.id=regr.gbm; learner.class=regr.gbm
Trained on: task.id = ap; obs = 1000; features = 5
Hyperparameters: distribution=gaussian,keep.data=FALSE,n.trees=50,interaction.depth=11
```



```r
saveRDS(tuned.model, "./tuned_models.RDS")
```

