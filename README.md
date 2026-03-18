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

- `has_meat`: whether the recipe contains meat (chicken, beef, pork, etc.)
- `has_veggies`: whether the recipe contains vegetables
- `high_sugar`: whether sugar PDV exceeds 50
- `high_sodium`: whether sodium PDV exceeds 50
- `is_dessert`: whether the recipe is tagged as a dessert (high sugar alone does not mean dessert)
- `is_vegetarian`: whether the recipe is vegetarian
- `is_healthy`: whether the recipe is tagged as healthy

Cuisine dummies were also created. This had the most thought applied to it, as I wanted to get a variety of cuisines, but it's impossible to go through all the tags and get literally every one. I tried to just get some of the most popular, and then make dummies based on region (african, american, asian, european, etc) as well as the top 5 specific popular cuisines (italian, mexican, chinese, indian, thai). I also made some dummies for specific regions of asia since there are a lot of asian recipes and I think there may be interesting patterns there.

A lot of the types of foods and cuisines I love the most are actually negatively correlated with the dataset. Then I remember what TTT actually measures. Even if some of these ratings are positively correlated with these dummy variables, they may not be correlated with TTT score because TTT score also takes into account the time it takes to make the recipe. For example, desserts may be highly rated but if they take a long time to make, they may have a low TTT score. A lot of my favorite cuisines and recipes involve a lot of steps and ingredients, which would also lower TTT score. 

For my personal subset and ratings, it makes a little more sense. I don't like vegetables that much, but do like east asian/asian foods in general. It's worth noting that this subset only consists of the top 100 recipes, which is why there are missing values for south asian and southeast asian, as those cuisines are not represented in the top 100. This is important, as we probably don't have many occurrences of these cuisines within the subset, and these correlations are to be taken with a heaping of salt because of that.

## Baseline Model

**Features:**
- Quantitative: `minutes`, `n_steps`, `n_ingredients`: standardized with StandardScaler so coefficients are comparable
- Nominal Binary: `has_meat`, `has_veggies`, `is_vegetarian`, `is_healthy`, `is_dessert`, `high_sugar`: already encoded as 1/0, passed through unchanged

A Linear Regression pipeline was used with StandardScaler on the numeric columns. Binary features are already 1/0 so they are passed through unchanged.

The model is not overfitting as train and test results are similar. The model is a lot less consistent for the personal subset, which makes sense as n=100 compared to the general dataset. The personal subset is also likely more noisy, as it is based on personal ratings which are more subjective and variable than the general dataset ratings. The model only explains 30% and 23% of the variance in TTT score for the general dataset and personal subset respectively, which means there is definitely room for improvement.

<iframe src="assets/baseline-gen.html" width="800" height="600" frameborder="0"></iframe>

<iframe src="assets/baseline-per.html" width="800" height="600" frameborder="0"></iframe>

## Final Model

While the regression baseline wasn't over/underfitting, the R^2 scores were pretty low, which means there was definitely room for improvement. The following improvements were made:

- Added `log_minutes` instead of raw `minutes`, as the relationship of minutes to TTT score seems to be non linear
- Added `calories` and `protein_pdv`, which were also correlated with TTT score, albeit less so than the features already in the baseline
- Added `is_dessert_and_sweet`, an interaction term combining `is_dessert` and `high_sugar`
- Switched to a RandomForestRegressor to capture any non linear relationships in the data that the linear regression can't

**General Dataset Parameters:**
n_estimators = 100, standard for random forest. max_depth not specified as the large dataset means overfitting is less of a concern. min_samples_leaf not specified for the same reason. random_state = 42 to ensure reproducibility.

**Personal Subset Parameters:**
Hyperparameter tuning for the personal subset is more important than for the general dataset, as the personal subset is smaller and more prone to overfitting. A grid search over max_depth and min_samples_leaf was run to find the best balance of bias and variance for this smaller dataset.

<iframe src="assets/final-pred-gen.html" width="800" height="600" frameborder="0"></iframe>

<iframe src="assets/final-resid-gen.html" width="800" height="600" frameborder="0"></iframe>

<iframe src="assets/final-pred-per.html" width="800" height="600" frameborder="0"></iframe>

<iframe src="assets/final-resid-per.html" width="800" height="600" frameborder="0"></iframe>

I'm much happier with the results of the general dataset, as the R^2 score is much higher than the regression baseline, indicating that the model is explaining a lot more of the variance in TTT score. The test RMSE is also lower than the regression baseline, which means the model is making more accurate predictions. The random forest model is likely capturing some non linear relationships in the data that the linear regression couldn't, which is why we see such an improvement in performance.

On the other hand, the personal subset model didn't perform much better than the baseline. The R^2 score actually got worse, and the RMSE is also worse on the test set. This could be due to the small dataset size (n=100), which makes it hard for the random forest to learn complex relationships without overfitting. Additionally, as I mentioned earlier, my personal subset has a lot more variance than the general dataset. That variance combined with the small dataset size may be why the model isn't performing well, as there may not be enough data for the model to learn from without overfitting. It trains on 80 samples and tests on 20 samples, which is a very small test set, so the results may not be very reliable. With such a small dataset, it may be better to stick with a simpler model like linear regression, or to gather more data for my personal ratings to improve the model's performance.

But since the main focus of this project is the general dataset, and my personal ratings were more of a fun side project, I'm happy with the results overall. The random forest model for the general dataset is a significant improvement over the regression and provides a much better fit to the data, while the personal subset model is a good learning experience and shows the challenges of working with small datasets.



## Fairness Analysis

Since this whole model and project is centered around time to cook, and the most correlated variable in the model is log_minutes, I ran a fairness test to see whether the model performs equally well on quick recipes or slow recipes. Quick recipes are defined as those that take less than 30 minutes to cook, and slow recipes as those that take 30 minutes or more.

**Group X:** Quick recipes (minutes < 30)

**Group Y:** Slow recipes (minutes >= 30)

**Evaluation Metric:** RMSE

**Null Hypothesis:** The model performs equally well on quick and slow recipes, there is no significant difference in RMSE between the two groups.

**Alternative Hypothesis:** The model performs better on quick recipes than on slow recipes, there is a significant difference in RMSE between the two groups.

**Significance Level:** 0.05

Since the TTT score is derived from minutes, and is negatively correlated with minutes, I would expect the model to perform better on quick recipes, as they tend to have higher TTT scores and may be easier for the model to predict accurately. Slow recipes, on the other hand, may have lower TTT scores and more variance, which could make them harder for the model to predict accurately.

<iframe src="assets/fairness-dist.html" width="800" height="600" frameborder="0"></iframe>

Defying my expectations, the model actually performs better on slow recipes than on quick recipes, as evidenced by the lower RMSE for slow recipes. Based on these results, I fail to reject the null hypothesis, as the p value is greater than the significance level of 0.05. This means that there is not enough evidence to conclude that the model performs better on quick recipes than on slow recipes.

In light of the unexpected result, I pivoted the alternative hypothesis to reflect the new direction of the observed difference.

**Updated Alternative Hypothesis:** The model performs better on slow recipes than on quick recipes, there is a significant difference in RMSE between the two groups.

The permutation test with the corrected direction shows that the observed difference in RMSE between quick and slow recipes is statistically significant, with a p value of 0. This means that we can reject the null hypothesis and conclude that there is a significant difference in model performance between quick and slow recipes, but with the model performing better on slow recipes, rather than quick recipes as I originally expected.

Some reasons why this may be the case is that slow recipes are far more represented in the dataset, nearly twice as much. Also quick recipes have a higher variance, which means the model may have a harder time predicting them accurately. Quick recipes have higher mean TTT scores, since the time penalty is lower, and also quick recipes are usually a lot more diverse. I saw quick recipes which were drinks, snacks, meals, desserts, etc. Slow recipes on the other hand were mostly meals, which may be easier for the model to predict accurately, as they may have more consistent features and patterns that the model can learn from.

Overall, this fairness test shows that the model does not perform equally well on quick and slow recipes, and actually performs better on slow recipes, which is an interesting and somewhat surprising finding. This is very strong evidence of unfairness, especially because a p value of 0 means not a single shuffle had a difference as large as the observed difference.

Unfortunately, while the model performs better on slow recipes, it still performs decently on quick recipes, with an RMSE of 0.35, which means that the model is still making reasonably accurate predictions for quick recipes. This means that this model is still useful for the intended purpose of this project, which was to make a model which can predict TTT score, allowing people to find recipes which are tasty and quick to make.

