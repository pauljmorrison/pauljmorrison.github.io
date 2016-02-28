---
layout: post
title: "Coursera - Machine Learning for Data Analysis - Assignment 4"
date: 2016-02-27
---

# Running a Cluster Analysis

### Assignment Description

Run a k-means cluster analysis to identify subgroups of observations in your data set that have similar patterns of response on a set of clustering variables. 



    %matplotlib inline
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    from sklearn import preprocessing
    from sklearn.cross_validation import train_test_split
    from sklearn import preprocessing
    from sklearn.cluster import KMeans


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
This k-means cluster analysis assignment will try to predict the depth of craters based on their latitude, longitude, diameter, and number of layers. All variables are quantitative. The training/test split will be 70/30.


    cluster_pre = data[['LATITUDE_CIRCLE_IMAGE','LONGITUDE_CIRCLE_IMAGE','DIAM_CIRCLE_IMAGE','NUMBER_LAYERS','DEPTH_RIMFLOOR_TOPOG']]

The predictors must all be scaled to have a mean zero and variance one.


    cluster = pd.DataFrame()
    cluster['LATITUDE_CIRCLE_IMAGE'] =preprocessing.scale(cluster_pre['LATITUDE_CIRCLE_IMAGE'].astype('float64'))
    cluster['LONGITUDE_CIRCLE_IMAGE']=preprocessing.scale(cluster_pre['LONGITUDE_CIRCLE_IMAGE'].astype('float64'))
    cluster['DIAM_CIRCLE_IMAGE']     =preprocessing.scale(cluster_pre['DIAM_CIRCLE_IMAGE'].astype('float64'))
    cluster['NUMBER_LAYERS']         =preprocessing.scale(cluster_pre['NUMBER_LAYERS'].astype('float64'))
    cluster['DEPTH_RIMFLOOR_TOPOG']  =preprocessing.scale(cluster_pre['DEPTH_RIMFLOOR_TOPOG'].astype('float64'))


    clus_train, clus_test = train_test_split(cluster, test_size=.3, random_state=123)
    
    print(clus_train.shape)
    print(clus_test.shape)

    (414, 5)
    (178, 5)
    

# Cluster Sweep

Sweep through a list from 1 to 9 to determine the best number of clusters to choose. Plot the average distance of each observation from its cluster centriod (cluster variance) against the cluster number.


    from scipy.spatial.distance import cdist
    clusters=range(1,10)
    meandist=[]
    
    for k in clusters:
        model=KMeans(n_clusters=k)
        model.fit(clus_train)
        clusassign=model.predict(clus_train)
        meandist.append(sum(np.min(cdist(clus_train, model.cluster_centers_, 'euclidean'), axis=1)) 
        / clus_train.shape[0])


    plt.plot(clusters, meandist)
    plt.xlabel('Number of clusters')
    plt.ylabel('Average distance')
    plt.title('Selecting k with the Elbow Method')




    <matplotlib.text.Text at 0xbbc37b8>




![Out1]({{ pauljmorrison.github.io }}/assets/C4W4_output_14_1.png)


Based on the above plot, the "elbow" occurs at two clusters. The ideal number of clusters to choose is a bit subjective. The guidline is to plot the average distance of each observation from its cluster centriod (cluster variance) against the cluster number. This is called an Elbow Curve and the appropriate number of clusters is where the slope of this curve changes steeply.


    model2=KMeans(n_clusters=2)
    model2.fit(clus_train)
    clusassign=model2.predict(clus_train)
    # plot clusters
    
    from sklearn.decomposition import PCA
    pca_2 = PCA(2)
    plot_columns = pca_2.fit_transform(clus_train)
    plt.scatter(x=plot_columns[:,0], y=plot_columns[:,1], c=model2.labels_,)
    plt.xlabel('Canonical variable 1')
    plt.ylabel('Canonical variable 2')
    plt.title('Scatterplot of Canonical Variables for 2 Clusters')
    plt.show()


![Out1]({{ pauljmorrison.github.io }}/assets/C4W4_output_16_0.png)



    clus_train.reset_index(level=0, inplace=True)


    assign_df = pd.DataFrame({'clusassign':clusassign})
    assign_df.shape




    (414, 1)




    clus_new = pd.concat([clus_train,assign_df],axis=1)
    clus_new.shape




    (414, 7)




    clus_new.clusassign.value_counts()




    1    299
    0    115
    dtype: int64




    clustergrp = clus_new.groupby('clusassign').mean()
    print ("Clustering variable means by cluster")
    print(clustergrp)

    Clustering variable means by cluster
                     index  LATITUDE_CIRCLE_IMAGE  LONGITUDE_CIRCLE_IMAGE  \
    clusassign                                                              
    0           414.121739              -0.769496                0.137616   
    1           247.792642               0.312659                0.001760   
    
                DIAM_CIRCLE_IMAGE  NUMBER_LAYERS  DEPTH_RIMFLOOR_TOPOG  
    clusassign                                                          
    0                    1.274889      -0.762010              1.038952  
    1                   -0.514324       0.253961             -0.426337  
    

The mean analysis indicates the clusters divide most strongly with diameter. The larger diameter craters are in cluster zero and smaller diameter craters are in cluster one. Consistent with previous analyses, the depth is strongly associated with diameter. The deeper craters are in the same cluster as the larger diameter craters. Interestingly, the latitude also shows a correlation with the larger craters clustering in the more southerly latitudes. Larger diameter circles also cluster with a fewer number of layers.
