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

| Column      | Description  |
|------------|-------------|
| `user_id`  | Unique identifier for the user who provided the rating |
| `recipe_id` | Unique identifier for the recipe being rated |
| `date`      | Date when the rating was submitted |
| `rating`    | Rating given by the user (typically on a 1-5 scale) |
| `review`    | Text review provided by the user |




## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

To prepare the dataset for analysis, a series of data cleaning and transformation steps were performed. These steps ensure that the data is structured correctly, missing values are handled appropriately, and relevant features are extracted for further analysis.

**1. Merging Recipes and Ratings & Handling Missing Values**  

To integrate recipe information with user ratings, the recipes dataset was merged with the interactions dataset using a left join on the recipe ID, ensuring all recipes remain in the dataset even if unrated. Some ratings were recorded as 0, which falls outside the valid range of 1 to 5 and likely represents missing values. To prevent distortion, all 0 ratings were replaced with NaN. Since recipes can receive multiple ratings, an average rating per recipe was calculated and merged back into the recipes dataset, preserving all attributes while incorporating the average rating. Unrated recipes retained a missing value for average rating.  

**2. Nutrition Processing and Macronutrient Proportions**

To analyze the impact of fat content on ratings, the nutrition data was processed, and macronutrient proportions were calculated.  

- **Extracting and Cleaning Nutrition Data**:  
  The `nutrition` column, initially stored as a string, was split into separate features: `Calories (#)`, `Total fat (PDV)`, `Sugar (PDV)`, `Sodium (PDV)`, `Protein (PDV)`, `Saturated fat (PDV)`, and `Carbohydrates (PDV)`. Each value was converted to a float, and the original `nutrition` column was dropped.  

- **Calculating Macronutrient Proportions**:  
  Macronutrient proportions were calculated relative to total caloric intake, ensuring comparisons across recipes. Calculations were performed only for recipes where `calories (#) > 0`, with others set to 0.  
  - `prop_protein`: 50g daily value, 1g = 4 kcal  
  - `prop_total_fat`: 50g daily value, 1g = 9 kcal  
  - `prop_saturated_fat`: 20g daily value, 1g = 9 kcal  
  - `prop_sugar`: 50g daily value, 1g = 4 kcal  
  - `prop_sodium`: Converted from PDV to decimal form  
  - `prop_carbohydrates`: 275g daily value, 1g = 4 kcal
  
  The original PDV-based columns were dropped after these transformations.
  
- **Identifying and Removing Invalid Entries**:  
  Entries where `saturated fat (PDV) > total fat (PDV)` were removed, as saturated fat cannot exceed total fat. Similarly, recipes where `total fat (PDV) = 0` but `saturated fat (PDV) > 0` were also removed. A total of 233 invalid entries were dropped.  

- **Computing the Proportion of Saturated Fat Relative to Total Fat**:  
  A new feature, `sat_fat_ratio`, was created to measure the proportion of saturated fat within total fat. If total fat was 0, the ratio was set to 0 to avoid division errors.  

- **Classifying High-Fat and High-Saturated-Fat Recipes**:  
  Recipes were categorized based on fat-derived calorie proportions:  
  - `is_high_fat`: **1** if total fat proportion is above the dataset mean, otherwise **0**.  
  - `is_high_saturated_fat`: **1** if saturated fat proportion is above the dataset mean, otherwise **0**.  
This classification helps analyze how both total fat and saturated fat influence user ratings.  

**3. Extracting Time Features**

To enable time-based analysis, the `submitted` column was converted to **datetime format**, and two new features, `year` and `month`, were extracted. Since the exact submission date was no longer needed, the `submitted` column was dropped. These time features allow for trend analysis, such as examining changes in recipe ratings or fat content over time.  

**4. Final Data Cleaning**

After processing the necessary transformations, a subset of relevant columns was selected for the final cleaned dataset. The selection focused on key attributes related to **recipe complexity, nutritional composition, fat-related properties, and user ratings**. The `year` and `month` features replaced the original `submitted` column to enable time-based analysis. This final dataset ensures that only the most relevant features are retained for further analysis.  

### Result  

Here are the columns of the cleaned dataframe:

| Column                 | Description                              |
|------------------------|------------------------------------------|
| `'id'`                | Recipe ID (int64)                        |
| `'minutes'`           | Time to prepare the recipe (int64)       |
| `'year'`              | Year the recipe was submitted (int64)    |
| `'month'`             | Month the recipe was submitted (int64)   |
| `'n_steps'`           | Number of steps in the recipe (int64)    |
| `'n_ingredients'`     | Number of ingredients used (int64)       |
| `'calories (#)'`      | Total calories in the recipe (float64)   |
| `'prop_protein'`      | Proportion of protein calories (float64) |
| `'prop_total_fat'`    | Proportion of total fat calories (float64) |
| `'prop_saturated_fat'` | Proportion of saturated fat calories (float64) |
| `'prop_sugar'`        | Proportion of sugar calories (float64)   |
| `'prop_carbohydrates'` | Proportion of carbohydrate calories (float64) |
| `'prop_sodium'`       | Proportion of sodium (float64)           |
| `'sat_fat_ratio'`     | Ratio of saturated fat to total fat (float64) |
| `'is_high_fat'`       | 1 if fat proportion is above dataset mean, else 0 (int64) |
| `'is_high_saturated_fat'` | 1 if saturated fat proportion is above dataset mean, else 0 (int64) |
| `'avg_rating'`        | Average rating of the recipe (float64)   |
| `'description'`       | Text description of the recipe (object)  |


This cleaned dataset is now ready for further analysis to explore the relationship between fat content and user ratings. Our cleaned dataframe consists of **82,789 rows and 18 columns**. Below is a preview of the first five unique recipes. Scroll right to view more columns.  

<div style="overflow-x: auto; white-space: nowrap;">

|     id |   minutes |   year |   month |   n_steps |   n_ingredients |   calories (#) |   prop_protein |   prop_total_fat |   prop_saturated_fat |   prop_sugar |   prop_carbohydrates |   prop_sodium |   sat_fat_ratio |   is_high_fat |   is_high_saturated_fat |   avg_rating | description                                                                                                                                                                                                                                                                                                                                                                       |
|-------:|----------:|-------:|--------:|----------:|----------------:|---------------:|---------------:|-----------------:|---------------------:|-------------:|---------------------:|--------------:|----------------:|--------------:|------------------------:|-------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 333281 |        40 |   2008 |      10 |        10 |               9 |          138.4 |      0.0433526 |         0.325145 |             0.24711  |    0.722543  |             0.476879 |          0.03 |        0.76     |             1 |                       1 |            4 | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven!                                                                                                              |
| 453467 |        45 |   2011 |       4 |        12 |              11 |          595.1 |      0.0436901 |         0.347841 |             0.15426  |    0.709125  |             0.480591 |          0.22 |        0.443478 |             1 |                       1 |            5 | this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don't have margarine or don't like it, then just use butter (softened) instead.                                                                                                                                            |
| 306168 |        40 |   2008 |       5 |         6 |               9 |          194.8 |      0.225873  |         0.462012 |             0.332649 |    0.0616016 |             0.169405 |          0.32 |        0.72     |             1 |                       1 |            5 | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 |
| 286009 |       120 |   2008 |       2 |         7 |               7 |          878.3 |      0.0455425 |         0.322783 |             0.252078 |    0.742343  |             0.488444 |          0.13 |        0.780952 |             1 |                       1 |            5 | why a millionaire pound cake?  because it's super rich!  this scrumptious cake is the pride of an elderly belle from jackson, mississippi.  the recipe comes from "the glory of southern cooking" by james villas.                                                                                                                                                                |
| 475785 |        90 |   2012 |       3 |        17 |              13 |          267   |      0.217228  |         0.505618 |             0.323596 |    0.0898876 |             0.082397 |          0.12 |        0.64     |             1 |                       1 |            5 | ready, set, cook! special edition contest entry: a mediterranean flavor inspired meatloaf dish. featuring: simply potatoes - shredded hash browns, egg, bacon, spinach, red bell pepper, and goat cheese.     |

</div>

### Univariate Analysis 

<iframe
  src="graphs/prop_total_fat_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
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
