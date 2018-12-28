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
 
 > str(test)
Classes ‘data.table’ and 'data.frame':	233599 obs. of  11 variables:
 $ User_ID                   : int  1000004 1000009 1000010 1000010 1000011 1000013 1000013 1000013 1000015 1000022 ...
 $ Product_ID                : Factor w/ 3491 levels "P00000142","P00000242",..: 1145 995 2673 1300 520 3241 1400 3438 1459 639 ...
 $ Gender                    : Factor w/ 2 levels "F","M": 2 2 1 1 1 2 2 2 2 2 ...
 $ Age                       : Factor w/ 7 levels "0-17","18-25",..: 5 3 4 4 3 5 5 5 3 2 ...
 $ Occupation                : int  7 17 1 1 1 1 1 1 7 15 ...
 $ City_Category             : Factor w/ 3 levels "A","B","C": 2 3 2 2 3 3 3 3 1 1 ...
 $ Stay_In_Current_City_Years: Factor w/ 5 levels "0","1","2","3",..: 3 1 5 5 2 4 4 4 2 5 ...
 $ Marital_Status            : int  1 0 1 1 0 1 1 1 0 0 ...
 $ Product_Category_1        : int  1 3 5 4 4 2 1 2 10 5 ...
 $ Product_Category_2        : int  11 5 14 9 5 3 11 4 13 14 ...
 $ Product_Category_3        : int  NA NA NA NA 12 15 15 9 16 NA ...
 - attr(*, ".internal.selfref")=<externalptr> 
```

The first thing to note is that the testing data is missing the Purchase column.  That makes sense as they're wanting us to predict it.

The Gender, Age, City_Category and Stay_In_Current_City_Years columns all have character data in them which I'll need to recode into integers for effective analysis.

I can also already see that Product_Category_2 and Product_Category_3 contain 'NA' values.  This will have to get recoded as well and I'll likely need to add a column to flag this later.

```
> summary (train)
    User_ID            Product_ID     Gender        Age           Occupation     City_Category
 Min.   :1000001   P00265242:  1880   F:135809   0-17 : 15102   Min.   : 0.000   A:147720     
 1st Qu.:1001516   P00025442:  1615   M:414259   18-25: 99660   1st Qu.: 2.000   B:231173     
 Median :1003077   P00110742:  1612              26-35:219587   Median : 7.000   C:171175     
 Mean   :1003029   P00112142:  1562              36-45:110013   Mean   : 8.077                
 3rd Qu.:1004478   P00057642:  1470              46-50: 45701   3rd Qu.:14.000                
 Max.   :1006040   P00184942:  1440              51-55: 38501   Max.   :20.000                
                   (Other)  :540489              55+  : 21504                                 
 Stay_In_Current_City_Years Marital_Status   Product_Category_1 Product_Category_2 Product_Category_3
 0 : 74398                  Min.   :0.0000   Min.   : 1.000     Min.   : 2.00      Min.   : 3.0      
 1 :193821                  1st Qu.:0.0000   1st Qu.: 1.000     1st Qu.: 5.00      1st Qu.: 9.0      
 2 :101838                  Median :0.0000   Median : 5.000     Median : 9.00      Median :14.0      
 3 : 95285                  Mean   :0.4097   Mean   : 5.404     Mean   : 9.84      Mean   :12.7      
 4+: 84726                  3rd Qu.:1.0000   3rd Qu.: 8.000     3rd Qu.:15.00      3rd Qu.:16.0      
                            Max.   :1.0000   Max.   :20.000     Max.   :18.00      Max.   :18.0      
                                                                NA's   :173638     NA's   :383247    
    Purchase    
 Min.   :   12  
 1st Qu.: 5823  
 Median : 8047  
 Mean   : 9264  
 3rd Qu.:12054  
 Max.   :23961  
                
> 
> summary (test)
    User_ID            Product_ID     Gender        Age          Occupation     City_Category
 Min.   :1000001   P00265242:   829   F: 57827   0-17 : 6232   Min.   : 0.000   A:62524      
 1st Qu.:1001527   P00112142:   717   M:175772   18-25:42293   1st Qu.: 2.000   B:98566      
 Median :1003070   P00025442:   695              26-35:93428   Median : 7.000   C:72509      
 Mean   :1003029   P00110742:   680              36-45:46711   Mean   : 8.085                
 3rd Qu.:1004477   P00046742:   646              46-50:19577   3rd Qu.:14.000                
 Max.   :1006040   P00184942:   626              51-55:16283   Max.   :20.000                
                   (Other)  :229406              55+  : 9075                                 
 Stay_In_Current_City_Years Marital_Status   Product_Category_1 Product_Category_2 Product_Category_3
 0 :31318                   Min.   :0.0000   Min.   : 1.000     Min.   : 2.00      Min.   : 3.00     
 1 :82604                   1st Qu.:0.0000   1st Qu.: 1.000     1st Qu.: 5.00      1st Qu.: 9.00     
 2 :43589                   Median :0.0000   Median : 5.000     Median : 9.00      Median :14.00     
 3 :40143                   Mean   :0.4101   Mean   : 5.277     Mean   : 9.85      Mean   :12.67     
 4+:35945                   3rd Qu.:1.0000   3rd Qu.: 8.000     3rd Qu.:15.00      3rd Qu.:16.00     
                            Max.   :1.0000   Max.   :18.000     Max.   :18.00      Max.   :18.00     
                                                                NA's   :72344      NA's   :162562   
```

Summarizing the data doesn't give me a whole lot more than I already knew but there is some interesting things to note here overall:
- The Gender with the most purchases is Male
- The most popular product is P00265242
- The Age group witht he most purchases is 26-35
- The City_Category with the most purchases is B

I'd like to be able to drill down into all of this further and see how these rankings relate with one another.

### Combining the Data Sets

To save on redundant work, I'll first combine the datasets

```
# add a Purchase column to test and then combine data sets
test[,Purchase := mean(train$Purchase)]
c <- list(train, test)
combin <- rbindlist(c)
```

Now with the data sets combined I can take another look at those columns that I noted before will need some reworking.

```
> combin[,prop.table(table(Gender))]
Gender
        F         M 
0.2470896 0.7529104 
> 
> #Age Variable
> combin[,prop.table(table(Age))]
Age
      0-17      18-25      26-35      36-45      46-50      51-55        55+ 
0.02722330 0.18113944 0.39942348 0.19998801 0.08329814 0.06990724 0.03902040 
> 
> combin[,prop.table(table(City_Category))]
City_Category
        A         B         C 
0.2682823 0.4207642 0.3109535 
> 
> combin[,prop.table(table(Stay_In_Current_City_Years))]
Stay_In_Current_City_Years
        0         1         2         3        4+ 
0.1348991 0.3527327 0.1855724 0.1728132 0.1539825 
> 
> length(unique(combin$Product_ID))
[1] 3677
> 
> length(unique(combin$User_ID))
[1] 5891
```

In addition though, now I know the total number of unique Product_IDs and User_IDs.  

Let's take a closer look at the missing values from the Product_Category columns.

```
> colSums(is.na(combin))
                   User_ID                 Product_ID                     Gender                        Age 
                         0                          0                          0                          0 
                Occupation              City_Category Stay_In_Current_City_Years             Marital_Status 
                         0                          0                          0                          0 
        Product_Category_1         Product_Category_2         Product_Category_3                   Purchase 
                         0                     245982                     545809                          0 
```

Thankfully the missing values are limited to just those two columns.  Honestly this makes sense, every product will have a primary category but only some will have secondary or tertiary categories.  The trick is how we're going to handle that for data analysis.  

```
combin[,Product_Category_2_NA := ifelse(sapply(combin$Product_Category_2, is.na) ==    TRUE,1,0)]
combin[,Product_Category_3_NA := ifelse(sapply(combin$Product_Category_3, is.na) ==  TRUE,1,0)]

combin[,Product_Category_2 := ifelse(is.na(Product_Category_2) == TRUE, "-999",  Product_Category_2)]
combin[,Product_Category_3 := ifelse(is.na(Product_Category_3) == TRUE, "-999",  Product_Category_3)]
```

First I added two new columns which act as flags for when the secondary and tertiary categories are missing.  Then I replace the NA values in the original columns with a -999 (it's an integer AND it stands out when looking through the data).

Now to recode the other columns.

```
levels(combin$Stay_In_Current_City_Years)[levels(combin$Stay_In_Current_City_Years) ==  "4+"] <- "4"

combin$Age <- as.numeric(combin$Age)


levels(combin$Age)[levels(combin$Age) == "0-17"] <- 0
levels(combin$Age)[levels(combin$Age) == "18-25"] <- 1
levels(combin$Age)[levels(combin$Age) == "26-35"] <- 2
levels(combin$Age)[levels(combin$Age) == "36-45"] <- 3
levels(combin$Age)[levels(combin$Age) == "46-50"] <- 4
levels(combin$Age)[levels(combin$Age) == "51-55"] <- 5
levels(combin$Age)[levels(combin$Age) == "55+"] <- 6

combin[, Gender := as.numeric(as.factor(Gender)) - 1]
```

Or simply put, I've replaced the value of '4+' with simply a '4' in the Stay_In_Current_City_Years column and converted the column into a numeric.  Then I assigned integer values for each of the Age ranges in the Age column, again converting it to numeric.

I think it will be useful to know more about each unique user and product so I'm going to add columns related to them.

```
#User Count
combin[, User_Count := .N, by = User_ID]

#Product Count
combin[, Product_Count := .N, by = Product_ID]

#Mean Purchase of Product
combin[, Mean_Purchase_Product := mean(Purchase), by = Product_ID]

#Mean Purchase of User
combin[, Mean_Purchase_User := mean(Purchase), by = User_ID]
```

So now for each row in the combined data set I have added the following:
 - User_Count = The total number of rows for this user
 - Product_Count = The total number of rows for this product
 - Mean_Purchase_User = The mean of Purchase amount for this user
 - Mean_Purchase_Product = The mean of Purchase amount for this product

Lastly I use the dummies package to break out the City_Category column into three columns (one for each of the categories).  This will make that data easier to work with in numeric (binary?) format.

```
combin <- dummy.data.frame(combin, names = c("City_Category"), sep = "_")
```

Now I think all of my data points are in place, time to double check and make sure they're in the correct formats.

```
> sapply(combin, class)
                   User_ID                 Product_ID                     Gender                        Age 
                 "integer"                   "factor"                  "numeric"                  "numeric" 
                Occupation            City_Category_A            City_Category_B            City_Category_C 
                 "integer"                  "integer"                  "integer"                  "integer" 
Stay_In_Current_City_Years             Marital_Status         Product_Category_1         Product_Category_2 
                  "factor"                  "integer"                  "integer"                "character" 
        Product_Category_3                   Purchase      Product_Category_2_NA      Product_Category_3_NA 
               "character"                  "numeric"                  "numeric"                  "numeric" 
                User_Count              Product_Count      Mean_Purchase_Product         Mean_Purchase_User 
                 "integer"                  "integer"                  "numeric"                  "numeric" 
```





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

