# A Machine Learning case-study on taxi demand prediction in NYC
<h2> Business Objective: </h2>
<p> Given a region and a particular time interval, predict the no of pickups as accurately as possible in that region and nearby regions.
 <br><b>Note: </b> There is no strict latency requirement and interpretability is also not very important for this problem. The driver just cares about the percentage errors in prediction.
<h2> Data Collection: </h2>
<p>
We get our data from : http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml .The data used in the attached datasets were collected and provided to the NYC Taxi and Limousine Commission (TLC). For simplicity, we are considering only the yellow taxis' data for the time period between Jan - Mar 2015 & Jan - Mar 2016 
</p>
<h2> Mapping to ML problem: Data</h2>
<p>Each row in the input csv file corresponds to a pickup and the data has many features(vendorID, pickupDateTime, dropoffDateTime, passengerCount, tripDistance, pickupLatitude, pickupLongitude, dropoffLatitude, dropoffLongitude, fareAmount, surcharge etc). As the dataset is in very large csv files, it becomes difficult to handle with pandas. So we use the purpose-built special library - <b>Dask</b>
</p>
<h2> Mapping to ML problem: Performance Metrices </h2>
<p>The problem here is a special type of regression : Time-series prediction/forecasting. Taking into consideration the problem type and the business constraints, we boil down to the following performance metrices:
<ol><b>MAPE</b>(Mean Absolute Percentage Error)</ol>
<ol><b>MSE</b>(Mean Squared Error)</ol>
</p>
<h2>Data Cleaning:</h2>
<p>Now we perform univariate analysis and remove outliers or illegitimate values(which may be caused due to some error)<br><br>
  
<b>Pickup/Drop-off Latitude and Longitude :</b> It is inferred from the source https://www.flickr.com/places/info/2459115 that New York   is bounded by the location coordinates(lat,long) - (40.5774, -74.15) & (40.9176,-73.7004). So, any coordinates not within these             coordinates are not considered by us as we are only concerned with pickups which originate within New York.<br>

<b>Trip Durations: </b> According to NYC Taxi & Limousine Commision Regulations the maximum allowed trip duration in a 24 hour interval is 12 hours. So we remove the points where the duration is >12 hr<br>

<b>Speed: </b> We found that the 99.9th percentile value of speed is 45.31 mph. So, we consider only the data points where the computed speed is <45.31mph. We also observed that the avg speed in New York is 12.45miles/hr, i.e. <b>a cab driver can travel 2 miles per 10min on avg</b><br>

<b>Trip Distance: </b>The 99.9th percentile value of the distance covered in a ride is ~23 miles. So we remove rows with large trip distances<br>

<b> Fare:</b> From percentile and graphical analysis, we set the upper limit of total fare to be 1000$ and consider only the data points which satisfies the limit 
</p>

<h2> Data preparation: </h2>
<p>
<b>Clustering/Segmentaion: </b> Now we break whole of NYC into clusters/segments/regions. We choose the (lat,long) of pickup as features and we apply K-Means clustering algorithm(we find the right K using GridSearch). On choosing a cluster size of 40, we find an optimal min inter-cluster distance. Finally, we assign the cluster no. to each data point<br>
  
<b>Time-binning: </b> We use the unix time-stamps to find the 10 min time-bins each data point belongs to(index of the 10min time interval to which that data point belongs)<br>

<b> Smoothing time-series data: </b> In our time-series data plot, there will be some 10-min bins where there are no pickups. And its not useful to predict zero pickups for a data point and moreover these points will create a problem in Moving Averages(using ratios). Thus, we smooth our training data(2015)(as in smoothing we are looking at future values) and fill with zero our test data(2016)<br>

<b> Fourier Transform: </b> From the Fourier transform plot of our time-series data we find the top amplitudes and corresponding frequencies
</p> 

<h2> Modeling : Baseline Models</h2>
<p> Now we get into modelling in order to forecast the pickup densities for the months of Jan, Feb and March of 2016 for which we are using multiple models with two variations : Using Ratios of the 2016 data to the 2015 data and Using Previous known values of the 2016 data itself to predict the future values.<br>
 We used <b>Simple Moving Averages</b>, <b>Weighted Moving Averages</b> and <b>Exponential Weighted Moving Averages</b>
</p>
<h2> Modeling : Regression Models</h2>
<p>
<b>Train-test split: </b>Before we start predictions using the tree based regression models we take 3 months of 2016 pickup data and split it such that for every region we have 70% data in train and 30% in test, ordered date-wise for every region<br>
 
 <b>Final features: </b> We have the following features in our final train and test data : the last 5 no of pickups, lat and long of the cluster centre, exponential average, top amplitudes and frequencies obtained from FFT<br>
 
 <b> Models used: </b> We tried out Linear Regression, Random Forest Regressor and xgBoost Regressor. We used hyperparameter tuning to find the best hyperparams for each of the models and then finally computed the performance metrices in each case.<br>
 
</p>

