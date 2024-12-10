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



**By performing these steps, the dataset was transformed into a clean and structured format that supports the intended analyses. My cleand data frame ended up with 234429 rows and 21 columns, here is the head of my cleaned data frame with the most relevant columns to my analyses shown: (scroll right to view all columns) **

| name                                 |     id |   minutes |    n_steps|   rating |   average rating |  time_category |  calories |  total fat |  sugar |  carbohydrates |  protein |
|:-------------------------------------|-------:|----------:|:----------|---------:|-----------------:|---------------:|----------:|-----------:|-------:|---------------:|---------:|
| 1 brownies in the world    best ever | 333281 |        40 |        10 |        4 |                4 |     30-60 mins |     138.4 |       10.0 |   50.0 |            6.0 |      3.0 |
| 1 in canada chocolate chip cookies   | 453467 |        45 |        12 |        5 |                5 |     30-60 mins |     595.1 |       46.0 |  211.0 |           26.0 |     13.0 |
| 412 broccoli casserole               | 306168 |        40 |         6 |        5 |                5 |     30-60 mins |     194.8 |       20.0 |    6.0 |            3.0 |     22.0 |
| 412 broccoli casserole               | 306168 |        40 |         6 |        5 |                5 |     30-60 mins |     194.8 |       20.0 |    6.0 |            3.0 |     22.0 |
| 412 broccoli casserole               | 306168 |        40 |         6 |        5 |                5 |     30-60 mins |     194.8 |       20.0 |    6.0 |            3.0 |     22.0 |



