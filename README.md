---
title: "Data Mining with R: Predict Diabetes"
output: html_document
---



```{r global_options, include=FALSE}
knitr::opts_chunk$set(echo=FALSE, warning=FALSE, message=FALSE)
```

```{r}
library(ggplot2)
library(dplyr)
library(gridExtra)
library(corrplot)
```

Look at the structure and the first few rows.

```{r}
diabetes <- read.csv('diabetes.csv')
dim(diabetes)
str(diabetes)
head(diabetes)
```

### Exploratory Data Analysis and Feature Selection

Check missing values 

```{r}
cat("Number of missing value:", sum(is.na(diabetes)), "\n")
```

Staitstical summary 

```{r}
summary(diabetes)
```

```{r}
diabetes$Outcome <- factor(diabetes$Outcome)
```

Histogram of numeric variables

```{r}
p1 <- ggplot(diabetes, aes(x=Pregnancies)) + ggtitle("Number of times pregnant") +
  geom_histogram(aes(y = 100*(..count..)/sum(..count..)), binwidth = 1, colour="black", fill="white") + ylab("Percentage")
p2 <- ggplot(diabetes, aes(x=Glucose)) + ggtitle("Glucose") +
  geom_histogram(aes(y = 100*(..count..)/sum(..count..)), binwidth = 5, colour="black", fill="white") + ylab("Percentage")
p3 <- ggplot(diabetes, aes(x=BloodPressure)) + ggtitle("Blood Pressure") +
  geom_histogram(aes(y = 100*(..count..)/sum(..count..)), binwidth = 2, colour="black", fill="white") + ylab("Percentage")
p4 <- ggplot(diabetes, aes(x=SkinThickness)) + ggtitle("Skin Thickness") +
  geom_histogram(aes(y = 100*(..count..)/sum(..count..)), binwidth = 2, colour="black", fill="white") + ylab("Percentage")
p5 <- ggplot(diabetes, aes(x=Insulin)) + ggtitle("Insulin") +
  geom_histogram(aes(y = 100*(..count..)/sum(..count..)), binwidth = 20, colour="black", fill="white") + ylab("Percentage")
p6 <- ggplot(diabetes, aes(x=BMI)) + ggtitle("Body Mass Index") +
  geom_histogram(aes(y = 100*(..count..)/sum(..count..)), binwidth = 1, colour="black", fill="white") + ylab("Percentage")
p7 <- ggplot(diabetes, aes(x=DiabetesPedigreeFunction)) + ggtitle("Diabetes Pedigree Function") +
  geom_histogram(aes(y = 100*(..count..)/sum(..count..)), colour="black", fill="white") + ylab("Percentage")
p8 <- ggplot(diabetes, aes(x=Age)) + ggtitle("Age") +
  geom_histogram(aes(y = 100*(..count..)/sum(..count..)), binwidth=1, colour="black", fill="white") + ylab("Percentage")
grid.arrange(p1, p2, p3, p4, p5, p6, p7, p8, ncol=2)
```

All the variables have reasonable broad distribution, therefore, will be kept for the regression analysis. 

Correlation Between Numeric Varibales

```{r}
numeric.var <- sapply(diabetes, is.numeric)
corr.matrix <- cor(diabetes[,numeric.var])
corrplot(corr.matrix, main="\n\nCorrelation Plot for Numerical Variables", order = "hclust", tl.col = "black", tl.srt=45, tl.cex=0.5, cl.cex=0.5)
```

The numeric variabls are almost not correlated. 

Correlation bewteen numeric variables and outcome. 

```{r}
attach(diabetes)
par(mfrow=c(2,4))
boxplot(Pregnancies~Outcome, main="No. of Pregnancies vs. Diabetes", 
        xlab="Outcome", ylab="Pregnancies")
boxplot(Glucose~Outcome, main="Glucose vs. Diabetes", 
        xlab="Outcome", ylab="Glucose")
boxplot(BloodPressure~Outcome, main="Blood Pressure vs. Diabetes", 
        xlab="Outcome", ylab="Blood Pressure")
boxplot(SkinThickness~Outcome, main="Skin Thickness vs. Diabetes", 
        xlab="Outcome", ylab="Skin Thickness")
boxplot(Insulin~Outcome, main="Insulin vs. Diabetes", 
        xlab="Outcome", ylab="Insulin")
boxplot(BMI~Outcome, main="BMI vs. Diabetes", 
        xlab="Outcome", ylab="BMI")
boxplot(DiabetesPedigreeFunction~Outcome, main="Diabetes Pedigree Function vs. Diabetes", xlab="Outcome", ylab="DiabetesPedigreeFunction")
boxplot(Age~Outcome, main="Age vs. Diabetes", 
        xlab="Outcome", ylab="Age")

```

Blood pressure and skin thickness show little variation with diabetes, they will be excluded from the model. Other variables show more or less correlation with diabetes, so will be kept.

### Logistic Regression 

```{r}
diabetes$BloodPressure <- NULL
diabetes$SkinThickness <- NULL
train <- diabetes[1:540,]
test <- diabetes[541:768,]

model <-glm(Outcome ~.,family=binomial(link='logit'),data=train)
summary(model)
```

The top three most relevant features are "Glucose", "BMI" and "Number of times pregnant" because of the low p-values.

"Insulin" and "Age" appear not statistically significant.

```{r}
anova(model, test="Chisq")
```

From the table of deviance, we can see that adding insulin and age have little effect on the residual deviance.

### Cross Validation

```{r}
fitted.results <- predict(model,newdata=test,type='response')
fitted.results <- ifelse(fitted.results > 0.5,1,0)
misClasificError <- mean(fitted.results != test$Outcome)
print(paste('Accuracy',1-misClasificError))
```

### Decision Tree

```{r}
library(rpart)
model2 <- rpart(Outcome ~ Pregnancies + Glucose + BMI + DiabetesPedigreeFunction, data=train,method="class")
```

```{r}
plot(model2, uniform=TRUE, 
  	main="Classification Tree for Diabetes")
text(model2, use.n=TRUE, all=TRUE, cex=.8)
```

This means if a person's BMI less than 45.4 and her diabetes digree function less than 0.8745, then she is more likely to have diabetes. 

Confusion table and accuracy

```{r}
treePred <- predict(model2, test, type = 'class')
table(treePred, test$Outcome)
mean(treePred==test$Outcome)
```

