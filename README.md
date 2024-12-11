# predicting_recipe_popularity

## **Introduction**

___

### **Abstract and Data**  
Nutritional information plays a key role in how people choose recipes; by examining nutritional values, such as calorie count, fat content, or protein levels, individuals can better understand how a recipe fits into their personal needs. Understanding preferences purely from nutritional data can empower people to make more informed choices, helping them identify recipes that suit their goals without needing to rely solely on trial and error. This project will focus in particular on understanding how users may unknowingly rate recipes based on nutritional content.  
  
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

![review screenshot](./img/food.com_review.png)

Additionally, pandas implicitly excludes `NaN` values from mean calculation, making it ideal for making sure the general shape of the data is correct, even if the true evaluation of a review was not present.

`recipe_reviews.head()`

<div class="table-wrapper" markdown="block">
| Name                           | ID     | Minutes | Contributor ID | Submitted   | Tags                                  | Nutrition                     | Steps | Steps Description                        | Description                        | Ingredients                              | Ingredients Count | Rating |
|--------------------------------|---------|----------|----------------|-------------|---------------------------------------|--------------------------------|-------|------------------------------------------|------------------------------------|------------------------------------------|-------------------|--------|
| 1 brownies in th...            | 333281  | 40       | 985201         | 2008-10-27  | [60-minutes-or-less, ...]             | [138.4, 10.0, 50...]          | 10    | [heat the oven to...]                   | these are the mo...               | [bittersweet chocolate, ...]            | 9                 | 4.0    |
| 1 in canada choc...            | 453467  | 45       | 1848091        | 2011-04-11  | [60-minutes-or-less, ...]             | [595.1, 46.0, 21...]          | 12    | [pre-heat oven t...]                    | this is the reci...               | [white sugar, brown sugar, ...]         | 11                | 5.0    |
| 412 broccoli cas...            | 306168  | 40       | 50969          | 2008-05-30  | [60-minutes-or-less, ...]             | [194.8, 20.0, 6...]           | 6     | [preheat oven to...]                    | since there are ...               | [frozen broccoli, ...]                  | 9                 | 5.0    |
| millionaire poun...            | 286009  | 120      | 461724         | 2008-02-12  | [time-to-make, cookies-and-brownies]  | [878.3, 63.0, 32...]          | 7     | [freheat the ove...]                    | why a millionair...               | [butter, sugar, ...]                    | 7                 | 5.0    |
| 2000 meatloaf                  | 475785  | 90       | 2202916        | 2012-03-06  | [time-to-make, comfort-food, ...]     | [267.0, 30.0, 12...]          | 17    | [pan fry bacon ,...]                    | ready, set, cook...               | [meatloaf mixture, ...]                 | 13                | 5.0    |

</div>

### **Univariate Analysis**



### **Bivariate Analysis**



## **Assessment of Missingness**

___

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


## **Baseline Model**

___

## **Final Model**

___


## **Fairness Analysis**

___
