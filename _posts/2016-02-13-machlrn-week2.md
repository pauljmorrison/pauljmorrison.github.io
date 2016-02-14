---
layout: post
title: "Coursera - Machine Learning for Data Analysis - Assignment 2"
date: 2016-02-13
---

# Running a Random Forest

### Assignment Description

Write a few sentences summarizing a random forest analysis to test nonlinear relationships among a series of explanatory variables and a binary, categorical response variable.




    %matplotlib inline
    import pandas as pd
    import numpy as np
    import subprocess
    from sklearn.cross_validation import train_test_split
    from sklearn.tree import DecisionTreeClassifier
    from sklearn.metrics import classification_report
    import sklearn.metrics
    
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.ensemble import ExtraTreesClassifier


    data = pd.read_csv('marscrater_pds.csv',low_memory=False, na_values=' ')
    #upper-case all DataFrame column names
    data.columns = map(str.upper,data.columns)



## Data Preparation for this Assignment
To prepare the data for this assignment, the dataset is restricted to to contain only named craters with only primary ejecta classifications.


    data = data[~pd.isnull(data['CRATER_NAME'])]
    data = data[~pd.isnull(data['MORPHOLOGY_EJECTA_1'])]
    data.head()




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CRATER_ID</th>
      <th>CRATER_NAME</th>
      <th>LATITUDE_CIRCLE_IMAGE</th>
      <th>LONGITUDE_CIRCLE_IMAGE</th>
      <th>DIAM_CIRCLE_IMAGE</th>
      <th>DEPTH_RIMFLOOR_TOPOG</th>
      <th>MORPHOLOGY_EJECTA_1</th>
      <th>MORPHOLOGY_EJECTA_2</th>
      <th>MORPHOLOGY_EJECTA_3</th>
      <th>NUMBER_LAYERS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>01-000001</td>
      <td>Korolev</td>
      <td>72.760</td>
      <td>164.464</td>
      <td>82.02</td>
      <td>1.97</td>
      <td>Rd/MLERS</td>
      <td>HuBL</td>
      <td>NaN</td>
      <td>3</td>
    </tr>
    <tr>
      <th>12</th>
      <td>01-000012</td>
      <td>Dokka</td>
      <td>77.170</td>
      <td>-145.681</td>
      <td>51.08</td>
      <td>1.74</td>
      <td>Rd</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>01-000028</td>
      <td>Louth</td>
      <td>70.173</td>
      <td>103.226</td>
      <td>36.28</td>
      <td>1.41</td>
      <td>SLERS</td>
      <td>HuBL</td>
      <td>NaN</td>
      <td>1</td>
    </tr>
    <tr>
      <th>68</th>
      <td>01-000068</td>
      <td>Escorial</td>
      <td>76.887</td>
      <td>-54.969</td>
      <td>22.11</td>
      <td>0.92</td>
      <td>SLEPd</td>
      <td>HuBL</td>
      <td>NaN</td>
      <td>1</td>
    </tr>
    <tr>
      <th>85</th>
      <td>01-000085</td>
      <td>Inuvik</td>
      <td>78.595</td>
      <td>-28.276</td>
      <td>20.02</td>
      <td>0.78</td>
      <td>SLERS</td>
      <td>SmAm</td>
      <td>NaN</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




    data.shape




    (592, 10)



To code the binary variable, a new parameter `SLERS_c` is created for the `SLERS` classification of `MORPHOLOGY_EJECTA_1`. This new variable will be coded one for the existence of `SLERS` and zero otherwise.


    def SLERS_c (row):
       if row['MORPHOLOGY_EJECTA_1']=='SLERS':
          return 1
       else:
          return 0
    data['SLERS_c'] = data.apply (lambda row: SLERS_c (row),axis=1)
    data['SLERS_c'].value_counts()




    0    520
    1     72
    dtype: int64



## Grouping the predictors and target variables


    predictors = data[['LATITUDE_CIRCLE_IMAGE','LONGITUDE_CIRCLE_IMAGE','DIAM_CIRCLE_IMAGE','DEPTH_RIMFLOOR_TOPOG']]
    target = data['SLERS_c']


    pred_train, pred_test, tar_train, tar_test  =   train_test_split(predictors, target, test_size=.4)
    
    print(pred_train.shape)
    print(pred_test.shape)
    print(tar_train.shape)
    print(tar_test.shape)

    (355, 4)
    (237, 4)
    (355,)
    (237,)
    

## Build model on training data


    classifier=RandomForestClassifier(n_estimators=25)
    classifier=classifier.fit(pred_train,tar_train)


    predictions=classifier.predict(pred_test)


    print(sklearn.metrics.confusion_matrix(tar_test,predictions))
    print(sklearn.metrics.accuracy_score(tar_test, predictions))

    [[201   8]
     [ 22   6]]
    0.873417721519
    

## Fit an Extra Trees model to the data


    model = ExtraTreesClassifier()
    model.fit(pred_train,tar_train)
    # display the relative importance of each attribute
    print(model.feature_importances_)

    [ 0.30623441  0.19718923  0.244072    0.25250436]
    

When using a random forest classfier to predict the SLERS characterization of crater ejecta morphology, an accuracy score of about 85% is obtained using the predictors latitude, longitude, crater diameter, and crater depth. The confusion matrix indicates that 202 observations are correctly classified and 35 are incorrectly classified. Based on the importance feature vector, both diameter and depth are important, each with about 25.9 scores. Latitude is most important with a 26.8 score and longitude is least important at a 21.4 importance score. Its interesting that these all have relatively equal importance and no factor stands out significantly from the others. 
