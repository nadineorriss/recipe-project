# Investigating the Relationship Between Cooking Time and Ratings
Author: Nadine Orriss
## Overview
This data science project, conducted at UCSD, explores the relationship between the average rating of recipes and their cooking times. 
## Introduction
In a world where time is precious, how do people choose what to cook? Are quick and easy recipes the key to satisfaction, or does effort still pay off in the kitchen?
According to the Bureau of Labor Statistics, the time spent on cooking and food preparation has decreased over the decades. In 1965, Americans spent an average of 112 minutes per day cooking, compared to 53 minutes per day in 2022. The average American works approximately 34.6 hours per week, spends an average of 27.6 minutes commuting one way to work, and over 60% of households have both partners working. People have busy schedules—driven by work commitments, commute times, and family responsibilities—which results in less time for cooking.
So, is there a strong preference for recipes that are quick and easy to prepare, and are they rated higher than their laborious, time-consuming counterparts? That is what I aim to answer in my analysis. The results of this could help Food Network and other recipe networks adapt to the shifting attitudes of Americans and cater to their evolving lifestyle demands. To do so, I am analyzing two datasets consisting of recipes and ratings posted since 2008 on Food.com. The original purpose of the datasets was for the recommender system research paper, Generating Personalized Recipes from Historical User Preferences by Majumder et al.

The first dataset, `recipe`, contains 83782 rows, indicating 83782 unique recipes, with 10 columns recording the following information:

| Column             | Description                                                                                                                                                                                       |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `'name'`           | Recipe name                                                                                                                                                                                       |
| `'id'`             | Recipe ID                                                                                                                                                                                         |
| `'minutes'`        | Minutes to prepare recipe                                                                                                                                                                         |
| `'contributor_id'` | User ID who submitted this recipe                                                                                                                                                                 |
| `'submitted'`      | Date recipe was submitted                                                                                                                                                                         |
| `'tags'`           | Food.com tags for recipe                                                                                                                                                                          |
| `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'`        | Number of steps in recipe                                                                                                                                                                         |
| `'steps'`          | Text for recipe steps, in order                                                                                                                                                                   |
| `'description'`    | User-provided description                                                                                                                                                                         |
| `'ingredients'`    | Text for recipe ingredients                                                                                                                                                                       |
| `'n_ingredients'`  | Number of ingredients in recipe                                                                                                                                                                   |

The second dataset, `interactions`, contains 731927 rows and each row contains a review from the user on a specific recipe. The columns it includes are:

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |

## Data Cleaning and Exploratory Data Analysis
To prepare the dataset for analysis, several key data cleaning steps were performed to ensure accuracy and usability. 
1. First, the recipes dataset (RAW_recipes.csv) was merged with the interactions dataset (RAW_interactions.csv) using a left join on the id and recipe_id columns. This ensured that each unique recipe was matched with its corresponding ratings and reviews, while retaining all recipes even if they lacked user interaction data.

2. Next, ratings of 0 were replaced with NaN. This decision was based on the understanding that a rating of 0 likely represents missing or invalid data rather than a legitimate user rating, as most rating systems use a scale starting at 1. Treating these values as missing ensures that they are excluded from calculations like averages, preventing them from artificially skewing statistics. This aligns with best practices for handling missing data in rating systems.

3. The dataset was then grouped by id, and the mean rating was calculated for each recipe to produce an average_rating column. This provides a summary statistic of recipe popularity and serves as a simplified metric for further analyses, such as understanding the relationship between recipe length and user satisfaction.

4. Columns such as user_id, date, recipe_id, contributor_id, and submitted were dropped as they were not directly relevant to the analysis goals. Removing these columns reduced redundancy and simplified the dataset, making it more manageable for subsequent steps.

5. Recipes were categorized into time bins (0-15 mins, 15-30 mins, 30-60 mins, and 1+ hours) based on their minutes column. This transformation created a time_category column, allowing for comparisons of average ratings across different preparation time ranges. This is particularly useful for analyzing whether shorter recipes are associated with higher ratings.

6. Columns such as tags, nutrition, steps, and ingredients contained data stored as list-like strings. These were converted into actual lists to enable further analysis, such as extracting specific tags or calculating nutritional statistics.

7. The nutrition column, which contained a list of nutritional values, was parsed into separate columns for individual metrics, such as calories, total fat, sugar, sodium, and others. This step was essential for the calorie prediction model, as it provided a clear and accessible format for the target variable (calories) and other nutritional predictors.



**By performing these steps, the dataset was transformed into a clean and structured format that supports the intended analyses. My cleand data frame ended up with 234429 rows and 21 columns, here is the head of my cleaned data frame with the most relevant columns to my analyses shown: (scroll right to view all columns)**

| name                                 |     id |   minutes |    n_steps|   rating |   average rating |  time_category |  calories |  total fat |  sugar |  carbohydrates |  protein |
|:-------------------------------------|-------:|----------:|:----------|---------:|-----------------:|---------------:|----------:|-----------:|-------:|---------------:|---------:|
| 1 brownies in the world    best ever | 333281 |        40 |        10 |        4 |                4 |     30-60 mins |     138.4 |       10.0 |   50.0 |            6.0 |      3.0 |
| 1 in canada chocolate chip cookies   | 453467 |        45 |        12 |        5 |                5 |     30-60 mins |     595.1 |       46.0 |  211.0 |           26.0 |     13.0 |
| 412 broccoli casserole               | 306168 |        40 |         6 |        5 |                5 |     30-60 mins |     194.8 |       20.0 |    6.0 |            3.0 |     22.0 |
| 412 broccoli casserole               | 306168 |        40 |         6 |        5 |                5 |     30-60 mins |     194.8 |       20.0 |    6.0 |            3.0 |     22.0 |
| 412 broccoli casserole               | 306168 |        40 |         6 |        5 |                5 |     30-60 mins |     194.8 |       20.0 |    6.0 |            3.0 |     22.0 |

### Univariate Analysis
The histogram below visualizes the distribution of recipe cooking times. The distribution is right-skewed, indicating that most recipes have shorter cooking times (e.g., under 60 minutes), while a smaller number require significantly longer preparation times, which could represent more elaborate or slow-cooked dishes.

<iframe
  src="fig_time.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


The histogram below visualizes the distribution of average recipe ratings, showing a left-skewed trend where the majority of recipes have high ratings (e.g., 4 or 5). This suggests that most recipes are well-received by users, with only a small proportion receiving lower ratings, indicating overall positive feedback for the dataset's recipes.

<iframe
  src="fig_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis
The bar chart below explores the relationship between cooking time categories and the average number of ratings per recipe. It reveals a decreasing trend, where shorter recipes (e.g., "0-15 mins") tend to receive more ratings on average compared to longer recipes (e.g., "1+ hours"). This suggests that users may engage more with quicker recipes, possibly due to their convenience or broader appeal.

<iframe
  src="fig.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Aggregate Analysis 
The table below aggregates data by cooking time categories to calculate the total number of recipes, total ratings, and average ratings per recipe within each category, formatting the results for clarity. This helps analyze recipe popularity, user engagement, and trends in cooking times, offering insights into which categories attract the most interaction and align with user preferences.

| Time Category     |     Total Recipes |   Total Ratings |    Avg. Ratings per Recipe |
|:------------------|------------------:|----------------:|:---------------------------|
|        0-15 mins  |            16,679 |          48,431 |                       2.90 |
|       15-30 mins  |            20,632 |          58,153 |                       2.82 |
|       30-60 mins  |            25,416 |          69,377 |                       2.73 |
|         1+ hours  |            21,054 |          55,688 |                       2.65 |

Shorter recipes recieve more ratings per recipe. Recipes in the 0-15 mins category have the highest average ratings per recipe (2.90). Enagagement gradually decreases as cooking time increases. Which we can interpret as users are more likely to engage with shorter recipes, which are quicker and easier to attempt. The average ratings per recipe decreases steadily with longer cooking times, reaching its lowest in the 1+ hours category. Longer recipes may deter users due to their time and effort requirements, leading to reduced engagement. Although longer categories like 30-60 mins and 1+ hours have the most recipes (25,416 and 21,054, respectively), their total ratings and average ratings per recipe are lower. Shorter categories attract more ratings per recipe despite having fewer recipes overall. Recipes in the 0-15 mins category, with 16,679 recipes, achieve the highest average engagement (2.90 ratings per recipe). Users are drawn to quick, simple recipes, which are perceived as more accessible and less intimidating. The data is telling us: to focus on developing and promoting shorter recipes (0-15 mins and 15-30 mins) to maximize user engagement.
## Assessment of Missingness
### NMAR Analysis
name, description, user_id, recipe_id, date, rating, and review all have missing values. Of these, I will focus on "review" because I believe "review" to be NMAR. The missingness in the review column is Not Missing At Random (NMAR) because it depends on users’ emotional responses to the recipe, which are unobserved in the dataset. Users with strong feelings—whether positive or negative—are more likely to leave a review, while those who feel indifferent are less inclined to invest the effort required to write one. Since this behavior is driven by the missing emotional response itself and cannot be explained by other observable variables, the missingness in review is NMAR.
### Missingness Dependency
Then, I proceeded to examine the missingness of the 'rating' column in the merged DataFrame by testing whether its missingness depends on the column 'n_steps' (the number of steps in the recipe) or the column 'minutes' (the cooking time of the recipe).
> Rating vs Cooking Time

**Null Hypothesis:** The missingness of the rating column does not depend on the minutes column (cooking time).

**Alternate Hypothesis:** The missingness of the rating column depends on the minutes column (cooking time).

**Test Statistic:** The test statistic is the absolute difference in mean minutes (cooking time) between rows where rating is missing and rows where it is not missing.

**Significance Level:** 0.05

I conducted a permutation test by shuffling the missingness mask of the rating column 1000 times to simulate the null hypothesis, which assumes no relationship between the missingness of rating and the cooking time. For each permutation, I calculated the mean difference in the two distributions (missing vs. non-missing rating), generating 1000 simulated test statistics to compare against the observed value.

<iframe
  src="fig1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed statistic of 51.4524 is indicated by the red vertical line on the graph. Since the p-value we calculated (0.1270) is greater than the significance level of 0.05, we fail to reject the null hypothesis. This suggests that the missingness of rating does not depend on the minutes (cooking time) column.

> Rating vs Number of Steps

**Null Hypothesis:** The missingness of the rating column does not depend on the n_steps column (number of steps).

**Alternate Hypothesis:** The missingness of the rating column depends on the n_steps column (number of steps).

**Test Statistic:** The test statistic is the absolute difference in mean number of steps (n_steps) between rows where rating is missing and rows where it is not missing.

**Significance Level:** 0.05

I conducted a permutation test by shuffling the missingness mask of the rating column 1000 times to simulate the null hypothesis, which assumes no relationship between the missingness of rating and the number of steps. For each permutation, I calculated the mean difference in the two distributions (missing vs. non-missing rating), generating 1000 simulated test statistics to compare against the observed value.

<iframe
  src="fig2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed statistic of 1.3386 is indicated by the red vertical line on the graph. Since the p-value we calculated (0.0000) is less than the significance level of 0.05, we reject the null hypothesis. This suggests that the missingness of rating does depend on the n_steps (number of steps) column.

## Hypothesis Testing
**Null Hypothesis:** The average rating for recipes that take 15 minutes or less is the same as the average rating for recipes that take more than 15 minutes.

**Alternate Hypothesis:** The average rating for recipes that take 15 minutes or less is different from the average rating for recipes that take more than 15 minutes.

**Test Statistic:** The absolute difference in mean ratings between the two groups (recipes that take 15 minutes or less vs. those that take more than 15 minutes).

**Significance Level:** 0.05

To evaluate the relationship between cooking time and average recipe ratings, I performed a permutation test:

1. Calculated the observed test statistic, which was the absolute difference in mean ratings between the two groups (obs_diff_15 = 0.0485).
2. Simulated the null hypothesis by randomly shuffling group labels (15 minutes or less vs. more than 15 minutes) 1000 times.
3. Recalculated the test statistic for each permutation and generated a null distribution of permuted mean differences.
4. Calculated the p-value as the proportion of permuted test statistics greater than or equal to the observed difference.

**Resulting p-value:** 0.0

**Conclusion:** Since the p-value is less than the significance level (0.0 < 0.05), we reject the null hypothesis. This indicates that there is a statistically significant difference in average ratings between recipes that take 15 minutes or less and those that take more than 15 minutes. The results support the conclusion that recipes with shorter cooking times are rated higher, aligning with the hypothesis that users prefer quick and easy recipes.

## Framing a Prediction Problem 

I am going to build a model using regression to predict the number of calories in a recipe. The response variable is calories, I chose it because i thought it would be the most straightforward and fun to implement. Because calories is a continuous response variable, prediction requires regression, not classification. I am using Root Mean Squared Error (RSME) as the metric for evaluation for a couple of reasons:
1. it measures the average magnitude of prediction errors in the same units as the response variable, calories.
2. it penalizes larger errors more heavily, which is appropriate for calorie prediction, where large deviations can be problematic.

I chose it over Mean Absolute Error (MAE) because large errors are highly undesireable, and MAE measures the average magnitude of errors, treating all errors equally regardless of their size.

The features I am going to use to predict calories are known before cooking the recipe, including preperation time "minutes", number of steps "n_steps", and the nutritional data that I already cleaned and split apart in the first part (fat, protein, carbs, sugar).

## Baseline Model

**This model is a baseline linear regression model that predicts the number of calories in a recipe based on the following features:**
- n_ingredients: Number of ingredients in the recipe (quantitative).
- minutes: Cooking time in minutes (quantitative).
- n_steps: Number of steps in the recipe (quantitative).

All three features (n_ingredients, minutes, n_steps) are numeric and continuous. Since all features are quantitative, no encoding was required. 

The features were standardized using StandardScaler to ensure that all features had a mean of 0 and a standard deviation of 1. This step was to prevent features with larger scales from dominating the model's coefficients, as well as address the outliers observed.

**Observed Metrics:** 
R-squared (Train): 0.0294
R-squared (Test): 0.0289
RMSE: 23.7689

As you can see with the above metrics, this is not what I consider to be a good model. The R-squared values are very low, indicating that the model explains less than 3% of the variability in the target variable (calories). The RMSE of 23.7689 is pretty high, suggesting that the model's predictions deviate substantially from the actual values. This tells me This indicates that the baseline features (n_ingredients, minutes, and n_steps) capture some variability in calories but are insufficient for accurate predictions.

## Final Model

For my final model, I used Lasso regression.
The features included were: n_ingredients, n_steps, total fat, protein, sugar, carbohydrates.
I excluded minutes due to its weak correlation with calories and potential for introducing noise. Cooking time does not reliably predict calorie content. total fat, protein, sugar, and carbohydrates were added because they are directly tied to calorie content through the data generating process (nutritional components contribute directly to calorie count). Including macronutrients aligns with known caloric contributions:
- Protein and carbohydrates contribute 4 calories per gram.
- Fat contributes 9 calories per gram.
- Sugar is a simple carbohydrate and a significant calorie contributor in many recipes.

I chose Lasso regression for its ability to perform feature selection by shrinking coefficients of less relevant features to zero, and reducing overfitting and improving interpretability.

For preprocessing, I standardized all numerical features (n_ingredients, n_steps, total fat, protein, sugar, carbohydrates) using StandardScaler.

I did Hyperparameter tuning using GridSearchCV with 5-fold-cross-validation. The best hyperparameters and their significance were:
- alpha = 0.1: Minimal regularization, allowing most features to contribute to the model.
- max_iter = 1000: Ensures convergence of the optimization algorithm.'
- tol = 0.001: Provides a good balance between precision and computational efficiency.

The final model’s R-squared values (0.9957) demonstrate that it captures nearly all variability in calorie content, while the RMSE of 5.9660 reflects highly precise predictions. Compared to the baseline model’s RMSE of 23.3495, the final model reduced the average prediction error by over 74%.

## Fairness Analysis

**Group X:** Recipes with a low number of ingredients (e.g., fewer than 5 ingredients).

**Group Y:** Recipes with a high number of ingredients (e.g., 5 or more ingredients).

**Evaluation Metric:** The R-squared value, which measures the proportion of variance explained by the model for predicting calorie counts.

**Null Hypothesis:** The model's performance does not depend on the number of ingredients in the recipe.

**Alternative Hypothesis:** The model's performance does depend on the number of ingredients in the recipe.

**Test Statistic:** The absolute difference in R-squared values between Group X and Group Y.

**Significance Level:** 0.05

Test Procedure:

1. The observed test statistic was calculated as the absolute difference in R-squared values between the two groups.

2. Under the null hypothesis, the labels indicating Group X and Group Y were shuffled randomly, breaking any true association between group membership and R-squared values.

3. This permutation process was repeated 1000 times to create a null distribution of the test statistic.

4. The p-value was calculated as the proportion of permuted test statistics greater than or equal to the observed test statistic.

**Resulting p-value:** 0.0

**Conclusion:** Since the p-value is 0.0, which is less than the significance level of 0.05, we reject the null hypothesis. This indicates that the model's performance, as measured by the R-squared difference, does depend on the number of ingredients in the recipe. The result suggests potential fairness concerns, as the model may perform differently for recipes with varying ingredient counts. Further investigation is needed to understand and address these disparities to ensure equitable performance across all recipe types.
