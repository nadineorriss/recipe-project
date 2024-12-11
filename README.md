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

The histogram below visualizes the distribution of average recipe ratings, showing a left-skewed trend where the majority of recipes have high ratings (e.g., 4 or 5). This suggests that most recipes are well-received by users, with only a small proportion receiving lower ratings, indicating overall positive feedback for the dataset's recipes.
### Bivariate Analysis
The bar chart below explores the relationship between cooking time categories and the average number of ratings per recipe. It reveals a decreasing trend, where shorter recipes (e.g., "0-15 mins") tend to receive more ratings on average compared to longer recipes (e.g., "1+ hours"). This suggests that users may engage more with quicker recipes, possibly due to their convenience or broader appeal.

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


The observed statistic of 51.4524 is indicated by the red vertical line on the graph. Since the p-value we calculated (0.1270) is greater than the significance level of 0.05, we fail to reject the null hypothesis. This suggests that the missingness of rating does not depend on the minutes (cooking time) column.

> Rating vs Number of Steps
**Null Hypothesis:** The missingness of the rating column does not depend on the n_steps column (number of steps).

**Alternate Hypothesis:** The missingness of the rating column depends on the n_steps column (number of steps).

**Test Statistic:** The test statistic is the absolute difference in mean number of steps (n_steps) between rows where rating is missing and rows where it is not missing.

**Significance Level:** 0.05
I conducted a permutation test by shuffling the missingness mask of the rating column 1000 times to simulate the null hypothesis, which assumes no relationship between the missingness of rating and the number of steps. For each permutation, I calculated the mean difference in the two distributions (missing vs. non-missing rating), generating 1000 simulated test statistics to compare against the observed value.


The observed statistic of 1.3386 is indicated by the red vertical line on the graph. Since the p-value we calculated (0.0000) is less than the significance level of 0.05, we reject the null hypothesis. This suggests that the missingness of rating does depend on the n_steps (number of steps) column.
