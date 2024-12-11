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


## **Data Cleaning and Exploratory Data Analysis**

___

### **Data Cleaning**

There were a few columns with missing values:
- `name` - 1 missing value
- `description` - 70 missing values
- `review` - 169 missing values

Since this project does not require `name` and `description` information (recipes are labelled by id, and nutritional information can be found outside of the description), these columns will be dropped to prevent confusion. 

In order to find sentiment regarding a recipe, the average rating for each recipe is found through left merging the interactions dataset into the recipes dataset, to ensure all reviews being used are correlated to a currently existing recipe. All 0 values were replaced with `NaN`; this is because the lowest star rating that a recipe can be given is 1, implying missing or erroneous data rather than a true evaluation.

![food.com review screenshot](./resource/food.com_review.png)

Additionally, pandas implicitly excludes `NaN` values from mean calculation, making it ideal for making sure the general shape of the data is correct, even if the true evaluation of a review was not present.

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

The dataset that will be used through this project is as follows:

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

### **Univariate Analysis**
First, we examine the distribution of average ratings through a histogram, rounded to the nearest whole number. Looking at the graph, it can be noted that the average rating seems heavily skewed towards favorable ratings. 

<iframe src="resources/univariate_1.html" width=800 height=600 frameBorder=0></iframe>

Next, we examine distributions of our main three nutritional contents.  
#### `calories`

<iframe src="resources/univariate_2.html" width=800 height=600 frameBorder=0></iframe>

#### `sugar`

<iframe src="resources/univariate_3.html" width=800 height=600 frameBorder=0></iframe>

#### `t_fat`

<iframe src="resources/univariate_4.html" width=800 height=600 frameBorder=0></iframe>

### **Bivariate Analysis**


### **Interesting Aggregates**


## **Assessment of Missingness**

___

### **NMAR** 


### **Missingness Dependency: MCAR vs MAR** 

#### **Review**
The hypotheses to analyze missingness of the `review` column are as follows:  
**Null hypothesis**: The missingness of `review` does not depend on any other columns.
**Alternate Hypothesis**: The missingness of `review` is dependent on `rating` with a significance level of 0.05.

## **Hypothesis Testing**

___


### **Hypotheses**
#### Null Hypothesis:


#### Alternative Hypothesis:



### **Testing**



### **Results**



## **Framing a Prediction Problem**

___

### **Problem Identification**


## **Baseline Model**

___

## **Final Model**

___


## **Fairness Analysis**

___
