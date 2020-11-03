---
title       : The Plant Predictor
subtitle    : 
author      : Nikolas
job         : 
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained
knit        : slidify::knit2slides
---

<style>
article p {
  font-size: 17px;
}
article code {
  font-size: 13px;
}

</style>


## The app
### Link
https://arxaio.shinyapps.io/PlantPredictor/

### What does it do?
The Plant predictor is a Shiny app hosted on www.shinyapps.io and is based on the iris data set. The user inputs 4 parameters, namely sepal length, sepal width, petal length and petal width, and based on that the app predicts whether the plant is a setosa, a versicolor or a virginica. 

### Why is it useful?
If you ever know your plant is a setosa, a versicolor or a virginica, however, you are not sure which one this app can predict your plant species. In other words, not so useful, however through this I was able to get used to the differences in syntax when making a shiny app. 

--- 

## How was this done? 
I used the iris data set and split it into a training (60%) and a testing subset (40%). I then used the decision trees model to train and then tested the algorithm on the testing subset to make sure that the accuracy was reasonable. 


```r
library(caret)
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
set.seed(1234)
intrain <- createDataPartition(iris$Species, p = 0.6, list = FALSE)
    
training <- iris[intrain, ]
testing <- iris[-intrain, ]

mod_trees <- train(Species~., method = "rpart", data = training)
pred_trees <- predict(mod_trees, newdata = testing)

confusionMatrix(table(pred_trees, testing$Species))$overall
```

```
##       Accuracy          Kappa  AccuracyLower  AccuracyUpper   AccuracyNull 
##   9.333333e-01   9.000000e-01   8.380132e-01   9.815382e-01   3.333333e-01 
## AccuracyPValue  McnemarPValue 
##   1.906794e-22            NaN
```

--- 

## How was this done? (ui.R)

```r
library(shiny); 

shinyUI(fluidPage(

    titlePanel("Predicticting plant species based on sepal and petal length and width"),

    sidebarLayout(
        sidebarPanel(
            h2("If you are not sure whether your plant is a setosa, a versicolor or a virginica, fill in the values below and find out"),
            numericInput("sl", "What is your plant's sepal length (pick a value between 1 and 10)",
                         value = 2, min = 1.0, max = 10.0),
            numericInput("sw", "What is your plant's sepal width (pick a value between 1 and 10)",
                         value = 5, min = 0.1, max = 5.0),
            numericInput("pl", "what is your plant's petal length (pick a value between 1 and 10)",
                         value = 4, min = 1.0, max = 10.0),
            numericInput("pw", "what is your plant's petal width (pick a value between 0.1 and 5)",
                         value = 2.3, min = 0.1, max = 5.0),
            submitButton("Submit")
        ),
        mainPanel(
            h2("Your plant is a:"),
            textOutput("prediction")
        )
    )
))
```

---

## How was this done? (server.R)

```r
library(shiny); library(caret); library(e1071)

shinyServer(function(input, output) {
    set.seed(1234)
    intrain <- createDataPartition(iris$Species, p = 0.6, list = FALSE)
    
    training <- iris[intrain, ]
    testing <- iris[-intrain, ]
    
    mod_trees <- train(Species~., method = "rpart", data = training)
    
    modelpred <- reactive({
        df <- data.frame("Sepal.Length" = input$sl, "Sepal.Width" = input$sw, "Petal.Length" = input$pl, "Petal.Width" = input$pw)
        predict(mod_trees, newdata = df)
    })
    
    output$prediction <- reactive({
        modelpred()
    })
        
})
```
