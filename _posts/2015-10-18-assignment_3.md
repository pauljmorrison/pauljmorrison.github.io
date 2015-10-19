---
layout: post
title: "Coursera - Data Management and Visualization - Assignment 3"
date: 2015-10-18
---
To efficiently code out missing data, the Python pandas `read_csv` function 
has the ability to specify missing values with the `na_values` option. To filter 
missing data from the Mars Crater dataset, set `na_values` equal to an empty
space.

    data = pd.read_csv('marscrater_pds.csv',low_memory=False, na_values=' ')
    
    #upper-case all DataFrame column names
    data.columns = map(str.upper,data.columns)

Since the NA values were specified at import, the `value_counts` function has 
an option to drop these values for the reported summary or to keep them using
the `dropna=True` or `dropna=False` settings. This is applied to the following
three variables: 

* `MORPHOLOGY_EJECTA_1`
* `MORPHOLOGY_EJECTA_2`
* `MORPHOLOGY_EJECTA_3`

These variables characterize the ejecta of the various impact craters. These 
variables each contain missing data and the number of missing datapoints for 
each variable is:

|       Variable        | Number Missing |
|-----------------------|----------------|
| `MORPHOLOGY_EJECTA_1` |    339718      |
| `MORPHOLOGY_EJECTA_2` |    364867      |
| `MORPHOLOGY_EJECTA_3` |    383050      |

`MORPHOLOGY_EJECTA_3` has the most missing variables. These missing content 
was coded out using the above procedure. The filtered frequency distributions
are found below (limited to top 5):
 
    morph_data=data.copy()
    print('Top 5 counts for MORPHOLOGY_EJECTA_1')
    c1 = data['MORPHOLOGY_EJECTA_1'].value_counts(dropna=False)[:5]
    print(c1)
    
    print('Top 5 normalized counts for MORPHOLOGY_EJECTA_1')
    n1 = data['MORPHOLOGY_EJECTA_1'].value_counts(dropna=False,normalize=True)[:5]
    print(n1)
    
    print('Top 5 counts for MORPHOLOGY_EJECTA_1 with NA removed')
    c11 = data['MORPHOLOGY_EJECTA_1'].value_counts(dropna=True)[:5]
    print(c11)

| `MORPHOLOGY_EJECTA_1` | Frequency |
|-----------------------|-----------|
|       Rd              |   24892   |
|       SLEPS           |    4949   |
|       SLERS           |    4828   |
|       SLEPC           |    2552   |
|       SLERC           |    1216   |

    print('Top 5 counts for MORPHOLOGY_EJECTA_2')
    c2 = data['MORPHOLOGY_EJECTA_2'].value_counts(dropna=False)[:5]
    print(c2)
    
    print('Top 5 normalized counts for MORPHOLOGY_EJECTA_2')
    n2 = data['MORPHOLOGY_EJECTA_2'].value_counts(dropna=False,normalize=True)[:5]
    print(n2)
    
    print('Top 5 counts for MORPHOLOGY_EJECTA_2 with NA removed')
    c21 = data['MORPHOLOGY_EJECTA_2'].value_counts(dropna=True)[:5]
    print(c21)

| `MORPHOLOGY_EJECTA_2` | Frequency |
|-----------------------|-----------|
|       HuSL            |    6109   |
|       HuBL            |    4424   |
|       SmSL            |    2713   |
|       HuAm            |    1356   |
|       Hu              |    1307   |

    print('Top 5 counts for MORPHOLOGY_EJECTA_3')
    c3 = data['MORPHOLOGY_EJECTA_3'].value_counts(dropna=False)[:5]
    print(c3)
    
    print('Top 5 normalized counts for MORPHOLOGY_EJECTA_3')
    n3 = data['MORPHOLOGY_EJECTA_3'].value_counts(dropna=False,normalize=True)[:5]
    print(n3)
    
    print('Top 5 counts for MORPHOLOGY_EJECTA_3 with NA removed')
    c31 = data['MORPHOLOGY_EJECTA_3'].value_counts(dropna=True)[:5]
    print(c31)

| `MORPHOLOGY_EJECTA_3` | Frequency |
|-----------------------|-----------|
| Pin-Cushion           |     349   |
| Small-Crown           |     264   |
| Pseudo-Butterfly      |     118   |
| Inner is Pin-Cushion  |      87   |
| Butterfly             |      76   |

    latlong_data = data[['LATITUDE_CIRCLE_IMAGE','LONGITUDE_CIRCLE_IMAGE']].copy()

Next, the latitude and longitude positions of these craters can be summarized.
It is effective to summarize these into bins of 20 degrees each. This binning
is accomplished using the Pandas package `cut` command.

    print('Binned Summary of LATITUDE_CIRCLE_IMAGE')
    c5_bins=[-100, -80, -60, -40, -20, 0, 20, 40, 60, 80, 100]
    latlong_data['LAT_CIR_IM_BIN'] = pd.cut(latlong_data['LATITUDE_CIRCLE_IMAGE'], c5_bins)
    c5 = latlong_data['LAT_CIR_IM_BIN'].value_counts(sort=False, dropna=True)
    print(c5)

|    Bin       |  Count |
|--------------|--------|
| (-100, -80]  |    631 |
| (-80, -60]   |  20511 |
| (-60, -40]   |  44154 |
| (-40, -20]   |  81081 |
| (-20, 0]     |  87079 |
| (0, 20]      |  62773 |
| (20, 40]     |  52355 |
| (40, 60]     |  24961 |
| (60, 80]     |  10754 |
| (80, 100]    |     44 |


    print('Binned Summary of LONGITUDE_CIRCLE_IMAGE')
    c6_bins=[-180, -140, -100, -60, -20, 20, 60, 100, 140, 180]
    latlong_data['LON_CIR_IM_BIN'] = pd.cut(latlong_data['LONGITUDE_CIRCLE_IMAGE'], c6_bins)
    c6 = latlong_data['LON_CIR_IM_BIN'].value_counts(sort=False, dropna=True)
    print(c6)

|    Bin       |  Count |
|--------------|--------|
| (-180, -140] |  35825 |
| (-140, -100] |  23468 |
| (-100, -60]  |  35521 |
| (-60, -20]   |  49500 |
| (-20, 20]    |  58256 |
| (20, 60]     |  52463 |
| (60, 100]    |  46155 |
| (100, 140]   |  43678 |
| (140, 180]   |  39477 |

In summary, the craters seem distributed heavily in the northern latitudes
and more evenly distributed in the longitude parameter but somewhat clustered
in the central longitudes.

    plt.figure()
    latlong_data['LATITUDE_CIRCLE_IMAGE'].plot(kind='hist',bins=c5_bins)
    plt.xlabel('Latitude [deg]')
    plt.ylabel('Count')

![Latitude]({{ pauljmorrison.github.io }}/assets/week3_latitude.png)

    plt.figure()
    latlong_data['LONGITUDE_CIRCLE_IMAGE'].plot(kind='hist',bins=c6_bins)
    plt.xlabel('Longitude [deg]')
    plt.ylabel('Count')

    plt.show()

![Longitude]({{ pauljmorrison.github.io }}/assets/week3_longitude.png)