<style>

.table-wrapper {
    overflow-x: scroll;
  }

</style>

# predicting_recipe_popularity

## **Introduction**

___

### **Abstract and Data**  
Nutritional information plays a key role in how people choose recipes; by examining nutritional values, such as calorie count, fat content, or protein levels, individuals can better understand how a recipe fits into their personal needs. Understanding preferences purely from nutritional data can empower people to make more informed choices, helping them identify recipes that suit their goals without needing to rely solely on trial and error. This project will focus in particular on understanding how users may unknowingly rate recipes based on nutritional content, specifically calorie and sugar content, through the question,  
  
**Do healthier recipes tend to have higher ratings?**  
  
This exploratory project utilizes data from Food.com, focusing on recipes and reviews posted since 2008. Two datasets, `RAW_recipes.csv` and `RAW_interactions.csv`, are analyzed.  
Below are the columns from both datasets:  

#### Recipes Dataset:  ``RAW_recipes.csv``  
- `name` The name of the recipe.  
- `id` A unique identifier for the recipe.  
- `minutes` The time required to prepare the recipe, in minutes.  
- `contributor_id` The ID of the user who submitted the recipe.  
- `submitted` The date the recipe was submitted.  
- `tags` A list of tags associated with the recipe, provided by Food.com.  
- `nutrition` A list containing the nutritional values for the recipe: calories, total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), and carbohydrates (PDV).  
- `n_steps` The number of steps required to prepare the recipe.  
- `steps` A list of textual instructions for preparing the recipe, in order.  
- `description` A user-provided description of the recipe.  

#### Ratings Dataset:  `RAW_interactions.csv`  
- `user_id` The ID of the user who interacted with a recipe.  
- `recipe_id` The unique ID of the recipe that was reviewed.  
- `date` The date the interaction occurred.  
- `rating` The numerical rating (out of 5) given to the recipe.  
- `review` The text of the user's review for the recipe.  

These datasets offer a detailed look at the interplay between recipe characteristics, nutritional content, and user feedback, enabling insights into how people select and evaluate recipes based on nutritional data.


<br>

## **Data Cleaning and Exploratory Data Analysis**

___

### **Data Cleaning**

There were a few columns with missing values:
- `name` - 1 missing value
- `description` - 70 missing values
- `review` - 169 missing values

Since this project does not require `name` and `description` information (recipes are labelled by id, and nutritional information can be found outside of the description), these columns will be dropped to prevent confusion. 

In order to find sentiment regarding a recipe, the average rating for each recipe is found through left merging the interactions dataset into the recipes dataset, to ensure all reviews being used are correlated to a currently existing recipe. All 0 values were replaced with `NaN`; this is because the lowest star rating that a recipe can be given is 1, implying missing or erroneous data rather than a true evaluation.

![food.com review screenshot](./resources/food.com_review.png)

Additionally, pandas implicitly excludes `NaN` values from mean calculation, making it ideal for making sure the general shape of the data is correct, even if the true evaluation of a review was not present.  

The following dataset is the result of merging average reviews back into the recipe dataset:  

`recipe_reviews.head()`  

<div class="table-wrapper" markdown="block">

| Name                           | ID     | Minutes | Contributor ID | Submitted   | Tags                                  | Nutrition                     | Steps | Steps Description                        | Description                        | Ingredients                              | Ingredients Count | Average Rating |
|--------------------------------|---------|----------|----------------|-------------|---------------------------------------|--------------------------------|-------|------------------------------------------|------------------------------------|------------------------------------------|-------------------|--------|
| 1 brownies in th...            | 333281  | 40       | 985201         | 2008-10-27  | [60-minutes-or-less, ...]             | [138.4, 10.0, 50...]          | 10    | [heat the oven to...]                   | these are the mo...               | [bittersweet chocolate, ...]            | 9                 | 4.0    |
| 1 in canada choc...            | 453467  | 45       | 1848091        | 2011-04-11  | [60-minutes-or-less, ...]             | [595.1, 46.0, 21...]          | 12    | [pre-heat oven t...]                    | this is the reci...               | [white sugar, brown sugar, ...]         | 11                | 5.0    |
| 412 broccoli cas...            | 306168  | 40       | 50969          | 2008-05-30  | [60-minutes-or-less, ...]             | [194.8, 20.0, 6...]           | 6     | [preheat oven to...]                    | since there are ...               | [frozen broccoli, ...]                  | 9                 | 5.0    |
| millionaire poun...            | 286009  | 120      | 461724         | 2008-02-12  | [time-to-make, cookies-and-brownies]  | [878.3, 63.0, 32...]          | 7     | [freheat the ove...]                    | why a millionair...               | [butter, sugar, ...]                    | 7                 | 5.0    |
| 2000 meatloaf                  | 475785  | 90       | 2202916        | 2012-03-06  | [time-to-make, comfort-food, ...]     | [267.0, 30.0, 12...]          | 17    | [pan fry bacon ,...]                    | ready, set, cook...               | [meatloaf mixture, ...]                 | 13                | 5.0    |

</div>

After finding average reviews, only the columns that are relevant to the project are kept, which includes `nutrition` (includes relevant information on nutrition facts), `ratings_avg` (to find if nutrition affects it), and `id` (to cross reference recipes later on if needed).

Then, nutrition is expanded into seven columns: `calories`, `t_fat` (total fat), `sugar`, `sodium`, `protein`, `s_fat` (saturated fat), `carbs`, and concatenated into the subsetted dataset. 

Looking into each column, it can be noted that there seems to be some recipes where there are unrealistic amounts of nutritional content (i.e., over 30000% daily value in sugar alone):

<div class="table-wrapper" markdown="block">

| Metric       | ID         | Avg Rating | Calories  | Total Fat | Sugar  | Sodium   | Protein | Saturated Fat | Carbs   |
|--------------|------------|------------|-----------|-----------|--------|----------|---------|---------------|---------|
| **Count**    | 83782.00   | 81173.00   | 83782.00  | 83782.00  | 83782.00 | 83782.00 | 83782.00 | 83782.00      | 83782.00 |
| **Mean**     | 381430.41  | 4.63       | 429.93    | 32.63     | 68.66  | 28.94    | 33.13   | 40.24         | 13.79   |
| **Std**      | 68715.48   | 0.64       | 636.63    | 60.15     | 247.24 | 144.97   | 51.03   | 80.91         | 28.76   |
| **Min**      | 275022.00  | 1.00       | 0.00      | 0.00      | 0.00   | 0.00     | 0.00    | 0.00          | 0.00    |
| **25%**      | 321548.50  | 4.50       | 171.33    | 8.00      | 9.00   | 5.00     | 6.00    | 6.00          | 4.00    |
| **50%**      | 374472.50  | 5.00       | 305.40    | 20.00     | 23.00  | 14.00    | 18.00   | 21.00         | 9.00    |
| **75%**      | 436200.75  | 5.00       | 498.70    | 39.00     | 61.00  | 32.00    | 49.00   | 49.00         | 16.00   |
| **Max**      | 537716.00  | 5.00       | 45609.00  | 3464.00   | 30260.00 | 29338.00 | 4356.00 | 6875.00       | 3007.00 |


</div>

Examining a few sample recipes, there may be an issue with scaling. Therefore, we will treat "unrealistic" nutritional values as errors or outliers that may skew the general population, and cut out extreme recipes that exceed 1000% in daily value for any of the nutritional values.

The dataset that will be used for hypothesis testing is as follows:

`nutrition_reviews.head()`

<div class="table-wrapper" markdown="block">

| ID     | Average Rating | Calories | Total Fat | Sugar | Sodium | Protein | Saturated Fat | Carbs |
|--------|--------|----------|-----------|-------|--------|---------|---------------|-------|
| 333281 | 4.0    | 138.4    | 10.0      | 50.0  | 3.0    | 3.0     | 19.0          | 6.0   |
| 453467 | 5.0    | 595.1    | 46.0      | 211.0 | 22.0   | 13.0    | 51.0          | 26.0  |
| 306168 | 5.0    | 194.8    | 20.0      | 6.0   | 32.0   | 22.0    | 36.0          | 3.0   |
| 286009 | 5.0    | 878.3    | 63.0      | 326.0 | 13.0   | 20.0    | 123.0         | 39.0  |
| 475785 | 5.0    | 267.0    | 30.0      | 12.0  | 12.0   | 29.0    | 48.0          | 2.0   |

</div>
Note: as aforementioned, calories are in cals, while other nutritional values are listed in percent daily value (PDV)

<br>

### **Univariate Analysis**
First, we examine the distribution of average ratings through a histogram, rounded to the nearest whole number. Looking at the graph, it can be noted that the average rating seems heavily skewed towards favorable ratings. 

<iframe src="resources/univariate_1.html" width=800 height=600 frameBorder=0></iframe>

Next, we examine distributions of our main three nutritional contents.  


#### `calories`
Calories are most distributed around the 100-400 range, with the most frequent range being 165-174.9 calories. Even after extreme nutritional content was removed, the recipe with the highest calorie count seems to be over 10,000, which is 3 times the recommended average calorie intake amount.

<iframe src="resources/univariate_2.html" width=800 height=600 frameBorder=0></iframe>


#### `sugar`
Amount of sugar used in a recipe seems to skew heavily to the right, with a majority of recipes lying between 0-30% in percent daily value, and the highest frequency bin being between 0-4%. 

<iframe src="resources/univariate_3.html" width=800 height=600 frameBorder=0></iframe>


#### `t_fat`
Total fat is also skewed heavily to the right, with a huge tick in recipes listed as 0% daily value in fat.

<iframe src="resources/univariate_4.html" width=800 height=600 frameBorder=0></iframe>

<br>

### **Bivariate Analysis**
After seeing trends in individual columns, bivariate analysis is conducted in order to understand interactions between different variables in the data.  
In each of the plots below, the three nutritional values of interest were plotted against average ratings.


#### `calories`

<iframe src="resources/biv_1.html" width=800 height=600 frameBorder=0></iframe>


#### `sugar`

<iframe src="resources/biv_2.html" width=800 height=600 frameBorder=0></iframe>


#### `t_fat`

<iframe src="resources/biv_3.html" width=800 height=600 frameBorder=0></iframe>


### **Interesting Aggregates**

<br>

## **Assessment of Missingness**

___

### **NMAR** 
In the first modified DataFrame, `recipe_ratings`, there were three missing columns: `name`, `description`, and `average rating`. Of these, `description` is likely to be NMAR since users can choose to add a description to their recipes; recipes with more complicated descriptions may be more likely to be missing. This also means that it is likely that the missingness of `description` may be dependent on how complicated a recipe is.

### **Missingness Dependency: MCAR vs MAR** 
For missingness analysis, we focus on `average rating`. 7.08% of this column is missing values in the original interactions dataset, resulting in 3.1% of the average ratings being missing as well.  

<br>

#### Year of submission
Because older recipes have more time to be seen, newer recipes have a higher chance of not yet having ratings. Therefore, `average rating` is likely to be MAR based on `submitted`, or when the recipe was published on food.com. To analyze this, a new column, `year`, which takes only the year out of the submission date will be used for simplicity.

The hypotheses to analyze missingness of the `average rating` column on `year` are as follows:  
**Null hypothesis**: The missingness of `average rating` does not depend on any other columns.
**Alternate Hypothesis**: The missingness of `average rating` is dependent on `year` with a significance level of 0.05.

We begin by visualizing distributions of missingness of ratings by year. From the plot, it seems like the distributions are similar in shape, but newer ratings seem to be missing more often than not.

<iframe src="resources/missingness_bar.html" width=800 height=600 frameBorder=0></iframe>

By calculating the TVD for the proportions of missing vs not missing values per year, an observed TVD of 0.11249 was found. This is much larger than majority of the permutated TVDs, which mostly lay between 0.01 to 0.03.

<iframe src="resources/missingness_dist.html" width=800 height=600 frameBorder=0></iframe>

From this, we find that the p-value of 0.000 and less than the significance level of 0.05, so it can be assumed that the missingness of `average rating` is MAR on `submission`.  

<br>

#### Number of ingredients
Recipes with larger numbers of ingredients may overwhelm users, which in turn would drive traffic away from a recipe and leave less users for review. Therefore, it should be tested if `average rating` is MAR on `n_ingredients`.

The hypotheses to analyze missingness of the `average rating` column on `n_ingredients` are as follows:  
**Null hypothesis**: The missingness of `average rating` does not depend on any other columns.  
**Alternate Hypothesis**: The missingness of `average rating` is dependent on `n_ingredients` with a significance level of 0.05.

Similarly to `year`, through visualizing the distributions, the distributions of the missing ratings and non-missing ratings seem to be similar in shape but shifted; therefore, this test will also be conducted using TVD.

<iframe src="resources/missingness_bar_ing.html" width=800 height=600 frameBorder=0></iframe>

When comparing missing vs non-missing values based on number of ingredients, a TVD of 0.04020 was found. However, with a p-values of 0.086, this does not meet the significance level of 0.05 that was needed to reject the null hypothesis, so it is likely that `average rating` is MCAR on `n_ingredients`.

<iframe src="resources/missingness_dist_ing.html" width=800 height=600 frameBorder=0></iframe>


## **Hypothesis Testing**

___


### **Hypotheses**
##### Null Hypothesis: In the population, the mean rating is the same across groups of recipes with similar fat content, and any observed differences in our sample are due to random chance.


##### Alternative Hypothesis: In the population, the average rating differs across groups of recipes with similar fat contents.



### **Testing**

In this test, the difference in average rating is being compared across recipes with different total fat content. Therefore, a permutation test is used to observe differences in sample distributions, to see if the distributions of ratings come from the same population. The significance level is set at the standard 0.05, for a confidence level of 95%.

#### Test statistic: Difference in Means
Because the distributions are non-normal, difference in means is an appropriate metric to summarize overall variation in ratings across different total fat content levels. The observed maximum difference in means was 0.9792.


### **Results**
Upon running 10000 permutation tests, the p-value came out to be 0.1155. Since (0.1155 > 0.05), we fail to reject the null hypothesis.  

<iframe src="resources/hypothesis_test.html" width=800 height=600 frameBorder=0></iframe>

## **Framing a Prediction Problem**

___

### **Problem Identification**

This model focuses on predicting recipe ratings, as it may be helpful for users to be able to quickly identify if they would enjoy a recipe or not before trying it out.  

Given the heavily left-skewed distribution of ratings, where most average ratings fall between 4-5, and the goal of predicting a continuous variable `rating`, the most appropriate base model would be one that can handle regression with skewed data and non-linear relationships between features and `rating`.  
While ratings themselves are categorical, the target variable of average ratings is narrow but continuous. A classifier would only predict fixed classes, and would be likely to only predict 4 or 5 as it would reduce error the most. Therefore, a regressor would be more appropriate over a classifier.

In this project, the baseline model I will be using will be a **Random Forest Regressor**, which would help model complex relationships between different features, while handling skewness in feature and target variables well due to it not assuming normal distribution. 

The following features were used: 
- `minutes`
- `contributor_id`
- `submitted`
- `tags`
- `nutrition`
- `avg_rating`

The features were chosen based on how easy it is for users to come across this information even before reading the recipe.

## **Baseline Model**

___

The initial baseline model was a Random Forest Regressor.  
The qualitative variables used were:  
- `contributor_id` (nominal) - use One Hot Encoding
- `tags` (nominal) - explode tags into binary columns

The quantitative variables used were:  
- `minutes`
- `submitted` - take out only the year for simplicity
- `nutrition` - expand from list format into 7 individual columns 
- `avg_rating`

## **Final Model**

___


## **Fairness Analysis**

___
