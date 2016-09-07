# Models  

We compare LSTM with 50 hidden neurons and 0.25 dropout probability before the output layer (batch_size=16) with 3 layer feed-forward neural network with 10 hidden neurons.

# Feature encoding 

LSTM can read sequences of varying length so we don't need to modify them, whereas a classical feed-forward neural network needs to have sequences of fixed length. Thus we use the `kmer_index_encoding()` which breaks non 9 mers into several 9 mers. The `kmer_index_encoding()` can be found in `mhcflurry/dataset.py`.

Labels (after being log-transformed) are being set to 0 if they are negative. 

# Embedding 

The features for the LSTM are index-encoded an then embedded into 32 dimensional space (while zeros are being masked). The indices given by `kmer_index_encoding()` for the feed-forward neural network are also being embedded into 32 dimensional space.  

# Training 

Training is done simultaneously on shuffled 3-fold cross-validation 

# Evaluation 

One uses the classical AUC score, where IC50 scores below 500 are being classified as 1 ("binders"), and above 500 as 0 ("non-binders"). 

# Plot 

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/Screen%20Shot%202016-09-07%20at%2010.20.52.png)

# Observation 

One clearly sees that LSTM has strong tendency to overfit (despite the dropout before the output layer), and that a classical neural network is performing slightly better, even though some information has been lost using the `kmer_index_encoding`. Thus our conclusion is that one should try more aggressive regularisation techniques on the LSTM, such as dropout on recurrent layer, or input layer. 
