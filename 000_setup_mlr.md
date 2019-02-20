---
title: "tune several models with mlr"
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



# install packages


```r
install.packages(c("mlr"))
install.packages(c("glmnet", "kernlab", "randomForest", "gbm"))

install.packages(c("DALEX", "iml"), dependencies = TRUE)
```

# create an mlr task and model

regression task for Boston dataset.


```r
data(apartments, package = "DALEX")
task <- makeRegrTask(id = "ap", data = apartments, target = "m2.price")
tune.ctrl <- makeTuneControlRandom()
res.desc  <- makeResampleDesc("CV", iters = 2)

learner <- NULL
par.set <- NULL
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

# tune model


```r
model.labels <- names(learner)
# print(model.names)

tuned.par.set <- NULL
tuned.model   <- NULL

for(model.name in model.labels) {
  
  print(model.name)
  tuned.par.set[[model.name]] <- tuneParams(
    learner[[model.name]], 
    task = task, 
    resampling = res.desc,
    par.set = par.set[[model.name]],
    control = tune.ctrl)
  
  # Create a new model using tuned hyperparameters
  tuned.learner <- setHyperPars(
    learner = learner[[model.name]],
    par.vals = tuned.par.set[[model.name]]$x
  )
  
  # Re-train parameters using tuned hyperparameters (and full training set)
  tuned.model[[model.name]] <- train(tuned.learner, task)
  
}
```

```
[1] "enet"
[1] "svm"
[1] "rf"
[1] "gbm"
```

```r
saveRDS(tuned.model, "./tuned_models.RDS")
```


```r
for(model.name in model.labels){
  print(model.name)
  print(tuned.model[[model.name]])
}
```

```
[1] "enet"
Model for learner.id=regr.glmnet; learner.class=regr.glmnet
Trained on: task.id = ap; obs = 1000; features = 5
Hyperparameters: s=9.46,alpha=0.835
[1] "svm"
Model for learner.id=regr.ksvm; learner.class=regr.ksvm
Trained on: task.id = ap; obs = 1000; features = 5
Hyperparameters: fit=FALSE,kernel=rbfdot,C=38.7,sigma=0.0124
[1] "rf"
Model for learner.id=regr.randomForest; learner.class=regr.randomForest
Trained on: task.id = ap; obs = 1000; features = 5
Hyperparameters: ntree=66
[1] "gbm"
Model for learner.id=regr.gbm; learner.class=regr.gbm
Trained on: task.id = ap; obs = 1000; features = 5
Hyperparameters: distribution=gaussian,keep.data=FALSE,n.trees=50,interaction.depth=7
```

