# General overview 

A general overview can be found here: http://www.cbs.dtu.dk/services/NetMHCpan/versions.php

# netMHCpan-1.0

## References

**NetMHCpan, a Method for Quantitative Predictions of Peptide Binding to Any HLA-A and -B Locus Protein of Known Sequence.** Nielsen M, et al. (2007) PLoS ONE 2(8): e796. doi:10.1371/journal.pone.0000796 

http://www.cbs.dtu.dk/services/NetMHCpan-1.0/PONE2_796.pdf

## Method

**Features** 
* conventional sparse encoding for both: peptide and HLA pseudo-sequence
* BLOSUM encoding for both: peptide and HLA pseudo-sequence
* sparse encoding for peptide and BLOSUM encoding for HLA pseudo-sequence 


**Model** 

* conventional 3-layer FFN with 22-86 hidden units

**Training**

# netMHCpan-2.0

## References

**NetMHCpan - MHC class I binding prediction beyond humans** 
Ilka Hoof, Bjoern Peters, John Sidney, Lasse Eggers Pedersen, Ole Lund, Soren Buus, and Morten Nielsen. Immunogenetics 61.1 (2009): 1-13 

http://www.ncbi.nlm.nih.gov/pubmed/19002680

## Method

**Features**  (same as netMHCpan-1.0: only extended to non-human alleles)
* conventional sparse encoding for both: peptide and MHC pseudo-sequence
* BLOSUM encoding for both: peptide and MHC pseudo-sequence
* sparse encoding for peptide and BLOSUM encoding for MHC pseudo-sequence 

**Model** (same as netMHCpan-1.0)

* conventional 3-layer FFN with 22-86 hidden units.

**Training**

# netMHCpan-3.0

## References

**NetMHCpan-3.0: improved prediction of binding to MHC class I molecules integrating information from multiple receptor and peptide length data sets**  Morten Nielsen and Massimo Andreatta 
Genome Medicine (2016): 8:33

https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-016-0288-x

## Method

**Features**

* 9 mer reduction of peptides 
* BLOSUM encoding of both peptides and MHC pseudo-sequence:
* deletion or insertion length of the 9 mer reduction
* length & composition of PFR (peptide flanking region)
* peptide length L: L =< 8, L = 9, L = 10, L >= 11

**Model** 

* conventional 3-layer FFN with 56 or 66 hidden units. 

**Training**
