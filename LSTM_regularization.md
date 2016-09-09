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

## test comparison LSTM111 vs LSTM111+FFN 

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/test_LSTM111_vs_LSTM111FFN%20.png)

## training + test comparison LSTM111 vs LSTM111+FFN

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/train%2Btest_LSTM111_vs_LSTM111FFN%20.png)

## Conclusion so far

Due to the steady behaviour of LSTM111 it seemed to me that it is the right moment to introduce so more model complexity. The most natural way to do this is either add a Dense(10) layer, or LSTM layer or maybe even increase the hidden layer size of the current LSTM. I chose to go with Dense(10) layer. 

Quite surprisingly, the behaviour didn't really improve which makes me wonder why. One potential reason could be that one has to widen the model (aka adding complexity) before "throwing out" information (aka "skinny jeans analogy"). Here we added complexity after doing all the dropout (aka after "throwing out" some information). So probably it is a good idea to introduce somethign before out LSTM model and maybe get rid of the embedding dropout as this one is usually pretty aggressive.

The next section I will just test LSTM100 against LSTM011 to see what the embedding dropout alone is actually doing, before adding more complexity. I will then get a better understanding which of both regularization approaches are better / stronger, and whether to keep going with LSTM1xxx or rather LSTM0xxxx.

## test comparison LSTM100 vs LSTM001

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/test_LSTM100_vs_LSTM011.png)

Here we clearly see some messed up behaviour from the LSTM100, so embedding dropout doesnt seem to be a good idea. 

1) It looks like here and there important bits of information are being thrown out, thus making the performance very shaking, unstable and the curve very non-smooth.  

2) One also recognizes that LSTM100 has a trend to overfit again. 

All in all lets forget about embedding dropout. The whole point of introducing dropout is to reduce overfitting, instead LSTM100 didnt really correct that but just introduced some weird jitteriness that is actually making things worse.

Lets just look at how the training curves look like just as a way to emphasize the overfitting tendency of LSTM100. 

## training + test comparison LSTM100 vs LSTM011 

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/training%2Btest_LSTM100_vs_LSTM011.png)

# Adding 'start' and 'stop' symbols

## test plot

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/test_lstm011ss_vs_lstm011.png)

## training and test plot

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/training%2Btest_lstm011ss_vs_lstm011.png)

OK looks like the initial activations are good enough to represent sequence boundaries. So no need for start and stop symbols. 

# Stacking two LSTM layers

let us compare LSTM011 with LSTM011-11 which is LSTM011 followed by same LSTM with 50 hidden units, `dropout_W =0.5` and  `dropout_U=0.5`. We want to see whether the reduction of variance in increase of bias we have got through adding dropout allows us to augment our model with more complexity and get higher performance. So one way of doing this is by adding another LSTM layer.

## test set comparison 

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/test_LSTM011-11_vs_LSTM011%20.png)

## training + test set comparison  

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/training%2Btest_LSTM011-11_vs_LSTM011%20.png)

## Conclusion

All in all it looks like LSTM is really not performing that well (at least not as we expected given the fact that it read sequences of multiple length without the need of a helper method that destroys the sequence and thus loses information). Call Y the thing in the LSTM that enables it to capture sequences of varying length.

Despit the richer input, the LSTM is still performaning worse than a classical feed-forward net. So this let me conclude that the LSTM doesn't handle the surplus in information it has been given, very effectively. Call X the thing in the LSTM that causes the model to not handle the surplus in information effectively.

So question is, can we have Y without being forced to have X at the same time? The following setps could look like this:
* a) filter out what X could be or come up with strategies that pinpoints what X is. 
* b) try to understand the minimal strucutre needed to have Y working. (call this minimal structure M)
* c) try to remove X while still having Y. 
* c') try on general model M how it performs and understand why it doesnt imply X automatically.
* d') find a way to bypass a way to improve performance of M while staying away form X as much as possible. 

