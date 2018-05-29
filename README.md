# protaxfungi
Probabilistic classification of fungi ITS

#################
# Preliminaries #
#################

Extract files from file 'protaxfungi.tgz' (tar xvfz protaxfungi.tgz). This will create directory 'protaxfungi' which contains PROTAX-Fungi version 1 (without sequence data and taxonomy files). Subdirectories under 'protaxfungi' are:

- model: taxonomy, reference sequences, and model parameters
- protaxscripts: Perl scripts needed for running PROTAX
- thirdparty: external tools to calculate sequence similarity predictors and make Krona pie charts

Things to set before running PROTAX:

1) Get sequence data files and taxonomy tree. 

Directory 'model' contains files with the name 'paramsR.levelL', where R is 'its1', 'its2', or 'itsfull'. For each sequence region there are parameters for all taxonomy levels, i.e. L=2..7 (level 1 represents kingdom but it is excluded since the classification starts directly from Fungi node in kingdom level). Levels are 2:phylum, 3:class, 4:order, 5:family, 6:genus, and 7:species. Model parameters are trained for UNITE sequence data and taxonomy using USEARCH version usearch10.0.240_i86linux32.

User should put sequence data and taxonomy tree in the directory 'model'. The needed files are:
- reference sequences in FASTA format for each sequence region: its1.fa, its2.fa, itsfull.fa. When models have been trained, ITS1 and ITS2 regions have been extracted using ITSx software and only those sequences have been used which represent full ITS region, i.e. both ITS1 and ITS2 have been extracted from the original sequence.
- ref.tax[2..7] files contain sequence ID and taxon name pairs for each taxonomy level (one line per sequence ID)
- rseqs[2..7] files contain the list of reference sequence IDs for each taxonomy node (one line per taxon)
- seqid2tax file contains full taxonomy path for each sequence ID
- taxonomy file contains taxonomy tree with all levels
- tax[L=2..7] files contain taxonomy tree for each level separately
- sintax predictor related files:
  -- seqid2tax.ascii7: seqid2tax file where special characters have been replaced
  -- taxonomy.ascii7: taxonomy file where special characters have been replaced
  -- sintax[its1,its2,itsfull]train.fa: reference data suitably formatted for sintax

2) Sequence similarity related predictors are based on 'usearch_global' and 'sintax' from USEARCH package. Download USEARCH from https://www.drive5.com/usearch/download.html

3) Put USEARCH binary 'usearch10.0.240_i86linux32' to subdirectory 'thirdparty'. 
4) Make reference sequences available to USEARCH by giving the following commands in directory 'model':

$ cd model
$ ../thirdparty/usearch10.0.240_i86linux32 -makeudb_usearch its1.fa -output its1.udb
$ ../thirdparty/usearch10.0.240_i86linux32 -makeudb_usearch its2.fa -output its2.udb
$ ../thirdparty/usearch10.0.240_i86linux32 -makeudb_usearch itsfull.fa -output itsfull.udb
$ ../thirdparty/usearch10.0.240_i86linux32 -makeudb_sintax sintaxits1train.fa -output sintaxits1.udb
$ ../thirdparty/usearch10.0.240_i86linux32 -makeudb_sintax sintaxits2train.fa -output sintaxits2.udb
$ ../thirdparty/usearch10.0.240_i86linux32 -makeudb_sintax sintaxitsfulltrain.fa -output sintaxitsfull.udb

5) Download Krona from https://github.com/marbl/Krona/wiki 
6) Put Krona tar package to directory 'thirdparty':

$ cd thirdparty
$ tar xvf KronaTools-2.6.1.tar
$ cd KronaTools-2.6.1
$ perl install.pl --prefix ../krona

and make Krona available in PATH (this is for Bash script 'runprotax'):

$ export PATH=$PATH:`pwd`/krona/bin

7) Set variable PROTAXDIR correctly in Bash script 'runprotax'. It should be the path to the present directory where the subdirectories 'model', 'protaxscripts' and 'thirdparty' are.

#####################
# How to use PROTAX #
#####################

Bash script 'runprotax' is used for running PROTAX using either ITS1, ITS2, or ITSfull model. ITSfull includes the entire ITS1_5.8S_ITS2 region. 

Two shell variables ITS and ODIR need to be set before running the script. Variable ITS defines the sequence model, it needs to be its1, its2, or itsfull.

Sequences to be classified should be in FASTA format in a file named 'query.fa'. Before running PROTAX, user should set variable ODIR to be the parent directory of this file. PROTAX will write all its output in this directory. E.g. if the original FASTA file is in ~/allseqs/sample1.fa and the sequences represent ITS2 region, the following commands will create a new directory for this sample and run PROTAX for it:

$ ODIR=~/myprotaxout/sample1
$ ITS=its2
$ mkdir $ODIR
$ cp ~/allseqs/sample1.fa $ODIR/query.fa
$ source runprotax

After runprotax is finished, PROTAX classifications and Krona piechart are in $ODIR. Outputs from each taxonomy level are in separate files:

- phylum classifications in $ODIR/query2.nameprob
- class classifications in $ODIR/query3.nameprob
- order classifications in $ODIR/query4.nameprob
- family classifications in $ODIR/query5.nameprob
- genus classifications in $ODIR/query6.nameprob
- species classifications in $ODIR/query7.nameprob

Note: when creating Krona piechart, if input FASTA file has ';' in ID lines, the 2nd column is interpreted as cluster size which sequence represents. Size information may also include prefix 'size='. Here are four examples of FASTA IDs:

>seqidA              =>   cluster size = 1  (since no other information given)
>seqidB;size=150     =>   cluster size = 150
>seqidC;40           =>   cluster size = 40
>seqidD;fungi        =>   cluster size = 1  (since 'fungi' is not a number)
