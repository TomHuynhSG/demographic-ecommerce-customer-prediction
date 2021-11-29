# demographic-ecommerce-customer-prediction
Demographic prediction system based on the product viewing activities in hierarchical E-commerce data in **the Data Mining Competition of PAKDD’15 Conference.** FYI, PAKDD is The Pacific-Asia Conference on Knowledge Discovery and Data Mining.

Customers' demographic characteristics, such as gender, provide crucial information for e-commerce service providers in marketing and online application customisation. Indeed, one of the most successful ways to anticipate user demographic information is to look at their online actions, such as browsing activity or catalog browsing statistics. Because users must do something on the system, such as visit the sites, click the items, or browse the catalog, the key advantage of this technique is that the data is available in most applications.

The experiments were conducted on the datasets and obtained the promising results.

## Requirements 

* All the library requirements can be installed through requirements.txt.

With pip:
```
pip install -r requirements.txt
```

With conda:
```
conda create --name <your_env_name> --file requirements.txt
conda activate <your_env_name>
```

## Data

The datasets are provided by FPT Corporation in the framework of Data Mining Competition in PAKDD’ 15 Conference.

It contains 15,000 records which correspond to the product viewing logs.

A single log in the training data file is composed of four types of information:
- Session ID
- Start time (including date)
- End time (including date)
- List of product IDs

The IDs of the items that the user has browsed throughout the session are listed in the list of product IDs. Because the products may fall into various categories, the IDs also include information about the categories. From the most broad categories (starting with " A") to subcategories (starting with "B" and "C") to individual products (starting with " D"), each product ID can be divided into four separate IDs.

An example of a single log:

u10008, 2014-11-17 19:20:06, 2014-11-17 19:21:54, A00001/B00001/C00001/D00001/; A00001/B00002/C00002/D00002

Browsing categories data can be considered to be multi-level hierarchical tree structure of categories/products like this for the illustration purpose:

A00002/B00003/C00006/D19760/; A00002/B00001/C00010/D18416; A00002/B00001/C00004/D19764/;A00002/B00003/C00008/D19761/; A00002/B00003/C00008/D08538/


![](https://i.imgur.com/nCxYVf8.png)

Here is the frequency of categories appeared in the customer browsering:

Category Level | Frequency 
--- | --- 
Level A (Categories) | 11
Level B (Subcategories) | 86
Level C (Sub-subcategories) | 383
Level D (Product ID) | 21880

Here is the frequency of the labels:

Label | Frequency 
--- | --- 
Females | 11703
Males | 3297

As you can see, this data is unbalanced! In the notebook, I tried several methods to mitigate this!

## Temporal Feature Preprocessing

The temporal features are related timestamp and frequency of viewing actions, for example, the time of day, the day of the week, holidays, viewing duration, and the number of products viewed in a single session are all factors to consider.

With only Start time (date) and End time (date) features, we need to apply feature engineering and try to create more meaningful features like so.

Temporal Feature | Numberic Range 
--- | --- 
Day in month | 31
Month | 12
Day of Week | 7
Start Hour | 24
End Hour | 24
Duration | seconds
No Of Products | number
Average Time Per Product | seconds

For these time-based features "DayOfWeek", "Month", "Day", "StartHour" and "EndHour", it is not ideal to keep them as numerical categories.

Because it does not capture the cycle of time (day, week, hour). Also, converting them into one-hot encoding might cause the curse of dimentionality where the matrix is too parse and the model will have a hard time to learn any meaning relationships.

Therefore, we will encode the cyclic features (cyclical occurring features) using the basic formulation of trigonometry, by computing the sin and cosine of the features.

![](https://i.imgur.com/u020cN6.png)

![](https://i.imgur.com/j5BQ2CG.png)

This will ensure we create meaningful extra features capturing the cycle of time without creating too many empty columns using one-hot encoding.

## Metrics

The balanced accuracy metric was employed to assess the model due to the problem of class imbalance. Balanced accuracy is defined as an average accuracy acquired on either class, and it can help prevent inflated performance estimates on unbalanced datasets.

- Balanced Accuracy Measure (BAC): 

![BAC](https://i.imgur.com/UzbAYvx.png/)


## Model Experiments

Here is the grand summary of the above results of each model for Balanced Accuracy Measure (BAC) of the test set:

- Logistic Regression (Baseline Model): 0.487
- Naive Baynes: 0.512
- Deep Learning (basic dense layers): 0.782
- Decision Tree: 0.863
- XGBoost: 0.843
- Random Forest: 0.890

As the results, Random Forest is the best model so far with the best BAC value among the models I have tried so far!

Also, the ROC of random forest shows the nice balanced predictions between two classes and look very good compared to others.

![](https://i.imgur.com/wFEvi5O.png)

## Conclusion

After selectecting Random Forest model due to the highest BAC metric value compared to others, we have fine-tuned our model to find the best hyper-parameters.

So here is my best model so far:

Random Forest with these hyper-parameters:

- 'n_estimators': 100,
- 'min_samples_split': 2,
- 'min_samples_leaf': 1,
- 'max_depth': 400

Here are a few things can be improved:

- For unbalanced data, instead of oversampling or undersampling, I could look into Cost-sensitive learning which add different weights to misclassification cost to make up for the missing quantitiy.
- For category column, instead of breaking each sessions with many products into multiple rows with single product, I could figure out how to setup hierarchy tree with mutiple-levels and also combine bi-gram to make use of the sequences of previous product to current product and to next product.
- Since I selected Random Forest model as the best model, I can spend way more time to fine tune the model with grid-search and random-search to select the optimal parameters. Also, Using cross-validation will provide less-bias way to select the test set and evaluate optimal parameters. But due to the time constrait, I keep this process is simple enough to demontrate the points.