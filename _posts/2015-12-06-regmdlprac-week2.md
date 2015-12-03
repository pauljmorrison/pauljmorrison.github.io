---
layout: post
title: "Coursera - Regression Modeling in Practice - Assignment 2"
date: 2015-12-06
---

# Test A Basic Linear Regression Model


    %matplotlib inline
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import seaborn as sns
    import statsmodels.api
    import statsmodels.formula.api as smf


    data = pd.read_csv('marscrater_pds.csv',low_memory=False, na_values=' ')
    
    #upper-case all DataFrame column names
    data.columns = map(str.upper,data.columns)

## Data Preparation for this Assignment
To prepare the data for this assignment, the dataset is restricted to to contain only named craters with primary and secondary ejecta classifications.


    data = data[~pd.isnull(data['CRATER_NAME'])]
    data = data[~pd.isnull(data['MORPHOLOGY_EJECTA_1'])]
    data = data[~pd.isnull(data['MORPHOLOGY_EJECTA_2'])]
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
    <tr>
      <th>204</th>
      <td>01-000204</td>
      <td>Lonar</td>
      <td>72.992</td>
      <td>38.304</td>
      <td>10.89</td>
      <td>1.10</td>
      <td>Rd/MLEPC/MLERS/MLERS/MLERS</td>
      <td>HuSL/HuBL/HuBL/HuSp</td>
      <td>NaN</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




    data.shape




    (300, 10)




    sns.regplot(x='DEPTH_RIMFLOOR_TOPOG',y='DIAM_CIRCLE_IMAGE',scatter=True,data=data)
    plt.xlabel('DEPTH_RIMFLOOR_TOPOG')
    plt.ylabel('DIAM_CIRCLE_IMAGE')




    <matplotlib.text.Text at 0x15472e48>




![Original]({{ pauljmorrison.github.io }}/assets/C3W2_output_6_1.png)


## Re-centering the variables
The quantitative response and explanatory variables are centered so that the mean = 0 by subtracting the mean of each.


    data['expl_centrd'] = data['DEPTH_RIMFLOOR_TOPOG'] - np.mean(data['DEPTH_RIMFLOOR_TOPOG'])
    data['resp_centrd'] = data['DIAM_CIRCLE_IMAGE'] - np.mean(data['DIAM_CIRCLE_IMAGE'])


    sns.regplot(x='expl_centrd',y='resp_centrd',scatter=True,data=data)
    plt.xlabel('re-centered DEPTH_RIMFLOOR_TOPOG')
    plt.ylabel('re-centered DIAM_CIRCLE_IMAGE')




    <matplotlib.text.Text at 0x5bbe9b0>




![Re-Centered]({{ pauljmorrison.github.io }}/assets/C3W2_output_9_1.png)


To verify that the re-centering is successful, the mean values of the re-centered variables should be very close to zero.


    print(np.mean(data['expl_centrd']))
    print(np.mean(data['resp_centrd']))

    -2.36847578587e-17
    -5.68434188608e-16
    

## Linear Regression Model
The linear regression results for the response versus explanatory variable is accomplished using the python statsmodels package.


    reg1 = smf.ols('resp_centrd ~ expl_centrd', data=data).fit()
    print (reg1.summary())

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:            resp_centrd   R-squared:                       0.648
    Model:                            OLS   Adj. R-squared:                  0.647
    Method:                 Least Squares   F-statistic:                     548.8
    Date:                Wed, 02 Dec 2015   Prob (F-statistic):           1.49e-69
    Time:                        23:07:50   Log-Likelihood:                -1066.6
    No. Observations:                 300   AIC:                             2137.
    Df Residuals:                     298   BIC:                             2145.
    Df Model:                           1                                         
    Covariance Type:            nonrobust                                         
    ===============================================================================
                      coef    std err          t      P>|t|      [95.0% Conf. Int.]
    -------------------------------------------------------------------------------
    Intercept   -7.216e-16      0.491  -1.47e-15      1.000        -0.966     0.966
    expl_centrd    23.5635      1.006     23.427      0.000        21.584    25.543
    ==============================================================================
    Omnibus:                      130.408   Durbin-Watson:                   1.345
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):              623.533
    Skew:                           1.778   Prob(JB):                    4.00e-136
    Kurtosis:                       9.102   Cond. No.                         2.05
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    

The $R^2$ is 0.648 indicating a strong correlation between the explanatory and response variable. The Intercept coefficient is zero which is expected since the response variable was re-centered on zero. The explanatory variable has a coefficient of 23 and a p-value of 0.000 which indicates this coefficient is statistically significant. The results of linear regression indicate that crater rim diameter is postively and significantly correlated with crater depth.


    
