# IEDB database: 
http://www.iedb.org/

One usually tries to train on IEDB 2009 and test on IEDB 2013

Link for IEDB 2009: 

Link for IEDB 2013: 

# Data loading and preparation 

```
df = pd.read_table("data.txt")
``` 
loads data into data the frame `df`

```
df.groupby('mhc').size().nlargest(11)
```  
shows you list of the 11 most frequent alleles.

```
df_h=df[df['mhc']=='HLA-A-0201'][['sequence','meas']]
``` 
loads HLA-A-0201 data into a new dataframe `df_h` (in case you are working on allele-specific predictor)

```
df_h['log_meas']=1-np.log(df_h['meas'])/math.log(50000) 
``` 
log-transforms the labels into [0,1] values.

# Amino acid encoding 
* Index encoding 

```
from mhcflurry.amino_acid import common_amino_acids
``` 
if you want to have a look at amino_acid.py here is the link: https://github.com/hammerlab/mhcflurry/blob/master/mhcflurry/amino_acid.py
```
def amino_acid_encoding(s, maxlen):
    a = 1+common_amino_acids.index_encoding([s],len(s)).flatten()
    return np.concatenate([a, np.zeros(maxlen-len(a),dtype=int)])
```
transforms peptide sequence string into integer vector, where empty is sent to 0, A is sent to 1 etc All vectors are of same length namely the length of the longest peptide sequence in the list aka `maxlen`, here it is 15. 


```
df_h['encoded_peptides'] = df_h.sequence.apply(lambda seq: amino_acid_encoding(seq, max_len))
```
applies it to every peptide sequence in the data frame, where 
```
max_len=df_h['sequence'].str.len().max()
```
 is the maximum length of all peptides in the data.
* Hotshot encoding

* MHC sequence encoding

https://github.com/ANHIG/IMGTHLA/blob/Latest/fasta/A_prot.fasta
not sure though how to properly read this one in
 
* MHC pseudo-sequence encoding 

* PFR (peptide flanking region) encoding

* C- terminal encoding 

* N- terminal encoding  

* BLOSUM encoding 

* Conventional sparse encoding 

* 9 mer reduction encoding 

* Peptide length encoding 

```
df_h['peptide_length'] = df_h['sequence'].str.len()
```
creates a new column with respective peptide lengths. 

* stability (NetMHCstab)
* cleavage sites of the human proteasome (NetChop)

* Which one is the best encoding method ? Advantages vs Disadvantages 

# Training / test split and cross-validation 

```
X_train = pd.DataFrame(list(df_h['encoded_peptides'].iloc[train])).values
Y_train = pd.DataFrame(list(df_h['log_meas'].iloc[train])).values
X_test = pd.DataFrame(list(df_h['encoded_peptides'].iloc[test])).values
Y_test = pd.DataFrame(list(df_h['log_meas'].iloc[test])).values
```
The choice of variables `train` and `test` determine what training / test split one wishes to have.
 
# Keras

* Useful packages to load
```
from keras import models, layers, optimisers
from keras.preprocessing import sequence
from keras.models import Model
from keras.layers import Dense, Dropout, Embedding, LSTM, Input, merge, Convolution1D, AveragePooling1D, Activation, Flatten
 ``` 

* How to compile a bidirectional LSTM in Keras

```
sequence = Input( shape= (max_len,), dtype='int32')
embedding_dimension = 100
embedded = Embedding(21, embedding_dimension , mask_zero = True)(sequence)
```

First we have to embed the sequences we wish to use as the input for our LSTM. 
Recall that `max_len` is defined above as maximal peptide length in the dataset. 

Note that `mask_zero = True` cuts out all the 0s and reduces the input integer vector to the largest zero-free subvector. One wishes to cut out the 0s before they get embedded as otherwise they get treated as amino acids and thus skew the result or performance. Also note that when using input vectors of varying length one cannot use a FFN straight away, as it requires to have vectors of same lengths, thats why the methods in Nielsen et al. use either only 9 mer peptides as inputs or some technique that reduces all non - 9 mer peptides to 9 mer peptides, which of course comes at the cost of a loss of information. Here we don't need to worry about having input data that has same length as we are using an LSTM and LSTMs can handle sequences of varying length. So if one wishes to compare the performance of LSTM and say 3-layer FFN one cannot do it directly, but either (a) reduce all peptides to 9 mers, or (b) compare only on 9 mers, or (c) have a FFN for each peptide length and then combine the output each a FFN in some way.

```
LSTM_hidden_layer_size = 80
forwards = LSTM(LSTM_hidden_layer_size)(embedded)
backwards = LSTM(LSTM_hidden_layer_size, go_backwards=True)(embedded)
merged = Merge([forwards, backwards], mode='concat', concat_axis=-1)
```
The `'Merge'` layer supports a number of pre-defined modes:

`'sum'` (default): element-wise sum
`'concat'`: tensor concatenation. You can specify the concatenation axis via the argument `'concat_axis'`, as we did.
`'mul'`: element-wise multiplication
`'ave'`: tensor average
`'dot'`: dot product. You can specify which axes to reduce along via the argument dot_axes.
`'cos'`: cosine proximity between vectors in 2D tensors.

You can also pass a function as the `'mode'` argument, allowing for arbitrary transformations such as: 
```
merged = Merge([left_branch, right_branch], mode=lambda x, y: x - y)
```
see: https://keras.io/getting-started/sequential-model-guide/
```
dropout_probability = 0.5
after_dp = Dropout(dropout_probability)(merged)
```
adds some regularisation to out LSTM via dropout with probability 0.5.
``` 
output = Dense(1, activation='hard_sigmoid')(after_dp)
model = Model(input=sequence, output=output)
``` 
finalizes the model with a single output layer. The choice of the activation function seems to be quite relevant for the overall performance. There are several choices for activation functions in Keras such as: `'softplus'` , `'softmax'`, `'softsign'`, `'linear'`,  `'sigmoid'`,  `'tanh'`,  `'relu'`, and `'hard_sigmoid'`.  see: [https://keras.io/activations/](url)

```
model.compile(optimizer = 'Adam', loss='mean_squared_error')
```
finally complies the model. There are several choices for optimizers:
`'Adagrad'`, `'Adamax'`,  `'Adadelta'`,  `'SGD'`,  `'RMSprop'`,  `'Adam'` and `'Nadam'` 
see: [https://keras.io/optimizers/](url)

as well as several choices for the loss function: 
`'mean_squared_error'`,  `'mean_absolute_error'`, `'mean_absolute_percentage_error'`, `'mean_squared_logarithmic_error'`,  `'squared_hinge'`,  `'hinge'`,  `'binary_crossentropy'`,  `'categorical_crossentropy'`, `'sparse_categorical_crossentropy'`,  `'kullback_leibler_divergence'`,  `'poisson'`,  `'cosine_proximity'`. 
see: [https://keras.io/objectives/](url)
* How to train LSTM on batch 

```
batch_size = 16
n_epochs = 100
epoch = 0
for epoch in range(n_epochs):
    for batch_idx in range(len(X_train) // batch_size):   
        model.train_on_batch(X_train[batch_idx * batch_size:(batch_idx + 1) * batch_size], 
                             Y_train[batch_idx * batch_size:(batch_idx + 1) * batch_size])
    train_pred = model.predict(X_train)
    test_pred = model.predict(X_test)
```
Note that by using X_train and X_test, we implicitly use " index encoding " as our amino acid encoding. 
 
* Evaluating LSTM performance via AUC

You might be confused why on earth I am talking about AUC, as this is only used for evaluating categorical output, whereas the log-transformed binding is continuous. Well, as it seems, there is a conventional binding affinity cutoff of 500 nM that distinguishes "binders" from "non-binders". Thus the definition of the following function:
```
def measured_affinity_less_than_500(Y):
    IC50 = 50000**(1-Y)
    return IC50 < 500
```
Note that the function is defined in terms of `Y`, a numpy array, which has the advantage of being more memory efficient. 

The reason we use AUC is merely because it has been used in previous literature, and thus we can directly compare our results to previous results in the literature, and hopefully outperform them.

At the end of the second for loop in the batch training (cf. previous bulletpoint), one can add the following print statement: 
```
print(epoch, batch_idx, roc_auc_score(measured_affinity_less_than_500(Y_train),train_pred), 
          roc_auc_score(measured_affinity_less_than_500(Y_test),test_pred))

```     
this gives you an overview what happens after each epoch. Note that the reason why used `measured_affinity_less_than_500` is because `roc_auc_score` requires as a first argument a categorical 0-1 value, whereas the second argument can be continuous. Don't forget to include:
```
from sklearn.metrics import roc_auc_score
```
 
* How to add convolutional filter to our LSTM

* How to add attentional mechanism to out LSTM  
