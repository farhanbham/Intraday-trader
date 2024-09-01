# Long-Short-Term-Memory (LSTM) Forecasting for Buy Signal Generation

> Task: Maximise Sharpe ratio using buy signals generated from our LSTM model.

In this project we attempt to generate buy signals using an LSTM model to forecast the next days price using the previous 24 hours of price data. LSTM models are effective time series modellers due to its ability to capture temporal dependecies and handle non-stationary sequential data. Our buy signals are generated by comparing the forecasted price against the current price, producing a buy signal when the forecasted price is above a certain threshold relative to the current price. Finally, we evalaute our model using the Sharpe ratio and compare it to a baseline buy and hold strategy. 


---


## Table of contents
- [Dataset](#dataset)
- [Data Preprocessing](#data-preprocessing)
- [LSTM Model](#lstm-model)
- [Forecast Evaluation](#forecast-evaluation)
- [Trading System](#trading-system)
- [Trading System Evaluation](#trading-system-evaluation)
- [Conclusion](#conclusion)


---


## Dataset

The dataset was downloaded through Binance's API. The dataset consists of hourly ETH-USD open, high, low, close and volume (OHLCV) data. The dataset contains no missing values. 


---


## Data Preprocessing

Several steps are taken to process the data for our LSTM model:

**Scaling -**
In this stage we make sure to scale our test data with parameters used to scale our training data. This is to ensure no data leakage.

**Creating Sequences -**
 Our LSTM model requires batches of sequences for training. We create batches of sequences of OHLCV data in which the length can be adjusted through the variable SEQ_LENGTH. A corresponding target value (the close price in 24 hours) is also created. 

**Auto-Encoder -**
 To prevent overfitting, we implement an autoencoder model. This reduces the dimensionality of our data to a specified dimension which can adjusted through the dimension variable. This model encodes the sequences into our specificied dimensionality and then attempts to decode it back to the orignal input values. Although we already have our encoded data we still need to decode it so we can identify if the encoded data retains the orignal information. We optimise our autoencoder model hyperparameters using Bayesian optimisation. This method learns from previous results in an attempt to reduce hyperparamter tuning speed as this is a time-intensive process.


---


## LSTM Model

Our LSTM model contains bi-directional, dropout and batch normalisation layers. The model hyperparamters are also tuned using bayesian optimisation. The model takes our encoded sequences as input, and trains using the target prices. Training and validation data is split in a 4:1 ratio. The model uses mean squared error (MSE) as its loss function, penalising larger error more harshly than mean absolute error (MAE). After training, the model predicts prices using unseen data which will be used for our trading simulation.


---


## Forecast Evaluation


We evalaute the model's performance using MSE, MAE and mean absolute percentage error (MAPE). We also evaluate the models performance of forecasting the direction of price movement. We produce a confusion matrix and calculate recall (percentage of correctly predicted upward movements) and specificity (percentage of correctly predicted downward movements) scores to do this. 


---


## Trading System

Here we create a trading system which buys, sells or holds its position every hour. Trades are based on the hourly ETH-USD close prices from January 2023 to June 2024. A slippage value of 0.0015 (0.15%) is applied to every trade to account for Binance's trading fee (0.1%) and margin loss (we estimate 0.05%). 

**Buy Signal -**
 We generate a buy signal by dividing the predicted price by the current price. If this value is above a certain threshold (eg. 1.04 for a 4% threshold) a buy signal is produced and the asset is purchased at the current price in the simulation if funds are available. 

**Sell Signal -**
 Our trading system sells when our position has increased by a certain threshold or if an open position is held beyond a certain maximum holding period. 

We are able to adjust position size, take_profit, max_hold_time and signal_strength parameter values according to individual risk preference.


---


## Trading System Evaluation

Our trading system is evaluated using annual return, maximum drawdown and the Sharpe ratio. These metrics are compared to that of a buy and hold strategy of the same asset throughout the trading period. The risk free rate used to calcualte the Sharpe ratio was estiamted using the average 1 year treasury bond yield during the trading period.


---


## Conclusion

We find that the LSTM model appears to predict the future price of Ethereum fairly accurately, however, this is due to the fact that the LSTM uses data from the previous 24 hour preiod. The model will therefore select prices close to the last seen price by the model, giving the false impression of prediction accuracy. Furthermore, we find that the model is actually a poor predicted of price movement direction as seen through our confusion matrix and corresponding metrics. The model should therefore be regarded as an innefective predictor of price movement. However, the model does show promising results as it produces better metrics accross the board compared to our baseline model. It can be argued that this may be due to our signal generation threshold. If strict criteria is set for signal generation (a predicted price much greater than current price) this may increase our confidence in gerenated buy signals but, also reduce the number of buy signals produced. We suggest that further research include an implementation of other deep learning buy signal generation methods supplemented with those produced by our model.
  
