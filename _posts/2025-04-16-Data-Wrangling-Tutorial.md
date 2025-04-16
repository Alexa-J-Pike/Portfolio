---
title: "What Game Should Nintendo Make Next? A Data Wrangling Tutorial"
date: 2025-04-16
output: html_document
---

### Alexa Pike

```{r include=FALSE}
library(tidyverse)
library(knitr)
```

```{r include=FALSE}
video_game_sales <- read_csv("video games sales.csv")
head(video_game_sales)
```

# Introduction

Data wrangling is one of the foundations of data analysis. Without it, it is essentially impossible for R to read and visualize data. Thankfully, there is a collection of R packages for this exact reason, introducing: *Tidyverse*. The Tidyverse R package contains several different packages, each of which help clean, transform, or visualize your data. One of these packages is called DPLYR, and will be the focus of this tutorial. DPLYR contains a set of functions that help transform data frames to make them more readable, and relevant to a research question. Of the many DPLYR functions, we will discuss six:

- filter()
- group_by()
- summarize()
- mutate()
- arrange()
- Pipe operators

To demonstrate the capabilities of these DPLYR functions, we will be transforming the [Video Games Sale](https://www.kaggle.com/datasets/zahidmughal2343/video-games-sale) data set created by Kaggle.com user Zahid Feroze. Feel free to download it and follow along. This data set contains information on video game sales since 1980 up to 2020 with eleven variables, three of which will be relevant in this tutorial:

- Publisher
- Genre
- Global Sales ($M)

Before we start transforming our data, we must come up with a research question. When creating a research question, you should consider what it is you'd like to learn about the data and how the results may be useful. Our research question will be:

**What is Nintendo's most profitable genre?**

This question may seem simple, but there are a lot of steps we need to take to in order to find the answer. So, let's get started!

\

# Tidyverse

Before we do anything, let's make sure the Tidyverse package is installed. This can be done directly in R Studio:

`install.packages("tidyverse")`

If you've already installed it, don't forget to load it in:

`library(tidyverse)`

\

# DPLYR

\

### filter()

In our data set, the Publisher variable contains the name of the company who published each game. There are many publishers contained in our data set, but we are only interested in Nintendo. In order to isolate the rows that only contain "Nintendo" as the publisher, we must use the filter() function. 

filter() extracts a subset of rows that meet one or more specific conditions. 

Format: filter(data, condition1, condition2, ...)

Conditions can be described by relational operators, like greater or less than, equal or not equal to.

Here, we filter the Publisher variable by "Nintendo"
```{r}
Filtered_Data <- filter(video_game_sales, Publisher == "Nintendo")
Filtered_Data
```
Now our data only contains games published by Nintendo.

\

### group_by()

Recalling our research question, we want to to group the games by genre so that in the next step we can calculate statistics based on genre. To do this, we'll use the group_by function on our **filtered** data.

group_by() splits a table into groups based on one or more columns.

Format: group_by(data, col1, col2, ...)

Here, we group the games by Genre.
```{r}
Grouped_Data <- group_by(Filtered_Data, Genre)
Grouped_Data
```
The data frame doesn't get reorganized, but the data has been grouped.

\

### summarize()

If we want to find the most profitable genre, we'll need to calculate the average sales per game for each genre. In order to do this, we must first calculate the total global sales and the total number of games in each genre. We can find both of these statistics by using the summarize() function on our **grouped** data.

summarize() summarizes each *group* using aggregate functions

Format: summarize(data, new_variable1 = function, ...)

Some examples of aggregate functions are mean(), sum(), and n().

Here, we'll calculate Nintendo's total global sales and number of games per genre.
```{r}
Sum_Data <- summarize(Grouped_Data,
                      Total_Global_Sales = sum(Global_Sales),
                      Number_Of_Games = n())
Sum_Data
```
We now have two new variables, Total_Global_Sales and Number_Of_Games.

\

### mutate()

Using the two new variables we made in the previous step, we can now calculate the average game sales in each genre. We'll need to divide the Total_Global_Sales by Number_Of_Games in each row, which can be accomplished using the mutate() function on our **summarized** data.

mutate() creates new columns based on calculations from existing columns

Format: (data, new_variable1 = expression1, ...)

Here, we'll create a new average sales per game column.
```{r}
Mutated_Data <- mutate(Sum_Data, 
                       Avg_Sales_Per_Game = Total_Global_Sales / Number_Of_Games)
Mutated_Data
```
We now have the Avg_Sales_Per_Game column, which shows the average profitability of a game in any given genre.

\

### arrange()

Even though we calculated our answer in the previous step, it's hard to find because our data is not well organized. It would be a lot easier to read if the data was listed by the highest average sales per game to the lowest. Thankfully, we can do just that by using the arrange() function on our **mutated** data.

arrange() changes the order of the rows based on the value of the columns

Format: arrange(data, column1, column2, ...)

By default, arrange() orders the columns from least to greatest. We can change this by adding in the desc() argument.

Here, we'll arrange the rows from highest to lowest based on average sales per game.
```{r}
Arrange_Data <- arrange(Mutated_Data, (desc(Avg_Sales_Per_Game)))
Arrange_Data
```
We can now clearly see that Racing games on average generate the most sales.

\

### Pipe operators (|>)

Now that we've answered our research question, we want visualize our data with a graph. However, if we used our arranged data frame to generate a graph, others will not be able to see all of the previous functions we used. This is when Pipe operators (|>) become incredibly useful. Pipe operators are used to chain commands together.

Here, I've used pipe operators to chain all of our functions together.
```{r}
Nintendo_Genre_Graph <-
  video_game_sales |>
  filter(Publisher == "Nintendo") |>
  group_by(Genre) |>
  summarize(Total_Global_Sales = sum(Global_Sales),
            Number_Of_Games = n()) |>
  mutate(Avg_Sales_Per_Game = Total_Global_Sales / Number_Of_Games) |>
  arrange(desc(Avg_Sales_Per_Game))
```

Now anyone will be able to understand how we generated our data, and we can use this to create a graph.

```{r echo=FALSE}
Nintendo_Genre_Graph |>
  ggplot(
    aes(x = reorder(Genre, -Avg_Sales_Per_Game), y = Avg_Sales_Per_Game)) +
  geom_bar(stat = "identity") +
  theme_classic() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1),
        axis.title.y = element_text(angle = 0, vjust = 0.5)) +
  labs(
    x = "Genre",
    y = "Average\nSales Per\nGame ($M)",
    title = "Nintendo's Average Game Sales By Genre",
    subtitle = "Games released between 1980 - 2020"
  )
```

# Conclusion

Based on this graph we can conclude that Nintendo's most profitable game genre is racing, since, on average, their racing games generate the most sales per game. This data could be extremely useful for Nintendo when deciding what kind of games they should be working on. For example, it would be a lot more beneficial for Nintendo to be putting money into making more racing, sports, and platform games since they have the highest returns. Additionally, this data might suggest that Nintendo should avoid making adventure and strategy games, since they have the lowest returns. As consumers, we can potentially use this data to determine what genres of games we should expect Nintendo to release next.

Data wrangling is one, if not the, most crucial steps in the data science process, and it is made easy with packages like DPLYR that are contained within the larger Tidyverse package. I hope that this short tutorial has helped you better understand the data wrangling process. Good luck in your data science travels!
