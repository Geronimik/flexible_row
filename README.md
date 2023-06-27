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

After creating the system, we will give a specific example of its operation based on a specific data set. As such, we use the real series of payments received, aggregated monthly by the total amount. The results of the system operation are shown below.
