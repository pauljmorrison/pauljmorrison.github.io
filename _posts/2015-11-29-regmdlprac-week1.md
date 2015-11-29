---
layout: post
title: "Coursera - Regression Modeling in Practice - Assignment 1"
date: 2015-11-29
---

# Sample

The sample of 378,540 Mars craters was drawn from high resolution imagery from the THermal EMission Imaging System (THEMIS) Daytime infrared (IR) imager between 2002 and 2011. The effective resolution of the images are 100 meters per pixel. The sample includes sizes and geographic position of craters as well as classification of their various ejecta.


# Procedure
The data is classified as observational data. The procedure to extract information from IR mosaic images made extensive use of *ArcGIS* software to manually identify and measure crater rims. The manual identification invlolved drawing a shape around each crater. These shapes were then fed into another software package, *Igor Pro* to fit circles using a non-linear least squares approximation. This procedure was repeated four times to ensure all visible craters were captured and characterized. 


# Measures

The circle fitting algorithm resulted in a dataset with three measures to characterize each crater: *x* position, *y* position, and radius of the crater. In addition to circular features, elliptical fits were also established using a similar procedure in *Igor Pro* but with the following parameters: semi-major axis, *a*; semi-minor axis, *b*; center longitude, *x_<sub>0</sub>*; center latitude, *y_<sub>0</sub>*, and major axis tilt, *theta*. Statistical measures such as standard deviation and goodness-of-fit are also provided for each observation.
