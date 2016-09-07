# General overview

A general overview can be found here: http://www.cbs.dtu.dk/services/NetMHC/versions.php 
 

# netMHC-1.0

## References

**Sensitive quantitative predictions of peptide-MHC binding by a 'Query by Committee' artificial neural network approach.** Buus S, Lauemoller SL, Worning P, Kesmir C, Frimurer T, Corbet S, Fomsgaard A, Hilden J, Holm A, Brunak S. Tissue Antigens., 62:378-84, 2003.

http://www.ncbi.nlm.nih.gov/pubmed/14617044

## Method

# netMHC-2.0 

## References

**Reliable prediction of T-cell epitopes using neural networks with novel sequence representations.** Nielsen M, Lundegaard C, Worning P, Lauemoller SL, Lamberth K, Buus S, Brunak S, Lund O. Protein Sci., 12:1007-17, 2003

http://www.ncbi.nlm.nih.gov/pubmed/12717023

**Improved prediction of MHC class I and class II epitopes using a novel Gibbs sampling approach.**
Nielsen M, Lundegaard C, Worning P, Hvid CS, Lamberth K, Buus S, Brunak S, Lund O.
Bioinformatics, 20(9):1388-97, 2004. 

http://www.ncbi.nlm.nih.gov/pubmed/14962912

## Method

**Features** 
* conventional sparse encoding: 9x20 = 180 features 
* BLOSUM50 encoding: 9x20 = 180 features
* Gibbs sampling: 9 features (optional: for comb II) 

**Model** 
* conventional 3-layer FFN with 2-10 hidden units

**Training**

* first round of training:
 * trained on 428 of the 528 peptides (Buus data set)
 * 5-fold cross validation on 428 peptides: the splitting is such that each fold has same distribution of high, low, non-binding peptides. test set being used for stopping criteria. 

* second round of training is done all 528 peptides using parameter of first round. 

* performance measured PCC (Pearson correlation coefficient)

* balanced training: there is an over representation of low and non-binders. One partitions training data into N bins such that i-th bin contains peptides with log-transformed binding affinity between (i-1)/N and i/N. The data from each bin is presented to the NN with equal frequency. 

* For each test/ training split, NN with H (= 2,3,4,6,8 or 10) hidden neurons are tested, and different number of bins N (= 1,2, 3, 4 or 5). For each split, the pair (H,N) with best PCC performance is selected. Say for split i, the model M_i = (H_i,N_i) was being selected, then the ensemble resulting from this cross-validation is the model that the arithmetic mean of M_1, ..., M_5. Afterwards that model is being trained in the second round on the 528 peptides.     
  
## Data 

 
# netMHC-3.0

## References

**Accurate approximation method for prediction of class I MHC affinities for peptides of length 8, 10 and 11 using prediction tools trained on 9mers.** Lundegaard C, Lund O, Nielsen M. Bioinformatics, 24(11):1397-98, 2008.

http://bioinformatics.oxfordjournals.org/content/24/11/1397

**NetMHC-3.0: accurate web accessible predictions of human, mouse and monkey MHC class I affinities for peptides of length 8-11** Lundegaard C, Lamberth K, Harndahl M, Buus S, Lund O, Nielsen M. Nucleic Acids Res. 1;36(Web Server issue):W509-12. 2008

http://www.ncbi.nlm.nih.gov/pubmed/18463140

## Method

# netMHC-4.0

## References

**Gapped sequence alignment using artificial neural networks: application to the MHC class I system.** Andreatta M, Nielsen M. Bioinformatics (2015)

http://www.ncbi.nlm.nih.gov/pubmed/26515819

## Method

**Features** 
* 9 mer reduction & conventional sparse encoding: 9x20 =180 features
* 9 mer reduction  & BLOSUM encoding: 9x20 =180 features  
* deletion or insertion length of the 9 mer reduction
* length & composition of PFR (peptide flanking region)
* peptide length L: L =< 8, L = 9, L = 10, L >= 11
 
**Model** 
* conventional 3-layer FFN with 5 hidden units  

**Training**
* _Hobohm1 five-clustering_: 
Peptides are clustered into five subsets (aka folds) by a Hobohm1-type algorithm. The motivation is to minimise peptide overlap between training and test set:
 * For a given allele, go through the set of all peptides in the data one by one. Every peptide will either put in our "NR list" of non-redundant peptides, or will be put into a cluster attributed to a peptide already belonging to the "NR list". 
 * If the peptide has no common 9 mer with any peptide already in the list, then the peptide is added to the list. Otherwise it is added to the cluster attributed to the first peptide in the list with which it has a common 9 mer. 
 * Next all peptides in the NR list (together with their cluster members) are split into five subsets. This doesn't necessarily guarantee that there is no 9 mer overlap in between elements of two distinct clusters, but apparently the overlap is small (along the lines of 0.5%-2%).

_[Maybe there is room for improvement on the above clustering algorithm, as the construction of the clusters is heavily dependant on the chronology in which the peptides are read in]_

* _Nested cross-validation_: 
 * First fix one of the five folds, and look at the four remaining folds. 
 * Then for each subset of three folds of those four folds, train the 3-layer 5-hidden-unit neural network, and use the fourth fold as a "stopping set", resulting in an ensemble four neural networks. The ensemble is then applied to the fifth fold.
 * Do this for every fold of those five folds.   
