# Peer-graded-Assignment-Prediction-Assignment-Writeup-

## Data

The training data is available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available in the following site:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

# Analysis

Loading the data:

```{r load data, warning=FALSE, message=FALSE, echo=TRUE}
training = read.csv("./pml-training.csv",na.strings=c("NA","#DIV/0!",""))
testing = read.csv("./pml-testing.csv",na.strings=c("NA","#DIV/0!",""))
# Data dimensions
dim(training)
dim(testing)
```

```{r first look, warning=FALSE, message=FALSE, eval= FALSE}
# First look at the data
head(training)
head(testing)
```


```{r cross-validation, warning=FALSE, message=FALSE, echo=TRUE}
# load packages
library(caret)
library(randomForest)
# Index for training dataset (70%) and testing dataset (30%) 
# from the pml-training data set
set.seed(12345)
inTrain = createDataPartition(y=training$classe,p=0.7, list=FALSE)
# training dataset
training.set = training[inTrain,]
# testing dataset
testing.set = training[-inTrain,]
```

```{r clean data, warning=FALSE, message=FALSE, echo=TRUE}
# Remove near zero variance predictors
ind.nzv = nearZeroVar(x = training, saveMetrics = T)
# Remove variables with more than 50% NA values
ind.NA = !as.logical(apply(training, 2, function(x){ mean(is.na(x)) >= 0.5}))
# Cleaning data
ind2 = ind.NA*1 + (!ind.nzv$nzv)*1
ind3 = ind2 == 2
sum(ind3)
#View(data.frame(ind.NA, !ind.nzv$nzv, ind2, ind3))
training.set = training.set[,ind3]
testing.set = testing.set[, ind3]
training.set = training.set[, -1]
testing.set = testing.set[, -1]
testing = testing[,ind3]
testing = testing[,-1]
# Coerce the data into the same type in order to avoid
# "Matching Error" when calling random forest model, due to different levels in variables
for (i in 1:length(testing) ) {
  for(j in 1:length(training.set)) {
    if( length( grep(names(training.set[i]), names(testing)[j]) ) == 1)  {
      class(testing[j]) <- class(training.set[i])
    }      
  }      
}
# To get the same class between testing and training.set
testing = testing[,-ncol(testing)]
testing <- rbind(training.set[2, -58] , testing)
testing <- testing[-1,]
```

```{r prediction with trees, warning=FALSE, message=FALSE, echo=TRUE}
# Prediction with Trees
# Build model
set.seed(12345)
tree.fit = train(y = training.set$classe,
                 x = training.set[,-ncol(training.set)],
                 method = "rpart")
# Plot classification tree
rattle::fancyRpartPlot(
  tree.fit$finalModel
)
# Predictions with rpart model
pred.tree = predict(tree.fit, testing.set[,-ncol(testing.set)])
# Get results (Accuracy, etc.)
confusionMatrix(pred.tree, testing.set$classe)
```

```{r random forest, warning=FALSE, message=FALSE, echo=TRUE}
# Prediction with Random Forest
# Build model
set.seed(12345)
rf.fit = randomForest(
  classe ~ .,
  data = training.set,
  ntree = 250)
# Plot the Random Forests model
plot(rf.fit)
# Predict with random forest model
pred2 = predict(
  rf.fit,
  testing.set[,-ncol(testing.set)]
)
# Get results (Accuracy, etc.)
confusionMatrix(pred2, testing.set$classe)
```

```{r pml-testing predictions, warning=FALSE, message=FALSE, echo=TRUE}
# Get predictions for the 20 observations of the original pml-testing.csv
pred.validation = predict(rf.fit, testing)
pred.validation
```

```{r saving results, warning=FALSE, message=FALSE, echo=TRUE, eval = FALSE}
# Saving predictions for testing dataset
testing$pred.classe = pred.validation
write.table(
  testing,
  file = "testing_with_predictions",
  quote = F
)
```
