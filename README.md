# foodcom-recipe-analysis
A data science project analyzing Food.com recipes and predicting ratings using machine learning. Created for DSC 80 at UCSD.
hellohihihi

# My Project Title

by Suraj Rampure (rampure@ucsd.edu)

***Note***: If you choose a repo name and title as uninspired as the ones here, I will be quite sad.

---

## Introduction

Food plays an essential role in daily life, not only as a necessity but also as a source of enjoyment and cultural expression. Many people find joy in cooking and exploring new recipes, often balancing taste preferences with health considerations. One common trade-off in food choices is between flavor and nutritional value—particularly when it comes to **fat content**. While fats can enhance taste and texture, excessive consumption is linked to various health concerns, including cardiovascular diseases.

In this study, I analyze data [Food.com](https://www.food.com) to **explore the relationship between fat content and recipe ratings**. I aim to understand whether higher fat content correlates with better ratings, reflecting the common perception that "fat equals flavor," or if health-conscious users rate high-fat recipes lower due to nutritional concerns. Beyond total fat, I also examine **the ratio of saturated fat to total fat**, as saturated fats are often considered less healthy than unsaturated fats. By analyzing this ratio, I investigate whether the type of fat in a recipe impacts user ratings.

To conduct this analysis, I utilize two datasets from Food.com, containing recipes and user ratings since 2008. These datasets were originally compiled for the research paper [*Generating Personalized Recipes from Historical User Preferences* by Majumder et al.](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf). which focused on recommender system applications. By leveraging these data, I seek to uncover trends in user preferences and gain insights into how fat content and fat quality influence recipe ratings.

### Dataset Overview

The first dataset `recipes` contains information about recipes, including their names, preparation time, nutritional content, and user-contributed descriptions. It contains 83782 rows(representing 83782 unique recipes), with 12 columns recording the following information:

| Column          | Description  |
|----------------|-------------|
| `id`          | Unique identifier for each recipe |
| `name`        | Recipe name |
| `minutes`        | Minutes to prepare recipe |
| `contributor_id`      | User ID who submitted this recipe |
| `submitted`   | Date when the recipe was submitted |
| `nutrition`   | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for "percentage of daily value" |
| `tags`        | Food.com tags for recipe |
| `n_steps`     | Number of steps in the recipe |
| `steps`       | Text for recipe steps, in order |
| `description` | User-provided description |
| `ingredients` | List of ingredients used in the recipe |
| `n_ingredients` | Number of ingredients in the recipe |

The second dataset `interactions` contains user interactions with recipes, including ratings and review texts. There are 731,927 rows in total, with 5 columns recording the following information:

#### Relevant Columns and Descriptions:

| Column      | Description  |
|------------|-------------|
| `user_id`  | Unique identifier for the user who provided the rating |
| `recipe_id` | Unique identifier for the recipe being rated |
| `date`      | Date when the rating was submitted |
| `rating`    | Rating given by the user (typically on a 1-5 scale) |
| `review`    | Text review provided by the user |




## Cleaning and EDA

To prepare the dataset for analysis, a series of data cleaning and transformation steps were performed. These steps ensure that the data is structured correctly, missing values are handled appropriately, and relevant features are extracted for further analysis.

1. Merging Recipes and Ratings & Handling Missing Values

- To integrate recipe information with user ratings, the recipes dataset was merged with the interactions dataset using a left join on the recipe ID. This ensures that all recipes remain in the dataset, even if they have not been rated. After merging, some ratings were recorded as 0, which falls outside the valid rating scale of 1 to 5. These zero values likely indicate missing ratings rather than actual user feedback. To prevent distortion in the analysis, all 0 ratings were replaced with missing values (NaN). Since a recipe can receive multiple ratings from different users, an average rating per recipe was calculated by grouping the dataset by recipe ID and computing the mean rating. This aggregated rating was then merged back into the recipes dataset, preserving all recipe attributes while incorporating the average rating. Recipes that had never been rated remained in the dataset with a missing value for average rating.

2. Nutrition Processing and Macronutrient Proportions  

To analyze the impact of fat content on recipe ratings, the nutrition data was processed, and macronutrient proportions were calculated.

- Extracting and Cleaning Nutrition Data:  
  The `nutrition` column originally stored a list of numerical values as a string, making it difficult to analyze. Square brackets were removed, and the column was split into individual features:  
  `Calories (#)`, `Total fat (PDV)`, `Sugar (PDV)`, `Sodium (PDV)`, `Protein (PDV)`, `Saturated fat (PDV)`, `Carbohydrates (PDV)`.  
  Each value was converted to a float for numerical operations, and the original `nutrition` column was dropped after extraction.

- Calculating Macronutrient Proportions:  
  To standardize nutrient comparisons, macronutrient proportions were calculated relative to total caloric intake. These calculations were performed only for recipes where `calories (#) > 0` to avoid division errors (others were set to 0).  
  - `prop_protein`: 50g daily value, 1g = 4 kcal  
  - `prop_total_fat`: 50g daily value, 1g = 9 kcal  
  - `prop_saturated_fat`: 20g daily value, 1g = 9 kcal  
  - `prop_sugar`: 50g daily value, 1g = 4 kcal  
  - `prop_sodium`: Converted directly from PDV to decimal form  
  - `prop_carbohydrates`: 275g daily value, 1g = 4 kcal  
  Since the PDV-based nutrient columns were no longer needed after the proportions were calculated, they were removed.

- Identifying and Removing Invalid Entries:  
  Entries where `saturated fat (PDV) > total fat (PDV)` were removed because it is impossible for saturated fat to exceed total fat. This likely indicates data errors. Entries where `total fat (PDV) = 0` but `saturated fat (PDV) > 0` were also removed, as a recipe with no fat cannot contain saturated fat. A total of 233 invalid entries were removed to maintain data integrity.  

- Computing the Proportion of Saturated Fat Relative to Total Fat:  
  A new feature `sat_fat_ratio` was created to measure the proportion of saturated fat within total fat. If total fat was 0, the ratio was set to 0 to prevent division errors.  This feature helps distinguish whether a recipe's fat content is primarily from saturated sources.  

- Classifying High-Fat and High-Saturated-Fat Recipes:  
  To differentiate recipes based on fat content:  
  - `is_high_fat`: Assigned **1** if the **total fat proportion** is **above the dataset mean**, otherwise **0**.  
  - `is_high_saturated_fat`: Assigned **1** if the **saturated fat proportion** is **above the dataset mean**, otherwise **0**.  
  This classification helps analyze how both total fat and saturated fat content influence user ratings. 

3. Extracting Time Features  

- To enable time-based analysis, the `submitted` column was converted to **datetime format**, allowing for easier extraction of relevant time components. From this, two new features, `year` and `month`, were created to capture when each recipe was submitted. Since the exact submission date was no longer necessary, the original `submitted` column was dropped after extracting these features. These time-based attributes allow for potential trend analysis, such as examining changes in recipe ratings or fat content over time.


<iframe src="assets/10-80-enrollment.html" width=800 height=600 frameBorder=0></iframe>

---

## Assessment of Missingness

Here's what a Markdown table looks like. Note that the code for this table was generated _automatically_ from a DataFrame, using

```py
print(counts[['Quarter', 'Count']].head().to_markdown(index=False))
```

| Quarter     |   Count |
|:------------|--------:|
| Fall 2020   |       3 |
| Winter 2021 |       2 |
| Spring 2021 |       6 |
| Summer 2021 |       4 |
| Fall 2021   |      55 |

---

## Hypothesis Testing


---
