# recipe-analysis
DSC 80 project, analyzing recipes and the relation between ingredients, rating, and time to prepare
# What Makes a Recipe Worth the Effort?
**Name:** Dhruv Sengupta
**Course:** DSC 80, UCSD

## Introduction
This project explores a dataset of recipes and user ratings from Food.com to answer a simple question: what makes a recipe worth the time it takes to cook? As someone who enjoys cooking but often has limited time, the goal is to understand the relationship between recipe complexity, cooking time, and quality. By doing this, I hope to identify what kinds of recipes offer the best return on time invested.

The dataset contains recipes submitted to Food.com along with user ratings and reviews. The key columns relevant to this analysis are:

| Column | Description |
|---|---|
| `minutes` | Time in minutes to prepare the recipe |
| `n_steps` | Number of steps in the recipe |
| `n_ingredients` | Number of ingredients required |
| `avg_rating` | Average community rating out of 5 |
| `ttt_score` | Tastiness-to-time score, a custom metric balancing rating and cook time |
| `ingredients` | List of ingredients used in the recipe |


## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
The raw dataset was cleaned in the following steps:

1. The recipes and interactions datasets were merged on recipe ID using a left join to keep all recipes
2. Ratings of 0 were replaced with `NaN` since a rating of 0 on Food.com means there was no rating, not an actual 0/5
3. Average rating per recipe was computed and added as a new column. The TTT score will be based on this
4. Recipes with <= 0 minutes or > 1440 minutes (24 hours) were removed as these are likely errors
5. A custom **Tastiness-to-Time (TTT) score** was created by dividing average rating by the log of cooking time, then scaling to a 1-10 range. The log transformation was chosen to reflect diminishing returns of longer cooking times. Essentially, the difference between a 5 and 10 minute recipe matters more than the difference between a 60 and 65 minute recipe

## Assessment of Missingness
...

## Hypothesis Testing
...

## Framing a Prediction Problem
...

## Baseline Model
...

## Final Model
...

## Fairness Analysis
...
