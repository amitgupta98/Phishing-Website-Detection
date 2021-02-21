## Deploying on Local Machine
##
![image](https://user-images.githubusercontent.com/56480620/102687956-4984fd00-4219-11eb-9d74-b825df49dc1a.png)

## Phishing-Website-Detection
##
![sec1674-fig-0004-m](https://user-images.githubusercontent.com/56480620/108626076-579f0600-7474-11eb-9c13-d586c4774091.jpg)

## 1. Business Problem

## 1.1 Data Description

The client will send data in multiple sets of files in batches at a given location. Data will contain Wafer names and 590 columns of different sensor values for each wafer. The last column will have the "Good/Bad" value for each wafer.
"Good/Bad" column will have two unique values +1 and -1.  
"+1" represents Bad wafer.
"-1" represents Good Wafer. 
Apart from training files, we also require a "schema" file from the client, which contains all the relevant information about the training files such as:
Name of the files, Length of Date value in FileName, Length of Time value in FileName, Number of Columns, Name of the Columns, and their datatype. 

## 1.2 Problem Statement

To build a classification methodology to predict whether a website is a phising website on the basis of given set of predictors.  

## 1.3 Real world/Business Objectives

We need to clasify based on different parameters and features.

## 1.4 Machine Learning Formulation
Here we suppose to classify chip, so it is basically **Classification** problem.

## 1.5 Perfomance Metric
We will be using confusion matrix as results. So it is **Classification** problem reducing **CM**.

## 2 Data Validation 
In this step, we perform different sets of validation on the given set of training files.  
1.	 Name Validation- We validate the name of the files based on the given name in the schema file. We have created a regex pattern as per the name given in the schema file to use for validation. After validating the pattern in the name, we check for the length of date in the file name as well as the length of time in the file name. If all the values are as per requirement, we move such files to "Good_Data_Folder" else we move such files to "Bad_Data_Folder."

2.	 Number of Columns - We validate the number of columns present in the files, and if it doesn't match with the value given in the schema file, then the file is moved to "Bad_Data_Folder."


3.	 Name of Columns - The name of the columns is validated and should be the same as given in the schema file. If not, then the file is moved to "Bad_Data_Folder".

4.	 The datatype of columns - The datatype of columns is given in the schema file. This is validated when we insert the files into Database. If the datatype is wrong, then the file is moved to "Bad_Data_Folder".


5.	Null values in columns - If any of the columns in a file have all the values as NULL or missing, we discard such a file and move it to "Bad_Data_Folder".

## Data Insertion in Database
1) Database Creation and connection - Create a database with the given name passed. If the database is already created, open the connection to the database. 
2) Table creation in the database - Table with name - "Good_Data", is created in the database for inserting the files in the "Good_Data_Folder" based on given column names and datatype in the schema file. If the table is already present, then the new table is not created and new files are inserted in the already present table as we want training to be done on new as well as old training files.     
3) Insertion of files in the table - All the files in the "Good_Data_Folder" are inserted in the above-created table. If any file has invalid data type in any of the columns, the file is not loaded in the table and is moved to "Bad_Data_Folder".

## Model Training
1) Data Export from Db - The data in a stored database is exported as a CSV file to be used for model training.
2) Data Preprocessing   
   a) Check for null values in the columns. If present, impute the null values using the KNN imputer.
   b) Check if any column has zero standard deviation, remove such columns as they don't give any information during model training.
3) Clustering - KMeans algorithm is used to create clusters in the preprocessed data. The optimum number of clusters is selected by plotting the elbow plot, and for the dynamic selection of the number of clusters, we are using "KneeLocator" function. The idea behind clustering is to implement different algorithms
   To train data in different clusters. The Kmeans model is trained over preprocessed data and the model is saved for further use in prediction.
4) Model Selection - After clusters are created, we find the best model for each cluster. We are using two algorithms, "Random Forest" and "XGBoost". For each cluster, both the algorithms are passed with the best parameters derived from GridSearch. We calculate the AUC scores for both models and select the model with the best score. Similarly, the model is selected for each cluster. All the models for every cluster are saved for use in prediction.

## Prediction
1) Data Export from Db - The data in the stored database is exported as a CSV file to be used for prediction.
2) Data Preprocessing   
   a) Drop columns not useful for training the model. Such columns were selected while doing the EDA.
   b) Replace the invalid values with numpy “nan” so remove them.
   d) Scale the training data.
   e) convert categorical to numerical 
3) Clustering - KMeans model created during training is loaded, and clusters for the preprocessed prediction data is predicted.
4) Prediction - Based on the cluster number, the respective model is loaded and is used to predict the data for that cluster.
5) Once the prediction is made for all the clusters, the predictions along with the original names before label encoder are saved in a CSV file at a given location and the location is returned to the client.


## Deployment
-We will be deploying the model to the Pivotal Web Services Platform. 

-This is a workflow diagram for the prediction of using the trained model.                  
