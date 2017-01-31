README for FactorNet
====================

README still under construction (1/30/17). As you can tell, there are a lot of similarly named files. Many lines of code were embarassingly copied in multiple scripts instead of simply shared. I will try to condense everything into 2 or 3 script files soon, but everything I used for the ENCODE-DREAM competition is here.

Citing FactorNet
================
Quang, D. and Xie, X. ``FactorNet: a deep learning framework for predicting cell-type specific transcription factor binding from nucleotide-resolution sequential data'', In preparation, 2017.

INSTALL
=======
FactorNet uses several bleeding edge packages that are sometimes not backwards compatible with older code or packages. Therefore, I have included the most recent version numbers of the packages for the configuration that worked for me. For the record, I am using Ubuntu Linux 14.04 LTS with an NVIDIA Titan X GPU (both Maxwell and Pascal architectures) and 128 GBs of RAM. Theoretically, about 24 GB of ram is needed to train one model, since the current version loads two copies of the genome into memory for efficient sequence extraction. If you would like a slightly slower version that saves memory by reading the genome from hard disk instead of RAM, let me know.

Required
--------
* [Python] (https://www.python.org) (2.7.11). The easiest way to install Python and all of the necessary dependencies is to download and install [Anaconda] (https://www.continuum.io) (4.0.0). I listed the versions of Python and Anaconda I used, but the latest versions should be fine. If you're curious as to what packages in Anaconda are used, they are: [numpy] (http://www.numpy.org/) (1.10.4), [scipy] (http://www.scipy.org/) (0.17.0). Standard python packages are: sys, os, errno, argparse, pickle and itertools. 
* [Theano] (https://github.com/Theano/Theano) (latest). At the time I wrote this, versoin 0.8.2 was the latest release of Theano. However, it is incompatible with the latest versions of CUDA (8.0) and cuDNN (5.1) I was using. You need to git clone the latest bleeding edge version since there is not a version number for it:
```
$ git clone git://github.com/Theano/Theano.git
$ cd Theano
$ python setup.py develop
```

* [pyfasta] (https://pypi.python.org/pypi/pyfasta) (0.5.2). Loads the hg19.fa FASTA file in the resources folder.

* [pyBigWig] (https://github.com/dpryan79/pyBigWig/releases/tag/0.2.8) (0.2.8). Efficiently loads bigWig continuous values into memory. Newer versions should be fine.

* [pybedtools] (https://pypi.python.org/pypi/pybedtools) (0.7.8). Also requires [bedtools2] (https://github.com/arq5x/bedtools2).

* [parmap] (https://pypi.python.org/pypi/parmap/1.3.0) (1.3.0). Helps parallelize certain functions like preprocessing multiple BED files.


* [keras] (https://github.com/fchollet/keras/releases/tag/1.1.1) (1.1.1). Deep learning package that uses Theano backend. Newer versions are likely to be fine, but Keras has a history of not being backwards compatible with older versions.

Optional
--------
* [CUDA] (https://developer.nvidia.com/cuda-toolkit) (8.0). Theano can use either CPU or GPU, but using a GPU is almost entirely necessary for a network and dataset this large.

* [cuDNN] (https://developer.nvidia.com/cudnn) (5.1). Significantly speeds up convolution and recurrent operations. 

USAGE
=====
Data
----
Before training, you must have a copy of the hg19 genome FASTA, hg19.fa, in the resources folder. Due to space issues, the FASTA file is not included in this repository. This file also cannot be compressed, so you will have to deal with it taking up quite a bit of space on the hard drive. You can get yourself a copy of the file with the following command lines:

```
$ wget http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/chromFa.tar.gz
$ tar zxvf chromFa.tar.gz
$ cat chr*.fa > hg19.fa
```

All training data are in the data folder. Training data are organized into one folder per cell line. The README in the data folder will give you an idea of how to format cell data if you want to use FactorNet on your own data. You will also need download additional files into the data folder before you can proceed with the example code below.

Training and prediction
-----------------------

There are five different type of models used for the ENCODE-DREAM competition. These models differ in their training method and types of features they incorporate. Training is primarily done with the train and predict scripts. Currently, the code is a little messy, so there is almost a train and predict script for each of the five different types of models. I plan on cleaning up the code into just one train and one predict script. I've also included a sample BED file in the resources folder (sample_ladder_regions.blacklistfiltered.bed.gz) to make predictions on. The README in the models folder describe what the program outputs once training completes.

The following are descriptions and code snppets for each model.

* multiTask
This model trains with multiple TF binding targets in a joint multi-task fashion. Each epoch, it draws . Bins labeled as ambiguous are treated as negative bins. This model can only train on one cell line.
```
$ python train.py -i data/HepG2 -oc multiTask_Unique35_DGF_HepG2
$ # If you want to remove the 35 bp Uniqueness track as features, remove that line from the bigwig.txt file in data/HepG2.
$ python predict.py -f FOXA2 -i data/liver -m multiTask_Unique35_DGF_HepG2 -b resources/sample_ladder_regions.blacklistfiltered.bed.gz -o FOXA2_liver.bed.gz
$ # The predict script can only predict one TF at a time, which must be specified with the -f option.
```

* onePeak
The onePeak model, and all subsequent models, are trained on a single TF in a single-task fashion. Unlike the multiTask model, it can leverage data from multiple reference cell lines and ignores ambiguous peaks. 
```
$ python train_onepeak.py -f MAX -i data/A549 data/GM12878 data/H1-hESC data/HCT116 data/HeLa-S3 data/HepG2 data/K562 -oc onePeak_Unique35_DGF_3n_50e_128k_64r_128d_MAX -k 128 -r 64 -d 128 -n 3 -e 50
$ python predict.py -f MAX -i data/liver -m onePeak_Unique35_DGF_3n_50e_128k_64r_128d_MAX -b resources/sample_ladder_regions.blacklistfiltered.bed.gz -o MAX_liver.bed.gz
```

* meta
```
$ python train_onepeak.py -f MAX -i data/A549 data/GM12878 data/H1-hESC data/HCT116 data/HeLa-S3 data/HepG2 data/K562 -oc onePeak_Unique35_DGF_3n_50e_128k_64r_128d_MAX -k 128 -r 64 -d 128 -n 3 -e 50
$ python predict.py -f MAX -i data/liver -m onePeak_Unique35_DGF_3n_50e_128k_64r_128d_MAX -b resources/sample_ladder_regions.blacklistfiltered.bed.gz -o MAX_liver.bed.gz
```

* GENCODE
This model is very similar to the meta model. Like the meta model, it incorporates non-sequential metadata features, except at the bin level instead of at the cell type level. Bins are annotated with GENCODE (promoters, introns, 5' UTR, 3' UTR, and CDS) and unmasked CpG islands. Promoters are defined as genomic regions up to 300 bps upstream and 100 bps downstream of a TSS.

```
$ python train_gencode.py -f REST -i data/H1-hESC data/HeLa-S3 data/HepG2 data/MCF-7 data/Panc1 -oc GENCODE_Unique35_DGF_64k_128d_REST -k 64 -d 128
$ python predict_gencode.py -f REST -i data/liver -m GENCODE_Unique35_DGF_64k_128d_REST -b resources/sample_ladder_regions.blacklistfiltered.bed.gz -o REST_liver.bed.gz
```

* metaGENCODE
This model includes the metadata features from both the meta and GENCODE models.
```
```




