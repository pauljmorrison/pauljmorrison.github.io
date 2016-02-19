---
layout: post
title: "Coursera - Machine Learning for Data Analysis - Assignment 3"
date: 2016-02-18
---

# Running a Lasso Regression

### Assignment Description

Run a lasso regression analysis using k-fold cross validation to identify a subset of predictors from a larger pool of predictor variables that best predicts a quantitative response variable.



    %matplotlib inline
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    from sklearn import preprocessing
    from sklearn.cross_validation import train_test_split
    from sklearn.linear_model import LassoLarsCV
    from sklearn.metrics import mean_squared_error


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



## Grouping the predictors and target variables
This lasso regression assignment will try to predict the depth of craters based on their latitude, longitude, diameter, and number of layers. All variables are quantitative. The training/test split will be 70/30.


    predictors_pre = data[['LATITUDE_CIRCLE_IMAGE','LONGITUDE_CIRCLE_IMAGE','DIAM_CIRCLE_IMAGE','NUMBER_LAYERS']]
    target = data['DEPTH_RIMFLOOR_TOPOG']

The predictors must all be scaled to have a mean zero and variance one.


    predictors = pd.DataFrame()
    predictors['LATITUDE_CIRCLE_IMAGE'] =preprocessing.scale(predictors_pre['LATITUDE_CIRCLE_IMAGE'].astype('float64'))
    predictors['LONGITUDE_CIRCLE_IMAGE']=preprocessing.scale(predictors_pre['LONGITUDE_CIRCLE_IMAGE'].astype('float64'))
    predictors['DIAM_CIRCLE_IMAGE']     =preprocessing.scale(predictors_pre['DIAM_CIRCLE_IMAGE'].astype('float64'))
    predictors['NUMBER_LAYERS']         =preprocessing.scale(predictors_pre['NUMBER_LAYERS'].astype('float64'))


    pred_train, pred_test, tar_train, tar_test  =   train_test_split(predictors, target, test_size=.3, random_state=123)
    
    print(pred_train.shape)
    print(pred_test.shape)
    print(tar_train.shape)
    print(tar_test.shape)

    (414, 4)
    (178, 4)
    (414,)
    (178,)
    

## Build the lasso model on training data
The `cv=10` argument to the `LassoLarsCV` function will allow a 10-fold cross validation on the regression.


    model=LassoLarsCV(cv=10, precompute=False).fit(pred_train,tar_train)

Print variable names and regression coefficients


    print(dict(zip(predictors.columns, model.coef_)))

    {'LONGITUDE_CIRCLE_IMAGE': -0.020387228747521323, 'NUMBER_LAYERS': 0.1344094164850248, 'LATITUDE_CIRCLE_IMAGE': -0.026424720461917156, 'DIAM_CIRCLE_IMAGE': 0.51084633325067863}
    

#Plot regression coefficients


    m_log_alphas = -np.log10(model.alphas_)
    ax = plt.gca()
    plt.plot(m_log_alphas, model.coef_path_.T)
    plt.axvline(-np.log10(model.alpha_), linestyle='--', color='k',
                label='alpha CV')
    plt.ylabel('Regression Coefficients')
    plt.xlabel('-log(alpha)')
    plt.title('Regression Coefficients Progression for Lasso Paths')



![Out1]({{ pauljmorrison.github.io }}/assets/C4W3_output_17_1.png)


    m_log_alphascv = -np.log10(model.cv_alphas_)
    plt.figure()
    plt.plot(m_log_alphascv, model.cv_mse_path_, ':')
    plt.plot(m_log_alphascv, model.cv_mse_path_.mean(axis=-1), 'k',
             label='Average across the folds', linewidth=2)
    plt.axvline(-np.log10(model.alpha_), linestyle='--', color='k',
                label='alpha CV')
    plt.legend()
    plt.xlabel('-log(alpha)')
    plt.ylabel('Mean squared error')
    plt.title('Mean squared error on each fold')





![Out2]({{ pauljmorrison.github.io }}/assets/C4W3_output_18_1.png)



    train_error = mean_squared_error(tar_train, model.predict(pred_train))
    test_error = mean_squared_error(tar_test, model.predict(pred_test))
    print ('training data MSE')
    print(train_error)
    print ('test data MSE')
    print(test_error)

    training data MSE
    0.21526645345
    test data MSE
    0.18322312517
    


    rsquared_train=model.score(pred_train,tar_train)
    rsquared_test=model.score(pred_test,tar_test)
    print ('training data R-square')
    print(rsquared_train)
    print ('test data R-square')
    print(rsquared_test)

    training data R-square
    0.528096290839
    test data R-square
    0.568602498863
    

The fit results show the diameter is the strongest predictor of circle depth followed by the number of layers. Latitude and longitude have coefficients very close to zero meaning there is little to no dependence of depth on these predictors. These results are supported by the lasso path traces.

Finally, the mean squared error and R-square of the lasso training model are consistent across both the training and test data sets indicating a model that is not overfit.
