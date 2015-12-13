---
layout: post
title: "Coursera - Regression Modeling in Practice - Assignment 3"
date: 2015-12-13
---

# Test A Multiple Regression Model

### Assignment Description

Summarize what you found. Discuss the results for the associations between all of your explanatory variables and your response variable. Make sure to include statistical results (Beta coefficients and p-values) in your summary.

Report whether or not your results supported your hypothesis for the association between your primary explanatory response variable.

Discuss whether or not there was evidence of confounding for the association between your primary explanatory and response variable.

Generate regression diagnostic plots and write a few sentences describing what these plots tell you about your regression model in terms of the distribution of the residuals, model fit, influential observations, and outliers.
1. q-q plot
2. standardized residuals for all observations
3. leverage plot

Include your multiple regression output in your blog.


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





## Data Exploration
Initial exploratory plots can be made using the `regplot` function in the `seaborn` package. The regression plots default to linear regression but polynomial regression can be accomplished by passing the addtional `order` argument where the numeric value of `order` is the order of the regression polynomial.


    sns.regplot(x='DEPTH_RIMFLOOR_TOPOG',y='DIAM_CIRCLE_IMAGE',scatter=True,data=data,label='Linear')
    sns.regplot(x='DEPTH_RIMFLOOR_TOPOG',y='DIAM_CIRCLE_IMAGE',order=2,scatter=True,data=data, label='Quadratic')
    plt.legend()
    plt.xlabel('DEPTH_RIMFLOOR_TOPOG')
    plt.ylabel('DIAM_CIRCLE_IMAGE')








![png]({{ pauljmorrison.github.io }}/assets/C3W3_output_8_1.png)




## Re-centering the variables
The quantitative response and explanatory variables are centered so that the mean = 0 by subtracting the mean of each.


    data['depth_centrd'] = data['DEPTH_RIMFLOOR_TOPOG'] - np.mean(data['DEPTH_RIMFLOOR_TOPOG'])
    data['diam_centrd'] = data['DIAM_CIRCLE_IMAGE'] - np.mean(data['DIAM_CIRCLE_IMAGE'])


    sns.regplot(x='depth_centrd',y='diam_centrd',scatter=True,data=data,label='Linear')
    sns.regplot(x='depth_centrd',y='diam_centrd',scatter=True,order=2,data=data,label='Quadratic')
    plt.xlabel('re-centered DEPTH_RIMFLOOR_TOPOG')
    plt.ylabel('re-centered DIAM_CIRCLE_IMAGE')








![png]({{ pauljmorrison.github.io }}/assets/C3W3_output_11_1.png)


To verify that the re-centering is successful, the mean values of the re-centered variables should be very close to zero.


    data[['depth_centrd','diam_centrd']].describe()




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>depth_centrd</th>
      <th>diam_centrd</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>3.000000e+02</td>
      <td>3.000000e+02</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>-2.368476e-17</td>
      <td>-5.684342e-16</td>
    </tr>
    <tr>
      <th>std</th>
      <td>4.886308e-01</td>
      <td>1.430202e+01</td>
    </tr>
    <tr>
      <th>min</th>
      <td>-9.322667e-01</td>
      <td>-1.259943e+01</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>-2.622667e-01</td>
      <td>-8.004433e+00</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>-7.726667e-02</td>
      <td>-5.524433e+00</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.027333e-01</td>
      <td>2.893067e+00</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2.097733e+00</td>
      <td>9.535057e+01</td>
    </tr>
  </tbody>
</table>
</div>





## Linear Regression Model
The linear regression results for the response versus explanatory variable is accomplished using the python `statsmodels` package.


    reg1 = smf.ols('diam_centrd ~ depth_centrd', data=data).fit()
    print (reg1.summary())

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:            diam_centrd   R-squared:                       0.648
    Model:                            OLS   Adj. R-squared:                  0.647
    Method:                 Least Squares   F-statistic:                     548.8
    Date:                Sun, 13 Dec 2015   Prob (F-statistic):           1.49e-69
    Time:                        16:05:41   Log-Likelihood:                -1066.6
    No. Observations:                 300   AIC:                             2137.
    Df Residuals:                     298   BIC:                             2145.
    Df Model:                           1                                         
    Covariance Type:            nonrobust                                         
    ================================================================================
                       coef    std err          t      P>|t|      [95.0% Conf. Int.]
    --------------------------------------------------------------------------------
    Intercept    -7.216e-16      0.491  -1.47e-15      1.000        -0.966     0.966
    depth_centrd    23.5635      1.006     23.427      0.000        21.584    25.543
    ==============================================================================
    Omnibus:                      130.408   Durbin-Watson:                   1.345
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):              623.533
    Skew:                           1.778   Prob(JB):                    4.00e-136
    Kurtosis:                       9.102   Cond. No.                         2.05
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    

It is clear from the linear regression results that diameter is significant and positively correlated with crater depth (P-value = 0.000, Beta = 23.56).



##Polynomial Regression Model
The polynomial regression results are accomplished in a similar method to the linear model with the addition of a quadratic term


    reg2 = smf.ols('diam_centrd ~ depth_centrd + I(depth_centrd**2)', data=data).fit()
    print (reg2.summary())

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:            diam_centrd   R-squared:                       0.770
    Model:                            OLS   Adj. R-squared:                  0.768
    Method:                 Least Squares   F-statistic:                     496.4
    Date:                Sun, 13 Dec 2015   Prob (F-statistic):           1.96e-95
    Time:                        16:05:41   Log-Likelihood:                -1003.0
    No. Observations:                 300   AIC:                             2012.
    Df Residuals:                     297   BIC:                             2023.
    Df Model:                           2                                         
    Covariance Type:            nonrobust                                         
    ========================================================================================
                               coef    std err          t      P>|t|      [95.0% Conf. Int.]
    ----------------------------------------------------------------------------------------
    Intercept               -2.9794      0.463     -6.431      0.000        -3.891    -2.068
    depth_centrd            18.4107      0.913     20.166      0.000        16.614    20.207
    I(depth_centrd ** 2)    12.5203      1.000     12.524      0.000        10.553    14.488
    ==============================================================================
    Omnibus:                      115.533   Durbin-Watson:                   1.436
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):              704.057
    Skew:                           1.446   Prob(JB):                    1.31e-153
    Kurtosis:                       9.926   Cond. No.                         3.07
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    

It is clear from the quadratic regression results that depth and depth squared are both significant and positively correlated (depth: P-value = 0.000, Beta = 18.41; depth squared: P-value = 0.000, Beta = 12.52; ).



## Additional Explanatory Variables
The following analysis will include latitude, longitude, and a cubic fit on depth in the regression to determine if these are significant contributors to crater diameter.


    reg3 = smf.ols('diam_centrd ~ depth_centrd + I(depth_centrd**2) + I(depth_centrd**3) + LATITUDE_CIRCLE_IMAGE + LONGITUDE_CIRCLE_IMAGE', data=data).fit()
    print (reg3.summary())

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:            diam_centrd   R-squared:                       0.776
    Model:                            OLS   Adj. R-squared:                  0.772
    Method:                 Least Squares   F-statistic:                     203.3
    Date:                Sun, 13 Dec 2015   Prob (F-statistic):           3.56e-93
    Time:                        16:05:41   Log-Likelihood:                -999.11
    No. Observations:                 300   AIC:                             2010.
    Df Residuals:                     294   BIC:                             2032.
    Df Model:                           5                                         
    Covariance Type:            nonrobust                                         
    ==========================================================================================
                                 coef    std err          t      P>|t|      [95.0% Conf. Int.]
    ------------------------------------------------------------------------------------------
    Intercept                 -3.3356      0.492     -6.782      0.000        -4.304    -2.368
    depth_centrd              19.2368      1.162     16.557      0.000        16.950    21.523
    I(depth_centrd ** 2)      13.0273      1.507      8.647      0.000        10.062    15.992
    I(depth_centrd ** 3)      -0.7414      1.096     -0.676      0.499        -2.899     1.416
    LATITUDE_CIRCLE_IMAGE      0.0354      0.014      2.511      0.013         0.008     0.063
    LONGITUDE_CIRCLE_IMAGE    -0.0057      0.005     -1.163      0.246        -0.015     0.004
    ==============================================================================
    Omnibus:                      102.869   Durbin-Watson:                   1.493
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):              584.133
    Skew:                           1.282   Prob(JB):                    1.44e-127
    Kurtosis:                       9.337   Cond. No.                         392.
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    

It is clear from the multiple regression results that depth and depth squared are still significant and positively correlated (depth: P-value = 0.000, Beta = 19.23; depth squared: P-value = 0.000, Beta = 13.03; ). However, the cubic term on depth is not significant (P-value = 0.499, Beta = -0.7414). The Latitude term is found to be significant (P-value = 0.013, Beta = 0.0354) but weakly correlated. The longitude term is not correlated (P-value = 0.246, Beta = -0.0057).



## Hypothesis Testing & Confounding
My results supported the hypothesis for the association between my primary explanatory response variable. Crater diameter is positively correlated with crater depth. The best fit is found to be a quadratic polynomial with order 2.

There was no evidence of confounding for the association between my primary explanatory and response variable because the regression coefficients for the best fit (2nd order polynomial) did not change appreciably with the addition of more explanatory variables.



## Normalized Residual Plot vs Observation Number
This normalized residual plot uncovers any missing trends in the fit. If this plot is not evenly distributed, it indicates an additional explanatory variable is needed.


    stdres=pd.DataFrame(reg3.resid_pearson)
    plt.plot(stdres, 'o', ls='None')
    l = plt.axhline(y=0, color='g')
    plt.ylabel('Standardized Residual')
    plt.xlabel('Observation Number')








![png]({{ pauljmorrison.github.io }}/assets/C3W3_output_25_1.png)


The normalized residuals appear centered around zero and have the expected distribution with very few outliers beyond the three sigma boundary. This indicates our regression is successful and captures the explanatory and response relationship.



## Q-Q Plot to Test for Residual Normality
The Q-Q plot is commonly used to visually assess the fit residuals for normality. Significant deviations from the diagonal line indicate non-Normality.


    fig4=sm.qqplot(reg3.resid, line='r')


![png]({{ pauljmorrison.github.io }}/assets/C3W3_output_28_0.png)


The departure of the fit residuals from the extremes of the red line indicate the tails of the residuals are not perfectly normal. This can be explained by the extreme variability in the dataset and uncertainty in the depth and diameter measurements themselves. 



## Leverage Plot


    fig3=sm.graphics.influence_plot(reg3, size=8)


![png]({{ pauljmorrison.github.io }}/assets/C3W3_output_31_0.png)


The leverage plot shows any outliers and their relative impact on the regression coefficients. Any highlighted points should be closely inspected in the original dataset. For this regression, point 80196 is not an outlier (y-axis) but does have a significant impact on the regression (x-axis).
