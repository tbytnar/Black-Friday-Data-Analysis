# Black Friday Data Analysis

This is a walkthrough of my analysis of the Black Friday (2018) data from Analytics Vidhya: https://datahack.analyticsvidhya.com/contest/black-friday/

I wanted to try my hand at analyzing real world sales data and came across the Black Firday 2018 data from Analytics Vidhya.  

## Environment

Everything here is built in R 3.5.2 "Eggshell Igloo" using RStudio on my home gaming pc (this way I'm not waiting around all day for models to execute).

### Required Packages

I tend to load all of my dependencies at the beginning of my scripts.

```
install.packages("data.table")
install.packages("ggplot2")
install.packages("gmodels")
install.packages("dummies")
install.packages("h2o")

library(data.table)
library(ggplot2)
library(gmodels)
library(dummies)
library(h2o)
```

### Studying the data

First things first, I read in the training and testing data using fread so that they are loaded as data tables.  Then (arguably) the most critical step is studying the data and determining what data is useful for analysis.

```
> train <- fread("train.csv", stringsAsFactors = T)
> test <- fread("test.csv", stringsAsFactors = T)


> dim(train)
[1] 550068     12
> dim(test)
[1] 233599     11
> str(train)
Classes ‘data.table’ and 'data.frame':	550068 obs. of  12 variables:
 $ User_ID                   : int  1000001 1000001 1000001 1000001 1000002 1000003 1000004 1000004 1000004 1000005 ...
 $ Product_ID                : Factor w/ 3631 levels "P00000142","P00000242",..: 673 2377 853 829 2735 1832 1746 3321 3605 2632 ...
 $ Gender                    : Factor w/ 2 levels "F","M": 1 1 1 1 2 2 2 2 2 2 ...
 $ Age                       : Factor w/ 7 levels "0-17","18-25",..: 1 1 1 1 7 3 5 5 5 3 ...
 $ Occupation                : int  10 10 10 10 16 15 7 7 7 20 ...
 $ City_Category             : Factor w/ 3 levels "A","B","C": 1 1 1 1 3 1 2 2 2 1 ...
 $ Stay_In_Current_City_Years: Factor w/ 5 levels "0","1","2","3",..: 3 3 3 3 5 4 3 3 3 2 ...
 $ Marital_Status            : int  0 0 0 0 0 0 1 1 1 1 ...
 $ Product_Category_1        : int  3 1 12 12 8 1 1 1 1 8 ...
 $ Product_Category_2        : int  NA 6 NA 14 NA 2 8 15 16 NA ...
 $ Product_Category_3        : int  NA 14 NA NA NA NA 17 NA NA NA ...
 $ Purchase                  : int  8370 15200 1422 1057 7969 15227 19215 15854 15686 7871 ...
 - attr(*, ".internal.selfref")=<externalptr> 
```

And repeat

```
until finished
```

End with an example of getting some data out of the system or using it for a little demo

## Running the tests

Explain how to run the automated tests for this system

### Break down into end to end tests

Explain what these tests test and why

```
Give an example
```

### And coding style tests

Explain what these tests test and why

```
Give an example
```

## Deployment

Add additional notes about how to deploy this on a live system

## Built With

* [Dropwizard](http://www.dropwizard.io/1.0.2/docs/) - The web framework used
* [Maven](https://maven.apache.org/) - Dependency Management
* [ROME](https://rometools.github.io/rome/) - Used to generate RSS Feeds

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Billie Thompson** - *Initial work* - [PurpleBooth](https://github.com/PurpleBooth)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Hat tip to anyone whose code was used
* Inspiration
* etc





#load data using fread
train <- fread("train.csv", stringsAsFactors = T)
test <- fread("test.csv", stringsAsFactors = T)

#No. of rows and columns in Train
dim(train)

#No. of rows and columns in Test
dim(test)

str(train)

#first prediction using mean
sub_mean <- data.frame(User_ID = test$User_ID, Product_ID = test$Product_ID, Purchase = mean(train$Purchase))
write.csv(sub_mean, file = "first_sub.csv", row.names = F)

summary (train)

summary (test)

#combine data set
test[,Purchase := mean(train$Purchase)]
c <- list(train, test)
combin <- rbindlist(c)

#analyzing gender variable
combin[,prop.table(table(Gender))] Gender

#Age Variable
combin[,prop.table(table(Age))]

#City Category Variable
combin[,prop.table(table(City_Category))]

#Stay in Current Years Variable
combin[,prop.table(table(Stay_In_Current_City_Years))]

#unique values in ID variables
length(unique(combin$Product_ID))

length(unique(combin$User_ID))

#missing values
colSums(is.na(combin))


#Age vs Gender
ggplot(combin, aes(Age, fill = Gender)) + geom_bar()

#Age vs City_Category
ggplot(combin, aes(Age, fill = City_Category)) + geom_bar()

#Show CrossTable of Occupation by City_Category
CrossTable(combin$Occupation, combin$City_Category)

#create a new variable for missing values
combin[,Product_Category_2_NA := ifelse(sapply(combin$Product_Category_2, is.na) ==    TRUE,1,0)]
combin[,Product_Category_3_NA := ifelse(sapply(combin$Product_Category_3, is.na) ==  TRUE,1,0)]

#impute missing values
combin[,Product_Category_2 := ifelse(is.na(Product_Category_2) == TRUE, "-999",  Product_Category_2)]
combin[,Product_Category_3 := ifelse(is.na(Product_Category_3) == TRUE, "-999",  Product_Category_3)]

#set column level
levels(combin$Stay_In_Current_City_Years)[levels(combin$Stay_In_Current_City_Years) ==  "4+"] <- "4"

#recoding age groups
levels(combin$Age)[levels(combin$Age) == "0-17"] <- 0
levels(combin$Age)[levels(combin$Age) == "18-25"] <- 1
levels(combin$Age)[levels(combin$Age) == "26-35"] <- 2
levels(combin$Age)[levels(combin$Age) == "36-45"] <- 3
levels(combin$Age)[levels(combin$Age) == "46-50"] <- 4
levels(combin$Age)[levels(combin$Age) == "51-55"] <- 5
levels(combin$Age)[levels(combin$Age) == "55+"] <- 6

#convert age to numeric
combin$Age <- as.numeric(combin$Age)

#convert Gender into numeric
combin[, Gender := as.numeric(as.factor(Gender)) - 1]

#User Count
combin[, User_Count := .N, by = User_ID]

#Product Count
combin[, Product_Count := .N, by = Product_ID]

#Mean Purchase of Product
combin[, Mean_Purchase_Product := mean(Purchase), by = Product_ID]

#Mean Purchase of User
combin[, Mean_Purchase_User := mean(Purchase), by = User_ID]

#Adding dummies to City_Category 
combin <- dummy.data.frame(combin, names = c("City_Category"), sep = "_")

#check classes of all variables
sapply(combin, class)

#converting Product Category 2 & 3
combin$Product_Category_2 <- as.integer(combin$Product_Category_2)
combin$Product_Category_3 <- as.integer(combin$Product_Category_3)


#Divide into train and test
c.train <- combin[1:nrow(train),]
c.test <- combin[-(1:nrow(train)),]


c.train <- c.train[c.train$Product_Category_1 <= 18,]

localH2O <- h2o.init(nthreads = -1)


h2o.init()

#data to h2o cluster
train.h2o <- as.h2o(c.train)
test.h2o <- as.h2o(c.test)

#check column index number
colnames(train.h2o)

#dependent variable (Purchase)
y.dep <- 14

#independent variables (dropping ID variables)
x.indep <- c(3:13,15:20)



#Multiple Regression in H2O
regression.model <- h2o.glm( y = y.dep, x = x.indep, training_frame = train.h2o, family = "gaussian")

h2o.performance(regression.model)


#make predictions
predict.reg <- as.data.frame(h2o.predict(regression.model, test.h2o))
sub_reg <- data.frame(User_ID = test$User_ID, Product_ID = test$Product_ID, Purchase =  predict.reg$predict)

write.csv(sub_reg, file = "sub_reg.csv", row.names = F)




#Random Forest
system.time(
  rforest.model <- h2o.randomForest(y=y.dep, x=x.indep, training_frame = train.h2o, ntrees = 1000, mtries = 3, max_depth = 4, seed = 1122)
)

h2o.performance(rforest.model)
h2o.varimp(rforest.model)

#making predictions on unseen data
system.time(predict.rforest <- as.data.frame(h2o.predict(rforest.model, test.h2o)))

#writing submission file
sub_rf <- data.frame(User_ID = test$User_ID, Product_ID = test$Product_ID, Purchase =  predict.rforest$predict)
write.csv(sub_rf, file = "sub_rf.csv", row.names = F)





#GBM
system.time(
  gbm.model <- h2o.gbm(y=y.dep, x=x.indep, training_frame = train.h2o, ntrees = 1000, max_depth = 4, learn_rate = 0.01, seed = 1122)
)

h2o.performance (gbm.model)

#making prediction and writing submission file
predict.gbm <- as.data.frame(h2o.predict(gbm.model, test.h2o))
sub_gbm <- data.frame(User_ID = test$User_ID, Product_ID = test$Product_ID, Purchase = predict.gbm$predict)
write.csv(sub_gbm, file = "sub_gbm.csv", row.names = F)




#deep learning models
system.time(
  dlearning.model <- h2o.deeplearning(y = y.dep,
                                      x = x.indep,
                                      training_frame = train.h2o,
                                      epoch = 60,
                                      hidden = c(100,100),
                                      activation = "Rectifier",
                                      seed = 1122
  )
)

h2o.performance(dlearning.model)


#making predictions
predict.dl2 <- as.data.frame(h2o.predict(dlearning.model, test.h2o))

#create a data frame and writing submission file
sub_dlearning <- data.frame(User_ID = test$User_ID, Product_ID = test$Product_ID, Purchase = predict.dl2$predict)
write.csv(sub_dlearning, file = "sub_dlearning_new.csv", row.names = F)

