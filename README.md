# flexible_row
Let's create the most flexible and versatile time series

The idea is this: to create a predictive system that will ask the User for data in the format of a time series and give him a value for one period into the future, as well as some familiarization characteristics of the series itself. The system should be as flexible as possible and ask the User for a minimum amount of additional data and settings.
To implement this task, first of all, it is necessary to determine the settlement and evaluation platform on which all this will work. The choice was initially among three options: generative statistics methods, machine learning algorithms or deep learning. Building a system based on generative statistics is quite time-consuming and resource-intensive, since it will be necessary to conduct volumetric studies of data of various kinds, quality and architecture. Deep learning and neural networks will require a huge amount of data and time to fully train. Machine learning algorithms, on the other hand, allow you to build a flexible and dynamic system on relatively small data (compared to deep learning) in optimal time.
Among other things, it is assumed that the system will receive the data on which to build a forecast at the time of their request from the User, and will not have access to them in advance. This means that the system will have to train on new data at the time of its work with the User's data, and not in advance, so the system will need to include an appropriate mechanism for additional training and quality control over it.
Next, you need to decide on the algorithm on which predictions will be based. After going through several options and models, it was decided to use a set of their two similar algorithms ARIMA and SARIMA. These algorithms were chosen because they are personally designed to work with time series and are very good in this area. Also, the SIGMA function was additionally created as an additional forecasting option (it will be discussed a little later).

How the system will work in general:
1. First, the User will be asked for data in the format of a temporary date. For specifics, in this case, the data is requested by a special request of the SQL from the DWH and they are preprocessed and prepared for immersion in the model. The training and test sets are defined by the ratio 80% / 20%.
2. Analysis of data for purity, noise, autocorrelation, seasonality and other things that will be further issued as general information about the series transmitted to the system.
3. The process of additional training of the system on a new row. Here the same mechanism will be implemented. Its goal is to find such a sample of training data in which the metrics for assessing the quality of trained algorithms produced the best results.
4. The process of forecasting one period into the future.
5. Upload results, introductory information and assessment metrics for study.

After creating the system, we will give a specific example of its operation based on a specific data set. As such, we use the real series of payments received, aggregated monthly by the total amount. The results of the system operation are shown below:
1. First, let's look at the structure of the series itself. We will display it on the chart, and in parallel with it we will display the moving average and the moving standard deviation:
![image (30)](https://github.com/Geronimik/flexible_row/assets/98644693/17fec869-b12a-4d1f-a578-a9f68c958d64)
In the chart below, the blue curve is in-kind payments (the scale is divided by 10^7)
                 green curve - moving average of payments for 12 months
                 orange curve - 12-month moving standard deviation
2. Next, we will study the series using decomposition: look at its noisiness, study seasonality, and so on. When decomposing, a multiplicative decomposition model was used:
First, let's look at the trend in order to get acquainted with the general trend of a series of payments, as well as to identify stationarity "approximately".
![image (31)](https://github.com/Geronimik/flexible_row/assets/98644693/42ee17c1-e779-4c49-a82a-781de72d1ebf)
From the trend line chart, a stable increase in the company's revenues until April-May 2018 is noticeable. Then the schedule goes to a plateau until 2020. This is due to the company's transition to a completely new work strategy and the consequences of this decision. Also, from April-May 2020 to April-May 2021, there is a steady stagnation of the chart. This is probably due to the consequences of the coronavirus pandemic: increased mortality, restriction of physical contact between people (which is very important in the work of the company), restrictions imposed on business by the state in connection with the rampant virus.
In general, purely intuitively, this series cannot be called stationary, but this should be confirmed by tests, which will be done further.
Next, let's look at the seasonality chart. It will be important to us when choosing a model, but now we will look at it purely informative:
![image (32)](https://github.com/Geronimik/flexible_row/assets/98644693/53ba57a6-3e28-4195-82d9-311723b38440)
Here seasonality is given for all years of the study, however, seasonality for any one year can be distinguished, since it is the same for each and, if necessary, consider it in detail.
Let's move on to studying the data for noise. This is a very important component of the series, since the noise, in a sense, determines the purity of the data and its suitability for training machine learning models.
![image (33)](https://github.com/Geronimik/flexible_row/assets/98644693/d18c41e3-f8a1-40b2-a4a4-652d807ffb08)
In general, the noise level of the row is more or less even, and the noise level is not high, except for some points. There are 3 peaks in 2017, 2019 and 2020, the highest being in 2017.
These peaks are well correlated in time with the peaks on the regular payment schedule. To identify the causes of these peaks, additional analysis is required, which cannot be provided here for some reason.
3. Next, we will analyze the series for stationarity, since this is very important for choosing a model. You can do the analysis using the autocorrelation chart (shown below for reference), but I trust the Dickey-Fuller test more.
![image (34)](https://github.com/Geronimik/flexible_row/assets/98644693/f1984438-76c1-4018-ad49-a790fa1ef14f)
As mentioned above, I gave the analysis of the series for stationarity to the automated Dickey-Fuller test with a confidence interval of 5%. This test showed that the series is stationary.
4. All the necessary parameters are defined, a number of primary and superficially studied, it's time to start selecting a model for training. As mentioned earlier, a special algorithm was developed for this purpose, which, by sequentially turning off older data, then retraining the ARIMA and SARIMAX models, enumerating their hyperparameters and measuring their quality, selects the most optimal set for training, the model, and the configuration of hyperparameters for it. For clarity, the model outputs the error function of the models for each set when combining hyperparameters, which gives the best quality in this case.
![image (35)](https://github.com/Geronimik/flexible_row/assets/98644693/32eef971-3a6e-4420-ae30-50c1a297f59e)
MAPE error (average absolute percentage error) was used for measurements. From the plots of the error functions for the models, it can be seen that the SARIMA model gives the best results in general for all data sets, which means that the seasonality factor is of great importance. However, by the end of the tested sets, the qualities of both models converge to almost the same values and to the minimum in the entire history of the test. It remains only to find out which of the models gives at least a slightly smaller error, having a higher quality. This operation is also automated, after determining the highest quality model, it gives us information about it in the following format:
![image (36)](https://github.com/Geronimik/flexible_row/assets/98644693/10ce16ca-48f8-4872-8650-16a911d805ae)
After choosing the most qualitative model and determining its qualitative and quantitative properties, it is necessary to retrain the model on the selected set, make predictions and compare them with the test set.
The results of the work were superimposed on the graph below:
![image (37)](https://github.com/Geronimik/flexible_row/assets/98644693/e71f03d8-8659-4eb0-ba53-82184531da25)
This graph overlays the real values of the test sample and the predicted values, plus predictions for the future. The blue curve is the actual payments of the test sample, the orange curve is the forecast payments.
Above, a system based on machine learning algorithms was described in general terms, which can accept any time series of any format and volume, analyze and describe it, and issue predictions for the future. I would like to clarify that all the actions described above performed by the system are fully automated, and the system itself starts literally "with one click of a button." Absolutely everything is automated: from collecting data from the DVH to uploading all the necessary documents and protocols on the results of the system run.
Also, this system can be easily rebuilt for other related production purposes.

And finally, the seasonality highlighted on a one-year scale is higher than the studied dataset (just for fun):
![image (38)](https://github.com/Geronimik/flexible_row/assets/98644693/a7c4ae75-23f5-4866-ae0d-cfc4aab234e4)
