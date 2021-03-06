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

After reading a bit about LSTMs, here: 

http://colah.github.io/posts/2015-08-Understanding-LSTMs/

I got the impression I should try several variations of LSTMs, such as GRU or just simply compare it with a simple RNN with same parameters, and see if there is significant difference, that might potentially hint us closer towards X.

# GRU vs LSTM 

here we compare LSTM011 with GRU011, same parameters for both, so we have a one to one comparison.

## test set

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/test_LSTM011_vs_GRU011.png)

## training and test set 

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/training_test_LSTM011_vs_GRU011.png)

We see that GRU is a lot more jittery and unstable in performance. The training performance of GRU is less than the one of the LSTM, so GRU overfits little bit less than LSTM.

# SimpleRNN vs LSTM

## test set comparison 

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/test_RNN_vs_LSTM.png)

## training + test set comparison

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/training_test_RNN_vs_LSTM.png)

## Conclusion & What's next

One sees that RNN clearly performs worse than LSTM. So none of SimpleRNN nor GRU performs better than their LSTM counterparts, so let us maybe come back to our LSTM and think more deeply about what parameters one can tweak. Let us think for a second which other parameters the LSTM might be particularly sensitive to, before moving on the bigger picture. Here a few candidtates: (before moving on let us just point out here that it was completely unknown to me that learning rate seems to be the single most important parameter to tweak, we will come back to the learning rate later on)
* optimization procedure (Since LSTMs are particularly sensitive to step size, we will be comparing ADAM vs RMSProp)
* embedding dimension (we have seen that applying dropout to embedding layer had quite a significant effect on the performance curve, so let us compare “big” vs. “small” sizes for embedding dimension)
* Batch Normalization
* hidden size (we already gridsearched this parameter and foudn 50, but we will do this just to see the plots and the behaviour over several epochs to get an idea how hidden size impacts performance and whether there is a lot of leverage this parameter, we will compare sizes 25 vs 50 vs 100 on a single plot.)

# LSTM "ADAM" vs "RMSProp"

## test 

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/test_RMSProp_vs_Adam.png)

## training & test 

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/training_test_RMSProp_vs_Adam.png)

As we see there is not much of a difference

# LSTM embedding dimension 32 vs 128

## test 

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/test_128_vs_32.png)

## training & test

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/training_test_128_vs_32.png)

Not much of a difference.

# LSTM hidden sizes 25 vs 50 vs 100

## test 

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/test_LSTM_25_50_100.png)

## training & test

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/training_test_LSTM_25_50_100.png)

Not much of a difference either, we just observe 100 hidden units overfitting slightly more

# LSTM with BatchNormalization vs without

## test

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/test_LSTM_vs_LSTMBatch.png)

## training & test

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/training_test_LSTM_vs_LSTMBatch.png)

# Conclusion 

I admit that the search for a meaningful analysis was a little bit all over the place, which has several reasons, but mainly it's me me not really understanding what is going inside an LSTM at every single time step, and thus having very inaccurate intuition what is happening and therefore very low intuition of what X could be. 

A) So in what follows I tried some more random ideas son the embedding side as it appears that it has great effect on the performance curve due to adding embedding dropout highly impacted the behaviour of the AUC performance curve. So two things we are going to try is:
* masking zeros vs padding 
* compare bidirectional LSTM with both LSTMs having the same embedding vs bidirectional LSTM with both LSTMs having individual embeddings (vs a simple LSTM)
* look at the embedded vectors and see what it looks like after PCA-ing or TSNE-ing them, maybe there is an interesting relationship to spot, or maybe there is anomly in the sense that some amino acids are far away form each other when they shoudln't. 


B) Another more fundamental question is how is LSTM performing wrt FFNN and sigmoid and linear model on 9 mers vs non 9 mers. One would expect that LSTM to perform better on non 9 mers due to the surplus on information, and if this is not true, this clearly hints towards a major inefficiency in the LSTM model. 

C) In hope of exposing X, we will give some toy dataset to the LSTM constructed upon the current dataset, and see whether there are some inefficiencies being exposed. One could feed the LSTM with some more or less obvious patterns to learn and see if it indeed gets 100% accuracy. 

D) Another parameter that we will explore is the learning rate, for some reason we have completely overlooked this parameter, and it appears to be the most important paramater for LSTMs. 

Lets dive into it. 

# A) Embedding analysis

## A1) Masking zeros vs padding in the embedding layer

## A2) embedding vs bi-embedding for bidirectional LSTM

## A3) Representing embedding vectors via PCA and t-SNE

# B) All vs 9 mer vs non- 9 mer analysis

# C) Toy datasets

# D) Learning rate analysis 
