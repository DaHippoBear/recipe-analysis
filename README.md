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

### Univariate Analysis

<iframe src="assets/cooking-time-dist.html" width="800" height="600" frameborder="0"></iframe>

The distribution of cooking times is heavily right skewed, with most recipes taking under 60 minutes. This confirms that the dataset is dominated by quick everyday recipes rather than elaborate long cooking dishes.

### Bivariate Analysis

<iframe src="assets/steps-vs-time.html" width="800" height="600" frameborder="0"></iframe>

There is a weak positive relationship between number of steps and cooking time, but there is a high spread. Many recipes with few steps still take a long time


### Interesting Aggregates

| step_group   |   ttt_score |   avg_rating |   minutes |
|:-------------|------------:|-------------:|----------:|
| 1-5          |        3.24 |         4.64 |     45.21 |
| 6-10         |        2.55 |         4.62 |     58.73 |
| 11-15        |        2.4  |         4.62 |     66.98 |
| 16-20        |        2.33 |         4.64 |     80.13 |
| 20+          |        2.25 |         4.65 |    103.4  |

## Assessment of Missingness

### MNAR Analysis
The `description` column is likely **MNAR** (Missing Not At Random). Reviewers may choose not to write a description just because the recipe is low effort or low quality, meaning the missingness is related to the quality of the recipe itself, which is unobserved in the data. Additional data that could explain this and make the missingness MAR would be the user's engagement level on Food.com, such as how active they are on the platform. A more engaged user would be more likely to write a description regardless of recipe quality.

### Missingness Dependency
The `description` column was analyzed to determine whether its missingness depends on other columns. Three permutation tests were run:

**Test 1: Does description missingness depend on `n_ingredients`?**
p-value: 0.0, observed difference: -1.40 ingredients. Since p < 0.05 and the gap is meaningful, description missingness depends on `n_ingredients`. Recipes with more ingredients may be more complex and worth describing, making a description more likely to be written. This is **MAR** with respect to `n_ingredients`.

**Test 2: Does description missingness depend on `minutes`?**
p-value: 0.344, observed difference: -10.96 minutes. Since p > 0.05, description missingness does not depend on minutes. This suggests missingness is **MCAR** with respect to minutes.

**Test 3: Does description missingness depend on `n_steps`?**
p-value: 0.186, observed difference: ~0.98 minutes. Since p > 0.05, description missingness does not depend on minutes. This suggests missingness is **MCAR** with respect to minutes.

<iframe src="assets/missingness-dist.html" width="800" height="600" frameborder="0"></iframe>

Recipes where the description is missing tend to have fewer ingredients on average, supporting the MAR conclusion that description missingness depends on n_ingredients.

## Hypothesis Testing

I made code which will cycle through the top 100 highest rated recipes, and ask the user (me) to input personal ratings for each recipe. The ratings are saved to a CSV file for later analysis. This is like a side project within the overall project, where I can reflect on how my personal tastes align with the community ratings and the TTT score. Since I made this project question for my own personal use, and I feel that my personal taste differs from the average, I thought it would be interesting to see how my ratings compare to the community ratings and whether the recipe features which are good for the general dataset are also good features to use as predictors for my personal subset. This also makes the project more fun for me :)

**Null Hypothesis:** Recipes containing meat with fewer than 9 steps have the same average TTT score as all other recipes

**Alternative Hypothesis:** Recipes containing meat with fewer than 9 steps have a higher average TTT score than other recipes

**Test Statistic:** Difference in mean TTT scores between low step meat recipes and all other recipes. I want to see if low step meat recipes have a higher TTT score on average, since I think they are a good balance of 
tastiness and time investment.

**Significance Level:** 0.05 (standard threshold for statistical significance)

**General Dataset Result:** It looks like for the general dataset, the observed difference is about -0.14, which means it is actually the case that low step meat recipes have a slightly lower average TTT score than other recipes. The p value is 1, so I fail to reject the null hypothesis. This suggests that there is no strong evidence that low step meat recipes have higher TTT scores than other recipes in the general dataset.

**Personal Subset Result:** For my personal ratings of the top 100 recipes, the observed difference is about 0.006, which means that low step meat recipes have a very slightly higher average TTT score than other recipes in my personal ratings. However, the p value is 0.467, so we fail to reject the null hypothesis. This suggests that there is no strong evidence that low step meat recipes have higher TTT scores than other recipes in my personal ratings either, although the observed difference is in the direction of the alternative hypothesis.

<iframe src="assets/gen-ts-dist.html" width="800" height="600" frameborder="0"></iframe>

<iframe src="assets/personal-ts-dist.html" width="800" height="600" frameborder="0"></iframe>

## Framing a Prediction Problem

Predicting the TTT score of a recipe based on its features (cooking time, has meat, number of steps, etc).

**Type:** Regression, since TTT score is a continuous numeric variable.

**Response Variable:** TTT score and personal TTT score. TTT score is the focus of this project, as the goal is to predict whether a recipe is quick and tasty based on its features. Personal TTT score uses the same approach but with personal ratings on the top 100 recipes.

**Evaluation Metrics:** RMSE and R². RMSE was chosen as it penalizes large errors, which is important for a recommendation system where we want to avoid recommending recipes that are very bad. R² gives a sense of how much variance in TTT score is explained by the model.

**Time of Prediction:** All features used in the model are properties of the recipe itself and custom columns based on those properties. These are all known when the recipe is submitted. Average rating is not used as a feature since it is not known at submission time, and it would also be too correlated a predictor since TTT score is based on it.

To identify the best features, I explored correlations between numeric and categorical features and TTT score for both the general dataset and personal subset.

<iframe src="assets/corr-general.html" width="800" height="600" frameborder="0"></iframe>


Starting with the general dataset, it looks like all the numeric features are negatively correlated with TTT_Score. The 3 most correlated columns are minutes, n_ingredients, and n_steps, which are all measures of recipe complexity. The minutes being correlated makes the most sense, as the TTT score is directly derived from minutes, so I would expect a strong negative correlation. The n_ingredients and n_steps correlations also make sense, as more complex recipes with more steps and ingredients may be less appealing to users, leading to lower ratings and thus lower TTT scores. After the top most correlated features, the correlations drop off significantly, with the next most correlated features being calories and sodium_pdv. This indicates that the most relevant features for predicting TTT scores are features which are related to recipe complexity, which makes intuitive sense. 

<iframe src="assets/corr-personal.html" width="800" height="600" frameborder="0"></iframe>

For the personal ratings subset, the correlations are generally weaker, but minutes, n_steps, and n_ingredients are still among the most negatively correlated features with personal TTT score. Interestingly, sodium is slightly positively correlated with personal TTT score, which is different from the general dataset where it is negatively correlated.  To me this makes intuitive sense, as I like recipes with more sodium as generally that means more flavor (I love instant ramen and things like that)

While I do have some features to work with here, the correlations are not super strong. I'm also interested in columns which aree not numeric, particularly the has_meat column I made earlier. I will do some OneHotEncodes and see if any categorical features correlate with TTT score as well, and then use all the features I found to be correlated to make a regression model to predict TTT score.

Since numeric features alone were not strongly correlated with TTT score, I made categorical dummy variables to capture additional patterns. The following dummy variables were created:

- `has_meat` — whether the recipe contains meat (chicken, beef, pork, etc.)
- `has_veggies` — whether the recipe contains vegetables
- `high_sugar` — whether sugar PDV exceeds 50
- `high_sodium` — whether sodium PDV exceeds 50
- `is_dessert` — whether the recipe is tagged as a dessert (high sugar alone does not mean dessert)
- `is_vegetarian` — whether the recipe is vegetarian
- `is_healthy` — whether the recipe is tagged as healthy

Cuisine dummies were also created. This had the most thought applied to it, as I wanted to get a variety of cuisines, but it's impossible to go through all the tags and get literally every one. I tried to just get some of the most popular, and then make dummies based on region (african, american, asian, european, etc) as well as the top 5 specific popular cuisines (italian, mexican, chinese, indian, thai). I also made some dummies for specific regions of asia since there are a lot of asian recipes and I think there may be interesting patterns there.

A lot of the types of foods and cuisines I love the most are actually negatively correlated with the dataset. Then I remember what TTT actually measures. Even if some of these ratings are positively correlated with these dummy variables, they may not be correlated with TTT score because TTT score also takes into account the time it takes to make the recipe. For example, desserts may be highly rated but if they take a long time to make, they may have a low TTT score. A lot of my favorite cuisines and recipes involve a lot of steps and ingredients, which would also lower TTT score. 

For my personal subset and ratings, it makes a little more sense. I don't like vegetables that much, but do like east asian/asian foods in general. It's worth noting that this subset only consists of the top 100 recipes, which is why there are missing values for south asian and southeast asian, as those cuisines are not represented in the top 100. This is important, as we probably don't have many occurrences of these cuisines within the subset, and these correlations are to be taken with a heaping of salt because of that.

## Baseline Model
...

## Final Model
...

## Fairness Analysis
...
