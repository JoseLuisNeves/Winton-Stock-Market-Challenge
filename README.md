## Winton Stock Market Challenge
# Summary
In this challenge, posted in the Kaggle plataform in 2016, Winton asked the participants to predict stock market returns using passed returns. This could be divided into intraday e interday returns, for each will develop separate models that will interact. 

# Data
It is given to us five day window of stock returns, days D-2, D-1, D, D+1, and D+2. We have the returns of the first two days and part of the third. We are asked to predict the rest of the stock returns of the five day window. In the third day the type of returns that we have its intraday, on a total 120 minutes of given data with wich we want to predict the next 60 minutes. The rest of the returns consists of the return that each stock makes since the opening of the day untill the opening of the stock market the next day. We are also provided with 25 anonymous features that may or may not be useful. 
The training comes with 40 000 samples and the testing with 120 000. The next image should give the reader an idea of the organization of the data.

![image](https://user-images.githubusercontent.com/90193839/132510555-c2bee661-3dec-47b4-ab6d-3746c423c30f.png)


# Data prepping

![image](https://user-images.githubusercontent.com/90193839/132510638-616f091d-9670-47be-8440-04d4044de57c.png)

The first thing we are going to talk about are the features. In the beginning of the challenge people started to try to discover the origin of this anonymous atributes of the stock. The contest's hosts noticed it and decided to mask all the data. This features correspond to static caracteristics of each sample but they differ in usefullness and may even hurt our results. For starters most some features have a large percentage of NaN values. In most circumstances NaNs may even give some information to the model about the data, but in some cases this percentage was so high that the model couldn't even finish training. The goal here is to let the deep learning model to the feature engineering for us, because it will probably be much better at it then a human. So for the model to work but without losing the information that the NaN values give to us we fill this empty spaces with big outliers(twenty times the value of the column's maximum). 
Lets talk about the returns now. We have two very distinct situations. Predicting intraday returns is much diferent than predicting the all return accomplished in one day. For minute predictions we have way to little data. So we are going to do two things. We are shifting the data format in this from return for each minute to a sliding window with various lenghts (we will test it to see wich lenght is the best to use), for example an interval of [1,50] minutes associated with minute 51 and so and so forth, increasing the number of training samples from a factor of one hundred. Also, the prediction will move in the way of a sliding window to. Continuing the example of 50 time steps lenght, we would predict the minute 51, throw away minute one, add to the end of the window the minute 51 and use this window to predict 52. The interday returns are going to be predicted in a similar fashion, with the sliding widnow going through the days instead of minutes.

![image](https://user-images.githubusercontent.com/90193839/132513435-74a59e4c-571f-43bc-ad5e-f98477a1b1c6.png)


# Model Building
So the model building part will too have to be separate. We will have a model for the intraday predictions and another one for the interday predictions, with the outputs of the first entering and helping training the second. However, this models will in many ways similar. They will both be multistep autoregressive models with multiple inputs. From the tensorflow page on time series forecasting we take the following diagram to show a general example of a multistep autoregressive:
![image](https://user-images.githubusercontent.com/90193839/132530014-dc86e117-cd8c-4790-8e9e-4b34efef00fb.png)
This is just a fancy name for the process that i talked about before, with the addition of the warm up process, in which the first layers will have the argument return_sequences = True, being the last layer the only one that is going to create an output. The first layers will not give us a prediction because we want to give the model time to improve it's results with better capacity for fitting more complex data that the last layers have. As for the multiple inputs part: in the first model we will need to have LSTM layers to analyse the time series wich we created as well as Feed Forward layers to analyse the static information given to us by the features. There are two ways to go about this: either we concatenate the two types of layers or we initialize the LSTM part with results of the Feed Forward. We choose the last one given the better results it was showing. The second model will have as input the features, the returns of the past days but the intraday predictions associated with that time series too. 

![image](https://user-images.githubusercontent.com/90193839/132532933-401405e2-3dea-45a3-8164-ecf8d536a764.png)
![image](https://user-images.githubusercontent.com/90193839/132533672-2e1ae37e-a23c-4ac6-8409-96c8ef8a1681.png)







