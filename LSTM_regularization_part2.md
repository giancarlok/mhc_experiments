# Introduction

Regularizations attempts in part 1 have been successfull in the sense that we could prevent the model from overfitting by tuning dropout and various other parameters. 
However we still dont have our LSTM at the performance we were hoping for, so the hyperparamater chasing continues.

A) So in what follows I tried some more random ideas son the embedding layer as it appears to have great effect on the performance curve: adding embedding dropout highly impacted the behaviour of the AUC performance curve, it became really jittery. So two things we are going to try:
* masking zeros vs padding 
* compare bidirectional LSTM with both LSTMs having the same embedding vs bidirectional LSTM with both LSTMs having individual embeddings (vs a simple LSTM)
* look at the embedded vectors and see what it looks like after PCA-ing or TSNE-ing them, maybe there is an interesting relationship to spot, or maybe there is anomly in the sense that some amino acids are far away form each other when they shoudln't. 


B) Another more fundamental question is how is LSTM performing wrt FFNN and sigmoid and linear model on 9 mers vs non 9 mers. One would expect that LSTM to perform better on non 9 mers due to the surplus on information, and if this is not true, this clearly hints towards a major inefficiency in the LSTM model. 

C) In hope of exposing X, we will give some toy dataset to the LSTM constructed upon the current dataset, and see whether there are some inefficiencies being exposed. One could feed the LSTM with some more or less obvious patterns to learn and see if it indeed gets 100% accuracy. 

D) A little side idea that came up (and maybe somehow related to C) is to approximate peptides with their first 3 amino acids and last three amino acids, and see how well LSTM and FFN are performing. This might give us an idea how relevant the middle piece really is, and whether more data is actually confusing the model rather than helping them. The idea comes from the fact that binding affinities tend to be rather influenced by what happens towards to ends of the peptide.

E) Another parameter that we will explore is the learning rate, for some reason we have completely overlooked this parameter, and it appears to be the most important paramater for LSTMs. 

Lets dive into it. 

# A) Embedding analysis

## A1) Masking zeros vs padding in the embedding layer 

[ notebook ](https://github.com/giancarlok/mhc_experiments/blob/master/LSTM%20mask%20zeros%20vs%20explicit%20padding.ipynb)

Here we compare our basic bi-LSTM model by looking at `mask_zero = True` vs `mask_zero = False`

### test set 

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/test_mask_zero_vs_padding.png)

### test and training set

![](https://raw.githubusercontent.com/giancarlok/mhc_experiments/master/training_est_mask_zero_vs_padding.png)

### Conclusion 

We see that padding confuses the LSTM initially (which we read from the performance drop), but eventually LSTM understands that zeros are to be ignored and performs eventually the same as when masking zeros. So there is no real gain by keeping the zeros.

## A2) embedding vs bi-embedding for bidirectional LSTM

[ notebook ](https://github.com/giancarlok/mhc_experiments/blob/master/LSTM%20vs%20biLSTM%20vs%20bi_embedded_LSTM.ipynb)

Recall the fact that we are actually working with a bidirectional LSTM. So far our bi-LSTM has had both its LSTMs coming from the same embedding layer. Now we will try to decouple the embedding for each LSTM, so that each LSTM has its own embedding layer and we' ll see whether there is a gain in performance by doing that. Pro forma we compare these two scenrios with a simple LSTM just as a baseline. 

### test set comparison



### test and training set comparison



## A3) Representing embedding vectors via PCA and t-SNE

# B) All vs 9 mer vs non- 9 mer analysis

# C) Toy datasets

# D) Approximation of peptides 

# E) Learning rate analysis 
