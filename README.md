# Analysis of the income pattern in US census data
Detailed analysis in  electricity_price.ipynb

## Model choice - XGBoost
XGBoost is an efficient implementation of gradient boosting for regression problems.

It is both fast and efficient, performing well, if not the best, on a wide range of predictive modeling tasks. Although not a time series model per se, it is very fast to compute and can be applied to a time series after transforming the time series dataset be  into a supervised learning problem first - whiich we did above.

Contrary to ARIMA models, XGBoost is agnostic about the underlying drivers of the time series, though is able to better fit the training data. Also, while LSTM and sequential neural networks may dedliver a better fit, they are very slow to train and would require more training data than the roughly 50,000 observations we have here - as the number of parameters to estimate wouldd be very large.

As an ensemble methodd, XGBoost fits a set of decision trees where new trees fix errors of those trees that are already part of the model. Trees are added until no further improvements can be made to the model.

* Target value: the 24h ahead spot power price
* Features: the prior week of half hourly data of the spot price and all 16 variables with retained with the PCA explaining 9-% of the cumulative variance

## Results
Display of the results on the test set for the month of March 2020

!

## Performance assessment
The RMSE achieved on the test set is still quite high at 29, where the average spot price oscilates around 45. As we can tell form training RMSE (though using the normalised values), it is steadily decreasing every training pass. Training it for longer would improive the fit substantially.

From the graph above, we see that the main issue is with the 'average' but that actually the variations - both postive and negative spikes have bee learnt very well!

## Improve performance
I have so far simply decided on the hyperparameters out of judgement (depth, features, learnig rate, regularization...). To improve the fit:
* train longer
* make learning rate adaptable
* hyperparameter tuning: grid search over model hyper parameters to select best architecture
* watch out for overfitting - could revise the feature selection threshold (from 70% to 95% of the cumulative variance)

In any case, to improve performance trainingmore than one mode is needed playing with its hyper-parameters so that cross comparisons can drive a proper model selection

## Productionise model
Once the model has been refined, we can productionise it. This requires automate the data pipeline to gather the sample on which to make the predictions, automate the model run and save the outputs before 7 BST every day.

#### Data pipeline
Every day, at 6:30 BST all half hourly data of the previous 8 days needs to be shaped to be fed to the trained model. This needs to prepare 48 datapoints of features - so tthe next 24h Spot price can be preddicted half-hourly.
* collection
* shaping
* formating / normalisation

#### Model run
Save the trained model parameters and run the output of the data pipeline every day at 6.30 once the data process has finished. 

#### Save 
The model runs will deliver a file with 48 data points, one for each spot price half hourly until 6.30 am the next day.

#### Infrastructure
Using AWS and terraform setting the infrastructures and task is simple. The infrastructures required are:
* database storing the features updated every day at 6.30
* instance EC2
* bucket S3

We should write a brief bash file to create a container that access the data base and implements the pipeline. Then the same executable calls the model and runs the new data through it, exporting the outoputs to the S3 wehre it can be accessed by the end user. The containter image can then be programmed using a recurrent task to start every day at 6.30.
