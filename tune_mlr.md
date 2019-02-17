---
title: "tune several models with mlr"
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



# create an mlr task and model

regression task for Boston dataset.


```r
data("Boston", package  = "MASS")
Boston.task = makeRegrTask(data = Boston, target = "medv")

models <- c("lasso", "svm", "rf", "xgb")

tune.ctrl <- makeTuneControlRandom()
res.desc  <- makeResampleDesc("CV", iters = 2)


learner <- NULL
par.set <- NULL
```

# choose model 


```r
listLearners() %>%
  filter(type == "regr") %>%
  select(class, short.name, package)
```

# Model setup 

## lasso


```r
# getLearnerParamSet(makeLearner("regr.glmnet"))
learner[["lasso"]]<- makeLearner("regr.glmnet", alpha = 1, intercept = FALSE)
par.set[["lasso"]] <- makeParamSet(
  makeIntegerParam("s",     lower=1, upper=10^3))

```

## SVM


```r
# getLearnerParamSet(makeLearner("regr.ksvm"))
learner[["svm"]]  <- makeLearner("regr.ksvm")
par.set[["svm"]]  <- makeParamSet(
  makeNumericParam("C", lower = -3, upper = 3, trafo = function(x) 10^x),
  makeNumericParam("sigma", lower = -3, upper = 3, trafo = function(x) 10^x))
```

## random forest


```r
# getLearnerParamSet(makeLearner("regr.randomForest"))
learner[["rf"]] <- makeLearner("regr.randomForest")
par.set[["rf"]] <- makeParamSet(
  makeIntegerParam("ntree", lower=50, upper=1000))
```

## XGBoost


```r
# getLearnerParamSet(makeLearner("regr.xgboost"))
learner[["xgb"]] <- makeLearner("regr.xgboost", objective = "reg:linear")
par.set[["xgb"]] <- makeParamSet(
  makeIntegerParam("nrounds",   lower = 3L, upper = 50L),
  makeIntegerParam("max_depth", lower = 3L, upper = 20L))
```

# tune model


```r
tuned.par.set <- NULL
tuned.model <- NULL

for(model.name in models) {
  
  print(model.name)
  tuned.par.set[[model.name]] <- tuneParams(
    learner[[model.name]], 
    task = Boston.task, 
    resampling = res.desc,
    par.set = par.set[[model.name]],
    control = tune.ctrl)
  
  # Create a new model using tuned hyperparameters
  tuned.learner <- setHyperPars(
    learner = learner[[model.name]],
    par.vals = tuned.par.set[[model.name]]$x
  )
  
  # Re-train parameters using tuned hyperparameters (and full training set)
  tuned.model[[model.name]] <- train(tuned.learner, Boston.task)
  
}
[1] "lasso"
[Tune] Started tuning learner regr.glmnet for parameter set:
     Type len Def     Constr Req Tunable Trafo
s integer   -   - 1 to 1e+03   -    TRUE     -
With control class: TuneControlRandom
Imputation value: Inf
[Tune-x] 1: s=588
[Tune-y] 1: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 2: s=344
[Tune-y] 2: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 3: s=810
[Tune-y] 3: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 4: s=542
[Tune-y] 4: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 5: s=16
[Tune-y] 5: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 6: s=738
[Tune-y] 6: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 7: s=408
[Tune-y] 7: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 8: s=758
[Tune-y] 8: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 9: s=23
[Tune-y] 9: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 10: s=155
[Tune-y] 10: mse.test.mean=356.0176896; time: 0.0 min
[Tune-x] 11: s=109
[Tune-y] 11: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 12: s=281
[Tune-y] 12: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 13: s=802
[Tune-y] 13: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 14: s=853
[Tune-y] 14: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 15: s=482
[Tune-y] 15: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 16: s=771
[Tune-y] 16: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 17: s=657
[Tune-y] 17: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 18: s=518
[Tune-y] 18: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 19: s=779
[Tune-y] 19: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 20: s=693
[Tune-y] 20: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 21: s=808
[Tune-y] 21: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 22: s=215
[Tune-y] 22: mse.test.mean=571.6919845; time: 0.0 min
[Tune-x] 23: s=105
[Tune-y] 23: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 24: s=219
[Tune-y] 24: mse.test.mean=580.8680498; time: 0.0 min
[Tune-x] 25: s=438
[Tune-y] 25: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 26: s=44
[Tune-y] 26: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 27: s=305
[Tune-y] 27: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 28: s=727
[Tune-y] 28: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 29: s=661
[Tune-y] 29: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 30: s=236
[Tune-y] 30: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 31: s=714
[Tune-y] 31: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 32: s=844
[Tune-y] 32: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 33: s=767
[Tune-y] 33: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 34: s=794
[Tune-y] 34: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 35: s=340
[Tune-y] 35: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 36: s=315
[Tune-y] 36: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 37: s=517
[Tune-y] 37: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 38: s=505
[Tune-y] 38: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 39: s=589
[Tune-y] 39: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 40: s=973
[Tune-y] 40: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 41: s=785
[Tune-y] 41: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 42: s=641
[Tune-y] 42: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 43: s=337
[Tune-y] 43: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 44: s=89
[Tune-y] 44: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 45: s=357
[Tune-y] 45: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 46: s=505
[Tune-y] 46: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 47: s=39
[Tune-y] 47: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 48: s=458
[Tune-y] 48: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 49: s=428
[Tune-y] 49: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 50: s=980
[Tune-y] 50: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 51: s=799
[Tune-y] 51: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 52: s=373
[Tune-y] 52: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 53: s=279
[Tune-y] 53: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 54: s=57
[Tune-y] 54: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 55: s=360
[Tune-y] 55: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 56: s=648
[Tune-y] 56: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 57: s=983
[Tune-y] 57: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 58: s=980
[Tune-y] 58: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 59: s=741
[Tune-y] 59: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 60: s=32
[Tune-y] 60: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 61: s=378
[Tune-y] 61: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 62: s=430
[Tune-y] 62: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 63: s=883
[Tune-y] 63: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 64: s=658
[Tune-y] 64: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 65: s=711
[Tune-y] 65: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 66: s=442
[Tune-y] 66: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 67: s=479
[Tune-y] 67: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 68: s=69
[Tune-y] 68: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 69: s=984
[Tune-y] 69: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 70: s=765
[Tune-y] 70: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 71: s=426
[Tune-y] 71: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 72: s=183
[Tune-y] 72: mse.test.mean=472.9394214; time: 0.0 min
[Tune-x] 73: s=540
[Tune-y] 73: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 74: s=114
[Tune-y] 74: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 75: s=972
[Tune-y] 75: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 76: s=89
[Tune-y] 76: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 77: s=948
[Tune-y] 77: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 78: s=13
[Tune-y] 78: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 79: s=982
[Tune-y] 79: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 80: s=835
[Tune-y] 80: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 81: s=245
[Tune-y] 81: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 82: s=577
[Tune-y] 82: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 83: s=12
[Tune-y] 83: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 84: s=638
[Tune-y] 84: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 85: s=49
[Tune-y] 85: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 86: s=396
[Tune-y] 86: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 87: s=716
[Tune-y] 87: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 88: s=1
[Tune-y] 88: mse.test.mean=312.0571202; time: 0.0 min
[Tune-x] 89: s=319
[Tune-y] 89: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 90: s=806
[Tune-y] 90: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 91: s=487
[Tune-y] 91: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 92: s=432
[Tune-y] 92: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 93: s=648
[Tune-y] 93: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 94: s=735
[Tune-y] 94: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 95: s=877
[Tune-y] 95: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 96: s=777
[Tune-y] 96: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 97: s=920
[Tune-y] 97: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 98: s=316
[Tune-y] 98: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 99: s=983
[Tune-y] 99: mse.test.mean=592.1469170; time: 0.0 min
[Tune-x] 100: s=580
[Tune-y] 100: mse.test.mean=592.1469170; time: 0.0 min
[Tune] Result: s=109 : mse.test.mean=312.0571202
[1] "svm"
[Tune] Started tuning learner regr.ksvm for parameter set:
         Type len Def  Constr Req Tunable Trafo
C     numeric   -   - -3 to 3   -    TRUE     Y
sigma numeric   -   - -3 to 3   -    TRUE     Y
With control class: TuneControlRandom
Imputation value: Inf
[Tune-x] 1: C=84.8; sigma=187
[Tune-y] 1: mse.test.mean=85.4372901; time: 0.0 min
[Tune-x] 2: C=60; sigma=1.23
[Tune-y] 2: mse.test.mean=41.1026880; time: 0.0 min
[Tune-x] 3: C=0.315; sigma=0.00738
[Tune-y] 3: mse.test.mean=31.4650683; time: 0.0 min
[Tune-x] 4: C=1.26; sigma=0.329
[Tune-y] 4: mse.test.mean=24.8901010; time: 0.0 min
[Tune-x] 5: C=0.0153; sigma=0.242
[Tune-y] 5: mse.test.mean=76.3150667; time: 0.0 min
[Tune-x] 6: C=0.00456; sigma=0.0025
[Tune-y] 6: mse.test.mean=84.5789787; time: 0.0 min
[Tune-x] 7: C=1.57; sigma=0.00502
[Tune-y] 7: mse.test.mean=23.3611788; time: 0.0 min
[Tune-x] 8: C=0.014; sigma=908
[Tune-y] 8: mse.test.mean=86.6502998; time: 0.0 min
[Tune-x] 9: C=173; sigma=0.00292
[Tune-y] 9: mse.test.mean=14.3596014; time: 0.0 min
[Tune-x] 10: C=503; sigma=0.00531
[Tune-y] 10: mse.test.mean=12.5955545; time: 0.0 min
[Tune-x] 11: C=44; sigma=24
[Tune-y] 11: mse.test.mean=83.8831935; time: 0.0 min
[Tune-x] 12: C=132; sigma=14.4
[Tune-y] 12: mse.test.mean=82.0240752; time: 0.0 min
[Tune-x] 13: C=0.0133; sigma=0.831
[Tune-y] 13: mse.test.mean=83.9137684; time: 0.0 min
[Tune-x] 14: C=26.6; sigma=139
[Tune-y] 14: mse.test.mean=85.4032227; time: 0.0 min
[Tune-x] 15: C=383; sigma=0.00683
[Tune-y] 15: mse.test.mean=12.8819052; time: 0.0 min
[Tune-x] 16: C=37.6; sigma=19.3
[Tune-y] 16: mse.test.mean=83.2499769; time: 0.0 min
[Tune-x] 17: C=7.03; sigma=0.223
[Tune-y] 17: mse.test.mean=18.5445176; time: 0.0 min
[Tune-x] 18: C=0.00331; sigma=0.0169
[Tune-y] 18: mse.test.mean=80.9989179; time: 0.0 min
[Tune-x] 19: C=0.0131; sigma=13.4
[Tune-y] 19: mse.test.mean=86.5946109; time: 0.0 min
[Tune-x] 20: C=0.0765; sigma=0.0115
[Tune-y] 20: mse.test.mean=47.2201604; time: 0.0 min
[Tune-x] 21: C=13.7; sigma=471
[Tune-y] 21: mse.test.mean=85.4629234; time: 0.0 min
[Tune-x] 22: C=77.2; sigma=0.0147
[Tune-y] 22: mse.test.mean=12.7172613; time: 0.0 min
[Tune-x] 23: C=0.00384; sigma=0.139
[Tune-y] 23: mse.test.mean=81.9122732; time: 0.0 min
[Tune-x] 24: C=11.8; sigma=0.966
[Tune-y] 24: mse.test.mean=36.3481345; time: 0.0 min
[Tune-x] 25: C=40; sigma=0.00113
[Tune-y] 25: mse.test.mean=20.8025902; time: 0.0 min
[Tune-x] 26: C=0.00171; sigma=412
[Tune-y] 26: mse.test.mean=86.7044415; time: 0.0 min
[Tune-x] 27: C=0.00371; sigma=0.0404
[Tune-y] 27: mse.test.mean=79.2219883; time: 0.0 min
[Tune-x] 28: C=26.8; sigma=0.0768
[Tune-y] 28: mse.test.mean=13.6741965; time: 0.0 min
[Tune-x] 29: C=0.023; sigma=0.0116
[Tune-y] 29: mse.test.mean=64.6844582; time: 0.0 min
[Tune-x] 30: C=15.8; sigma=0.211
[Tune-y] 30: mse.test.mean=18.2408690; time: 0.0 min
[Tune-x] 31: C=182; sigma=460
[Tune-y] 31: mse.test.mean=85.4628779; time: 0.0 min
[Tune-x] 32: C=0.37; sigma=29.5
[Tune-y] 32: mse.test.mean=86.4598185; time: 0.0 min
[Tune-x] 33: C=0.462; sigma=207
[Tune-y] 33: mse.test.mean=86.9205925; time: 0.0 min
[Tune-x] 34: C=0.825; sigma=0.872
[Tune-y] 34: mse.test.mean=42.9607146; time: 0.0 min
[Tune-x] 35: C=0.0442; sigma=0.0125
[Tune-y] 35: mse.test.mean=54.8480775; time: 0.0 min
[Tune-x] 36: C=0.264; sigma=0.0922
[Tune-y] 36: mse.test.mean=26.9622918; time: 0.0 min
[Tune-x] 37: C=3.31; sigma=1.99
[Tune-y] 37: mse.test.mean=51.4958030; time: 0.0 min
[Tune-x] 38: C=0.21; sigma=177
[Tune-y] 38: mse.test.mean=86.5638720; time: 0.0 min
[Tune-x] 39: C=3.16; sigma=11.3
[Tune-y] 39: mse.test.mean=80.5801554; time: 0.0 min
[Tune-x] 40: C=0.0199; sigma=742
[Tune-y] 40: mse.test.mean=86.6391865; time: 0.0 min
[Tune-x] 41: C=0.111; sigma=0.0289
[Tune-y] 41: mse.test.mean=34.1471741; time: 0.0 min
[Tune-x] 42: C=0.0218; sigma=472
[Tune-y] 42: mse.test.mean=86.6279779; time: 0.0 min
[Tune-x] 43: C=20.3; sigma=296
[Tune-y] 43: mse.test.mean=85.4589206; time: 0.0 min
[Tune-x] 44: C=25.9; sigma=10.5
[Tune-y] 44: mse.test.mean=80.0089020; time: 0.0 min
[Tune-x] 45: C=506; sigma=449
[Tune-y] 45: mse.test.mean=85.4628180; time: 0.0 min
[Tune-x] 46: C=65.7; sigma=0.00176
[Tune-y] 46: mse.test.mean=17.2128330; time: 0.0 min
[Tune-x] 47: C=325; sigma=0.00259
[Tune-y] 47: mse.test.mean=14.0698808; time: 0.0 min
[Tune-x] 48: C=4.1; sigma=0.0545
[Tune-y] 48: mse.test.mean=13.4777519; time: 0.0 min
[Tune-x] 49: C=0.00424; sigma=0.00333
[Tune-y] 49: mse.test.mean=84.1878880; time: 0.0 min
[Tune-x] 50: C=0.0863; sigma=0.634
[Tune-y] 50: mse.test.mean=69.4986205; time: 0.0 min
[Tune-x] 51: C=0.299; sigma=0.145
[Tune-y] 51: mse.test.mean=29.3394225; time: 0.0 min
[Tune-x] 52: C=21.1; sigma=0.00391
[Tune-y] 52: mse.test.mean=16.7211256; time: 0.0 min
[Tune-x] 53: C=2.03; sigma=99.7
[Tune-y] 53: mse.test.mean=85.4340690; time: 0.0 min
[Tune-x] 54: C=24.4; sigma=0.105
[Tune-y] 54: mse.test.mean=14.6134680; time: 0.0 min
[Tune-x] 55: C=0.143; sigma=49.1
[Tune-y] 55: mse.test.mean=86.4351023; time: 0.0 min
[Tune-x] 56: C=4.12; sigma=20.4
[Tune-y] 56: mse.test.mean=83.4250483; time: 0.0 min
[Tune-x] 57: C=16.3; sigma=0.119
[Tune-y] 57: mse.test.mean=15.0832618; time: 0.0 min
[Tune-x] 58: C=0.706; sigma=0.00228
[Tune-y] 58: mse.test.mean=35.1897680; time: 0.0 min
[Tune-x] 59: C=0.102; sigma=0.0563
[Tune-y] 59: mse.test.mean=35.0427176; time: 0.0 min
[Tune-x] 60: C=0.00308; sigma=3.28
[Tune-y] 60: mse.test.mean=86.5667550; time: 0.0 min
[Tune-x] 61: C=31.5; sigma=82.5
[Tune-y] 61: mse.test.mean=85.2814872; time: 0.0 min
[Tune-x] 62: C=36; sigma=435
[Tune-y] 62: mse.test.mean=85.4627295; time: 0.0 min
[Tune-x] 63: C=2.14; sigma=243
[Tune-y] 63: mse.test.mean=85.5193969; time: 0.0 min
[Tune-x] 64: C=0.00347; sigma=45.6
[Tune-y] 64: mse.test.mean=86.7055296; time: 0.0 min
[Tune-x] 65: C=0.0117; sigma=29.6
[Tune-y] 65: mse.test.mean=86.6406696; time: 0.0 min
[Tune-x] 66: C=45.5; sigma=28.9
[Tune-y] 66: mse.test.mean=84.2913297; time: 0.0 min
[Tune-x] 67: C=0.0252; sigma=437
[Tune-y] 67: mse.test.mean=86.6356721; time: 0.0 min
[Tune-x] 68: C=410; sigma=9.31
[Tune-y] 68: mse.test.mean=79.0458560; time: 0.0 min
[Tune-x] 69: C=0.00415; sigma=1.17
[Tune-y] 69: mse.test.mean=86.0589610; time: 0.0 min
[Tune-x] 70: C=1.28; sigma=0.00113
[Tune-y] 70: mse.test.mean=36.5179864; time: 0.0 min
[Tune-x] 71: C=0.157; sigma=17.6
[Tune-y] 71: mse.test.mean=86.1356979; time: 0.0 min
[Tune-x] 72: C=360; sigma=0.619
[Tune-y] 72: mse.test.mean=29.0065332; time: 0.0 min
[Tune-x] 73: C=143; sigma=7.3
[Tune-y] 73: mse.test.mean=76.5221138; time: 0.0 min
[Tune-x] 74: C=0.0047; sigma=0.00281
[Tune-y] 74: mse.test.mean=84.2970947; time: 0.0 min
[Tune-x] 75: C=0.0574; sigma=93.6
[Tune-y] 75: mse.test.mean=86.6407057; time: 0.0 min
[Tune-x] 76: C=48.9; sigma=0.00206
[Tune-y] 76: mse.test.mean=17.2063189; time: 0.0 min
[Tune-x] 77: C=864; sigma=0.00292
[Tune-y] 77: mse.test.mean=12.9514507; time: 0.0 min
[Tune-x] 78: C=0.00185; sigma=0.00121
[Tune-y] 78: mse.test.mean=86.2164307; time: 0.0 min
[Tune-x] 79: C=455; sigma=1.76
[Tune-y] 79: mse.test.mean=48.6870862; time: 0.0 min
[Tune-x] 80: C=0.00211; sigma=0.00185
[Tune-y] 80: mse.test.mean=85.8948999; time: 0.0 min
[Tune-x] 81: C=0.184; sigma=517
[Tune-y] 81: mse.test.mean=86.5609310; time: 0.0 min
[Tune-x] 82: C=0.394; sigma=28.5
[Tune-y] 82: mse.test.mean=86.4522234; time: 0.0 min
[Tune-x] 83: C=0.00308; sigma=421
[Tune-y] 83: mse.test.mean=86.7070326; time: 0.0 min
[Tune-x] 84: C=64; sigma=0.962
[Tune-y] 84: mse.test.mean=36.2682213; time: 0.0 min
[Tune-x] 85: C=0.00222; sigma=1.43
[Tune-y] 85: mse.test.mean=86.3970361; time: 0.0 min
[Tune-x] 86: C=160; sigma=345
[Tune-y] 86: mse.test.mean=85.4612797; time: 0.0 min
[Tune-x] 87: C=1.91; sigma=152
[Tune-y] 87: mse.test.mean=85.5403261; time: 0.0 min
[Tune-x] 88: C=0.0023; sigma=0.506
[Tune-y] 88: mse.test.mean=85.7560425; time: 0.0 min
[Tune-x] 89: C=0.00243; sigma=0.0437
[Tune-y] 89: mse.test.mean=81.5625611; time: 0.0 min
[Tune-x] 90: C=0.689; sigma=0.00659
[Tune-y] 90: mse.test.mean=26.1530698; time: 0.0 min
[Tune-x] 91: C=0.295; sigma=0.185
[Tune-y] 91: mse.test.mean=31.8592087; time: 0.0 min
[Tune-x] 92: C=0.00628; sigma=2.61
[Tune-y] 92: mse.test.mean=86.3673100; time: 0.0 min
[Tune-x] 93: C=0.00252; sigma=16.3
[Tune-y] 93: mse.test.mean=86.6987018; time: 0.0 min
[Tune-x] 94: C=0.00784; sigma=80.5
[Tune-y] 94: mse.test.mean=86.6812477; time: 0.0 min
[Tune-x] 95: C=58.2; sigma=1.29
[Tune-y] 95: mse.test.mean=42.1518376; time: 0.0 min
[Tune-x] 96: C=0.849; sigma=2.75
[Tune-y] 96: mse.test.mean=66.6833816; time: 0.0 min
[Tune-x] 97: C=0.0152; sigma=0.0693
[Tune-y] 97: mse.test.mean=66.9179046; time: 0.0 min
[Tune-x] 98: C=22.1; sigma=149
[Tune-y] 98: mse.test.mean=85.4131631; time: 0.0 min
[Tune-x] 99: C=0.00199; sigma=0.362
[Tune-y] 99: mse.test.mean=85.5202016; time: 0.0 min
[Tune-x] 100: C=5.15; sigma=0.0817
[Tune-y] 100: mse.test.mean=13.2472797; time: 0.0 min
[Tune] Result: C=503; sigma=0.00531 : mse.test.mean=12.5955545
[1] "rf"
[Tune] Started tuning learner regr.randomForest for parameter set:
         Type len Def      Constr Req Tunable Trafo
ntree integer   -   - 50 to 1e+03   -    TRUE     -
With control class: TuneControlRandom
Imputation value: Inf
[Tune-x] 1: ntree=151
[Tune-y] 1: mse.test.mean=13.1353047; time: 0.0 min
[Tune-x] 2: ntree=644
[Tune-y] 2: mse.test.mean=13.6973350; time: 0.0 min
[Tune-x] 3: ntree=908
[Tune-y] 3: mse.test.mean=13.6936320; time: 0.0 min
[Tune-x] 4: ntree=369
[Tune-y] 4: mse.test.mean=13.6389426; time: 0.0 min
[Tune-x] 5: ntree=702
[Tune-y] 5: mse.test.mean=13.6969548; time: 0.0 min
[Tune-x] 6: ntree=147
[Tune-y] 6: mse.test.mean=13.5217214; time: 0.0 min
[Tune-x] 7: ntree=594
[Tune-y] 7: mse.test.mean=13.8943053; time: 0.0 min
[Tune-x] 8: ntree=493
[Tune-y] 8: mse.test.mean=13.4260160; time: 0.0 min
[Tune-x] 9: ntree=494
[Tune-y] 9: mse.test.mean=13.8570528; time: 0.0 min
[Tune-x] 10: ntree=706
[Tune-y] 10: mse.test.mean=13.7668151; time: 0.0 min
[Tune-x] 11: ntree=591
[Tune-y] 11: mse.test.mean=13.6149329; time: 0.0 min
[Tune-x] 12: ntree=956
[Tune-y] 12: mse.test.mean=13.7261819; time: 0.0 min
[Tune-x] 13: ntree=614
[Tune-y] 13: mse.test.mean=13.6667441; time: 0.0 min
[Tune-x] 14: ntree=820
[Tune-y] 14: mse.test.mean=13.6925929; time: 0.0 min
[Tune-x] 15: ntree=55
[Tune-y] 15: mse.test.mean=15.0736288; time: 0.0 min
[Tune-x] 16: ntree=426
[Tune-y] 16: mse.test.mean=13.5748238; time: 0.0 min
[Tune-x] 17: ntree=867
[Tune-y] 17: mse.test.mean=13.6221894; time: 0.0 min
[Tune-x] 18: ntree=388
[Tune-y] 18: mse.test.mean=13.5993097; time: 0.0 min
[Tune-x] 19: ntree=583
[Tune-y] 19: mse.test.mean=13.8851485; time: 0.0 min
[Tune-x] 20: ntree=543
[Tune-y] 20: mse.test.mean=13.6340810; time: 0.0 min
[Tune-x] 21: ntree=752
[Tune-y] 21: mse.test.mean=13.6126653; time: 0.0 min
[Tune-x] 22: ntree=220
[Tune-y] 22: mse.test.mean=13.7120007; time: 0.0 min
[Tune-x] 23: ntree=74
[Tune-y] 23: mse.test.mean=13.5486854; time: 0.0 min
[Tune-x] 24: ntree=266
[Tune-y] 24: mse.test.mean=13.7384414; time: 0.0 min
[Tune-x] 25: ntree=84
[Tune-y] 25: mse.test.mean=14.1014786; time: 0.0 min
[Tune-x] 26: ntree=945
[Tune-y] 26: mse.test.mean=13.6723311; time: 0.0 min
[Tune-x] 27: ntree=633
[Tune-y] 27: mse.test.mean=13.7522716; time: 0.0 min
[Tune-x] 28: ntree=287
[Tune-y] 28: mse.test.mean=13.3221586; time: 0.0 min
[Tune-x] 29: ntree=948
[Tune-y] 29: mse.test.mean=13.5829461; time: 0.0 min
[Tune-x] 30: ntree=97
[Tune-y] 30: mse.test.mean=13.9852395; time: 0.0 min
[Tune-x] 31: ntree=804
[Tune-y] 31: mse.test.mean=13.5992419; time: 0.0 min
[Tune-x] 32: ntree=328
[Tune-y] 32: mse.test.mean=13.5669581; time: 0.0 min
[Tune-x] 33: ntree=701
[Tune-y] 33: mse.test.mean=13.6871320; time: 0.0 min
[Tune-x] 34: ntree=799
[Tune-y] 34: mse.test.mean=13.8207616; time: 0.0 min
[Tune-x] 35: ntree=188
[Tune-y] 35: mse.test.mean=13.9036690; time: 0.0 min
[Tune-x] 36: ntree=281
[Tune-y] 36: mse.test.mean=13.8436546; time: 0.0 min
[Tune-x] 37: ntree=580
[Tune-y] 37: mse.test.mean=13.7040020; time: 0.0 min
[Tune-x] 38: ntree=156
[Tune-y] 38: mse.test.mean=13.8319607; time: 0.0 min
[Tune-x] 39: ntree=837
[Tune-y] 39: mse.test.mean=13.7358285; time: 0.0 min
[Tune-x] 40: ntree=352
[Tune-y] 40: mse.test.mean=13.6373988; time: 0.0 min
[Tune-x] 41: ntree=684
[Tune-y] 41: mse.test.mean=13.7094839; time: 0.0 min
[Tune-x] 42: ntree=894
[Tune-y] 42: mse.test.mean=13.5254221; time: 0.0 min
[Tune-x] 43: ntree=828
[Tune-y] 43: mse.test.mean=13.8032489; time: 0.0 min
[Tune-x] 44: ntree=896
[Tune-y] 44: mse.test.mean=13.7219122; time: 0.0 min
[Tune-x] 45: ntree=461
[Tune-y] 45: mse.test.mean=13.6932357; time: 0.0 min
[Tune-x] 46: ntree=332
[Tune-y] 46: mse.test.mean=13.6051745; time: 0.0 min
[Tune-x] 47: ntree=784
[Tune-y] 47: mse.test.mean=13.7090018; time: 0.0 min
[Tune-x] 48: ntree=102
[Tune-y] 48: mse.test.mean=13.2273064; time: 0.0 min
[Tune-x] 49: ntree=67
[Tune-y] 49: mse.test.mean=14.3171141; time: 0.0 min
[Tune-x] 50: ntree=52
[Tune-y] 50: mse.test.mean=14.3181702; time: 0.0 min
[Tune-x] 51: ntree=770
[Tune-y] 51: mse.test.mean=13.5494005; time: 0.0 min
[Tune-x] 52: ntree=572
[Tune-y] 52: mse.test.mean=13.4786836; time: 0.0 min
[Tune-x] 53: ntree=116
[Tune-y] 53: mse.test.mean=13.3322116; time: 0.0 min
[Tune-x] 54: ntree=466
[Tune-y] 54: mse.test.mean=13.6237170; time: 0.0 min
[Tune-x] 55: ntree=859
[Tune-y] 55: mse.test.mean=13.5474555; time: 0.0 min
[Tune-x] 56: ntree=714
[Tune-y] 56: mse.test.mean=13.7730251; time: 0.0 min
[Tune-x] 57: ntree=463
[Tune-y] 57: mse.test.mean=13.5181853; time: 0.0 min
[Tune-x] 58: ntree=226
[Tune-y] 58: mse.test.mean=13.7464438; time: 0.0 min
[Tune-x] 59: ntree=662
[Tune-y] 59: mse.test.mean=13.8719245; time: 0.0 min
[Tune-x] 60: ntree=915
[Tune-y] 60: mse.test.mean=13.9028470; time: 0.0 min
[Tune-x] 61: ntree=488
[Tune-y] 61: mse.test.mean=13.7709979; time: 0.0 min
[Tune-x] 62: ntree=396
[Tune-y] 62: mse.test.mean=14.0924689; time: 0.0 min
[Tune-x] 63: ntree=320
[Tune-y] 63: mse.test.mean=13.5225552; time: 0.0 min
[Tune-x] 64: ntree=514
[Tune-y] 64: mse.test.mean=13.7792450; time: 0.0 min
[Tune-x] 65: ntree=605
[Tune-y] 65: mse.test.mean=13.9861130; time: 0.0 min
[Tune-x] 66: ntree=910
[Tune-y] 66: mse.test.mean=13.4858534; time: 0.0 min
[Tune-x] 67: ntree=831
[Tune-y] 67: mse.test.mean=13.7021329; time: 0.0 min
[Tune-x] 68: ntree=978
[Tune-y] 68: mse.test.mean=13.8949655; time: 0.0 min
[Tune-x] 69: ntree=759
[Tune-y] 69: mse.test.mean=13.6677972; time: 0.0 min
[Tune-x] 70: ntree=59
[Tune-y] 70: mse.test.mean=14.9603674; time: 0.0 min
[Tune-x] 71: ntree=95
[Tune-y] 71: mse.test.mean=14.0566352; time: 0.0 min
[Tune-x] 72: ntree=574
[Tune-y] 72: mse.test.mean=13.6392521; time: 0.0 min
[Tune-x] 73: ntree=692
[Tune-y] 73: mse.test.mean=13.6836798; time: 0.0 min
[Tune-x] 74: ntree=55
[Tune-y] 74: mse.test.mean=13.9362242; time: 0.0 min
[Tune-x] 75: ntree=235
[Tune-y] 75: mse.test.mean=13.9598062; time: 0.0 min
[Tune-x] 76: ntree=288
[Tune-y] 76: mse.test.mean=13.6908268; time: 0.0 min
[Tune-x] 77: ntree=496
[Tune-y] 77: mse.test.mean=13.8416086; time: 0.0 min
[Tune-x] 78: ntree=433
[Tune-y] 78: mse.test.mean=13.9668706; time: 0.0 min
[Tune-x] 79: ntree=554
[Tune-y] 79: mse.test.mean=13.6971713; time: 0.0 min
[Tune-x] 80: ntree=246
[Tune-y] 80: mse.test.mean=13.8151076; time: 0.0 min
[Tune-x] 81: ntree=348
[Tune-y] 81: mse.test.mean=13.9843219; time: 0.0 min
[Tune-x] 82: ntree=704
[Tune-y] 82: mse.test.mean=13.6739478; time: 0.0 min
[Tune-x] 83: ntree=537
[Tune-y] 83: mse.test.mean=13.6358696; time: 0.0 min
[Tune-x] 84: ntree=482
[Tune-y] 84: mse.test.mean=13.6583236; time: 0.0 min
[Tune-x] 85: ntree=414
[Tune-y] 85: mse.test.mean=13.3591933; time: 0.0 min
[Tune-x] 86: ntree=610
[Tune-y] 86: mse.test.mean=13.8064945; time: 0.0 min
[Tune-x] 87: ntree=499
[Tune-y] 87: mse.test.mean=13.6958101; time: 0.0 min
[Tune-x] 88: ntree=312
[Tune-y] 88: mse.test.mean=13.6731001; time: 0.0 min
[Tune-x] 89: ntree=409
[Tune-y] 89: mse.test.mean=13.9744267; time: 0.0 min
[Tune-x] 90: ntree=541
[Tune-y] 90: mse.test.mean=13.4894813; time: 0.0 min
[Tune-x] 91: ntree=571
[Tune-y] 91: mse.test.mean=13.7500286; time: 0.0 min
[Tune-x] 92: ntree=202
[Tune-y] 92: mse.test.mean=14.2396077; time: 0.0 min
[Tune-x] 93: ntree=743
[Tune-y] 93: mse.test.mean=13.5812491; time: 0.0 min
[Tune-x] 94: ntree=155
[Tune-y] 94: mse.test.mean=14.0110734; time: 0.0 min
[Tune-x] 95: ntree=204
[Tune-y] 95: mse.test.mean=13.7123178; time: 0.0 min
[Tune-x] 96: ntree=919
[Tune-y] 96: mse.test.mean=13.6663117; time: 0.0 min
[Tune-x] 97: ntree=211
[Tune-y] 97: mse.test.mean=13.7494775; time: 0.0 min
[Tune-x] 98: ntree=331
[Tune-y] 98: mse.test.mean=13.6433587; time: 0.0 min
[Tune-x] 99: ntree=645
[Tune-y] 99: mse.test.mean=13.5547939; time: 0.0 min
[Tune-x] 100: ntree=729
[Tune-y] 100: mse.test.mean=13.6279040; time: 0.0 min
[Tune] Result: ntree=151 : mse.test.mean=13.1353047
[1] "xgb"
[Tune] Started tuning learner regr.xgboost for parameter set:
             Type len Def  Constr Req Tunable Trafo
nrounds   integer   -   - 3 to 50   -    TRUE     -
max_depth integer   -   - 3 to 20   -    TRUE     -
With control class: TuneControlRandom
Imputation value: Inf
[Tune-x] 1: nrounds=18; max_depth=11
[Tune-y] 1: mse.test.mean=13.3927559; time: 0.0 min
[Tune-x] 2: nrounds=46; max_depth=6
[Tune-y] 2: mse.test.mean=12.2778863; time: 0.0 min
[Tune-x] 3: nrounds=31; max_depth=10
[Tune-y] 3: mse.test.mean=13.1903715; time: 0.0 min
[Tune-x] 4: nrounds=38; max_depth=3
[Tune-y] 4: mse.test.mean=12.7382400; time: 0.0 min
[Tune-x] 5: nrounds=43; max_depth=8
[Tune-y] 5: mse.test.mean=13.0111497; time: 0.1 min
[Tune-x] 6: nrounds=12; max_depth=17
[Tune-y] 6: mse.test.mean=14.0606909; time: 0.0 min
[Tune-x] 7: nrounds=13; max_depth=4
[Tune-y] 7: mse.test.mean=13.3678639; time: 0.0 min
[Tune-x] 8: nrounds=23; max_depth=20
[Tune-y] 8: mse.test.mean=13.2481199; time: 0.0 min
[Tune-x] 9: nrounds=35; max_depth=12
[Tune-y] 9: mse.test.mean=13.1336100; time: 0.0 min
[Tune-x] 10: nrounds=6; max_depth=4
[Tune-y] 10: mse.test.mean=26.1536661; time: 0.0 min
[Tune-x] 11: nrounds=50; max_depth=9
[Tune-y] 11: mse.test.mean=13.3116873; time: 0.1 min
[Tune-x] 12: nrounds=19; max_depth=8
[Tune-y] 12: mse.test.mean=13.0828598; time: 0.0 min
[Tune-x] 13: nrounds=50; max_depth=8
[Tune-y] 13: mse.test.mean=13.0153350; time: 0.0 min
[Tune-x] 14: nrounds=46; max_depth=3
[Tune-y] 14: mse.test.mean=12.5728905; time: 0.0 min
[Tune-x] 15: nrounds=23; max_depth=17
[Tune-y] 15: mse.test.mean=13.2603956; time: 0.0 min
[Tune-x] 16: nrounds=11; max_depth=6
[Tune-y] 16: mse.test.mean=13.5268146; time: 0.0 min
[Tune-x] 17: nrounds=31; max_depth=6
[Tune-y] 17: mse.test.mean=12.3641851; time: 0.0 min
[Tune-x] 18: nrounds=23; max_depth=10
[Tune-y] 18: mse.test.mean=13.2167251; time: 0.0 min
[Tune-x] 19: nrounds=12; max_depth=14
[Tune-y] 19: mse.test.mean=14.0938368; time: 0.0 min
[Tune-x] 20: nrounds=42; max_depth=11
[Tune-y] 20: mse.test.mean=13.2866441; time: 0.0 min
[Tune-x] 21: nrounds=10; max_depth=18
[Tune-y] 21: mse.test.mean=15.0614982; time: 0.0 min
[Tune-x] 22: nrounds=24; max_depth=16
[Tune-y] 22: mse.test.mean=13.2330722; time: 0.1 min
[Tune-x] 23: nrounds=15; max_depth=14
[Tune-y] 23: mse.test.mean=13.4769344; time: 0.0 min
[Tune-x] 24: nrounds=12; max_depth=16
[Tune-y] 24: mse.test.mean=14.0606909; time: 0.0 min
[Tune-x] 25: nrounds=42; max_depth=15
[Tune-y] 25: mse.test.mean=13.3038864; time: 0.1 min
[Tune-x] 26: nrounds=15; max_depth=5
[Tune-y] 26: mse.test.mean=12.7421704; time: 0.0 min
[Tune-x] 27: nrounds=32; max_depth=13
[Tune-y] 27: mse.test.mean=13.1978834; time: 0.0 min
[Tune-x] 28: nrounds=28; max_depth=20
[Tune-y] 28: mse.test.mean=13.2230779; time: 0.0 min
[Tune-x] 29: nrounds=35; max_depth=10
[Tune-y] 29: mse.test.mean=13.1807631; time: 0.0 min
[Tune-x] 30: nrounds=13; max_depth=3
[Tune-y] 30: mse.test.mean=14.1786867; time: 0.0 min
[Tune-x] 31: nrounds=50; max_depth=9
[Tune-y] 31: mse.test.mean=13.3116873; time: 0.0 min
[Tune-x] 32: nrounds=8; max_depth=8
[Tune-y] 32: mse.test.mean=17.7969841; time: 0.0 min
[Tune-x] 33: nrounds=30; max_depth=11
[Tune-y] 33: mse.test.mean=13.2924168; time: 0.0 min
[Tune-x] 34: nrounds=49; max_depth=9
[Tune-y] 34: mse.test.mean=13.3123488; time: 0.1 min
[Tune-x] 35: nrounds=37; max_depth=8
[Tune-y] 35: mse.test.mean=13.0293472; time: 0.0 min
[Tune-x] 36: nrounds=14; max_depth=12
[Tune-y] 36: mse.test.mean=13.5368723; time: 0.0 min
[Tune-x] 37: nrounds=47; max_depth=18
[Tune-y] 37: mse.test.mean=13.2118138; time: 0.1 min
[Tune-x] 38: nrounds=46; max_depth=7
[Tune-y] 38: mse.test.mean=13.0517904; time: 0.0 min
[Tune-x] 39: nrounds=38; max_depth=15
[Tune-y] 39: mse.test.mean=13.3053507; time: 0.1 min
[Tune-x] 40: nrounds=4; max_depth=6
[Tune-y] 40: mse.test.mean=55.3304853; time: 0.0 min
[Tune-x] 41: nrounds=9; max_depth=8
[Tune-y] 41: mse.test.mean=15.9675821; time: 0.0 min
[Tune-x] 42: nrounds=46; max_depth=5
[Tune-y] 42: mse.test.mean=11.9969642; time: 0.0 min
[Tune-x] 43: nrounds=12; max_depth=7
[Tune-y] 43: mse.test.mean=13.9011234; time: 0.0 min
[Tune-x] 44: nrounds=27; max_depth=4
[Tune-y] 44: mse.test.mean=12.0099452; time: 0.0 min
[Tune-x] 45: nrounds=49; max_depth=20
[Tune-y] 45: mse.test.mean=13.2048759; time: 0.1 min
[Tune-x] 46: nrounds=29; max_depth=19
[Tune-y] 46: mse.test.mean=13.2298028; time: 0.0 min
[Tune-x] 47: nrounds=44; max_depth=10
[Tune-y] 47: mse.test.mean=13.1748810; time: 0.0 min
[Tune-x] 48: nrounds=32; max_depth=8
[Tune-y] 48: mse.test.mean=13.0421917; time: 0.0 min
[Tune-x] 49: nrounds=45; max_depth=20
[Tune-y] 49: mse.test.mean=13.2053812; time: 0.1 min
[Tune-x] 50: nrounds=25; max_depth=5
[Tune-y] 50: mse.test.mean=12.0846688; time: 0.0 min
[Tune-x] 51: nrounds=39; max_depth=19
[Tune-y] 51: mse.test.mean=13.2200767; time: 0.1 min
[Tune-x] 52: nrounds=44; max_depth=9
[Tune-y] 52: mse.test.mean=13.3119583; time: 0.0 min
[Tune-x] 53: nrounds=18; max_depth=11
[Tune-y] 53: mse.test.mean=13.3927559; time: 0.0 min
[Tune-x] 54: nrounds=4; max_depth=7
[Tune-y] 54: mse.test.mean=55.2907438; time: 0.0 min
[Tune-x] 55: nrounds=15; max_depth=18
[Tune-y] 55: mse.test.mean=13.4803814; time: 0.1 min
[Tune-x] 56: nrounds=6; max_depth=13
[Tune-y] 56: mse.test.mean=26.9490423; time: 0.0 min
[Tune-x] 57: nrounds=47; max_depth=7
[Tune-y] 57: mse.test.mean=13.0536628; time: 0.0 min
[Tune-x] 58: nrounds=11; max_depth=4
[Tune-y] 58: mse.test.mean=14.2965015; time: 0.0 min
[Tune-x] 59: nrounds=16; max_depth=6
[Tune-y] 59: mse.test.mean=12.6340261; time: 0.0 min
[Tune-x] 60: nrounds=14; max_depth=11
[Tune-y] 60: mse.test.mean=13.6875426; time: 0.0 min
[Tune-x] 61: nrounds=6; max_depth=16
[Tune-y] 61: mse.test.mean=26.9490423; time: 0.0 min
[Tune-x] 62: nrounds=40; max_depth=6
[Tune-y] 62: mse.test.mean=12.2950786; time: 0.0 min
[Tune-x] 63: nrounds=6; max_depth=14
[Tune-y] 63: mse.test.mean=26.9490423; time: 0.0 min
[Tune-x] 64: nrounds=10; max_depth=5
[Tune-y] 64: mse.test.mean=14.2732871; time: 0.0 min
[Tune-x] 65: nrounds=25; max_depth=17
[Tune-y] 65: mse.test.mean=13.2372900; time: 0.0 min
[Tune-x] 66: nrounds=44; max_depth=4
[Tune-y] 66: mse.test.mean=11.7057562; time: 0.0 min
[Tune-x] 67: nrounds=41; max_depth=7
[Tune-y] 67: mse.test.mean=13.0605508; time: 0.0 min
[Tune-x] 68: nrounds=37; max_depth=18
[Tune-y] 68: mse.test.mean=13.2144298; time: 0.1 min
[Tune-x] 69: nrounds=30; max_depth=4
[Tune-y] 69: mse.test.mean=11.9119738; time: 0.0 min
[Tune-x] 70: nrounds=4; max_depth=16
[Tune-y] 70: mse.test.mean=55.2377079; time: 0.0 min
[Tune-x] 71: nrounds=38; max_depth=8
[Tune-y] 71: mse.test.mean=13.0259020; time: 0.0 min
[Tune-x] 72: nrounds=38; max_depth=11
[Tune-y] 72: mse.test.mean=13.2872887; time: 0.0 min
[Tune-x] 73: nrounds=47; max_depth=8
[Tune-y] 73: mse.test.mean=13.0154225; time: 0.1 min
[Tune-x] 74: nrounds=29; max_depth=9
[Tune-y] 74: mse.test.mean=13.3251644; time: 0.0 min
[Tune-x] 75: nrounds=48; max_depth=5
[Tune-y] 75: mse.test.mean=11.9885711; time: 0.0 min
[Tune-x] 76: nrounds=23; max_depth=10
[Tune-y] 76: mse.test.mean=13.2167251; time: 0.0 min
[Tune-x] 77: nrounds=17; max_depth=7
[Tune-y] 77: mse.test.mean=13.2230576; time: 0.0 min
[Tune-x] 78: nrounds=9; max_depth=14
[Tune-y] 78: mse.test.mean=16.1448636; time: 0.0 min
[Tune-x] 79: nrounds=6; max_depth=14
[Tune-y] 79: mse.test.mean=26.9490423; time: 0.0 min
[Tune-x] 80: nrounds=38; max_depth=7
[Tune-y] 80: mse.test.mean=13.0690053; time: 0.0 min
[Tune-x] 81: nrounds=9; max_depth=16
[Tune-y] 81: mse.test.mean=16.1448636; time: 0.0 min
[Tune-x] 82: nrounds=30; max_depth=13
[Tune-y] 82: mse.test.mean=13.2039259; time: 0.1 min
[Tune-x] 83: nrounds=10; max_depth=18
[Tune-y] 83: mse.test.mean=15.0614982; time: 0.0 min
[Tune-x] 84: nrounds=47; max_depth=9
[Tune-y] 84: mse.test.mean=13.3129507; time: 0.1 min
[Tune-x] 85: nrounds=32; max_depth=20
[Tune-y] 85: mse.test.mean=13.2133378; time: 0.1 min
[Tune-x] 86: nrounds=33; max_depth=11
[Tune-y] 86: mse.test.mean=13.2895829; time: 0.0 min
[Tune-x] 87: nrounds=36; max_depth=8
[Tune-y] 87: mse.test.mean=13.0253936; time: 0.0 min
[Tune-x] 88: nrounds=30; max_depth=13
[Tune-y] 88: mse.test.mean=13.2039259; time: 0.1 min
[Tune-x] 89: nrounds=38; max_depth=5
[Tune-y] 89: mse.test.mean=12.0376940; time: 0.0 min
[Tune-x] 90: nrounds=40; max_depth=20
[Tune-y] 90: mse.test.mean=13.2071935; time: 0.1 min
[Tune-x] 91: nrounds=18; max_depth=12
[Tune-y] 91: mse.test.mean=13.3003237; time: 0.0 min
[Tune-x] 92: nrounds=33; max_depth=8
[Tune-y] 92: mse.test.mean=13.0400210; time: 0.0 min
[Tune-x] 93: nrounds=13; max_depth=4
[Tune-y] 93: mse.test.mean=13.3678639; time: 0.0 min
[Tune-x] 94: nrounds=32; max_depth=7
[Tune-y] 94: mse.test.mean=13.0652559; time: 0.1 min
[Tune-x] 95: nrounds=49; max_depth=5
[Tune-y] 95: mse.test.mean=11.9757321; time: 0.0 min
[Tune-x] 96: nrounds=4; max_depth=5
[Tune-y] 96: mse.test.mean=54.6098787; time: 0.0 min
[Tune-x] 97: nrounds=42; max_depth=8
[Tune-y] 97: mse.test.mean=13.0153720; time: 0.1 min
[Tune-x] 98: nrounds=4; max_depth=14
[Tune-y] 98: mse.test.mean=55.2377079; time: 0.0 min
[Tune-x] 99: nrounds=31; max_depth=5
[Tune-y] 99: mse.test.mean=12.0657889; time: 0.0 min
[Tune-x] 100: nrounds=14; max_depth=4
[Tune-y] 100: mse.test.mean=13.1046638; time: 0.0 min
[Tune] Result: nrounds=44; max_depth=4 : mse.test.mean=11.7057562

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

saveRDS(tuned.model, "./tuned_models.RDS")
```

