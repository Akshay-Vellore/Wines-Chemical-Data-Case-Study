# Case Study: Wine Chemical Data Analysis

## INTRODUCTION

For this case study, we aim to analyze the alcohol percentage in wine by
examining its relationship with various chemical indicators. The main 
indicators/predictors we consider here are fixed acidity, volatile acidity, 
citric acid, residual sugar, chlorides, free sulfur dioxide, total sulfur 
dioxide, density, pH, sulfates, and wine type.

We will analyze this data using a variety of statistical linear models, such
as multiple linear regression, stepwise & greedy model selection, Ridge/LASSO
shrinkage, and principle components regression. We will also try to validate our
best model (indicated by smallest training RMSE) by analyzing the base linear model 
assumptions and by finding the RMSE of the model on a test data.

By understanding these relationships, we can gain insights into the factors that 
contribute to variations in alcohol percentage. Note we are using a significance level of
0.05 throughout this study.

**Read the data**

    wines = read.csv('wines.csv')
    head(wines)

    ##   fixed.acidity volatile.acidity citric.acid residual.sugar chlorides free.sulfur.dioxide total.sulfur.dioxide density   pH
    ## 1           7.4             0.70        0.00            1.9     0.076                  11                   34  0.9978 3.51
    ## 2           7.8             0.88        0.00            2.6     0.098                  25                   67  0.9968 3.20
    ## 3           7.8             0.76        0.04            2.3     0.092                  15                   54  0.9970 3.26
    ## 4          11.2             0.28        0.56            1.9     0.075                  17                   60  0.9980 3.16
    ## 5           7.4             0.70        0.00            1.9     0.076                  11                   34  0.9978 3.51
    ## 6           7.4             0.66        0.00            1.8     0.075                  13                   40  0.9978 3.51
    ##   sulphates alcohol    type
    ## 1      0.56     9.4 redwine
    ## 2      0.68     9.8 redwine
    ## 3      0.65     9.8 redwine
    ## 4      0.58     9.8 redwine
    ## 5      0.56     9.4 redwine
    ## 6      0.56     9.4 redwine

## Exploratory Data Analysis

1.  Dimensions of our dataset

<!-- -->

    dim(wines)

    ## [1] 6497   12

1.  Histogram for the Alcohol percentage for Wines

<!-- -->

    hist(wines$alcohol, xlab = "Alcohol Pecentage", 
         main = "Histogram for the Alcohol percentage for Wines")

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-3-1.png)

1.  Box PLot for Alcohol Percentage

<!-- -->

    boxplot(x = wines$alcohol, ylab = "Alcohol Pecentage", 
         main = "Box Plot for the Alcohol percentage in Wines")

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-4-1.png)

1.  Number of missing values (if any) per column

<!-- -->

    colSums(is.na(wines))

    ##        fixed.acidity     volatile.acidity          citric.acid       residual.sugar            chlorides  free.sulfur.dioxide 
    ##                    0                    0                    0                    0                    0                    0 
    ## total.sulfur.dioxide              density                   pH            sulphates              alcohol                 type 
    ##                    0                    0                    0                    0                    0                    0

1.  Correlation matrix

<!-- -->

    wines_cont = wines[, -c(12)]
    cor(wines_cont)

    ##                      fixed.acidity volatile.acidity citric.acid residual.sugar   chlorides free.sulfur.dioxide
    ## fixed.acidity           1.00000000       0.21900826  0.32443573     -0.1119813  0.29819477         -0.28273543
    ## volatile.acidity        0.21900826       1.00000000 -0.37798132     -0.1960112  0.37712428         -0.35255731
    ## citric.acid             0.32443573      -0.37798132  1.00000000      0.1424512  0.03899801          0.13312581
    ## residual.sugar         -0.11198128      -0.19601117  0.14245123      1.0000000 -0.12894050          0.40287064
    ## chlorides               0.29819477       0.37712428  0.03899801     -0.1289405  1.00000000         -0.19504479
    ## free.sulfur.dioxide    -0.28273543      -0.35255731  0.13312581      0.4028706 -0.19504479          1.00000000
    ## total.sulfur.dioxide   -0.32905390      -0.41447619  0.19524198      0.4954816 -0.27963045          0.72093408
    ## density                 0.45890998       0.27129565  0.09615393      0.5525170  0.36261466          0.02571684
    ## pH                     -0.25270047       0.26145440 -0.32980819     -0.2673198  0.04470798         -0.14585390
    ## sulphates               0.29956774       0.22598368  0.05619730     -0.1859274  0.39559331         -0.18845725
    ## alcohol                -0.09545152      -0.03764039 -0.01049349     -0.3594148 -0.25691558         -0.17983843
    ##                      total.sulfur.dioxide     density          pH    sulphates      alcohol
    ## fixed.acidity                 -0.32905390  0.45890998 -0.25270047  0.299567744 -0.095451523
    ## volatile.acidity              -0.41447619  0.27129565  0.26145440  0.225983680 -0.037640386
    ## citric.acid                    0.19524198  0.09615393 -0.32980819  0.056197300 -0.010493492
    ## residual.sugar                 0.49548159  0.55251695 -0.26731984 -0.185927405 -0.359414771
    ## chlorides                     -0.27963045  0.36261466  0.04470798  0.395593307 -0.256915580
    ## free.sulfur.dioxide            0.72093408  0.02571684 -0.14585390 -0.188457249 -0.179838435
    ## total.sulfur.dioxide           1.00000000  0.03239451 -0.23841310 -0.275726820 -0.265739639
    ## density                        0.03239451  1.00000000  0.01168608  0.259478495 -0.686745422
    ## pH                            -0.23841310  0.01168608  1.00000000  0.192123407  0.121248467
    ## sulphates                     -0.27572682  0.25947850  0.19212341  1.000000000 -0.003029195
    ## alcohol                       -0.26573964 -0.68674542  0.12124847 -0.003029195  1.000000000

There is one notable pair of highly correlated variables: free sulphur
dioxide content and total sulphur dioxide (0.72)

1.  Scatter plots to visualize highly correlated variables

<!-- -->

    par(mfrow = c(1, 2))
    plot(x = wines$free.sulfur.dioxide, wines$total.sulfur.dioxide, 
         ylab = "total sulfur dioxide Percentage", 
         xlab = "free sulfur dioxide Percentage")

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-7-1.png)

The scatter plot do show some correlation for our variables. It would be
interesting to explore their relationships further in our report.

1.  Some summary statistics for the Alcohol content:

<!-- -->

    summary(wines$alcohol)

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##    8.00    9.50   10.30   10.49   11.30   14.90

Minimum alcohol percent for a wine in the dataset = 8% Mean alcohol
percent for a wine in the dataset = 10.49% Maximum alcohol percent for a
wine in the dataset = 14.9%

    wines[which(wines$alcohol == max(wines$alcohol)), ]

    ##     fixed.acidity volatile.acidity citric.acid residual.sugar chlorides free.sulfur.dioxide total.sulfur.dioxide density   pH
    ## 653          15.9             0.36        0.65            7.5     0.096                  22                   71  0.9976 2.98
    ##     sulphates alcohol    type
    ## 653      0.84    14.9 redwine

## Linear Model Development

We will create a 80-20 train and test split on our data.

    set.seed(2345)
    train = sample(1:nrow(wines), 0.8*nrow(wines))
    train.df = wines[train,]
    test.df = wines[-train,]
    dim(train.df)

    ## [1] 5197   12

    dim(test.df)

    ## [1] 1300   12

Training and Testing Error for the full model

    wines.training = lm(alcohol~. , train.df)
    rmse = function(x, y) {
      sqrt(mean((x-y)^2))
    }
    rmse(fitted(wines.training), train.df$alcohol)

    ## [1] 0.5155815

    rmse(predict(wines.training, test.df), test.df$alcohol)

    ## [1] 0.452872

This is the RMSE for the training and testing data using the full model

train RMSE = 0.51558 test RMSE = 0.45287

1.  Greedy Algorithms for Model selection using information criteria

Greedy Algorithm using **AIC** for model selection

    step(lm(alcohol ~., data = train.df), direction = "both")

    ## Start:  AIC=-6861.61
    ## alcohol ~ fixed.acidity + volatile.acidity + citric.acid + residual.sugar + 
    ##     chlorides + free.sulfur.dioxide + total.sulfur.dioxide + 
    ##     density + pH + sulphates + type
    ## 
    ##                        Df Sum of Sq    RSS     AIC
    ## - total.sulfur.dioxide  1       0.5 1382.0 -6861.8
    ## <none>                              1381.5 -6861.6
    ## - chlorides             1       3.8 1385.3 -6849.3
    ## - free.sulfur.dioxide   1       7.6 1389.1 -6835.0
    ## - citric.acid           1      23.4 1404.8 -6776.5
    ## - volatile.acidity      1      27.0 1408.5 -6763.0
    ## - sulphates             1      87.2 1468.7 -6545.5
    ## - type                  1     223.2 1604.7 -6085.2
    ## - pH                    1     553.3 1934.8 -5113.0
    ## - fixed.acidity         1     796.3 2177.8 -4498.2
    ## - residual.sugar        1    1498.7 2880.2 -3045.3
    ## - density               1    3654.5 5036.0  -141.6
    ## 
    ## Step:  AIC=-6861.81
    ## alcohol ~ fixed.acidity + volatile.acidity + citric.acid + residual.sugar + 
    ##     chlorides + free.sulfur.dioxide + density + pH + sulphates + 
    ##     type
    ## 
    ##                        Df Sum of Sq    RSS     AIC
    ## <none>                              1382.0 -6861.8
    ## + total.sulfur.dioxide  1       0.5 1381.5 -6861.6
    ## - chlorides             1       3.8 1385.7 -6849.6
    ## - free.sulfur.dioxide   1      15.8 1397.7 -6804.8
    ## - citric.acid           1      22.9 1404.9 -6778.3
    ## - volatile.acidity      1      26.6 1408.5 -6764.9
    ## - sulphates             1      86.7 1468.7 -6547.5
    ## - type                  1     349.6 1731.5 -5691.9
    ## - pH                    1     556.7 1938.6 -5104.8
    ## - fixed.acidity         1     814.7 2196.7 -4455.3
    ## - residual.sugar        1    1535.8 2917.8 -2980.0
    ## - density               1    4108.0 5490.0   305.0

    ## 
    ## Call:
    ## lm(formula = alcohol ~ fixed.acidity + volatile.acidity + citric.acid + 
    ##     residual.sugar + chlorides + free.sulfur.dioxide + density + 
    ##     pH + sulphates + type, data = train.df)
    ## 
    ## Coefficients:
    ##         (Intercept)        fixed.acidity     volatile.acidity          citric.acid       residual.sugar            chlorides  
    ##           6.667e+02            5.315e-01            6.243e-01            5.788e-01            2.382e-01           -9.749e-01  
    ## free.sulfur.dioxide              density                   pH            sulphates        typewhitewine  
    ##          -3.698e-03           -6.735e+02            2.713e+00            1.036e+00           -1.218e+00

According to the AIC criteria for model selection:

The ideal model with the lowest AIC is:

alcohol ~ fixed.acidity + volatile.acidity + citric.acid +
residual.sugar + chlorides + free.sulfur.dioxide + density + pH +
sulphates + type

Greedy Algorithm using **BIC** for model selection

    n = nrow(train.df)
    step(lm(alcohol~., data = train.df), direction = "both", k = log(n))

    ## Start:  AIC=-6782.94
    ## alcohol ~ fixed.acidity + volatile.acidity + citric.acid + residual.sugar + 
    ##     chlorides + free.sulfur.dioxide + total.sulfur.dioxide + 
    ##     density + pH + sulphates + type
    ## 
    ##                        Df Sum of Sq    RSS     AIC
    ## - total.sulfur.dioxide  1       0.5 1382.0 -6789.7
    ## <none>                              1381.5 -6782.9
    ## - chlorides             1       3.8 1385.3 -6777.1
    ## - free.sulfur.dioxide   1       7.6 1389.1 -6762.9
    ## - citric.acid           1      23.4 1404.8 -6704.4
    ## - volatile.acidity      1      27.0 1408.5 -6690.9
    ## - sulphates             1      87.2 1468.7 -6473.4
    ## - type                  1     223.2 1604.7 -6013.1
    ## - pH                    1     553.3 1934.8 -5040.9
    ## - fixed.acidity         1     796.3 2177.8 -4426.1
    ## - residual.sugar        1    1498.7 2880.2 -2973.2
    ## - density               1    3654.5 5036.0   -69.5
    ## 
    ## Step:  AIC=-6789.7
    ## alcohol ~ fixed.acidity + volatile.acidity + citric.acid + residual.sugar + 
    ##     chlorides + free.sulfur.dioxide + density + pH + sulphates + 
    ##     type
    ## 
    ##                        Df Sum of Sq    RSS     AIC
    ## <none>                              1382.0 -6789.7
    ## - chlorides             1       3.8 1385.7 -6784.1
    ## + total.sulfur.dioxide  1       0.5 1381.5 -6782.9
    ## - free.sulfur.dioxide   1      15.8 1397.7 -6739.3
    ## - citric.acid           1      22.9 1404.9 -6712.7
    ## - volatile.acidity      1      26.6 1408.5 -6699.3
    ## - sulphates             1      86.7 1468.7 -6481.9
    ## - type                  1     349.6 1731.5 -5626.3
    ## - pH                    1     556.7 1938.6 -5039.2
    ## - fixed.acidity         1     814.7 2196.7 -4389.8
    ## - residual.sugar        1    1535.8 2917.8 -2914.4
    ## - density               1    4108.0 5490.0   370.6

    ## 
    ## Call:
    ## lm(formula = alcohol ~ fixed.acidity + volatile.acidity + citric.acid + 
    ##     residual.sugar + chlorides + free.sulfur.dioxide + density + 
    ##     pH + sulphates + type, data = train.df)
    ## 
    ## Coefficients:
    ##         (Intercept)        fixed.acidity     volatile.acidity          citric.acid       residual.sugar            chlorides  
    ##           6.667e+02            5.315e-01            6.243e-01            5.788e-01            2.382e-01           -9.749e-01  
    ## free.sulfur.dioxide              density                   pH            sulphates        typewhitewine  
    ##          -3.698e-03           -6.735e+02            2.713e+00            1.036e+00           -1.218e+00

The ideal model calculated using BIC critera is the same as calculated
using the AIC criteria, i.e:

alcohol ~ fixed.acidity + volatile.acidity + citric.acid +
residual.sugar + chlorides + free.sulfur.dioxide + density + pH +
sulphates + type

Calculating the RMSE for the training and testing data using the model
selected from the Greedy Algorithm.

    wines.training_greedy = lm(alcohol ~ fixed.acidity + volatile.acidity + 
        citric.acid + residual.sugar + chlorides + free.sulfur.dioxide + density + 
        pH + sulphates + type, train.df)

    rmse(fitted(wines.training_greedy), train.df$alcohol)

    ## [1] 0.5156707

    rmse(predict(wines.training_greedy, test.df), test.df$alcohol)

    ## [1] 0.4527027

This is the RMSE for the training and testing data using the model
chosen with greedy algorithms

train RMSE = 0.5156707 test RMSE = 0.4527027

1.  Leap-and-bounds algorithm using at least 3 different criteria.

<!-- -->

    library(leaps)

    regsubsets_selection = regsubsets(alcohol~., data = train.df)

    rs = summary(regsubsets_selection)

    rs$which

    ##   (Intercept) fixed.acidity volatile.acidity citric.acid residual.sugar chlorides free.sulfur.dioxide total.sulfur.dioxide density
    ## 1        TRUE         FALSE            FALSE       FALSE          FALSE     FALSE               FALSE                FALSE    TRUE
    ## 2        TRUE         FALSE            FALSE       FALSE          FALSE     FALSE               FALSE                FALSE    TRUE
    ## 3        TRUE         FALSE            FALSE       FALSE           TRUE     FALSE               FALSE                FALSE    TRUE
    ## 4        TRUE          TRUE            FALSE       FALSE           TRUE     FALSE               FALSE                FALSE    TRUE
    ## 5        TRUE          TRUE            FALSE       FALSE           TRUE     FALSE               FALSE                FALSE    TRUE
    ## 6        TRUE          TRUE            FALSE       FALSE           TRUE     FALSE               FALSE                FALSE    TRUE
    ## 7        TRUE          TRUE             TRUE       FALSE           TRUE     FALSE               FALSE                FALSE    TRUE
    ## 8        TRUE          TRUE             TRUE        TRUE           TRUE     FALSE               FALSE                FALSE    TRUE
    ##      pH sulphates typewhitewine
    ## 1 FALSE     FALSE         FALSE
    ## 2 FALSE     FALSE          TRUE
    ## 3 FALSE     FALSE          TRUE
    ## 4 FALSE     FALSE          TRUE
    ## 5  TRUE     FALSE          TRUE
    ## 6  TRUE      TRUE          TRUE
    ## 7  TRUE      TRUE          TRUE
    ## 8  TRUE      TRUE          TRUE

Adjusted-R^2 - The best model is the one with the largest Adjusted R^2

    rs$adjr2

    ## [1] 0.4627120 0.5240231 0.6245030 0.7061382 0.7911530 0.8025771 0.8049373 0.8074568

Using Adjusted R^2 values, the ideal model is Model 8 - Adj R^2 =
0.8074568

alcohol ~ fixed.acidity + volatile.acidity + citric.acid +
residual.sugar + density + pH + sulphates + type

CP Mallows - The best model is the one with the lowest Cp value

    rs$cp

    ## [1] 9506.49280 7828.59458 5080.14947 2848.02961  524.38009  213.03119  149.49637   81.62912

Using Cp mallows, the ideal model is Model 8 - Cp = 81.62912

alcohol ~ fixed.acidity + volatile.acidity + citric.acid +
residual.sugar + density + pH + sulphates + type

BIC - The best model is the one with the lowest BIC value

    rs$bic

    ## [1] -3212.374 -3834.513 -5059.264 -6325.707 -8092.967 -8377.765 -8432.713 -8492.725

Using the BIC, the ideal model is Model 8 - BIC = -8492.725

alcohol ~ fixed.acidity + volatile.acidity + citric.acid +
residual.sugar + density + pH + sulphates + type

All three criterias state that the model 8 is the best model when using
the Leaps and Bounds algorithm.

Using the RMSE for the training and testing data using the Model 8

    wines.training_l_and_b = lm(alcohol ~ fixed.acidity + volatile.acidity + 
        citric.acid + residual.sugar + density + pH + sulphates + type, train.df)

    rmse(fitted(wines.training_l_and_b), train.df$alcohol)

    ## [1] 0.5193281

    rmse(predict(wines.training_l_and_b, test.df), test.df$alcohol)

    ## [1] 0.4531508

This is the RMSE for the training and testing data using the model
chosen with Leaps and Bounds algorithm

train RMSE = 0.5193281 test RMSE = 0.4531508

1.  Principal Components Regression

<!-- -->

    library(pls)

    wines.pcr = pcr(alcohol~., data = train.df, scale = TRUE, ncomp = 11,
                    validation = "CV")

    summary(wines.pcr)

    ## Data:    X dimension: 5197 11 
    ##  Y dimension: 5197 1
    ## Fit method: svdpc
    ## Number of components considered: 11
    ## 
    ## VALIDATION: RMSEP
    ## Cross-validated using 10 random segments.
    ##        (Intercept)  1 comps  2 comps  3 comps  4 comps  5 comps  6 comps  7 comps  8 comps  9 comps  10 comps  11 comps
    ## CV           1.185    1.184    1.026   0.9367   0.9331   0.9291   0.9234   0.9237   0.9016   0.8726    0.7888    0.5252
    ## adjCV        1.185    1.184    1.026   0.9367   0.9330   0.9290   0.9228   0.9235   0.9014   0.8724    0.7887    0.5247
    ## 
    ## TRAINING: % variance explained
    ##           1 comps  2 comps  3 comps  4 comps  5 comps  6 comps  7 comps  8 comps  9 comps  10 comps  11 comps
    ## X        34.45104    53.69    66.81    75.65    82.16    87.20    92.11    95.50    98.07     99.32    100.00
    ## alcohol   0.06578    25.01    37.61    38.20    38.76    39.63    39.64    42.56    46.22     56.03     81.05

Choosing the optimal number of components using a Scree Plot:

    wines.pca = prcomp(train.df[,-c(11, 12)])
    plot(wines.pca$sdev[1:11], ylab = "PCAs Std Dev", xlab = "PCA number", 
         type = "l")

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-29-1.png)

Five PCs is a reasonable selection according to the scree plot since
this is where the PCA Standard Deviation levels out

Calculating RMSE with 5 components:

    wines_pcr_pred_5 = predict(wines.pcr, test.df, ncomp = 5)
    rmse(wines_pcr_pred_5, test.df$alcohol)

    ## [1] 0.940054

With 5 PCs, we get a RMSE = 0.940054

But we want to select enough number of PCs to minimize RMSE, we use the
RMSEP to select PCs instead:

    set.seed(135)
    pcr.mse = RMSEP(wines.pcr, newdata = test.df)
    plot(pcr.mse)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-31-1.png)

Using the plot, the model with 11 components gives the lowest RMSEP
value.

    wines_pcr_pred = predict(wines.pcr, test.df, ncomp = 11)
    rmse(wines_pcr_pred, test.df$alcohol)

    ## [1] 0.452872

With 11 PCs, we get a RMSE = 0.452872, which is the same as the full
model.

1.  Ridge and LASSO Regression:

**Ridge**

    library(MASS)

    train.df$type <- as.factor(train.df$type)
    wines.ridge <- lm.ridge(alcohol ~ fixed.acidity + volatile.acidity + 
        citric.acid + total.sulfur.dioxide + residual.sugar + chlorides + 
        free.sulfur.dioxide + density + pH + sulphates, data = train.df, 
        lambda = seq(0, 1, len = 200))
    summary(wines.ridge)

    ##        Length Class  Mode   
    ## coef   2000   -none- numeric
    ## scales   10   -none- numeric
    ## Inter     1   -none- numeric
    ## lambda  200   -none- numeric
    ## ym        1   -none- numeric
    ## xm       10   -none- numeric
    ## GCV     200   -none- numeric
    ## kHKB      1   -none- numeric
    ## kLW       1   -none- numeric

    matplot(wines.ridge$lambda, coef(wines.ridge), type="l", 
            xlab=expression(lambda), ylab=expression(hat(beta)), col=1)
    which.min(wines.ridge$GCV)

    ## 0.165829146 
    ##          34

    abline(v = 0.165829146)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-35-1.png)

Our chosen lambda which is a regularization parameter that controls the
penalty applied to regression coefficients is 0.165829146

Train RMSE calculation

    wines.pred.train <- cbind(1, as.matrix(train.df[,-c(11, 12)]))
    wines.pred.train <- wines.pred.train %*% coef(wines.ridge)[34,]
    rmse(wines.pred.train, train.df$alcohol)

    ## [1] 3.019105

Train RMSE = 3.019105

Test RMSE calculation

    wines.pred.test <- cbind(1, as.matrix(test.df[,-c(11, 12)])) 
    wines.pred.test <- wines.pred.test %*% coef(wines.ridge)[34,]
    rmse(wines.pred.test, test.df$alcohol)

    ## [1] 2.963918

Test RMSE = 2.963918

Ridge Regression gave us very high RMSEs for training and testing,
indicating that it is a poor choice for this dataset.

**Lasso**

    train.y = train.df$alcohol
    train.x = as.matrix(train.df[,-c(11, 12)])

    library(lars)

    ## Loaded lars 1.3

    wineslasso = lars(train.x, train.y)

Selecting parameters using Cross Validation

    set.seed(123)
    cv.ml = cv.lars(train.x, train.y)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-40-1.png)

    which.min(cv.ml$cv)

    ## [1] 99

    svm<-cv.ml$index[which.min(cv.ml$cv)]
    svm

    ## [1] 0.989899

Predicted value of the train data

    trainx<-as.matrix(train.df[,-c(11, 12)])

    predlasso<-predict(wineslasso, trainx, s=svm, mode="fraction")
    rmse(train.df$alcohol, predlasso$fit)

    ## [1] 0.5557236

Predicted value of the test data

    testx<-as.matrix(test.df[,-c(11, 12)])

    predlasso<-predict(wineslasso, testx, s=svm, mode="fraction")
    rmse(test.df$alcohol, predlasso$fit)

    ## [1] 0.5050367

For LASSO regression:

Train RMSE: 0.5557 Test RMSE: 0.5050

Comparing all of the different methods, we found that the greedy
algorithm with AIC and BIC gave us the model with the best (i.e. lowest)
Training and Test RMSE, and therefore it is the most optimal model out
of all the methods we tested.

**Therefore, our best linear model (based on minimizing RMSE) is:**

alcohol ~ fixed.acidity + volatile.acidity + citric.acid +
residual.sugar + chlorides + free.sulfur.dioxide + density + pH +
sulphates + type

## Model Diagnostics

**1. UNUSUAL OBSERVATIONS**

**High Leverage Points**

    wines.1 = wines.training_greedy
    wines.leverages = lm.influence(wines.1)$hat
    head(wines.leverages)

    ##         4590         5479         2259         2895         2523         2115 
    ## 0.0027272061 0.0032065821 0.0084471519 0.0010445367 0.0014259972 0.0007294002

Checking for Good and bad - high leverage Points:

    library(faraway)
    halfnorm(wines.leverages, nlab=6,
    labs=as.character(1:length(wines.leverages)), ylab="Leverages")

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-46-1.png)

In the plot above, we do not necessarily look for a straight line fit.
We use this plot to identify any leverages that do not follow the
pattern suggested by the rest of the data. In this case, some of the
leverages seem to be unusually large. Next, we determine which of the
leverages exceed the 2p/n threshold. p = no. of predictors for the
Model.

    wines.leverages.high =
    wines.leverages[wines.leverages>2*(10/dim(wines)[1])]
    length(wines.leverages.high)

    ## [1] 642

We identify 465 High Leverage Points in our samples. next we classify
the Leverage Points as Good or Bad:

Percentage of High Leverage Points

    465/6497

    ## [1] 0.07157149

Classifying as Good or Bad High Leverage Points:

    # Calculate the IQR for the dependent variable
    IQR_y = IQR(wines$alcohol)
    #Define a range with its lower limit being (Q1 - IQR) and upper limit being
    #(Q3 + IQR)
    QT1_y = quantile(wines$alcohol,0.25)
    QT3_y = quantile(wines$alcohol,0.75)
    lower_lim_y = QT1_y - IQR_y
    upper_lim_y = QT3_y + IQR_y
    vector_lim_y = c(lower_lim_y,upper_lim_y)
    # Range for y variable
    vector_lim_y

    ##  25%  75% 
    ##  7.7 13.1

    wines.highlev = wines[wines.leverages>2*10/6497,]
    wines.highlev_lower =
    wines.highlev[wines.highlev$alcohol < vector_lim_y[1], ]
    wines.highlev_upper =
    wines.highlev[wines.highlev$alcohol > vector_lim_y[2], ]
    wines.highlev2 = rbind(wines.highlev_lower,wines.highlev_upper)
    wines.highlev2

    ##      fixed.acidity volatile.acidity citric.acid residual.sugar chlorides free.sulfur.dioxide total.sulfur.dioxide density   pH
    ## 143            5.2            0.340        0.00           1.80     0.050                  27                   63 0.99160 3.68
    ## 145            5.2            0.340        0.00           1.80     0.050                  27                   63 0.99160 3.68
    ## 456           11.3            0.620        0.67           5.20     0.086                   6                   19 0.99880 3.22
    ## 468            8.8            0.460        0.45           2.60     0.065                   7                   18 0.99470 3.32
    ## 1229           5.1            0.420        0.00           1.80     0.044                  18                   88 0.99157 3.68
    ## 2986           5.6            0.490        0.13           4.50     0.039                  17                  116 0.99070 3.42
    ## 4194           5.4            0.500        0.13           5.00     0.028                  12                  107 0.99079 3.48
    ## 4393           6.5            0.370        0.30           2.20     0.033                  39                  107 0.98894 3.22
    ## 4844           6.7            0.210        0.36           8.55     0.020                  20                   86 0.99146 3.19
    ## 5058           5.8            0.320        0.20           2.60     0.027                  17                  123 0.98936 3.36
    ## 5107           7.1            0.360        0.28           2.40     0.036                  35                  115 0.98936 3.19
    ## 5364           5.4            0.460        0.15           2.10     0.026                  29                  130 0.98953 3.39
    ## 5385           5.6            0.190        0.31           2.70     0.027                  11                  100 0.98964 3.46
    ## 5515           4.7            0.455        0.18           1.90     0.036                  33                  106 0.98746 3.21
    ## 5749           5.8            0.240        0.28           1.40     0.038                  40                   76 0.98711 3.10
    ## 5795           7.1            0.450        0.24           2.70     0.040                  24                   87 0.98862 2.94
    ## 5950           7.0            0.360        0.25           5.70     0.015                  14                   73 0.98963 2.82
    ## 6110           6.8            0.330        0.30           2.10     0.047                  35                  147 0.98860 3.24
    ## 6160           5.4            0.270        0.22           4.60     0.022                  29                  107 0.98889 3.33
    ## 6356           6.0            0.380        0.26           3.50     0.035                  38                  111 0.98872 3.18
    ##      sulphates alcohol      type
    ## 143       0.79    14.0   redwine
    ## 145       0.79    14.0   redwine
    ## 456       0.69    13.4   redwine
    ## 468       0.79    14.0   redwine
    ## 1229      0.73    13.6   redwine
    ## 2986      0.90    13.7 whitewine
    ## 4194      0.88    13.5 whitewine
    ## 4393      0.53    13.5 whitewine
    ## 4844      0.22    13.4 whitewine
    ## 5058      0.78    13.9 whitewine
    ## 5107      0.44    13.5 whitewine
    ## 5364      0.77    13.4 whitewine
    ## 5385      0.40    13.2 whitewine
    ## 5515      0.83    14.0 whitewine
    ## 5749      0.29    13.9 whitewine
    ## 5795      0.38    13.4 whitewine
    ## 5950      0.59    13.2 whitewine
    ## 6110      0.56    13.4 whitewine
    ## 6160      0.54    13.8 whitewine
    ## 6356      0.47    13.6 whitewine

**We have 27 bad High leverage Points in our dataset.**

**OUTLIERS**

Bonferroni Correction for Outlier test:

    wines.resid = rstudent(wines.1);
    # Critical value WITH Bonferroni correction #
    bonferroni_cv = qt(.05/(2*6497), 6497-10-1)
    bonferroni_cv

    ## [1] -4.477101

Now we need to find out what studentized residuals exceed the Bonferroni
critical value:

    wines.resid.sorted = sort(abs(wines.resid), decreasing=TRUE)[1:10]
    print(wines.resid.sorted)

    ##      4381      5501      3126      3263      3253       560       565       396       354       494 
    ## 34.297168  7.425364  6.811445  5.677112  5.677112  5.566477  5.566477  5.371891  5.239486  4.963335

Comparing the values against the absolute critical value from the
Bonferroni outlier test:

    wines.outliers =
    wines.resid.sorted[abs(wines.resid.sorted) > abs(bonferroni_cv)]
    print(wines.outliers)

    ##      4381      5501      3126      3263      3253       560       565       396       354       494 
    ## 34.297168  7.425364  6.811445  5.677112  5.677112  5.566477  5.566477  5.371891  5.239486  4.963335

**We have 10 outliers in our dataset**

**Influential Points**

To check for high influential points, we will use Cook’s distance with
the cooks.distance R function:

    wines.cooks = cooks.distance(wines.1)
    sort(wines.cooks, decreasing = TRUE)[1:10]

    ##       4381       3126       3263       3253       5501       1320        227        354       1373        560 
    ## 6.76714900 0.06827438 0.02509836 0.02509836 0.02184777 0.01774149 0.01725407 0.01633500 0.01484062 0.01480395

    plot(wines.cooks)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-55-1.png)

**We have one influential point - data point 4381 since it’s cook’s
distance is greater than 1**

As point 4381 is also an outlier, we will remove it from the dataset,
since it may be causing an unusual change in behavior in our model that
is not consistent with the rest of the data. We will rerun the model on
the new dataset.

    wines_reduced = wines[-4381,]
    dim(wines_reduced)

    ## [1] 6496   12

Creating a new model based on our reduced dataset:

    wines.1.red = lm(data = wines_reduced, alcohol ~ fixed.acidity + 
        volatile.acidity + citric.acid + residual.sugar + chlorides + 
        free.sulfur.dioxide + density + pH + sulphates + type)
    summary(wines.1.red)

    ## 
    ## Call:
    ## lm(formula = alcohol ~ fixed.acidity + volatile.acidity + citric.acid + 
    ##     residual.sugar + chlorides + free.sulfur.dioxide + density + 
    ##     pH + sulphates + type, data = wines_reduced)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -3.7176 -0.2768 -0.0329  0.2391  3.9108 
    ## 
    ## Coefficients:
    ##                       Estimate Std. Error  t value Pr(>|t|)    
    ## (Intercept)          6.980e+02  4.278e+00  163.154  < 2e-16 ***
    ## fixed.acidity        5.626e-01  7.718e-03   72.896  < 2e-16 ***
    ## volatile.acidity     5.294e-01  5.007e-02   10.573  < 2e-16 ***
    ## citric.acid          4.466e-01  4.968e-02    8.990  < 2e-16 ***
    ## residual.sugar       2.421e-01  2.523e-03   95.953  < 2e-16 ***
    ## chlorides           -6.803e-01  2.105e-01   -3.232  0.00123 ** 
    ## free.sulfur.dioxide -2.127e-03  3.907e-04   -5.443 5.45e-08 ***
    ## density             -7.052e+02  4.391e+00 -160.593  < 2e-16 ***
    ## pH                   2.718e+00  4.786e-02   56.790  < 2e-16 ***
    ## sulphates            1.065e+00  4.627e-02   23.007  < 2e-16 ***
    ## typewhitewine       -1.308e+00  2.712e-02  -48.251  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.4618 on 6485 degrees of freedom
    ## Multiple R-squared:  0.8503, Adjusted R-squared:  0.8501 
    ## F-statistic:  3684 on 10 and 6485 DF,  p-value: < 2.2e-16

1.  **Constant variance Assumption**

The Breusch-Pagan test:

null hypothesis (Ho): the variance of error terms is constant (i.e.,
there is homoscedasticity).

Alternative Hypothesis (H1): The variance of error terms is not constant
(i.e., there is heteroscedasticity).

    library(lmtest)

    bptest(wines.1.red)

    ## 
    ##  studentized Breusch-Pagan test
    ## 
    ## data:  wines.1.red
    ## BP = 558.42, df = 10, p-value < 2.2e-16

**The variance of error terms is not constant** Based on the p-value of
the BP test = 2.2e-16 &lt; 0.05 -&gt; the p-value at the significance
level, we reject the null hypothesis and conclude that the Variance is
not constant.

Residuals vs Fitted Plot:

    plot(wines.1.red, which = 1)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-59-1.png)

Although the points do seem to be scattered along the zero line - almost
forming a football shaped cloud, there are some departures from the line
that are not accounted for.

To handle this, we will perform a **variance stabilizing
transformation** on our response:

We aim to find a function of the response, **h(Y)** that helps achieve
constant variance. From the above graph, we can say that the Var(Y) is
proportional to E(Y).

We will perform a square root transformation of the response:

    wines_reduced$sqrt_alcohol = sqrt(wines_reduced$alcohol)
    wines.1.red_var = lm(data = wines_reduced, sqrt_alcohol ~ fixed.acidity + 
        volatile.acidity + citric.acid + residual.sugar + chlorides + 
        free.sulfur.dioxide + density + pH + sulphates + type)
    summary(wines.1.red_var)

    ## 
    ## Call:
    ## lm(formula = sqrt_alcohol ~ fixed.acidity + volatile.acidity + 
    ##     citric.acid + residual.sugar + chlorides + free.sulfur.dioxide + 
    ##     density + pH + sulphates + type, data = wines_reduced)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.56602 -0.04208 -0.00461  0.03736  0.57778 
    ## 
    ## Coefficients:
    ##                       Estimate Std. Error  t value Pr(>|t|)    
    ## (Intercept)          1.076e+02  6.520e-01  165.068  < 2e-16 ***
    ## fixed.acidity        8.626e-02  1.176e-03   73.338  < 2e-16 ***
    ## volatile.acidity     7.490e-02  7.631e-03    9.815  < 2e-16 ***
    ## citric.acid          6.386e-02  7.572e-03    8.434  < 2e-16 ***
    ## residual.sugar       3.645e-02  3.844e-04   94.799  < 2e-16 ***
    ## chlorides           -1.115e-01  3.207e-02   -3.477  0.00051 ***
    ## free.sulfur.dioxide -3.303e-04  5.955e-05   -5.547 3.02e-08 ***
    ## density             -1.071e+02  6.692e-01 -160.025  < 2e-16 ***
    ## pH                   4.175e-01  7.293e-03   57.239  < 2e-16 ***
    ## sulphates            1.624e-01  7.052e-03   23.026  < 2e-16 ***
    ## typewhitewine       -1.980e-01  4.132e-03  -47.921  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.07038 on 6485 degrees of freedom
    ## Multiple R-squared:  0.8505, Adjusted R-squared:  0.8503 
    ## F-statistic:  3691 on 10 and 6485 DF,  p-value: < 2.2e-16

    bptest(wines.1.red_var)

    ## 
    ##  studentized Breusch-Pagan test
    ## 
    ## data:  wines.1.red_var
    ## BP = 595.8, df = 10, p-value < 2.2e-16

Based on the p-value of the BP test = 2.2e-16 &lt; 0.05 -&gt; the
p-value at the significance level, we again reject the null hypothesis
and conclude that the Variance after transformation of the response is
not constant.

A look at the Residuals vs. Fitted plot:

    plot(wines.1.red_var, which = 1)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-62-1.png)

The statistical BP test still fails for the transformed data, we
conclude that the transformations did not help in stabilizing and
upholding the constant variance assumption. Another approach that may
resolve the problem is a Weighted Least Squares method. Let’s use this
on top of the response transformation.

    wt <- lm(abs(wines.1.red$residuals) ~ wines.1.red$fitted.values)$fitted.values^2
    wt <- 1/wt
    #perform weighted least squares regression
    wls_wines.1.red <- lm(alcohol ~ fixed.acidity + volatile.acidity + citric.acid +
    residual.sugar + chlorides + free.sulfur.dioxide + density + pH +
    sulphates + type, data = wines_reduced, weights=wt)

    #view summary of model
    summary(wls_wines.1.red)

    ## 
    ## Call:
    ## lm(formula = alcohol ~ fixed.acidity + volatile.acidity + citric.acid + 
    ##     residual.sugar + chlorides + free.sulfur.dioxide + density + 
    ##     pH + sulphates + type, data = wines_reduced, weights = wt)
    ## 
    ## Weighted Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -11.6724  -0.8071  -0.0975   0.7043  11.3549 
    ## 
    ## Coefficients:
    ##                       Estimate Std. Error  t value Pr(>|t|)    
    ## (Intercept)          7.007e+02  4.245e+00  165.063  < 2e-16 ***
    ## fixed.acidity        5.617e-01  7.703e-03   72.924  < 2e-16 ***
    ## volatile.acidity     5.316e-01  4.994e-02   10.644  < 2e-16 ***
    ## citric.acid          4.513e-01  4.973e-02    9.075  < 2e-16 ***
    ## residual.sugar       2.431e-01  2.516e-03   96.646  < 2e-16 ***
    ## chlorides           -6.759e-01  2.121e-01   -3.186  0.00145 ** 
    ## free.sulfur.dioxide -2.143e-03  3.911e-04   -5.480 4.41e-08 ***
    ## density             -7.078e+02  4.357e+00 -162.445  < 2e-16 ***
    ## pH                   2.715e+00  4.780e-02   56.787  < 2e-16 ***
    ## sulphates            1.069e+00  4.609e-02   23.189  < 2e-16 ***
    ## typewhitewine       -1.318e+00  2.704e-02  -48.740  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1.357 on 6485 degrees of freedom
    ## Multiple R-squared:  0.8532, Adjusted R-squared:  0.853 
    ## F-statistic:  3769 on 10 and 6485 DF,  p-value: < 2.2e-16

    bptest(wls_wines.1.red)

    ## 
    ##  studentized Breusch-Pagan test
    ## 
    ## data:  wls_wines.1.red
    ## BP = 50749, df = 10, p-value < 2.2e-16

It seems that even the Weighted Least Squares approach could not resolve
this issue with non-constant variance. This is a potential shortcoming
of our final model. Conclusions that are drawn from our selected model
may not be reliable.

1.  **Test for Normality of Residuals**

We fist plot a histogram of the residuals for our model:

    hist(wines.1.red$residuals)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-65-1.png)

The residuals for the response do look like they follow a Normal
distribution, to confirm this, we can run a statistical Test for
Normality:

As we have a large number of samples in our data set, the statistical
test to check for normality in this case will be a Kolmogorov-Smirnov
test:

H0: The residuals are normally distributed (normality assumption holds)
Ha: The residuals aren’t normally distributed (normality assumption
doesn’t hold)

    ks.test(wines.1.red$residuals, "pnorm")

    ## 
    ##  Asymptotic one-sample Kolmogorov-Smirnov test
    ## 
    ## data:  wines.1.red$residuals
    ## D = 0.20828, p-value < 2.2e-16
    ## alternative hypothesis: two-sided

**The Normality assumption is not met** Based on the p-value of the KS
test = 2.2e-16 &lt; 0.05 -&gt; the p-value at the significance level, we
reject the null hypothesis and conclude that the Normality assumption
for the error terms is not met.

To handle this, we will perform a Box-Cox transformation of the
response:

    winesB.transformation = boxcox(wines.1.red,
    lambda=seq(-2, 2, length=400))

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-67-1.png)

    best_lambda <- winesB.transformation$x[which.max(winesB.transformation$y)]
    best_lambda

    ## [1] -0.3759398

The 95% Interval for the Box-Cox plot contains *λ*s near -0.5, so we
will perform an inverse square root transformation to try and fix the
issues with normality.

    wines_reduced$alcohol_trans = (wines_reduced$alcohol^(best_lambda) - 1)
    wines_reduced$alcohol_trans = wines_reduced$alcohol_trans/(best_lambda) 
    # Re-transforming the data set 
    # back to the original reduced data set and performing a log transformation 

    wines.1.red_norm = lm(data = wines_reduced, alcohol_trans ~ fixed.acidity + 
                           volatile.acidity + citric.acid + 
        residual.sugar + chlorides + free.sulfur.dioxide + density + 
        pH + sulphates + type)
    summary(wines.1.red_norm)

    ## 
    ## Call:
    ## lm(formula = alcohol_trans ~ fixed.acidity + volatile.acidity + 
    ##     citric.acid + residual.sugar + chlorides + free.sulfur.dioxide + 
    ##     density + pH + sulphates + type, data = wines_reduced)
    ## 
    ## Residuals:
    ##       Min        1Q    Median        3Q       Max 
    ## -0.141840 -0.010666 -0.000634  0.009427  0.137414 
    ## 
    ## Coefficients:
    ##                       Estimate Std. Error  t value Pr(>|t|)    
    ## (Intercept)          2.757e+01  1.655e-01  166.551  < 2e-16 ***
    ## fixed.acidity        2.187e-02  2.986e-04   73.234  < 2e-16 ***
    ## volatile.acidity     1.632e-02  1.937e-03    8.423  < 2e-16 ***
    ## citric.acid          1.406e-02  1.922e-03    7.315 2.88e-13 ***
    ## residual.sugar       8.944e-03  9.760e-05   91.635  < 2e-16 ***
    ## chlorides           -3.123e-02  8.143e-03   -3.835 0.000127 ***
    ## free.sulfur.dioxide -8.552e-05  1.512e-05   -5.657 1.60e-08 ***
    ## density             -2.669e+01  1.699e-01 -157.108  < 2e-16 ***
    ## pH                   1.062e-01  1.852e-03   57.355  < 2e-16 ***
    ## sulphates            4.085e-02  1.790e-03   22.820  < 2e-16 ***
    ## typewhitewine       -4.903e-02  1.049e-03  -46.738  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.01787 on 6485 degrees of freedom
    ## Multiple R-squared:  0.8479, Adjusted R-squared:  0.8477 
    ## F-statistic:  3616 on 10 and 6485 DF,  p-value: < 2.2e-16

    hist(wines.1.red_norm$residuals)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-69-1.png)

Although the histogram of the residuals looks more like it follows a
Normal distribution, We will look at the QQ plot for normality:

    plot(wines.1.red_norm, which = 2)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-70-1.png)
The QQ plot sees departures from the line.

    ks.test(wines.1.red_norm$residuals, "pnorm")
    
    ## 
    ##  Asymptotic one-sample Kolmogorov-Smirnov test
    ## 
    ## data:  wines.1.red_norm$residuals
    ## D = 0.47643, p-value < 2.2e-16
    ## alternative hypothesis: two-sided

Based on the p-value of the KS test on the transformed response =
2.2e-16 &lt; 0.05 -&gt; the p-value at the significance level, we reject
the null hypothesis and conclude that the Normality assumption for the
error terms in our transformed model is again not met.

This is a potential shortcoming in our model (the normality assumption
not being met). This may cause our predictions to be unreliable.

1.  **Test for Collinearity**

Removing the intercept and categorical variable:

Variance Inflation Factor:

    x = model.matrix(wines.1.red)[,-c(1, 11)]
    round(vif(x), dig=2)

    ##       fixed.acidity    volatile.acidity         citric.acid      residual.sugar           chlorides free.sulfur.dioxide 
    ##                2.93                1.75                1.56                3.81                1.61                1.40 
    ##             density                  pH           sulphates 
    ##                4.55                1.73                1.37

Note that the standard error for the coefficient associated with
`density` is 2.133 = (4.55)^0.5 times larger than it would have been
without collinearity.

Computing the Condition Number

    x = x - matrix(apply(x,2, mean), 6496,9, byrow=TRUE)
    x = x / matrix(apply(x, 2, sd), 6496,9, byrow=TRUE)
    eigenvalues.x = eigen(t(x) %*% x) 
    eigenvalues.x$val

    ## [1] 16110.6209 13229.2885  8886.1052  6015.5170  4429.2525  3516.7822  3258.4464  2356.2294   652.7579

    sqrt(eigenvalues.x$val[1]/eigenvalues.x$val[9])

    ## [1] 4.967983

The condition number is 4.968, lower than 30, so we conclude that
collinearity is absent.

1.  **Tests for Linearity**

To check that all of our predictors varied linearly with the response,
we have constructed added variable plots for each predictor, as seen
below.

    # Fixed Acidity
    y = update(wines.1.red, .~. -fixed.acidity)$res
    x = lm(fixed.acidity ~ ., data = wines_reduced[,-c(11)])$res
    plot(x, y, xlab="Fixed Acidity Residuals", ylab="Alcohol Residuals", 
         col='Darkblue', pch=3, size=3)
    abline(lm(y ~ x), col='Darkblue', lwd=2)
    abline(v = 0, col="red", lty=3)
    abline(h = 0, col="red", lty=3)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-75-1.png)

    # Volatile Acidity
    y = update(wines.1.red, .~. -volatile.acidity)$res
    x = lm(volatile.acidity ~ ., data = wines_reduced[,-c(11)])$res
    plot(x, y, xlab="Volatile Acidity Residuals", ylab="Alcohol Residuals", 
         col='Darkblue', pch=3, size=3)

    abline(lm(y ~ x), col='Darkblue', lwd=2)
    abline(v = 0, col="red", lty=3)
    abline(h = 0, col="red", lty=3)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-75-2.png)

    # Citric Acid
    y = update(wines.1.red, .~. -citric.acid)$res
    x = lm(citric.acid ~ ., data = wines_reduced[,-c(11)])$res
    plot(x, y, xlab="Citric Acid Residuals", ylab="Alcohol Residuals", 
         col='Darkblue', pch=3, size=3)

    abline(lm(y ~ x), col='Darkblue', lwd=2)
    abline(v = 0, col="red", lty=3)
    abline(h = 0, col="red", lty=3)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-75-3.png)

    # Residual Sugar
    y = update(wines.1.red, .~. -residual.sugar)$res
    x = lm(residual.sugar ~ ., data = wines_reduced[,-c(11)])$res
    plot(x, y, xlab="Residual Sugar Residuals", ylab="Alcohol Residuals", 
         col='Darkblue', pch=3, size=3)

    abline(lm(y ~ x), col='Darkblue', lwd=2)
    abline(v = 0, col="red", lty=3)
    abline(h = 0, col="red", lty=3)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-75-4.png)

    # Chlorides
    y = update(wines.1.red, .~. -chlorides)$res
    x = lm(chlorides ~ ., data = wines_reduced[,-c(11)])$res
    plot(x, y, xlab="Chlorides Residuals", ylab="Alcohol Residuals", 
         col='Darkblue', pch=3, size=3)

    abline(lm(y ~ x), col='Darkblue', lwd=2)
    abline(v = 0, col="red", lty=3)
    abline(h = 0, col="red", lty=3)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-75-5.png)

    # Free Sulfur Dioxide
    y = update(wines.1.red, .~. -free.suldur.dioxide)$res
    x = lm(free.sulfur.dioxide ~ ., data = wines_reduced[,-c(11)])$res
    plot(x, y, xlab="Free Sulfur Dioxide Residuals", ylab="Alcohol Residuals", 
         col='Darkblue', pch=3, size=3)
         
    abline(lm(y ~ x), col='Darkblue', lwd=2)
    abline(v = 0, col="red", lty=3)
    abline(h = 0, col="red", lty=3)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-75-6.png)

    # Density
    y = update(wines.1.red, .~. -density)$res
    x = lm(density ~ ., data = wines_reduced[,-c(11)])$res
    plot(x, y, xlab="Density Residuals", ylab="Alcohol Residuals", col='Darkblue', 
         pch=3, size=3)

    abline(lm(y ~ x), col='Darkblue', lwd=2)
    abline(v = 0, col="red", lty=3)
    abline(h = 0, col="red", lty=3)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-75-7.png)

    # pH
    y = update(wines.1.red, .~. -pH)$res
    x = lm(pH ~ ., data = wines_reduced[,-c(11)])$res
    plot(x, y, xlab="pH Residuals", ylab="Alcohol Residuals", col='Darkblue',
         pch=3, size=3)

    abline(lm(y ~ x), col='Darkblue', lwd=2)
    abline(v = 0, col="red", lty=3)
    abline(h = 0, col="red", lty=3)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-75-8.png)

    # Sulphates
    y = update(wines.1.red, .~. -sulphates)$res
    x = lm(sulphates ~ ., data = wines_reduced[,-c(11)])$res
    plot(x, y, xlab="Sulphates Residuals", ylab="Alcohol Residuals", 
         col='Darkblue', pch=3, size=3)

    abline(lm(y ~ x), col='Darkblue', lwd=2)
    abline(v = 0, col="red", lty=3)
    abline(h = 0, col="red", lty=3)

![](Wine_Chemical_Data_Case_Study_files/figure-markdown_strict/unnamed-chunk-75-9.png)

As we can see, for every added variable plot, the points are scattered
roughly symmetrically across the best fit line (except for maybe the
chlorides plot). Therefore, the linearity assumption is met for our MLR
model.

**Differences/Similarities in Model A and B**

Model A uses testing and training data sets and uses a greedy algorithm
for model selection based on the lowest BIC and AIC values. Model B uses
testing procedures for model selection. The model selection is done
based on p-values of the predictors through a backward elimination
testing procedure. Both methods ultimately resulted in the same model
-&gt; model A and model B have the same predictors.

**RESULTS**

    confint(wines.training_greedy)

    ##                             2.5 %        97.5 %
    ## (Intercept)          6.563066e+02  6.770255e+02
    ## fixed.acidity        5.126614e-01  5.503508e-01
    ## volatile.acidity     5.017003e-01  7.468765e-01
    ## citric.acid          4.565062e-01  7.011865e-01
    ## residual.sugar       2.320104e-01  2.443106e-01
    ## chlorides           -1.482588e+00 -4.671555e-01
    ## free.sulfur.dioxide -4.640142e-03 -2.755417e-03
    ## density             -6.841061e+02 -6.628386e+02
    ## pH                   2.596597e+00  2.829332e+00
    ## sulphates            9.234153e-01  1.148560e+00
    ## typewhitewine       -1.283423e+00 -1.151620e+00

**Insights:**

All 3 acidity measures (fixed, volatile, citric), sugar, and sulphates
are all positively correlated with alcohol content, whereas chlorides,
free *S**O*<sub>2</sub>, and density are negatively correlated with
alcohol content. Free *S**O*<sub>2</sub> in particular, seems to have
much smaller coefficients (by magnitude), and by proxy a weaker
correlation, but this could also be due to a choice in units. White
wines also tend to have lower alcohol contents than red wines.

## CONCLUSION

After fitting various types of linear models to our dataset, we found
that a full multiple linear regression excluding total sulfur dioxide as
a predictor minimized both training and testing RMSE the best. However,
this approach consistently encountered departures from key model
assumptions, particularly the normality and constant variance of
residuals.

We attempted to address these violations through remedial measures, such
as transformations of the response variable and model adjustments, but
we were unsuccessful in resolving the assumption failures. This is a
potential limitation in the reliability of our models for accurately
predicting alcohol percentage of new wines.

One significant contributing factor to this issue was the presence of
numerous unusual observations in the dataset, which may have introduced
noise or distortions to the modeling process. Addressing this limitation
by exploring alternative models, or reassessing the predictors selected
could improve future outcomes.

Despite these challenges, our systematic approach to model selection,
guided by metrics such as RMSE for prediction accuracy, allowed us to
identify the most optimal model for the task. To our surprise, the
leaps-and-bounds approach, Ridge/LASSO shrinkage, and Principal
Components methods all fell short of the basic greedy model building
method. This suggests that the connection between a wine’s alcohol
content and chemical indicators of the wine is fairly simple and
straightforward.

