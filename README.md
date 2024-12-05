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

**Given the datasets, I am investigating whether people rate quick and easy recipes higher than laborious, time-consuming counterparts.** To facilitate the investigation of my question, I categorized recipes into distinct time ranges based on their total preparation and cooking time: "0-15 mins," "15-30 mins," "30-60 mins," "1-2 hours," and "2+ hours." This information 'time_category' column was created by binning the original 'total_time' column, which represents the time it takes to prepare and cook each recipe.
Additionally, I calculated the average rating for each time category to compare the popularity of recipes within these groups. The most relevant columns to answer my question are 'time_category,' described above, 'rating,' which is the rating a user gave a recipe, and 'average rating,' which is the average of all ratings for each unique recipe.
By seeking an answer to my question, I aim to provide valuable insights into people’s preferences for recipes based on their time commitments. These insights can help contributors on Food.com and similar platforms adapt their recipes to align better with public interests, particularly in catering to busy lifestyles. Moreover, this analysis can serve as a foundation for future work exploring how time constraints influence dietary choices and culinary preferences.

