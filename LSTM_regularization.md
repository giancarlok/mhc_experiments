# Motivation 

Based on the analysis comparing test and training performances between a classical feed-forward neural network and an LSTM, it became clear that we should use more aggressive regularisation techniques in order to avoid overfitting. Here we compare several dropout regularisation approaches: 
* no additional dropout (beside the one we already have at the last layer) "LSTM00", 
* dropout (with probability 0.5) on input gates "LSTM10" , 
* dropout (with probability 0.5) on recurrent connections "LSTM01", and
* both "LSTM11". 

# Plots
## test set (40 epochs)

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/Screen%20Shot%202016-09-07%20at%2016.02.58.png)

## training set (40 epochs)

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/Screen%20Shot%202016-09-07%20at%2016.03.07.png)

## test set (100 epochs)

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/Screen%20Shot%202016-09-08%20at%2010.30.27.png)

## training & test for LSTM11 (250 epochs)

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/Screen%20Shot%202016-09-08%20at%2011.49.43.png)

# Observation 

We see that even after 40 epochs, the regularised models still have the potential to improve on the test set, whereas "LSTM00" clearly goes down before 40 epochs. 


# Adding embedding dropout 0.25

We now coin LSTM011 the previous LSTM11 and LSTM111 to be the previous LSTM11 but with embedding dropout of 0.25

## test set comparison (85 epochs)
![](https://github.com/giancarlok/mhc_experiments/blob/master/Screen%20Shot%202016-09-08%20at%2013.38.11.png)

## training and test set comparison (85 epochs)

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/Screen%20Shot%202016-09-08%20at%2013.38.27.png)
