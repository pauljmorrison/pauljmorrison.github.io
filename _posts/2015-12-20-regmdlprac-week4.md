---
layout: post
title: "Coursera - Regression Modeling in Practice - Assignment 4"
date: 2015-12-20
---

# Test A Logistic Regression Model

### Assignment Description

Write a blog entry that summarize in a few sentences:

1. what you found, making sure you discuss the results for the associations between all of your explanatory variables and your response variable. Make sure to include statistical results (odds ratios, p-values, and 95% confidence intervals for the odds ratios) in your summary. 

2. Report whether or not your results supported your hypothesis for the association between your primary explanatory variable and your response variable. 

3. Discuss whether or not there was evidence of confounding for the association between your primary explanatory and the response variable (Hint: adding additional explanatory variables to your model one at a time will make it easier to identify which of the variables are confounding variables).



    %matplotlib inline
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import seaborn as sns
    import statsmodels.api as sm
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



To code the logistic variable, a new parameter is created to test the hypothesis of depth versus diameter relationship on the `SLERS` classification of `MORPHOLOGY_EJECTA_1`. This new variable will be coded one for the existence of `SLERS` and zero otherwise.


    def SLERS_c (row):
       if row['MORPHOLOGY_EJECTA_1']=='SLERS':
          return 1
       else:
          return 0
    data['SLERS_c'] = data.apply (lambda row: SLERS_c (row),axis=1)
    data['SLERS_c'].value_counts()




    0    229
    1     71
    dtype: int64





## Re-centering the variables
The quantitative response and explanatory variables are centered so that the mean = 0 by subtracting the mean of each.


    data['depth_centrd'] = data['DEPTH_RIMFLOOR_TOPOG'] - np.mean(data['DEPTH_RIMFLOOR_TOPOG'])
    data['diam_centrd'] = data['DIAM_CIRCLE_IMAGE'] - np.mean(data['DIAM_CIRCLE_IMAGE'])


    sns.regplot(x='depth_centrd',y='diam_centrd',scatter=True,data=data,label='Linear')
    sns.regplot(x='depth_centrd',y='diam_centrd',scatter=True,order=2,data=data,label='Quadratic')
    plt.legend()
    plt.xlabel('re-centered DEPTH_RIMFLOOR_TOPOG')
    plt.ylabel('re-centered DIAM_CIRCLE_IMAGE')








![png]({{ pauljmorrison.github.io }}/assets/C3W4_output_11_1.png)


To verify that the re-centering is successful, the mean values of the re-centered variables should be very close to zero.


    data[['SLERS_c','depth_centrd','diam_centrd']].describe()




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>SLERS_c</th>
      <th>depth_centrd</th>
      <th>diam_centrd</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>300.000000</td>
      <td>3.000000e+02</td>
      <td>3.000000e+02</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>0.236667</td>
      <td>-2.368476e-17</td>
      <td>-5.684342e-16</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.425746</td>
      <td>4.886308e-01</td>
      <td>1.430202e+01</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>-9.322667e-01</td>
      <td>-1.259943e+01</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.000000</td>
      <td>-2.622667e-01</td>
      <td>-8.004433e+00</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>0.000000</td>
      <td>-7.726667e-02</td>
      <td>-5.524433e+00</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>0.000000</td>
      <td>2.027333e-01</td>
      <td>2.893067e+00</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1.000000</td>
      <td>2.097733e+00</td>
      <td>9.535057e+01</td>
    </tr>
  </tbody>
</table>
</div>





## Logistic Regression Model
The logistic regression results for the response versus explanatory variable is accomplished using the python `statsmodels` package.


    lreg1 = smf.logit('SLERS_c ~ depth_centrd', data=data).fit()
    print (lreg1.summary())
    print ("Odds Ratios")
    print (np.exp(lreg1.params))

    Optimization terminated successfully.
             Current function value: 0.537077
             Iterations 6
                               Logit Regression Results                           
    ==============================================================================
    Dep. Variable:                SLERS_c   No. Observations:                  300
    Model:                          Logit   Df Residuals:                      298
    Method:                           MLE   Df Model:                            1
    Date:                Sun, 20 Dec 2015   Pseudo R-squ.:                 0.01851
    Time:                        17:40:34   Log-Likelihood:                -161.12
    converged:                       True   LL-Null:                       -164.16
                                            LLR p-value:                   0.01369
    ================================================================================
                       coef    std err          z      P>|z|      [95.0% Conf. Int.]
    --------------------------------------------------------------------------------
    Intercept       -1.2043      0.140     -8.609      0.000        -1.478    -0.930
    depth_centrd    -0.7356      0.311     -2.365      0.018        -1.345    -0.126
    ================================================================================
    Odds Ratios
    Intercept       0.299909
    depth_centrd    0.479231
    dtype: float64
    

It is clear from the logistic regression results that the odds ratio of depth for SLERS craters is 0.479. This ratio being less than 1 indicates that as depth increases the likelihood of the crater being classified as SLERS decreases.

The p-value for this depth variable is 0.018. This indicates it is statistically significant.

The 95% confidence intervals are provided below. We can be 95% certain that the odds ratio for SLERS crater classification as a function of depth is 0.260 to 0.881.


    params = lreg1.params
    conf = lreg1.conf_int()
    conf['OR'] = params
    conf.columns = ['Lower CI', 'Upper CI', 'OR']
    print (np.exp(conf))

                  Lower CI  Upper CI        OR
    Intercept     0.227988  0.394519  0.299909
    depth_centrd  0.260514  0.881575  0.479231
    



## Additional Explanatory Variables
The following analysis will include latitude, and longitude in the logistic regression to determine if these are significant contributors to crater diameter.


    lreg2 = smf.logit('SLERS_c ~ depth_centrd + LATITUDE_CIRCLE_IMAGE + LONGITUDE_CIRCLE_IMAGE', data=data).fit()
    print (lreg2.summary())
    print ("Odds Ratios")
    print (np.exp(lreg2.params))

    Optimization terminated successfully.
             Current function value: 0.536915
             Iterations 6
                               Logit Regression Results                           
    ==============================================================================
    Dep. Variable:                SLERS_c   No. Observations:                  300
    Model:                          Logit   Df Residuals:                      296
    Method:                           MLE   Df Model:                            3
    Date:                Sun, 20 Dec 2015   Pseudo R-squ.:                 0.01881
    Time:                        17:40:34   Log-Likelihood:                -161.07
    converged:                       True   LL-Null:                       -164.16
                                            LLR p-value:                    0.1034
    ==========================================================================================
                                 coef    std err          z      P>|z|      [95.0% Conf. Int.]
    ------------------------------------------------------------------------------------------
    Intercept                 -1.2086      0.149     -8.131      0.000        -1.500    -0.917
    depth_centrd              -0.7296      0.314     -2.321      0.020        -1.346    -0.114
    LATITUDE_CIRCLE_IMAGE      0.0008      0.005      0.174      0.862        -0.009     0.010
    LONGITUDE_CIRCLE_IMAGE    -0.0005      0.002     -0.280      0.780        -0.004     0.003
    ==========================================================================================
    Odds Ratios
    Intercept                 0.298604
    depth_centrd              0.482092
    LATITUDE_CIRCLE_IMAGE     1.000839
    LONGITUDE_CIRCLE_IMAGE    0.999528
    dtype: float64
    

It is clear from the additional logistic regression results that depth is still significant (OR=0.482, p-value=0.02) and an indicator of SLERS classification. However, latitude and longitude. Both of these explanatory variables have Odds Ratios very near 1 (1.00 and 0.999 respectively) and p-values greater than 0.05 (0.862 and 0.780 respectively).



## Hypothesis Testing & Confounding
My results supported the hypothesis for the association between my primary explanatory response variable. Crater depth is a good indicator of SLERS classification.

There was no evidence of confounding for the association between my primary explanatory and response variable because the logistic regression odds ratio did not change appreciably with the addition of more explanatory variables.
