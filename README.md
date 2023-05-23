# ML Project for "Pump it Up: Data Mining the Water Table" (competition hosted by DrivenData.org) 

First of all, a great shout-out to the other co-owner of this project, github.com/zhrgns ! We put our effort to this analysis collectively.

## Problem Description

This is a multi-class classification problem regarding water pumps in Tanzania, where the goal is to classify water pumps' status either as: "functional", "needs repair", or "nonfunctional". The data is provided by the Tanzanian Ministry of Water. If a good statistical model is built, pump maintenance can be improved, which would lead to better access to water.

![](RackMultipart20230523-1-jin9lm_html_86e043b01d91e024.png)

**Figure 1:** Distribution of water pump status in the training dataset.

## Exploratory Data Analysis, Preprocessing

We started by exploring the dataset and performing basic data cleaning and preprocessing steps. The training data set contains 59,400 observations with the labels and the test set contains 14,850 observations (without the labels).

In training data set there were several number of numerical columns containing high number of zero values. So, we decided to handle zero values using categorical location-based data columns like "subvillage", "lga" and ward etc. These location columns had high cardinality, therefore after imputation of numerical location-based data columns we dropped some of the categorical locations columns because of two main reasons; (1) categorical data is required to be encoded into multiple binary columns thus would hinder the performance of the model (2) having a numerical longitude/latitude data would yield the same result therefore, would cause redundancy.

### Non-Categorical Features

#### Longitude, Latitude and GPS Height

![](RackMultipart20230523-1-jin9lm_html_759e8e64440133ce.png) ![](RackMultipart20230523-1-jin9lm_html_d859eee8bb7137c9.png)

**Figure 2 & 3:** Histograms GPS Height and Longitude in the training dataset.

In longitude and gps\_height columns there were high number 0 values. For Tanzania, these geographical values can not be 0. Whereas in latitude column, there were data points lying outside of Tanzania, on the ocean. Thus, zero and improper latitude values are replaced by NaN and then imputed by using mean value of each column for each unique grouping of 'basin', 'region', 'lga', 'ward', 'subvillage' variables.

![](RackMultipart20230523-1-jin9lm_html_5df425736f15e494.png)After value imputation, gps\_height plot shows that in lower heights of Tanzania the ratio of non functional pumps are higher than higher ratios. The functionality between 1000 and 2000 is better.

**Figure 4:** Histogram of imputed GPS Height in training dataset, grouped by pump status.

![](RackMultipart20230523-1-jin9lm_html_e2b4b044a7bdab38.png)

**Mapping Coordinates**

Creating a scatter plot with longitude and latitude, we can see that the number of functional water pumps is higher in some areas, while in some areas it is less, in some areas there are no records at all. The regions, latitude, longitude and gps height is in relation with pump functionality.

**Figure 5:** Scatter plot of imputed longitude and latitude in training dataset, grouped by pump status.

#### Construction Year

To handle missing values in this column, computed probability distribution of construction year and replaced missing values with random samples drawn from the normalized probability distribution and categorized all values to decades at the end.

![](RackMultipart20230523-1-jin9lm_html_894a886c593a884f.png)Not surprisingly, the older pumps have a higher ratio in non-functioning pumps compared to the newer water pumps. Besides, more than half of the water pump are constructed last 20 years.

**Figure 6:** Bar chart of construction year in training dataset, grouped by pump status.

#### Population

In population, several observations had missing values. Similar to the method stated for longitude-latitude-gps\_height above, missing values are replaced using the mean value of each column for each unique combination of 'basin', 'region', 'lga', 'ward', 'subvillage' variables.

### Categorical Features

#### Quantity Group

![](RackMultipart20230523-1-jin9lm_html_4b90cfa9ec40c1d5.png)The distribution of classes among variables are relatively even, except dry and unknown water pumps are largely non-functional.

**Figure 7:** Bar chart of quantity group in training dataset, grouped by pump status.

#### Quality Group

![](RackMultipart20230523-1-jin9lm_html_65eb01fd8106b9fc.png)Over 50k of waterpumps have good quality of water. However, people cannot reach this quality type due to half of these are non-functional waterpumps. Unlike the others, the rate of non-functional waterpumps in unknown variable is very high.

**Figure 8:** Bar chart of quality group in training dataset, grouped by pump status.

#### Waterpoint Type

![](RackMultipart20230523-1-jin9lm_html_766936ddfec44771.png)According to waterpoint types of water pump, other class have too much more non-functional pumps than functional. Almost half of waterpoint\_type is communal standpipe and more than half of this type is functional.

**Figure 9:** Bar chart of waterpoint type in training dataset, grouped by pump status.

#### Source Class

The distribution of classes among all variables are relatively ev ![](RackMultipart20230523-1-jin9lm_html_aa32bc5cd2e52757.png)en. But, nearly 50k of water pumps use groundwater.

**Figure 10:** Bar chart of source class in training dataset, grouped by pump status.

#### Extraction Type

![](RackMultipart20230523-1-jin9lm_html_3306e2b1a3a2dd0c.png)According to extraction types of water pump, other and mono class may not well maintained, and they have much more non-functional pumps than functional. Almost half of extraction class is gravity and these are mostly functional waterpumps.

**Figure 11:** Bar chart of extraction type in training dataset, grouped by pump status.

#### Recorded Date

![](RackMultipart20230523-1-jin9lm_html_442050d4a9302e6b.png)We extracted record month from date\_recorded column to see season effect. And, the ratio of functional water pumps in spring season records is slightly higher than other months. It may be a proof of season effect in records. To observe this in the analysis, we separated the date data into three numerical columns of DD-MM-YYYY.

**Figure 12:** Bar chart of recordMonth in training dataset, grouped by pump status.

#### Payment

W ![](RackMultipart20230523-1-jin9lm_html_db490e7ee11b2ee4.png)
 e converted payment data to binary where never pay is 0, and other cases as 1 for ease of analysis. The ratio of functional water pumps with payment is higher than no payment.

**Figure 13:** Bar chart of payment in training dataset, grouped by pump status.

#### Permit, Public meeting, Scheme Management

_Permit, Public Meeting, Scheme\_Management_ _columns seems to have 5-6% missing. "__scheme\_management"_ , "_permit"_, "public meeting" were highly orrelated with _management_ with a few exceptions. Therefore, they are grouped by management and filled by respective group's mode. At the end, permit and public meeting are binarized and false columns were dropped.

![](RackMultipart20230523-1-jin9lm_html_8d30e633d7c28434.png) ![](RackMultipart20230523-1-jin9lm_html_ebf1bb97e4929c3e.png)Scheme management and public meeting columns have imbalanced distribution.

**Figure 14 & 15:** Bar charts of scheme\_management (left) and public meeting (right) in training dataset, grouped by pump status.

## Feature Selection

Some of the columns stated below are dropped for different reasons:

- _Subvillage_ _column is only 0.6% missing. But it is a categoric varibale with high cardinality. Longitude, Latitude and GPS Height columns were imputed by the help of this and other categoric location based columns (__'subvillage', 'ward', 'region', 'basin', 'lga'..)_ and also a high cardinality column at the end would yield numerous encoded columns therefore this and other categoric location columns _(__'subvillage', 'ward', 'region', 'basin', 'lga'..)_were removed.
- _'recorded\_by'_ only has 1 unique value, would have no effect on the model and therefore removed.
- Some columns had very high number of unique values (high cardinality) and would hinder model consistency_: 'funder', 'installer', 'lga', 'scheme\_name', 'subvillage', 'ward', 'wpt\_name'._
- _#Amount\__tsh is 70% empty, _num\_private_ 98.7% empty therefore will be removed.
- _Construction\_year_ will be dropped since it is divided into seperate columns.
- _Extraction\_type_ and _extraction \_type\_group_ are omitted since same as _extraction\_type\_class_.
- _Similar to above, WaterQuality/Quality Group , Quantity/Quantity group, Management/Management Group, Region/RegionCode, Source/SourceType/SourceClass, WaterpointType/Group, Payment/PaymentType_ were columns stating the same detail for observations and were repeating themselves. Thus, these repetitions are dropped.
- _Scheme\_name__dropped considering that 47% missing data._

![](RackMultipart20230523-1-jin9lm_html_6d2575ca9b471046.png)

**Figure 16:** Chart of columns in training dataset having missing values, prior to imputation.

At the end we have columns of :

'id', 'gps\_height', 'longitude', 'latitude', 'region\_code',

'district\_code', 'population', 'public\_meeting', 'scheme\_management',

'permit', 'extraction\_type', 'payment', 'quality\_group',

'quantity\_group', 'source\_class', 'waterpoint\_type'

'recordMonth'.

### Dummifying categorical variables:

At the end of the preprocessing, we dummified features that can be used in models due to dataset contains categorical variables. Binary columns' false dummies were dropped.

## Model Assessment

After engineering our features, we have trained and evaluated various classification models to see which one performs best on the data. Trained models are based on; K-Nearest Neighbors (i.e. KNN) model, Logistic Regression, Random Forest, Random Forest with Grid-Search, and Gradient Boosting Classifier.

Accuracy of each model is stated below:

- Random Forest Classifier = 0.80
- XGBoost = 0.78
- Gradient Boosting Classifier = 0.74
- Logistic Regression = 0.71
- KNN = 0.52

### KNN

For KNN, we had to select a K hyperparameter, which governs the number of points to be taken under consideration for a given observation in the model.

Charting the error rate for K values in between range 1 and 10, it was observed that K=8 had the minimum error rate thus K=8 is selected.

![](RackMultipart20230523-1-jin9lm_html_9aeb379135ba5a90.png)

**Figure 17:** Error Rate per K values in KNN model trials.

Upon analysis of KNN model, confusion matrix provided the results below. 0.51 test set accuracy was quite low, therefore we had to search for alternative model types.

![](RackMultipart20230523-1-jin9lm_html_67d14cf35e7c104e.png)

**Figure 18:** Classification report for KNN model based on K=8.

### Logistic Regression

As our second model, we have used a multiclass logistic regression model. We used a multiclass model because the regular "logit" model under statsmodels was not supporting multiple classifiers in the target column (in this case, status\_group of the pumps).

![](RackMultipart20230523-1-jin9lm_html_ca326149c7a301a5.png)

**Figure 19:** Classification report for Logistic Regression model.

As seen in the figure above, LogisticRegression model has achieved a 71% accuracy. While this was pretty accurate, we were curious about exploring tree-based statistical models.

### Gradient Boosting Classifier

Gradient Boosting is a known model type in classification and regression alike. We have decided to use the Gradient Boosting Classifier model in a hope to increase accuracy of the model.

![](RackMultipart20230523-1-jin9lm_html_ca54bb378f76ea13.png)

**Figure 20:** Confusion Matrix and classification report for Gradient Boosting model.

As it is seen from the figure above, we have achieved an accuracy of 74%, ahead of Logistic Regression model. While this is quite good, we had to use XGBoost, an improved version of Gradient Boosting model, to see whether it would provide better accuracy over the data.

### XGBoost

XGBoost (eXtreme Gradient Boosting) is an improved version of the Gradient Boosting model using GB with decision trees and is superior in terms of speed, accuracy, data handling and so on. In this project, we expected an accuracy score above the Gradient Boosting model.

![](RackMultipart20230523-1-jin9lm_html_10f9e47c4dcfbd1e.png)

**Figure 21:** Confusion Matrix and classification report for XGBoost model.

As a result, we have achieved an accuracy of 78%, much higher than Gradient Boosting model. As hyperparameters, we got our score based on n\_estimators=100, reg\_alpha=1 for L1 and reg\_lambda=0.01 for L2 regularization.

### Random Forest Classifier

As the 5th and final model, we decided to opt for a Random Forest Classifier model. We have trained a Random Forest Classifier model having maximum depth of the trees in the forest to 20 and used it to make predictions on the test data. In this case, our output showed the accuracy of the test set as 80%, achieving our best test accuracy score at all.

![](RackMultipart20230523-1-jin9lm_html_b5171cec2355052c.png)

**Figure 22:** Classification report of Random Forest Classifier model.

## Conclusion

To sum up, we carefully explored and preprocessed the data, engineered informative features, and experimented with different classification models to see which one works best for this problem. Our best model, Random Forest Classifier, predicted the functionality of the pumps with an accuracy of 80%.

![](RackMultipart20230523-1-jin9lm_html_5e159c51a2096d81.png)

**Figure 23:** Submission Score of Drivendata for results of our Random Forest Classifier model.
