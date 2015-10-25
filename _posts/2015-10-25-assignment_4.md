---
layout: post
title: "Coursera - Data Management and Visualization - Assignment 4"
date: 2015-10-25
---

The objective of this assignment is to visualize various aspects of our chosen dataset. In this blog post, univariate and bivariate analyses support the conclusion that craters are approximately geographically uniformly distributed with the most common ejecta classification being 'Rd'.

# Categorical Variable Analysis:

To reasonably narrow the analysis, limit the dataset to the top 5 morphologies:
    
    MORPH_EJ_1_LIST = ['Rd','SLEPS','SLERS','SLEPC','SLERC'];
    MORPH_EJ_2_LIST = ['HuSL','HuBL','SmSL','HuAm','Hu'];

    MORPH_EJ_1_data = data[data['MORPHOLOGY_EJECTA_1'].isin(MORPH_EJ_1_LIST)]
    MORPH_EJ_2_data = data[data['MORPHOLOGY_EJECTA_2'].isin(MORPH_EJ_2_LIST)]

The following plots show univariate bar graphs for categorical variables. The first bar graph shows the primary crater morphology ejecta classification. This shows a strong skew toward 'Rd' categorized ejecta. The second bar graph is the secondary morphology ejecta classification and shows a maximum in the 'HuSL' category with the 'HuBL' category as second highest.

    plt.figure()
    seaborn.countplot(x='MORPHOLOGY_EJECTA_1', data=MORPH_EJ_1_data);
    plt.ylabel('Count')
    plt.xlabel('Primary Crater Ejecta Morphology')
    plt.title('Univariate Plot of MORPHOLOGY_EJECTA_1')
    seaborn.plt.show();

![MORPHOLOGY_EJECTA_1]({{ pauljmorrison.github.io }}/assets/week4_MORPH_EJ_1_bar.png)

    plt.figure()
    seaborn.countplot(x='MORPHOLOGY_EJECTA_2', data=MORPH_EJ_2_data);
    plt.ylabel('Count')
    plt.xlabel('Secondary Crater Ejecta Morphology')
    plt.title('Univariate Plot of MORPHOLOGY_EJECTA_2')
    seaborn.plt.show();

![MORPHOLOGY_EJECTA_2]({{ pauljmorrison.github.io }}/assets/week4_MORPH_EJ_2_bar.png)


# Quantitative Variable Analysis

    latlong_data = data[['LATITUDE_CIRCLE_IMAGE','LONGITUDE_CIRCLE_IMAGE']]

The following plots show univariate histograms for quantitative variables. The first histogram shows the crater latitude distribution. The graph is unimodal with the highest peak in the -20 to -30 degree range. The second histogram shows the crater longitude distribution. The graph is tri- or quad-modal with peaks in the -180, -50, -5, and +75 degree longitudes.

    plt.figure()
    seaborn.distplot(latlong_data['LATITUDE_CIRCLE_IMAGE'], kde=False);
    plt.ylabel('Count')
    plt.xlabel('Crater Latitude [degrees North]')
    plt.title('Crater Latitude in the Mars Crater Dataset')
    seaborn.plt.show();

![Latitude_hist]({{ pauljmorrison.github.io }}/assets/week4_Latitude_hist.png)

    plt.figure()
    seaborn.distplot(latlong_data['LONGITUDE_CIRCLE_IMAGE'], kde=False);
    plt.ylabel('Count')
    plt.xlabel('Crater Longitude [degrees East]')
    plt.title('Crater Longitude in the Mars Crater Dataset')
    seaborn.plt.show();

![Longitude_hist]({{ pauljmorrison.github.io }}/assets/week4_Longitude_hist.png)


# Bivariate Analysis

The full dataset is too large to be effectively visualized in a bivariate scatter plot. The `.sample()` method of a Pandas DataFrame randomly samples a specified fraction of the original dataset.

    biv_data = latlong_data.sample(frac=0.005)

The final plot shows a bivariate analysis of the 0.5% randomly sampled latitudes versus longitudes for the full crater dataset. The data shows a higher density of data just south of the equator. A large crater-less region in the -100 to -150 degree longitudes skews the longitudinal distribution to the east. These conclusions support the above conclusions drawn from the univariate histograms.

    plt.figure()
    seaborn.regplot(x='LONGITUDE_CIRCLE_IMAGE', y='LATITUDE_CIRCLE_IMAGE', fit_reg=False, data=biv_data)
    plt.ylabel('Crater Latitude [degrees North]')
    plt.ylim(-90,90)
    plt.xlabel('Crater Longitude [degrees East]')
    plt.xlim(-180,180)
    plt.title('Crater Latitude vs Longitude in the Mars Crater Dataset (0.5% Random Sample)')
    seaborn.plt.show();

![Lat-vs-Long_scatter]({{ pauljmorrison.github.io }}/assets/week4_Lat-vs-Long_scatter.png)


