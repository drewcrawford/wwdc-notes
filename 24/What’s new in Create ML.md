Explore updates to Create ML, including interactive data source previews and a new template for building object tracking models for visionOS apps. We'll also cover important framework improvements, including new time-series forecasting and classification APIs.

# Overview
[[Explore machine learning on Apple platforms]] - overview of ML frameworks

* app
* framework
* components

output: model.  Deploy into your app via frameworks.
# App enhancements

Predict contents of images, videos, tabular data.  Object detection, etc.

Preview your datasource to verify that your annotations match your expectations.  Now can explore input data in createML app.

# Object tracking

ML for Apple Vision Pro.  

start with a 3D asset.  We generate all the training data for you.

[[Explore object tracking for visionOS]]


# Components

Signals changing over time
* accelerometer
* GPS location
* Temperature
* Sales

TimeSeriesClassifier.  Ex, pinch, snap, clench using accelerometer data.

More powerful, general-purpose time series classifier component.  

Forecasting.  Learns from historical data to predict future values over a period of time.  Forecaster is a general-purpose component, so you can use it to forecast anything.  Audio, accelerometer, sales, etc!

Tabular data framework - [[Explore and manipulate data in Swift with TabularData]]


Consider extracting the day of week.  Month of year.  

We've introduced a DateFeatureExtractor component to make it easy to extract features from a date.

Prediction window.  Your context should be longer than your prediction window.  

# Wrap up
* data source preview
* Object tracking
* Time series forecasting
[[Explore machine learning on Apple platforms]]
