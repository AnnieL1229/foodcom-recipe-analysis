# Food.com Recipe Analysis: xxx

# My Project Title

by Yixin Liu (yil165@ucsd.edu)

## Overview


## Introduction

Food plays an essential role in daily life, not only as a necessity but also as a source of enjoyment and cultural expression. Many people find joy in cooking and exploring new recipes, often balancing taste preferences with health considerations. One common trade-off in food choices is between flavor and nutritional value—particularly when it comes to **fat content**. While fats can enhance taste and texture, excessive consumption is linked to various health concerns, including cardiovascular diseases.

In this study, we analyze data [Food.com](https://www.food.com) to **explore the relationship between fat content and recipe ratings**. We aim to understand whether higher fat content correlates with better ratings, reflecting the common perception that "fat equals flavor," or if health-conscious users rate high-fat recipes lower due to nutritional concerns. Beyond total fat, we also examine **the ratio of saturated fat to total fat**, as saturated fats are often considered less healthy than unsaturated fats. By analyzing this ratio, we investigate whether the type of fat in a recipe impacts user ratings.

To conduct this analysis, we utilize two datasets from Food.com, containing recipes and user ratings since 2008. These datasets were originally compiled for the research paper [*Generating Personalized Recipes from Historical User Preferences* by Majumder et al.](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf). By uncovering patterns in how fat content and its composition influence user ratings, this study provides insights into consumer food preferences, which could help recipe creators and food brands balance taste and health considerations to better meet user expectations.

The first dataset `recipes` includes details about each recipe, such as its name, preparation time, nutritional content, and user-contributed descriptions." It contains 83,782 rows(representing 83,782 unique recipes), with 12 columns recording the following information:

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
| `rating`    | Rating given by the user |
| `review`    | Text review provided by the user |




## Data Cleaning and Exploratory Data Analysis (EDA)

### Data Cleaning

To prepare the dataset for analysis, a series of data cleaning and transformation steps were performed. 

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

After processing the necessary transformations, a subset of relevant columns was selected for the final cleaned dataset. The selection focused on key attributes related to recipe complexity, nutritional composition, fat-related properties, and user ratings.  

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


This cleaned dataset is now ready for further analysis to explore the relationship between fat content and user ratings. Our cleaned dataframe consists of **82,789 rows and 18 columns**. Below is a preview of the first five rows. 

|    |     id |   minutes |   year |   month |   n_steps |   n_ingredients |   calories (#) |   prop_protein |   prop_total_fat |   prop_saturated_fat |   prop_sugar |   prop_carbohydrates |   prop_sodium |   sat_fat_ratio |   is_high_fat |   is_high_saturated_fat |   avg_rating | description                                                                                                                                                                                                                                                                                                                                                                       |\n|---:|-------:|----------:|-------:|--------:|----------:|----------------:|---------------:|---------------:|-----------------:|---------------------:|-------------:|---------------------:|--------------:|----------------:|--------------:|------------------------:|-------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|\n|  0 | 333281 |        40 |   2008 |      10 |        10 |               9 |          138.4 |      0.0433526 |         0.325145 |             0.24711  |    0.722543  |             0.476879 |          0.03 |        0.76     |             1 |                       1 |            4 | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you\'ll ever make.....sereiously! there\'s no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they\'re pure heaven!                                                                                                              |\n|  1 | 453467 |        45 |   2011 |       4 |        12 |              11 |          595.1 |      0.0436901 |         0.347841 |             0.15426  |    0.709125  |             0.480591 |          0.22 |        0.443478 |             1 |                       1 |            5 | this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don\'t have margarine or don\'t like it, then just use butter (softened) instead.                                                                                                                                            |\n|  2 | 306168 |        40 |   2008 |       5 |         6 |               9 |          194.8 |      0.225873  |         0.462012 |             0.332649 |    0.0616016 |             0.169405 |          0.32 |        0.72     |             1 |                       1 |            5 | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don\'t think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell\'s soup. but i think mine is better since i don\'t like cream of mushroom soup.submitted to "zaar" on may 28th,2008 |\n|  3 | 286009 |       120 |   2008 |       2 |         7 |               7 |          878.3 |      0.0455425 |         0.322783 |             0.252078 |    0.742343  |             0.488444 |          0.13 |        0.780952 |             1 |                       1 |            5 | why a millionaire pound cake?  because it\'s super rich!  this scrumptious cake is the pride of an elderly belle from jackson, mississippi.  the recipe comes from "the glory of southern cooking" by james villas.                                                                                                                                                                |\n|  4 | 475785 |        90 |   2012 |       3 |        17 |              13 |          267   |      0.217228  |         0.505618 |             0.323596 |    0.0898876 |             0.082397 |          0.12 |        0.64     |             1 |                       1 |            5 | ready, set, cook! special edition contest entry: a mediterranean flavor inspired meatloaf dish. featuring: simply potatoes - shredded hash browns, egg, bacon, spinach, red bell pepper, and goat cheese.                                                                                                                                                                         |

### Univariate Analysis 

To understand the distribution of key variables, I analyzed **total fat proportion, saturated fat ratio, and recipe ratings**.

**1. Distribution of Total Fat Proportion**

The histogram below shows the distribution of **total fat proportion**, representing the proportion of calories derived from fat. Most recipes have a fat proportion between **0.2 and 0.5**. The boxplot highlights a few outliers, where certain recipes have unusually high fat-derived calorie proportions.

<iframe
  src="graphs/prop_total_fat_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**2. Distribution of Saturated Fat Ratio**

The histogram below shows the distribution of **saturated fat ratio**, which measures the proportion of total fat that comes from saturated sources. A ratio of **0** can indicate:
- The recipe **contains no fat at all** (total fat = 0).  
- The recipe contains fat, but **none of it is saturated fat** (only unsaturated fats).  

There are two peaks:  
- **Low saturated fat (0.2 - 0.3):** Likely from healthier fats (e.g., olive oil, nuts).  
- **High saturated fat (0.8+):** Likely due to butter, cream, or deep-fried foods.  

<iframe
  src="graphs/saturated_fat_ratio_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**3.Distribution of Recipe Ratings**

The histogram of **recipe ratings** shows a strong skew toward high ratings, with a majority of ratings clustering around **5**. While lower ratings exist, they are much less frequent. This suggests a positive bias in user reviews, where most users tend to rate recipes favorably. 

<iframe
  src="graphs/recipe_ratings_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis: Fat Content and Recipe Ratings  

Our analysis of low-fat and high-fat recipes shows that **recipes with more fat tend to receive slightly higher ratings**. Both types of recipes have a median rating close to 5, meaning most recipes are rated well. However, low-fat recipes have more low ratings compared to high-fat recipes. This suggests that fat might help improve flavor and texture, making dishes more enjoyable for users

<iframe
  src="graphs/high_low_fat_rating_boxplot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="graphs/high_low_fat_mean_rating_bar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Similarly, **high-saturated-fat recipes exhibit slightly higher average ratings** than low-saturated-fat ones. This suggests that saturated fat, commonly found in butter and cream-based dishes, may enhance taste perception and appeal. This insight highlights a potential trade-off between health and taste, where users may prioritize enjoyment over healthfulness when rating recipes.

<iframe
  src="graphs/high_low_sat_fat_rating_boxplot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="graphs/high_low_sat_fat_mean_rating_bar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates: Fat Proportion Across Recipe Complexity

To explore how recipe complexity relates to fat content, we grouped recipes by **number of steps (`n_steps`)** and calculated key aggregate statistics for **total fat proportion (`prop_total_fat`)**. 

| `n_steps` | Mean (`prop_total_fat`) | Median (`prop_total_fat`) | Min (`prop_total_fat`) | Max (`prop_total_fat`) |
|-----------|-------------------------|---------------------------|-------------------------|-------------------------|
| 1         | 0.24                    | 0.17                      | 0.0                     | 0.78                    |
| 2         | 0.25                    | 0.20                      | 0.0                     | 0.78                    |
| 3         | 0.27                    | 0.26                      | 0.0                     | 0.78                    |
| ...       | ...                      | ...                        | ...                     | ...                     |
| 21        | 0.33                    | 0.33                      | 0.0                     | 0.75                    |
| 22        | 0.33                    | 0.33                      | 0.0                     | 0.73                    |
| 23        | 0.33                    | 0.34                      | 0.0                     | 0.76                    |


The line plot below confirms a **positive correlation between the number of steps and fat proportion**. The sharp increase in fat content for recipes with 2–5 steps suggests that moderately complex recipes often incorporate additional fat sources.

<iframe
  src="graphs/nsteps_rating_agg.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Assessment of Missingness

### Not Missing at Random (NMAR) Analysis

One NMAR example could be the **"description"** column because missing values here are likely intentional rather than random. Since the description is an optional summary rather than essential cooking instructions, users may choose to skip it. A missing description might indicate that the recipe is too simple to require one or that the contributor put minimal effort into the submission. For example, a gourmet dish may have a detailed description to highlight its uniqueness, while a basic recipe may have none at all.

### Missingness Dependency

We explored the missingness of the **"rating"** column. Since user ratings reflect how recipes are perceived, understanding whether missing ratings occur randomly or are influenced by specific factors is crucial. To investigate whether missing ratings (`avg_rating`) depend on other features, we conducted permutation tests on `prop_total_fat` and `n_steps`.

**1. Testing Dependency on `prop_total_fat`**  

- **Null Hypothesis (H₀):** The missingness of `avg_rating` is independent of `prop_total_fat`.
- **Alternative Hypothesis (H₁):** The missingness of `avg_rating` depends on `prop_total_fat`, meaning recipes with missing ratings systematically differ in fat content. 
- **Test Statistic:** Difference in mean prop_total_fat between recipes with and without missing ratings. 
- **Significance Level:** 0.05

<iframe
  src="graphs/missing_distr_fat.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

To test this, we performed 1,000 permutations, shuffling the missingness labels and recalculating the difference in mean `prop_total_fat` each time. This created an empirical distribution of differences under the null hypothesis.

<iframe
  src="graphs/MAR_fat.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed difference of 0.0032 (marked by the red vertical line) falls within the expected range under the null distribution. The computed p-value (0.3590) is greater than 0.05, so we **fail to reject the null hypothesis**. This suggests that there is no strong evidence that missing ratings are dependent on fat content (`prop_total_fat`).

**2. Testing Dependency on `n_steps`**

- **Null Hypothesis (H₀):** The missingness of `avg_rating` is independent of `n_steps`.  
- **Alternative Hypothesis (H₁):** The missingness of `avg_rating` depends on `n_steps`, meaning recipes with missing ratings systematically differ in the number of steps.  
- **Test Statistic:** Difference in mean `n_steps` between recipes with and without missing ratings.  
- **Significance Level:** 0.05  

<iframe  
  src="graphs/missing_distr_nsteps.html"  
  width="800"  
  height="600"  
  frameborder="0"  
></iframe>  

Similarly, we performed 1,000 permutations, shuffling the missingness labels and recalculating the difference in mean `n_steps` each time. 

<iframe  
  src="graphs/MAR_nsteps.html"  
  width="800"  
  height="600"  
  frameborder="0"  
></iframe>  

The observed difference of 89.2767 (marked by the red vertical line) falls far outside the expected range under the null distribution. The computed p-value (0.0000) is far below 0.05, so we **reject the null hypothesis**. This provides strong evidence that the missingness of `avg_rating` is not random but is related to `n_steps`.  

## Hypothesis Testing

Our interest lies in understanding the relationship between fat content and recipe ratings. As observed in our EDA, recipes with higher fat proportions (`prop_total_fat`) tend to have higher ratings, suggesting that users may prioritize taste over health.

Beyond total fat, we also examine fat quality, measured by the saturated fat ratio (`sat_fat_ratio`), which represents the proportion of total fat that comes from saturated sources. Saturated fats in ingredients like butter, cream, and fried foods create rich flavors and textures that often characterize indulgent dishes. Our initial analysis showed that recipes with a higher saturated fat ratio also tend to receive higher ratings, further reinforcing the idea that users favor richer, more flavorful meals.

To determine whether these observed differences are statistically significant, we conducted two hypothesis tests:  
1. Testing whether high-fat recipes (`is_high_fat = 1`) receive significantly higher ratings than low-fat recipes. (`is_high_fat = 1` if `prop_total_fat` is above the mean)
2. Testing whether high-saturated-fat recipes (`is_high_saturated_fat = 1`) receive significantly higher ratings than low-saturated-fat recipes. (`is_high_saturated_fat = 1` if `sat_fat_ratio` is above the mean) 

Since the true population distribution of ratings is unknown, we use **permutation tests**.  

**Test 1: `is_high_fat` and Recipe Ratings**  

- Null Hypothesis (H₀): There is no difference in average ratings between high-fat and low-fat recipes.  
- Alternative Hypothesis (H₁): High-fat recipes receive **higher** ratings than low-fat recipes.  
- Test Statistic: Difference in mean `avg_rating` between high-fat (`is_high_fat = 1`) and low-fat (`is_high_fat = 0`) recipes.  
- Significance Level: 0.05  

<iframe  
  src="graphs/HT_fat.html"  
  width="800"  
  height="600"  
  frameborder="0"  
></iframe>  

The observed mean difference for high-fat recipes was 0.0349, which is substantially larger than what we would expect under the null hypothesis. The computed p-value is 0.0000, which is well below the significance level of 0.05. Thus, we **reject the null hypothesis**, suggesting that high-fat recipes tend to receive higher ratings than low-fat ones. One possible explanation is that higher-fat recipes often incorporate richer ingredients that enhance flavor, making them more appealing to users.


**Test 2: `is_high_saturated_fat` and Recipe Ratings**  

- Null Hypothesis (H₀): There is no difference in average ratings between high satrated ratio and low-fat recipes.  
- Alternative Hypothesis (H₁): High-fat recipes receive **higher** ratings than low-fat recipes.  
- Test Statistic: Difference in mean `avg_rating` between high-fat (`is_high_fat = 1`) and low-fat (`is_high_fat = 0`) recipes.  
- Significance Level: 0.05  

<iframe  
  src="graphs/HT_sat_fat.html"  
  width="800"  
  height="600"  
  frameborder="0"  
></iframe>  

The observed mean difference was 0.0228, with a p-value of 0.0000, well below the 0.05 threshold. Thus, we **reject the null hypothesis**. There is strong evidence that people rate higher for recipes with larger saturated fat proportions. This may be because saturated fats—found in butter, cream, and fried foods—enhance texture and flavor, making dishes more enjoyable, whereas unsaturated fats—found in avocados, olive oil, and nuts—are generally considered healthier but are associated with lighter, heart-healthy options.

## Framing a Prediction Problem

The goal of this prediction task is to determine whether a recipe is likely to receive a high rating. Instead of predicting the exact rating, we frame this as a binary classification problem, where:

- `binary_rating = 1` if `avg_rating ≥ 4.5` (highly rated)
- `binary_rating = 0` if `avg_rating < 4.5` (not highly rated)

Recipe ratings on the platform are highly skewed toward higher ratings, meaning most recipes receive relatively positive feedback. Instead of small variations in scores, we focus on distinguishing between exceptionally well-rated recipes (≥4.5) and those that did not achieve such high ratings. This allows us to better understand what factors contribute to top-tier recipe ratings, making the results more actionable.

Since ratings are highly skewed toward higher values, accuracy alone would not be a reliable metric. Instead, we use the weighted F1-score, which accounts for class imbalance and ensures a fair evaluation across both highly rated and non-highly rated recipes. By framing the problem as a classification task and choosing an appropriate metric, we ensure that the model captures the true distribution of ratings and effectively differentiates top-rated recipes from others.

## Baseline Model

We use a **Random Forest Classifier** as our baseline model due to its ability to handle nonlinear relationships and capture feature interactions without extensive preprocessing. The baseline model includes **two quantitative features**: `prop_total_fat` (The proportion of total fat in the recipe), and `sat_fat_ratio` (The proportion of total fat that is saturated fat). Since both features are continuous numerical variables, no additional encoding was required.  

The dataset was split into training (80%) and testing (20%) sets. The Random Forest Classifier achieved a weighted F1-score of 0.6190. Specifically, F1-score for highly rated recipes (class 1) is 0.77, and F1-score for low-rated recipes (class 0) is 0.21. While this score provides a reasonable baseline, the model struggles with predicting low-rated recipes (class 0) due to class imbalance, leading to a low F1-score for this class.  

This suggests that fat content alone is insufficient for accurately predicting ratings. Additional features may improve model performance. Future iterations will explore expanding feature selection and optimizing hyperparameters to enhance predictive accuracy.  

## Final model 

### Feature Selection

For our final model, we selected the following features based on exploratory data analysis (EDA) and hypothesis testing:

- `minutes`
  - From our bivariate analysis, recipes that take longer to prepare tend to receive higher ratings. This pattern suggests that users may associate longer preparation times with higher-quality dishes, possibly due to the effort and complexity involved. We expect this feature to positively contribute to our model’s predictive performance.  

- `month`
  - As observed in the seasonal trend analysis, recipe ratings fluctuate throughout the year (see graph below). Ratings peak during certain months (e.g., spring and early summer), which may reflect seasonal food trends, holiday cooking patterns, or user activity changes. Including `month` allows our model to capture seasonal variations in user preferences.  

<iframe  
  src="graphs/monthly_trend.html"  
  width="800"  
  height="600"  
  frameborder="0"  
></iframe>  

- `prop_total_fat` (Proportion of Calories from Fat)  
  - Our hypothesis test provided strong evidence that fat content influences ratings, with higher-fat recipes receiving higher ratings. Since fat enhances taste and texture, it is an essential predictor of user preferences.  

- `sat_fat_ratio` (Proportion of Saturated Fat in Total Fat)  
  - Similar to total fat, our hypothesis test showed that higher saturated fat ratios are associated with higher ratings. Saturated fats are known for their rich flavor and texture, which may contribute to higher user satisfaction. This feature helps distinguish between different types of fat, allowing the model to capture nuanced user preferences.

By incorporating these features, our model aims to predict recipe ratings more effectively by leveraging key factors that influence user preferences, including **preparation effort, seasonality, and fat content**.

### Feature Transformations & Hyperparameter Tuning

To enhance our model’s predictive performance, we implemented feature transformations, class balancing, and hyperparameter tuning.

- Feature Transformations
  - Cyclic Encoding for `month`: Since `month` represents a cyclical feature (January and December are close), we applied **sine and cosine transformations** to capture seasonal patterns effectively. This prevents incorrect interpretations where December (`month = 12`) is numerically distant from January (`month = 1`).

  - Robust Scaling for Numeric Features: `minutes`, `prop_total_fat`, and `sat_fat_ratio` were **scaled using RobustScaler**, which is resistant to outliers. 

- Addressing Class Imbalance with SMOTE
  - Our dataset exhibits imbalance, with significantly more high-rated recipes than low-rated ones. To mitigate this, we used **Synthetic Minority Over-sampling Technique (SMOTE)** to generate synthetic samples for the minority class, improving our model’s ability to predict low-rated recipes.

- Hyperparameter Tuning (GridSearchCV)
  - We performed hyperparameter tuning** using **GridSearchCV** to optimize our Random Forest model. The parameters tested included:
    - `n_estimators`: [50, 100, 200] (Number of trees in the forest)  
    - `max_depth`: [5, 10, 20, None] (Tree depth)  
    - `min_samples_split`: [2, 5, 10] (Minimum samples needed to split a node)  
    - `class_weight`: [“balanced”, None] (To handle class imbalance)

### Result

Overall, the weighted F1-score **improved from 0.6190 to 0.6428**. The most notable improvement was in the prediction of low-rated recipes (Class 0), where the recall and F1-score increased from 0.21 to 0.25. The use of SMOTE successfully mitigated the impact of class imbalance, preventing the model from over-prioritizing high-rated recipes.

##











