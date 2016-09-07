#General overview 

http://www.cbs.dtu.dk/services/NetMHCII/versions.php

#SMM-align

##References 

**Prediction of MHC class II binding affinity using SMM-align, a novel stabilization matrix alignment method.**
Morten Nielsen, Claus Lundegaard, and Ole Lund.BMC Bioinformatics: 8: 238, 2007.

[http://www.ncbi.nlm.nih.gov/pubmed/17608956](http://www.ncbi.nlm.nih.gov/pubmed/17608956)

##Method

###Features

* every 9mer binding motif using one of two sequence-encoding schemes (sparse feature encoding: 9x20 or BLOSUM encoding: 9x20 )
the following features are added as variations of the core SMM-align method: 
* Gibbs sampler derived position specific weight matrices 
* direct encoding of PRFs (at its core: the SMM-align method is based on the assumption that the binding affinity is solely determined by the nine amino acids in the binding core, which is an oversimplification)
* penalties for longer peptides and short amino terminal peptide flanking residues. 
* peptide length

###Model

* For each of the encodings one tries to identify a weight matrix that optimally reproduces the log-transformed measured IC50 values.

* The score of a peptide is then given by the highest nonamer score within the peptide.

* The optimal weight matrix is found via a Metropolis Monte Carlo procedure where the energy function E that corresponds to regularised L2- loss function. The probability of accepting a move in the Monte Carlo search is given by P = min(1, exp(-dE/T)) where dE is difference in energy between the start and end configuration and T being a scalar.

* Each MC search was initiated with random weights, T was initiated with 0.01 and lowered to 10^{-6} in 20 uniform steps. At each value T, 2500 MC moves were performed.

* The final output is the average of the above mentioned methods using sparse encoding and BLOSUM encoding.

###Training

* In order to minimise the overlap between training set and evaluation data, one uses the Hobohm1 inspired algorithm. It works as follows: 
for each allele, add new peptide to list of non-redundant peptide sequences NR.
if it has no identical nonamer peptide overlap with any of the peptides already on the NR list.
otherwise: peptide is added to the cluster defined by the first hit on the NR list. 
next all the peptides in the NR list (together with their cluster members) are split into five subsets.
the overlap will be minor, for most alleles in the order of 0.5%-2%

* For each allele the method is trained using five-fold cross validation. One of the five sets was left out for evaluation and the others were using for training.

###Evaluation

Performance measured in terms of AUC, PCC and Spearman’s rank correlation. The average predicted AUC- performance for the SMM-align methods over all alleles is around 0.73- 0.75

#NN-align 

##References 

**NN-align. An artificial neural network-based alignment algorithm for MHC class II peptide binding prediction** Nielsen M, Lund O. BMC Bioinformatics. 2009 Sep 18;10:296. doi: 10.1186/1471-2105-10-296.

http://www.ncbi.nlm.nih.gov/pubmed/19765293

##Method 

###Features
for each 9 mer peptide core we produce the following 227-input feature vector:

* BLOSUM encoding of the 9 mer amino acids sequence: 9 x 20 = 180 inputs

* PFRs : 2 x 20 = 40 inputs 
L_{PFR} / 3 and 1- L_{PFR} /3 , where L_{PFR}: length of the PFR

* peptide length: 2 inputs 
L_{PEP’} and 1-L_{PEP’}, where L_{PEP’} = 1/ (1+exp((L-15)/2)), and L: peptide length

* length of C- and N- terminals: 2x2 = 4 inputs

* P1- PSSM score: 1 input
cf ref 10: for a given allele: a log-odds position specific matrix PSSM was constructed using the Gibbs sampler method for class II binding prediction (using the peptide binders in the training data for the allele in question). the core P1 amino acid was encoded as the P1-PSSM score for that given amino acid

TOTAL = 227 inputs

###Model

3-layer neural network: input 227, hidden: 2,10, 20, 40, and 60. (5 network architectures)

###Training

* dataset modification so that data overlap between training and test set is minimised using Hobohm1.

* For each test/training set configuration, one produces 50 networks in the following way: 
for each of the 5 architectures, we run 10 trainings with different initial configuration values (due to large set of local minima). Each run has 500 cycles (where in each cycle one identifies the optimal peptide as being the 9 mer with highest affinity, and updates the weights to lower the predictive error)

* out of these 50 networks one pick the 10 networks with the highest test-set PCC

* For each allele, the above procedure is run using a five-fold cross validation, where for each cross-validation 1/5 of the data is left out for evaluation and 4/5 for four-fold cross-validation in the following way: each four-fold cross-validation results in 40 networks, considering the previous point. The binding core was identified as being the peptide that had the highest affinity most of the time (aka by majority vote among the 40 networks). Until here no peptide was included in the training nor in the peptide core identification.

###Evaluation

* All comparisons are made using one-tailed binomial tests (p-values less than 0.05 were taken to be significant).Previous methods included in the evaluation: SMM -align, TEPITOPE

We compare variations of the above mentioned method:

* NN: standard NN- based method

* NN-W: NN- based method including data redundancy step-size rescaling

* NN-P1: NN-based method including PSSM-P1 amino acid encoding

* NN-W-P1: NN-based method including both the data redundancy step-size rescaling as well as the PSSM-P1 amino acid encoding

* NN-xPFR: NN-W-P1 method excluding peptide-flanking residue encoding

* NN- align methods (0.80-0.81 depending how average is taken and which alleles are taken into account) outperform SMM-align methods (0.75-0.78 depending how average is taken and which alleles are taken into account) and TEPITOPE (0.71-0.72).


